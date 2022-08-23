---
title: "Github Actions"
category:
  - DEVOPS
tag:
  - CI/CD
  - Github Action
sidebar_main: true
author_profile: true
use_math: true
published: true
---

## Github Actions 란?


github action을 위한 설정파일은 github repository root 폴더 내에 .github/workflows 폴더 내에 xxx.yml 파일의 형태로 추가한다. 그러면 github repository의 Actions 탭에서 workflows 목록에 뜨게 된다. 물론 복수개의 yml파일을 추가해도 된다.


```yaml
name: nipa-visualization

on:
  #  push:
  #    branches:  
  #      - main  
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-20.04

    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v1

      - name: Login to Enkis Docker Registry
        uses: docker/login-action@v1
        with:
          registry: docker.enkis.co.kr
          username: ${{ secrets.ENKIS_REGISTRY_USERNAME }}
          password: ${{ secrets.ENKIS_REGISTRY_PASSWORD }}

      - name: Build and push
        id: docker_build
        uses: docker/build-push-action@v2
        with:
          push: true
          tags: |
            docker.enkis.co.kr/nipa-visualization-server:latest
            docker.enkis.co.kr/nipa-visualization-server:${{ github.sha }}
  deploy:
    needs: build
    name: Deploy
    runs-on: ubuntu-20.04
    steps:
      - name: deploy to cluster
        uses: steebchen/kubectl@v2.0.0
        with:
          config: ${{ secrets.KUBE_CONFIG }}
          command: set image --namespace=visualization deployments/visualization-server-rams-deployment visualization-server=docker.enkis.co.kr/nipa-visualization-server:${{ github.sha }}
      - name: verify deployment
        uses: steebchen/kubectl@v2.0.0
        with:
          config: ${{ secrets.KUBE_CONFIG }}
          version: v1.21.5 # specify kubectl binary version explicitly          command: rollout status --namespace=visualization deployments/visualization-server-rams-deployment
```

name: 워크플로우 이름이다. 원하는 이름을 지어주면 된다.

on: 이 워크플로우가 실행되는 트리거를 지정할 수 있다. workflow_dispatch 는 github 홈페이지에서 수동으로 트리거 할 경우 쓰이는 옵션이며 특정 브랜치에서 푸시가 있을경우 실행되게 할수도 있는 등 여러가지 옵션이 있으며 github actions 문서를 참고하자.

jobs: 실행할 일들을 목록으로 추가할 수 있다. 위 예제에서는 build 와 deploy 라는 job을 정의하고 있다.

## jobs 내부 세부옵션

name: 해당 Job의 이름이다. 적절한 이름을 붙여준다.

runs-on: 해당 action이 실행될 runner 이다. 위 예제에서는 ubuntu-20.04 runner가 지정되어 있다. 정해진 runner 외에 사용자가 직접 만든 costom runner 또한 설정 가능하다. [About self-hosted runners - GitHub Docs](https://docs.github.com/en/actions/hosting-your-own-runners/about-self-hosted-runners){: target="_blank"} 여기를 참고한다.

needs: 해당 job이 실행 되기 전에 충족해야 하는 의존성으로 위 예제에서는 build step 이다.

steps: job 은 순차적으로 실행되는 여러 스텝으로 이루어 진다.

&nbsp;&nbsp;name: 해당 스텝의 이름이다. 적절한 이름을 붙여준다.
&nbsp;&nbsp;uses: 어떠한 action을 쓸지 적어준다. 필요한 action 목록은 [GitHub Marketplace: actions to improve your workflow](https://docs.github.com/en/actions/hosting-your-own-runners/about-self-hosted-runners){: target="_blank"} 에서 찾을수 있다.
&nbsp;&nbsp;with: 해당 action에서 필요한 설정값들을 적어준다. 필요한 설정값은 action 마다 다르며 해당 action 의 상세 페이지를 참고한다.

{% raw %}
변수 부분은 ${{  변수  }} 로 표현할 수 있으며 ${{ github.sha }}  의 형태로 github에서 제공하는 sha  를 변수로 제공받을 수 있다. secrets.   으로 접근하면 해당 repository 나 organization 의 Settings > Security > Secrets 에 정의되는 변수에 접근 가능하다.
{% endraw %}