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


## 1. Pyinstaller 설치

```console
$ pip install fastapi

$ pip install uvicorn
```

- uvicorn 은 ASGI Web 어플리케이션 서버 이다. 내장 모듈로 [uvloop](https://github.com/MagicStack/uvloop, "uvloop link") (liuv + Cyphon) 을 사용 한다.
- uvicorn에 대한 자세한 내용은 [https://www.uvicorn.org](https://www.uvicorn.org, "uvicorn link") 에서 볼 수 있다.

## 2. 실행해보기

- 공홈에 있는 hello world 소스

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
async def root():
    return {"message": "Hello World"}
```

- 아래 명령어로 실행한다.


```console
$ uvicorn main:app --reload
```

- Fastapi는 자체적으로 api 문서 기능을 제공한다.(fast api 공홈 참조)

![swagger image](https://fastapi.tiangolo.com/img/index/index-01-swagger-ui-simple.png)