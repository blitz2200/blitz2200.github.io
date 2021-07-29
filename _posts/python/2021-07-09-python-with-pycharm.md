---
title: "맥북프로 M1에서 PyCharm으로 개발환경 설정하기"
categories:
  - PYTHON
tags:
  - Python
  - PyCharm
  - Virtualenv
  - venv
sidebar_main: false
author_profile: true
use_math: true
published: true
---

# 파이썬 설치

파이썬2, 파이썬3… 어떤버전을 써야할까 고민하지 말라고 누가 pyenv 라는걸 만들어 놨다. 다양한 버전의 파이썬을 설치하고 골라서 쓸수 있게 해주는 거다. sdkman 이나 nvm과 비슷한 거다. 맥에서는 brew로 설치하면 편하다.
```zsh
$ brew update
$ brew install pyenv
```

설치 후 쓰고 있는 쉘에 맞게 설정을 해줘야 한다.<br>
Zsh 의 경우 아래를 입력하고 엔터.
```zsh
echo 'eval "$(pyenv init -)"' >> ~/.zshrc
```

사용법은 쉽다.

````zsh
$ pyenv install -l       # 설치 가능한 목록을 표시
$ pyenv install 3.9.6    # 해당 버전을 설치한다. 설치되는 경로는 ~/.pyenv/versions
$ cat ~/.pyenv/versions  # 설치된 버전 조회
$ pyenv local 3.9.6      # 실행한 폴더 내에 .python-version 파일이 생성되며
                         #  해당폴더 내에서는 .python-version 내에 명시된 python 버전이 사용됨   
````

# 가상환경 구성

파이썬에는 가상 환경이라는게 있다. 노드의 node_modules 폴더처럼 프로젝트별로 디펜던시를 관리할 수 있게 해주는 것으로 이해하면 편하다. 프로젝트 마다 진행된 시기별로 다양한 버전의 라이브러리를 가져다 쓸텐데 각 시기별로 쓰인 라이브러리가 최신라이브러리와는 호환이 안되는 등 디펜던시들이 꼬이는 일이 생길 수 있는데 가상환경을 사용하면 프로젝트별로 독립적으로 환경을 잡아주므로 위의 문제에서 자유롭다. 가상환경 잡는 방법은 꽤나 다양하다.

* venv : venv는 virtualenv의 subset, Python 3.3 버전 이후 부터 기본모듈에 포함됨
* virtualenv : Python 2 버전부터 사용해오던 가상환경 라이브러리, Python 3에서도 사용가능
* conda : Anaconda Python을 설치했을 시 사용할 수있는 모듈
* pyenv : pyenv의 경우 Python Version Manger임과 동시에 가상환경 기능을 플러그인 형태로 제공<br><br>

## venv를 이용하여 가상환경 잡기
<br>
가상 환경을 잡을 경로에 가서 아래 명령을 실행한다.<br><br>

```zsh
$ python -m venv venv
```

첫번째 venv 는 venv라는 파이썬 모듈이름이고 뒤의 이름은 지정하고자 하는 가상환경의 폴더이름이다.
쉘에서 위 명령을 실행하면 venv 라는 폴더가 생성되고 아래와 같은 파일들이 생성된다.<br><br>

![venv_filetree](/images/pycharm_venv_filetree.png){:.align-center width="30%"}

디렉토리 구조를 보면 venv/bin 아래에 실행파일들이 있고 dependency들이 설치 되는 venv/lib/python3.x/site-packages 폴더가 있다.
가상환경이 생성은 되었지만 현재 쉘에서 그 가상환경에 진입하도록 활성화를 시켜 주어야 한다. 그러지 않으면 그냥 이 상태로 쉘에서 python 명령을 치거나 pip install을 했을때 가상환경에 있는 python 을 사용하지 않고 쉘의 path에 등록된 python을 사용하게 된다.
아래 명령어로 가상환경을 활성화 해 준다.

```zsh
$ source venv/bin/activate   #쉘에 맞는 것을 선택해서 실행
(venv) $ 
```

가상환경이 활성화 되면 위와 같이 쉘 명령프롬프트 앞에 가상환경이름이 괄호와 함께 붙는다. 다시 비활성화 하고 싶다면

```zsh
$ deactivate  #이미 가상환경내의 bin 폴더가 PATH로 잡혀 있으므로 경로에 상관없이 실행가능
$ 
```

# PyCharm 에서 Virtualenv를 이용한 가상 환경 구성

* 기존 프로젝트에서 가상 환경 구성

기존 프로젝트를 PyCharm 에서 열면 오른쪽 하단에 아래 그림과 같이 python version 나와있는 곳을 눌러 interpreter settings... 라고 되어 있는 context 메뉴를 클릭한다. preference 에서 python interpreter 로 검색 해도 된다.

![pycharm_tray](/images/pycharm_interpreter_tray.png){:.align-center width="30%"}


창 우측 상단의 톱니 버튼을 눌러 나온 context 메뉴의 Add... 를 클릭한다.

![pycharm_interpreter_setting](/images/pycharm_interpreter_setting.png){:.align-center width="80%"}

그러면 아래와 같이 여러 형태의 가상환경을 만들 수 있는 창이 뜬다.

![pycharm_interpreter_add](/images/pycharm_interpreter_add.png){:.align-center width="80%"}

PyCharm 에는 Virtualenv 가 번들로 포함되어 있어서 Virtualenv를 따로 설치하지 않아도 Virtualenv 를 이용하여 가상 환경을 잡아줄 수 있다. New environment 의 Location 에는 가상환경의 폴더 위치를 지정해 주고 Base interpreter 에서 원하는 파이썬 바이너리를 선택해 준다. 시스템에 설치된 파이썬을 선택할 수도 있고 pyenv 를 이용해 설치한 파이썬 바이너리를 선택할 수도 있다. 물론 Existing environment를 선택해 기존에 생성한 가상환경을 선택할 수 도 있다.

* 새 프로젝트 만들면서 가상 환경 구성

새 프로젝트를 만들 때에도 아래와 같이 가상환경 설정이 가능하다.

![pycharm_create_project](/images/pycharm_create_project.png){:.align-center width="80%"}

# PyCharm 에서 Docker / Docker Compose 를 이용한 가상 환경 구성