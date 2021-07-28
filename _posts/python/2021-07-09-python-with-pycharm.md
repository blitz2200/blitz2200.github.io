---
title: "맥북프로 M1에서 PYCHARM으로 개발환경 설정하기"
category:
  - PYTHON
tag:
  - Python
  - PYCHARM
  - Virtualenv
  - venv
  
sidebar_main: false
author_profile: true
use_math: true
published: true
---

## 파이썬 설치

파이썬2, 파이썬3… 어떤버전을 써야할까 고민하지 말라고 누가 pyenv 라는걸 만들어 놨다. 다양한 버전의 파이썬을 설치하고 골라서 쓸수 있게 해주는 거다. sdkman 이나 nvm 비슷한 거다. 맥에서는 brew로 설치하면 편하다.
```
$ brew update
$ brew install pyenv
```

설치 후 쓰고 있는 쉘에 맞게 설정을 해줘야 한다.

* Zsh 의 경우 아래를 입력하고 엔터.
```
echo 'eval "$(pyenv init -)"' >> ~/.zshrc
```

사용법은 쉽다.

````
pyenv install -l       //설치 가능한 목록을 표시

pyenv install 3.9.6    //해당 버전을 설치한다. 설치되는 경로는 ~/.pyenv/versions

cat ~/.pyenv/versions  //설치된 버전 조회

pyenv local 3.9.6      //실행한 폴더 내에 .python-version 파일이 생성되며 해당폴더 내에서는 .python-version 내에 명시된 python 버전이 사용됨   
````

## 가상환경 설정하기

파이썬에는 가상환경이라는게 있다. 노드의 node_modules 폴더처럼 프로젝트별로 디펜던시를 관리할 수 있게 해주는 것으로 이해하면 편하다. 프로젝트 마다 진행된 시기별로 다양한 버전의 라이브러리를 가져다 쓸텐데 각 시기별로 쓰인 라이브러리가 최신라이브러리와는 호환이 안되는 등 디펜던시들이 꼬이는 일이 생길 수 있는데 가상환경을 사용하면 프로젝트별로 독립적으로 환경을 잡아주므로 위의 문제에서 자유롭다.

가상환경 잡는 방법은 꽤나 다양하다.

#### venv? virtualenv?
venv는 virtualenv의 subset이다. 즉 venv는 virtualenv의 일부기능인 거다. 그리고 이 venv는 파이썬 3.3버전부터 파이썬에 통합되었다. 그래서 venv를 업그레이드 하려면 파이썬 자체를 업그레이드 해야한다. 반면에 virtualenv는 pip로 설치하고 업데이트 된다.