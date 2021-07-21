---
title: "FastAPI"
category:
  - PYTHON
tag:
  - Web
  - FastAPI
  - Python
sidebar_main: true
author_profile: true
use_math: true
published: true
---

FastAPI란?

> FastAPI는 현대적이고, 빠르며(고성능), 파이썬 표준 타입 힌트에 기초한 Python3.6+의 API를 빌드하기 위한 웹 프레임워크입니다. (공홈 참고)


## 1. FastAPI 설치

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