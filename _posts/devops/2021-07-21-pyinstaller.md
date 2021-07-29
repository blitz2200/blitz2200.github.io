---
title: "Pyinstaller 를 이용하여 파이썬 코드 숨기기"
category:
  - DEVOPS
tag:
  - Python
  - Python 코드 숨기기
  - Docker
sidebar_main: true
author_profile: true
use_math: true
published: true
---

## Pyinstaller란?


py 파일을 실행파일로 만들어주는 프로그램이다.  실행파일로 만들어 주는 동시에 파이썬 코드를 볼 수 없기에 파이썬 코드를 숨겨서 배포하고자 할 경우에 유용하다.  덕분에 꽤 크고 많은 프로젝트에 두루 쓰이는 듯 하다.  [여기에 가면 어디에 쓰였는지 볼 수 있다.](https://github.com/pyinstaller/pyinstaller/wiki/Projects-Using-PyInstaller){: target="_blank"}


## PyPI에서 Pyinstaller 설치

```zsh
$ pip install pyinstaller
```

## Pyinstaller 실행

```zsh
$ pyinstaller main.py
```

위의 명령줄을 실행하면 build와 dist 폴더에 아래와 같이 결과물 파일들이 생성된다. 

![pyinstaller_bundle_sample](/images/pyinstaller_bundle_sample.png){:.align-center width="30%"}

dist 폴더 내의 main 파일이 실행파일이다. (MacOS 기준)

build 폴더는 빌드과정에서 쓰인 파일들이라 필요없고 dist폴더내의 다른 파일들은 전부 다 있어야 제대로 실행된다. 번잡해 보이지만 다행히 pyinstaller 는 아래와 같은 몇 가지 유용한 옵션들을 제공한다.

| Option | Description |
| --- | --- |
| -n NAME, --name NAME | 번들 앱이나 spec 파일의 이름을 지정해준다. 기본값은 첫 스크립트의 이름이다. |
| -F, --onefile | 하나의 실행파일로 만들어준다. |
| -w, --windowed, --noconsole | 표준입출력을 위한 콘솔을 제공하지 않는다. *NIX시스템에서는 적용되지 않는다. |
| --add-data <SRC;DEST or SRC:DEST> | 추가적인 non-binary 파일이나 폴더를 번들에 추가해주는 옵션이다. 경로 구분자가 플랫폼별로 다르다. 윈도우에서는 ; 그리고 맥이나 대부분의 유닉스시스템에서는 : 가 쓰인다. 여러번 반복해서 이 옵션을 추가해 줄 수 있다. |
| --hidden-import MODULENAME, --hidenimport MODULENAME | 스크립트 코드에 보이지 않는 모듈을 명시적으로 임포트 해 준다. |

[pyinstaller 공식문서 참조](https://pyinstaller.readthedocs.io/en/latest/){: target="_blank"}

```zsh
$ pyinstaller -F main.py
```

위와 같이 -F 옵션을 주면 dist/main 파일 하나만 생성이 된다.
 
사실 사용법 자체는 굉장히 간단하다. 하지만 실제 프로젝트에서 쓰다보면 여러가지 문제가 생기곤 했다. 실제 부딪혔던 문제들을 기록으로 남긴다.

## pyenv를 사용할 때 모듈 의존성 문제

pyenv 를 사용하여 python을 설치하여 사용하는 경우 빌드 자체는 잘 되지만 실행 파일을 실행 하는 경우 아래와 같이 main.py 파일 내에서 다른 모듈을 import 하고 있는 경우 아래와 같이 그 의존성을 찾지 못하는 에러가 난다. 생성된 파일의 용량 또한 정상적으로 빌드됐을 때 보다 현저하게 작다.

```zsh
Traceback (most recent call last):
  File "main.py", line 1, in <module>
ModuleNotFoundError: No module named 'flask'
[11787] Failed to execute script 'main' due to unhandled exception!
```
해결방법은 pyenv 를 이용해서 python을 설치할 때 --enable-shared 라는 옵션을 이용해서 설치해 주면 된다. [https://github.com/pyenv/pyenv/wiki#how-to-build-cpython-with---enable-shared](https://github.com/pyenv/pyenv/wiki#how-to-build-cpython-with---enable-shared, "pyenv official docs") 에서 자세한 내용을 확인할 수 있다

## 번들에서 외부파일 사용하도록 연결하기

코드상에 아래와 같이 외부 파일을 읽어오는 부분이 있다면 --add-data 옵션을 사용하여 해결할 수 있다.

``` python
@staticmethod
    def __load():
        config_file = 'conf/application.yml'

        with open(config_file, 'r') as stream:
            try:
                Configure.__conf = yaml.safe_load(stream)
            except yaml.YAMLError as e:
                print(e)
```
```zsh
$ pyinstaller --add-data "conf/application.yml:conf" ./main.py
```
그러면 dist 디렉토리 내에 conf 라는 디렉토리를 만들고 그 안에 application.yml 파일을 같이 포함 시켜 준다. 그런데 여기서 문제가 하나 있다. -F 옵션으로 하나의 파일로 패키징 하게 되면 해당 파일을 찾지 못하는 에러가 난다.

pyinstaller에서 하나의 파일로 패키징 된 실행파일을 실행하면 모든 파일들을 TEMP 디렉토리에 압축을 풀고 스크립트를 실행하게 된다. 임시 디렉토리의 경로는 매번 실행할 때마다 달라지지만 그 위치는 sys._MEIPASS 로 참조 가능하다. 즉 파이썬 코드에서 sys._MEIPASS 경로를 붙여주면 된다. 

```python
# data_files/data.txt
hello
world

# myScript.py
import sys
import os

def resource_path(relative_path):
    """ Get absolute path to resource, works for dev and for PyInstaller """
    try:
        # PyInstaller creates a temp folder and stores path in _MEIPASS
        base_path = sys._MEIPASS
    except Exception:
        base_path = os.path.abspath(".")

    return os.path.join(base_path, relative_path)

def print_file(file_path):
    file_path = resource_path(file_path)
    with open(file_path) as fp:
        for line in fp:
            print(line)

if __name__ == '__main__':
    print_file('data_files/data.txt')
```
```zsh
$ pyinstaller --onefile --add-data="data_files/data.txt;data_files" myScript.py
```
[스텍 오버 플로우 질문글 참조](https://stackoverflow.com/questions/51060894/adding-a-data-file-in-pyinstaller-using-the-onefile-option?answertab=votes#tab-top){:target="_blank"}

## --hidden-import 옵션

파이썬 스크립트 내에서는 import 된 것이 보이지 않지만 쓰이는 모듈들이 있다. 그런 모듈의 경우 dist 에 산출물로 포함되지 않아 모듈을 찾지 못한다는 에러가 나는 경우가 있는데 이런 경우 --hidden-import 옵션을 이용해 주면 된다.

```zsh
$ pyinstaller --hidden-import module_name main.py
```
