---
layout: post
categories: [miscellaneous]
title: 맥 환경에서 도커 세팅하고 사용하기
author: tngtied
date: 2023-11-01
---

## 도커란

도커는 반가상화를 지원하는 어플리케이션이다. 반가상화란 무거운 가상화 머신 없이도 원하는 환경에서 원하는 코드/executable을 돌릴 수 있게 해주는 걸 의미한다.
os 유형에 의존적인 system call을 사용하는 코드를 짜는데 linux 환경을 제시받았다고 해서 매번 데스크탑에서 무거운 vmware를 돌릴 필요가 없게 되는 것이다.

## brew로 도커 설치

GUI를 제공하는 cask를 함께 설치할 수 있고, 하지 않을 수도 있다.

`brew install --cask docker`

`brew install docker`

나의 경우 cask가 있는 쪽을 골랐다.

## 도커로 컨테이너 만들기, 호스트 시스템 마운트하기

컨테이너 생성 시 호스트 시스템 마운트를 병행하는 것을 권장한다.
컨테이너에 호스트 시스템 디렉토리를 직접적으로 마운트하는 것이 아닌,도커 volume을 생성하여 마운트시키는 방법도 존재한다. 해당 방법의 경우 host system에서 직접적인 조작이 어렵다는 단점이 존재한다. 도커 볼륨이 호스트 시스템으로부터 추상화(abstraction)되어 관리되기 때문이다. 그러나 이 추상화로 인하여 container 단위에서의 모듈화가 쉬워진다는 장점도 존재한다.

나는 호스트 시스템에 존재하는 코드 에디터를 사용하여 코드를 작성하는 것에 익숙하기 때문에, 생산성을 위해 직접 마운트하는 방법을 사용하였다.

사용된 명령어는 아래와 같다.

```
docker run -it --name container-name -v /existing/directory/in/host:/new/directory/in/container ubuntu:20.04 /bin/bash
```

`container-name` : 이 자리에 본인이 원하는 컨테이너 네임을 넣으면 된다.

`/existing/directory/in/host` : 이 자리에 컨테이너 내부와 즉시 동기화되길 바라는 호스트 시스템의, 존재하는 디렉토리 경로를 넣으면 된다.

`/new/directory/in/container` : 컨테이너 내부에 새로이 생성할 디렉토리의 경로이다. 나는 간단하게 /source로 입력해 접근하기 쉽도록 했다.

`ubuntu:20.04` : 원하는 os image를 넣으면 되는데, syntax는 기본적으로 os_name:version 으로 이와 같다. 지원하는 os image는 아래에서 확인 가능하다.
<a href="https://hub.docker.com/search?q="> hub.docker.com </a>

## 볼륨 생성해서 마운트하기

간단하게 볼륨을 생성해서 마운트하는 방법 또한 알아보겠다.
볼륨을 생성하는 커맨드는 일단 아래와 같다.

```
docker volume create py-volume
```

컨테이너 생성 시점에서 마운트하는 명령어는 아래와 같다.

```
docker run -v py-volume:/myapp -it --name py-env ubuntu:20.04 /bin/bash
```

미리 존재하는 컨테이너에 마운트하는 명령어는 아래와 같다.

## 도커 컨테이너 실행하기

생성 이후, 만약 기존에 생성한 컨테이너를 다시 사용하고 싶어졌을 때 사용하는 커맨드는 아래와 같다.

```
docker start my-container
```

이는 컨테이너를 실행시켜준다.

```
docker exec -it <container_name_or_id> /bin/bash
```

이는 실행된 컨테이너 내부의 터미널과 상호작용할 수 있게 해 준다.

만약 동일한 환경에서 여러 개의 터미널를 띄우고 싶다면 호스트 시스템 상의 터미널을 여러 창 띄워두고 위의 exec 커맨드를 각각 입력하면 된다.

## 도커 컨테이너 내부 조작

나는 해당 환경에서 python 3.9.13이 돌아가도록 만들어야 했다. 이를 위해 아래와 같은 커맨드를 터미널 상에서 돌렸다.

```
apt update
apt install python3.9
```

## 호스트 시스템 포트와 도커 내부 포트 연결하기

포트 포워딩은 오로지 docker run 혹은 create command로만 가능하다. 그 말인 즉슨, 이미 생성된 컨테이너는 port binding이 이론상 불가능하다는 것이다. 하지만 말이 이론상이지, 약간의 꼼수만 쓴다면 가능하다.

먼저 docker run과 create에서 포워딩하는 방법을 하술하겠다.

### container 생성 시 포트 포워딩하기

```
docker run -p host_port:docker_port docker_image
```

기본적인 문법은 위와 같다. `host_port` 자리에 호스트 시스템에서 열 포트를 넣고, `docker_port`자리에 컨테이너 내부에서 사용할 포트를 넣으면 된다. `docker_image` 자리에는, 상술했다시피, ubuntu:20.04 같은 specific한 이미지 종류를 넣는다.

만약 프로토콜을 명시하고 싶다면 아래와 같이 `/tcp` 혹은 `/udp`를 뒤에 붙여주면 된다.

```
docker run -p host_port:docker_port/tcp docker_image
```

### 이전에 생성된 컨테이너에 포트 포워딩하기

미리 존재하는 컨테이너를 포워딩하는 방법은 아래와 같다. 대상이 될 컨테이너가 정지된 상태이어야만 한다.

```
docker commit container_name new_image_name
```

해당 커맨드는 `container_name`의 이름을 가진 컨테이너를 기반으로 `new_image_name`이라는 이름의 커스텀 도커 이미지를 생성해준다.

```
docker run -p host_port:docker_port new_image_name
```

이후 `new_image_name`을 기반으로 포트 포워딩 옵션을 넣어 새로운 컨테이너를 만들어주면 된다.

이 외의 옵션이 궁금하다면 아래의 문서를 참고하면 좋다.
<a href="https://docs.docker.com/network/"> docx.docker.com/network </a>
