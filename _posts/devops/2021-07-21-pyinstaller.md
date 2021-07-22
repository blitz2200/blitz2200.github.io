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

Pyinstaller란?


py 파일을 실행파일로 만들어주는 프로그램이다.  실행파일로 만들어 주는 동시에 파이썬 코드를 볼 수 없기에 파이썬 코드를 숨겨서 배포하고자 할 경우에 유용하다.  덕분에 꽤 크고 많은 프로젝트에 두루 쓰이는 듯 하다.  [여기에 가면 어디에 쓰였는지 볼 수 있다.](https://github.com/pyinstaller/pyinstaller/wiki/Projects-Using-PyInstaller){: target="_blank"}


## 1. PyPI에서 Pyinstaller 설치

```console
$ pip install pyinstaller
```

## 2. 프로그램 폴더에 가서 실행

```console
$ pyinstaller main.py
```

위의 명령줄을 실행하면 build와 dist 폴더에 아래와 같이 결과물 파일들이 생성된다.  
![pyinstaller_bundle_sample](/images/pyinstaller_bundle_sample.png)
dist 폴더 내의 main 파일이 실행파일이다. (MacOS 기준)

build 폴더는 빌드과정에서 쓰인 파일들이라 필요없고 dist폴더내의 다른 파일들은 전부 다 있어야 제대로 실행된다. 번잡해 보이지만 다행히 pyinstaller 는 아래와 같은 몇 가지 유용한 옵션들을 제공한다.

* -F, --onefile  
: 하나의 실행파일로 만들어준다.
* -w, --windowed, --noconsole  
: 표준입출력을 위한 콘솔을 제공하지 않는다. *NIX시스템에서는 적용되지않는다.


```console
$ pyinstaller -F main.py
```
위와 같이 -F 옵션을 주면 dist/main 파일 하나만 생성이 된다.
 
사실 사용법 자체는 굉장히 간단하다. 하지만 실제 프로젝트에서 쓰다보면 여러가지 문제가 생기곤 했다. 실제 부딪혔던 문제들을 기록으로 남긴다.

## 3. pyenv를 사용할 때 모듈 의존성 문제

pyenv 를 사용하여 python을 설치하여 사용하는 경우 빌드 자체는 잘 되지만 실행 파일을 실행 하는 경우 아래와 같이 main.py 파일 내에서 다른 모듈을 import 하고 있는 경우 아래와 같이 그 의존성을 찾지 못하는 에러가 난다. 생성된 파일의 용량 또한 정상적으로 빌드됐을 때 보다 현저하게 작다.

```console
Traceback (most recent call last):
  File "main.py", line 1, in <module>
ModuleNotFoundError: No module named 'flask'
[11787] Failed to execute script 'main' due to unhandled exception!
```


해결방법은 pyenv 를 이용해서 python을 설치할 때 --enable-shared 라는 옵션을 이용해서 설치해 주면 된다. 자세한 내용은 링크로 갈음한다.
[https://github.com/pyenv/pyenv/wiki#how-to-build-cpython-with---enable-shared](https://github.com/pyenv/pyenv/wiki#how-to-build-cpython-with---enable-shared, "pyenv official docs")

## 4.