---
layout: post
categories: cppstudy
title: assignment of 10_13
author: tngtied
date: 2023-10-14
---


## 블로그 만들기


페이지를 띄웠을 때 포스팅 페이지 endpoint가 .html로 끝나는 게 이상하다. 분명 나는 .md 파일에 작성하고 있는데도 그러하기 때문이다. 이런 스태틱 웹페이지를 작성한지 너무 오래돼서 path를 어떻게 설정하는지 개인적으로 다시 한 번 알아봤다. 

작성중인 .md file 윗 부분에 아래와 같은 문법으로 작성하고
```
layout: post
categories: cppstudy
title: assignment of 10/13
author: tngtied
date: 2023-10-14
```
_config.yml file 내부에서 아래와 같이 선언해주면 endpoint가 지저분하게 보이던 것이 사라진다.
```
permalink: /:categories/:title
```

또한 목록 선언 시 url path를 아래와 같이 수정하는 것을 잊지 말아야 한다. 나의 프로젝트의 경우 pages.yml 내부에 선언되어 있었다. 참고로 baseurl은 _config.yml안에 정의되어 있으며, 보통 "/"가 디폴트이다.
```
<span class="post-date">
    <a href="\{\{ site.baseurl \}\}\{\{page.categories\}\}\{\{ page.url\}\}">
        # \{\{ page.date | date_to_string \}\} by \{\{ page.author \}\}
    </a>
</span>
```

고양이 사진의 layout 문제로 한 시간 가량을 소비했는데도 결국 못 고쳤다. 더 여기에 시간을 투자할 수 없다. 
해당 사진의 웃긴 점은  브라우저 상에서 f12를 눌러 개발자도구를 킬 경우 고양이가 위로 올라가 버린다는 거다.

## About Process Layout

운영체제를 수강할 경우 process 단원에서 배우게 되는 내용이다.

<center><a><img src="/static/img/231014-processLayout.png" width = "50"></a></center>
C:\Users\tngti\Documents\GitHub\tngtied.github.io\static\img\231014-processLayout.png
프로세스는 위와 같이 구성되어 있으며, c를 기반으로 한 언어인 cpp로 프로그래밍할 경우 실제 메모리 기반으로 돌아가기 때문에 메모리 관리를 주의하며 코드를 작성해야 한다.

기본적으로 우리가 선언하는 변수들은 전부 스택으로 쌓이며, 함수 또한 마찬가지이다. 함수에서 특정 값을 반환할 때, copy()동작이 내부적으로 일어나며 함수 바깥으로 값을 꺼낸다.

## l-value, r-value의 동작

```c
int a = 3;
```
과 같은 코드를 선언할 시 =를 기준으로 좌측 항, int a는 l-value이며 3은 r-value이다. l-value와 달리 r-value는 휘발적이다. 

```c
int b = 3;
int a = b;
```
위와 같은 코드의 경우, b 또한 l-value이지만, 둘째 줄에서 b에서 a로의 copy()가 실행된다.

```c 
int b = 3;
int a = move(b);
```
move()함수의 경우에는 stack 내부에서 a가 b로 바뀐다. 



-----

## boj 24728 팬케이크맛 쿠키


