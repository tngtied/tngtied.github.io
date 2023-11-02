---
layout: post
categories: miscellaneous
title: 맥 환경에서 도커 세팅하고 사용하기
author: tngtied
date: 2023-11-01
---

## 도커란

도커는 반가상화를 지원하는 어플리케이션이다. 반가상화란 무거운 가상화 머신 없이도 원하는 환경에서 원하는 코드/executable을 돌릴 수 있게 해주는 걸 의미한다.
os 유형에 의존적인 system call을 사용하는 코드를 짜는데 linux 환경을 제시받았다고 해서 매번 데스크탑에서 무거운 vmware를 돌릴 필요가 없게 되는 것이다.

## brew로 도커 설치
GUI를 제공하는 cask를 함께 설치할 수 있고, 하지 않을 수도 있다.

{%highlight%}
brew install --cask docker
{%endhighlight%}
{%highlight%}
brew install docker
{%endhighlight%}

나의 경우 cask가 있는 쪽을 골랐다.

## 도커로 컨테이너 만들기, 호스트 시스템 마운트하기

컨테이너 생성 시 호스트 시스템 마운트를 병행하는 것을 권장한다. 
컨테이너에 호스트 시스템 디렉토리를 직접적으로 마운트하는 것이 아닌,도커 volume을 생성하여 마운트시키는 방법도 존재한다. 해당 방법의 경우 host system에서 직접적인 조작이 어렵다는 단점이 존재한다. 도커 볼륨이 호스트 시스템으로부터 추상화(abstraction)되어 관리되기 때문이다. 그러나 이 추상화로 인하여 container 단위에서의 모듈화가 쉬워진다는 장점도 존재한다. 

나는 호스트 시스템에 존재하는 코드 에디터를 사용하여 코드를 작성하는 것에 익숙하기 때문에, 생산성을 위해 직접 마운트하는 방법을 사용하였다.

사용된 명령어는 아래와 같다.

{%highlight%}
docker run -it --name container-name -v /existing/directory/in/host:/new/directory/in/container ubuntu:20.04 /bin/bash
{%endhighlight%}

container-name : 이 자리에 본인이 원하는 컨테이너 네임을 넣으면 된다.
/existing/directory/in/host : 이 자리에 컨테이너 내부와 즉시 동기화되길 바라는 호스트 시스템의, 존재하는 디렉토리 경로를 넣으면 된다.
/new/directory/in/container : 컨테이너 내부에 새로이 생성할 디렉토리의 경로이다. 나는 간단하게 /source로 입력해 접근하기 쉽도록 했다.
ubuntu:20.04 : 원하는 os image를 넣으면 되는데, syntax는 기본적으로 os_name:version 으로 이와 같다. 지원하는 os image는 아래에서 확인 가능하다.
<a href="https://hub.docker.com/search?q="> hub.docker.com </a>

## 볼륨 생성해서 마운트하기

간단하게 볼륨을 생성해서 마운트하는 방법 또한 알아보겠다.
볼륨을 생성하는 커맨드는 일단 아래와 같다.
{%highlight%}
docker volume create py-volume
{%endhighlight%}

컨테이너 생성 시점에서 마운트하는 명령어는 아래와 같다.
{%highlight%}
docker run -v py-volume:/myapp -it --name py-env ubuntu:20.04 /bin/bash
{%endhighlight%}

미리 존재하는 컨테이너에 마운트하는 명령어는 아래와 같다. 

## 도커 컨테이너 실행하기

생성 이후, 만약 기존에 생성한 컨테이너를 다시 사용하고 싶어졌을 때 사용하는 커맨드는 아래와 같다. 

{%highlight%}
docker start my-container
{%endhighlight%}
이는 컨테이너를 실행시켜준다.

{%highlight%}
docker exec -it <container_name_or_id> /bin/bash
{%endhighlight%}
이는 실행된 컨테이너 내부의 터미널과 상호작용할 수 있게 해 준다.

만약 동일한 환경에서 여러 개의 터미널를 띄우고 싶다면 호스트 시스템 상의 터미널을 여러 창 띄워두고 위의 exec 커맨드를 각각 입력하면 된다.

## 도커 컨테이너 내부 조작

나는 해당 환경에서 python 3.9.13이 돌아가도록 만들어야 했다. 이를 위해 아래와 같은 커맨드를 터미널 상에서 돌렸다.

{%highlight%}
apt update
apt install python3.9
{%endhighlight%}
