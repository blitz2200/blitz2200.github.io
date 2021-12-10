---
title: "Django 프로젝트를 쿠버네티스에 배포하기"
categories:
  - PYTHON
tags:
  - Python
  - 파이썬
  - django
  - 장고
  - kubernetes
  - k8s
  - 쿠버네티스
  - nginx
sidebar_main: false
author_profile: true
use_math: true
published: false
---

# [Django 배포하기](https://docs.djangoproject.com/ko/3.2/howto/deployment/){: target="_blank"}

웹 프레임워크인 Django는 작동하기 위해 웹 서버가 필요하다. 그리고 대부분의 웹 서버는 기본적으로 Python을 사용하지 않기 때문에, 우리는 Django와 웹서버 사이의 커뮤니케이션이 이루어지도록 인터페이스가 필요하다.

Django는 현재 WSGI(Web Server Gateway Interface)와 ASGI(Asynchronous Server Gateway Interface)의 두 가지 인터페이스를 지원한다.
* WSGI는 웹 서버와 애플리케이션 간의 통신을 위한 주요 파이썬 표준이지만 동기식 코드만 지원한다.
* ASGI는 Django 사이트에서 비동기 Python 기능과 비동기 Django 기능을 개발하면서 사용할 수 있게 해주는 새로운 비동기 친화적인 표준이다.

ASGI는 request/response로 이루어진 WSGI와 달리 send/ receive 로 되어 있어 비동기적으로 이벤트 처리가 가능하다. 그로인해 여러 송수신 이벤트가 가능하다. 또한 WSGI의 상위 집합으로 설계되어있으며 asgiref 라이브러리를 통해 ASGI 서버 내에서 WSGI를 실행 할 수도 있다고 한다.

우리는 비동기 친화적이면서 WSGI와 호환도 되는 [ASGI](https://asgi.readthedocs.io/en/latest/introduction.html){: target="_blank"}를 사용하기로 한다.

[ASGI와 WSGI 분석](https://nitro04.blogspot.com/2020/01/django-python-asgi-wsgi-analysis-of.html){: target="_blank"}





# ASGI를 사용하여 배포하는 방법

Django에는 다음 ASGI 서버에 대한 시작 설명서가 포함되어 있다.
* [다프네와 함께 django를 사용하는 방법](https://docs.djangoproject.com/ko/3.2/howto/deployment/asgi/daphne/){: target="_blank"}
* [하이퍼콘과 함께 짱고를 사용하는 방법](https://docs.djangoproject.com/ko/3.2/howto/deployment/asgi/hypercorn/){: target="_blank"}
* [django와 유비콘 사용방법](https://docs.djangoproject.com/ko/3.2/howto/deployment/asgi/uvicorn/){: target="_blank"}

[어느 방법을 쓸것인가?](https://stackoverflow.com/questions/61700881/django-3-x-which-asgi-server-uvicorn-vs-daphne){: target="_blank"} 

위 링크를 참고하여 가볍고 심플한 uvicorn을 사용하기로 한다.

Uvicorn은 속도를 중시하는 《uvloop》과 《http tools》에 기반을 둔 ASGI 서버이다. 그리고 Uvicorn은 프로세스 매니저가 아니고 gunicorn의 워커로 쓰인다.


# GUNICORN 그리고 NGINX

> Gunicorn 'Green Unicorn' is a Python WSGI HTTP Server for UNIX. It's a pre-fork worker model.
> The Gunicorn server is broadly compatible with various web frameworks,
> simply implemented, light on server resources, and fairly speedy.

위는 Gunicorn 공홈에 있는 문구이다.

Gunicorn은 Python WSGI로 WEB Server(Nginx, Apache 등)로부터 서버사이드 요청을 받으면 WSGI (또는 ASGI)를 통해 서버 애플리케이션(Django, Flask 등)으로 전달해주는 역할을 수행한다. Django의 runserver 역시도 똑같은 역할을 수행하지만 단일 프로세스, 단일 쓰레드라서 production 환경에서는 사용하기에는 맞지 않다.(개발용으로는 유용하다)

gunicorn의 프로세스는 프로세스 기반의 처리 방식을 채택하고 있으며, 이는 내부적으로 크게 master process와 worker process로 나뉘어 집니다. gunicorn이 실행되면, 그 프로세스 자체가 master process이며, fork(완전히 분리된 *nix 프로세스)를 사용하여 설정에 부여된 worker 수대로 worker process가 생성 됩니다. master process는 worker process를 관리하는 역할을 하고, worker process는 웹어플리케이션을 임포트하며, 요청을 받아 웹어플리케이션 코드로 전달하여 처리하도록 하는 역할을 한다.

gunicorn만 있어도 http request를 처리할 수는 있지만, Gunicorn에는 없고 nginx에는 있는 기능 때문에 보통 둘을 연동해서 쓴다.
1. Django media, css등 static한 요청은 직접처리하고 다이나믹한 요청을 gunicorn에 넘긴다. Gunicorn으로 넘어가는 순간 자원사용이 크게늘어 스태틱한 요청을 따로 처리해주는게 중요하다.

2. nginx는 c로 구현되어 속도와 메모리 사용 측면에서 뛰어나다.
둘을 연동하면 동시에 많은 요청을 처리할 수 있고, 훨씬 안정화 된 서버를 구축 할 수 있게 된다.

# Django 앱을 위한 docker-compose

도커를 이용해서 Django 앱을 배포하고 앞단에 proxy로 nginx를 붙여주기 위해서 docker-compose 같은 서비스를 이용하여 묶어줄 수 있다.

Django는 runserver를 이용시 기본적으로 css나 image 같은 static 파일들을 static이라는 경로에 모아놓고 serve 한다. 하지만 Gunicorn 을 사용하고 nginx를 앞단에 붙여 static파일을 serve 하게 하려면 ```python3 manage.py collectstatic```  를 이용해 static 파일들을 특정경로 - {project_root}/.static_root 에 모아주고 해당 파일들을 /static 이라는 URL로 serve 할 수 있도록 nginx 도 설정을 해 주어야 한다.

아래의 nginx 설정파일을 nginx 컨테이너의 /etc/nginx/conf.d 경로에 추가되도록 아래와 같이 설정한다.

* nginx.conf
```zsh
upstream django {
    ip_hash;
    server 127.0.0.1:8000;
}

server {
    listen 80;

    location / {
        proxy_pass http://django;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /static/ {
        alias /static/;
    }
}
```
* Dockerfile
```dockerfile
FROM --platform=linux/amd64 python:3.8.11
RUN apt-get update && apt-get upgrade -y
ENV PYTHONUNBUFFERED=1
WORKDIR /app
COPY requirements.txt ./
RUN pip install --upgrade pip && pip install --no-cache-dir -r requirements.txt
COPY . .
```
* docker-compose.yaml
```zsh
version: "2"
services:
  telstar-auth:
    image: telstar-auth
    container_name : telstar-auth
    volumes: 
      -  path-to-static_root:/app/.static_root
    command: > 
      bash -c "python3 manage.py makemigrations
      && python3 manage.py migrate
      && python3 manage.py collectstatic --noinput
      && gunicorn config.asgi:application -b 0.0.0.0:8000 -k uvicorn.workers.UvicornWorker"
    ports: 
      - "8000:8000"
  nginx:
    image: nginx
    container_name : nginx
    volumes: 
      - path-to-config:/etc/nginx/conf.d
      - path-to-static_root/:/static
    ports:
      - "80:80"
    depends_on : 
      - telstar-auth
```
telstar-auth 서비스의 path-to-static_root 경로와 nginx 서비스의 path-to-static_root 는 Host의 같은 경로를 가리킴에 주의 한다.

docker-compose.yaml 파일 내의 command 부분을 좀 더 들여다 보겠다.
```zsh
gunicorn config.asgi:application -b 0.0.0.0:8000 -k uvicorn.workers.UvicornWorker
```
위 command를 보면 Gunicorn을 이용해 Django 앱을 실행시키고 있으며 worker로 uvicornWorker를 지정해 주었다. 물론 requirements.txt 내에 gunicorn uvicorn 이 기술 되어 있어 도커 이미지 내에서 해당 파이썬 패키지가 설치 되어 있어야 함에 주의 한다. gunicorn 뒤에 config.asgi:application으로 적어주는 이유는 ```django-admin startproject app_name```으로 Django 앱 스케폴딩시 생성되는 해당 앱 내의 asgi.py 내의 application을 가리키기 위함이고(위 예제에서는 앱이름이 config 이다.) -b 옵션으로 ip 및 포트를 바인딩 해 준다.



# Django 앱을 위한 쿠버네티스 Manifest
* Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: telstar-auth
  namespace: telstar
  labels:
    app: telstar-auth
    project: telstar-auth
spec:
  replicas: 1
  selector:
    matchLabels:
      app: telstar-auth #관리할 Pod의 검색조건
  template:
    metadata:
      labels:
        app: telstar-auth #생성할 Pod의 라벨
        project: telstar-auth
        service-name : telstar-auth  #grafana dashboard 조회용
    spec:
      containers: 
      - name: telstar-auth-django
        image: docker.enkis.co.kr/telstar-auth:latest
        envFrom:
          - secretRef:
              name: telstar-auth-secret
          - configMapRef:
              name: telstar-auth-configmap
        volumeMounts:
        - name: telstar-auth-static
          mountPath: /app/.static_root
        - name: telstar-auth-nginx-config
          mountPath: /app/nginx-config
        - name: telstar-auth-log
          mountPath: /app/logs
        ports:
        - containerPort: 8000
        command: ['/bin/bash']
        args: ["-c", "python3 manage.py makemigrations && python3 manage.py migrate
          && python3 manage.py collectstatic --no-input && gunicorn config.asgi:application
       --bind 0.0.0.0:8000 -k uvicorn.workers.UvicornWorker"]
      - name: telstar-auth-nginx
        image: nginx
        imagePullPolicy: IfNotPresent
        volumeMounts:
          - mountPath: /static
            name: nginx-static
          - mountPath: /etc/nginx/conf.d
            name: nginx-config
          - mountPath: /var/log/nginx
            name: nginx-log
        ports:
          - containerPort: 80
      imagePullSecrets:
        - name: enkis-nexus
      nodeSelector:
        process-unit-kind: cpu
        site: telstar
      volumes:
        - name: telstar-auth-static #shared
          hostPath:
            path: /home/ubuntu/telstar-auth/static
        - name: telstar-auth-nginx-config #shared
          hostPath:
            path: /home/ubuntu/telstar-auth/nginx-config
        - name: telstar-auth-log
          hostPath:
            path: /home/ubuntu/telstar-auth/nginx-log
        - name: nginx-static #shared
          hostPath:
            path: /home/ubuntu/telstar-auth/static
        - name: nginx-config #shared
          hostPath:
            path: /home/ubuntu/telstar-auth/nginx-config
        - name: nginx-log
          hostPath:
            path: /home/ubuntu/telstar-auth/django-log
```
* Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: telstar-auth
  namespace: telstar
  labels:
    app: telstar-auth
spec:
  type: NodePort
  selector:
    app: telstar-auth
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30000
```
* ConfigMap
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: telstar-auth-configmap
  namespace: telstar
data:
  DEBUG: 'False'
  ALLOWED_HOSTS: '*'
  ACCESS_TOKEN_LIFETIME_MINUTES: '5'
  REFRESH_TOKEN_LIFETIME_DAYS: '1'
```
* Secret
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: telstar-auth-secret
  namespace: telstar
type: Opaque
data:
  DB_ENGINE: ZGphbm--base64 encoded value--cmVzcWw=
  DB_NAME: dV--base64 encoded value--cg==
  DB_HOST: dGVsc--base64 encoded value--5jb20=
  DB_USER: cG--base64 encoded value--XM=
  DB_PASSWORD: RW8--base64 encoded value--SUnk=
  JWT_SIGNING_KEY: dGVs--base64 encoded value--a2V5
```
Django에서 사용하는 env파일은 필요한 보안 정도에 따라 secret(PW,SECRET KEY 등)과 configmap(DEBUG, ALLOWED HOST등) 으로 나누어 설정 한다.






참고링크 [Django 프로젝트 K8S 배포하기](https://velog.io/@seokbin/Django-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-K8S-%EB%B0%B0%ED%8F%AC%ED%95%98%EA%B8%B0){: target="_blank"}