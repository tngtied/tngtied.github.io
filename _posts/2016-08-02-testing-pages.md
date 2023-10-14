---
layout: post
category: cppstudy
title: assignment of 10_13
author: tngtied
date: 2023-10-14
---

## 블로그 만들기

페이지를 띄웠을 때 포스팅 페이지 endpoint가 .html로 끝나는 게 이상하다. 분명 나는 .md 파일에 작성하고 있는데도 그러하기 때문이다. 이런 스태틱 웹페이지를 작성한지 너무 오래돼서 path를 어떻게 설정하는지 개인적으로 알아봤다. 

.md file 윗 부분에 아래와 같은 문법으로 작성하고
```
layout: post
category: cppstudy
title: assignment of 10_13
author: tngtied
date: 2023-10-14
```
_config.yml file 내부에서 
```
permalink: /:category/:title
```
와 같이 선언해주면 endpoint가 지저분하게 보이던 것이 사라진다.

고양이 사진의 layout 문제로 한 시간 가량을 소비했는데도 결국 못 고쳤다. 더 여기에 시간을 투자할 수 없다. 
해당 사진의 웃긴 점은  브라우저 상에서 f12를 눌러 개발자도구를 킬 경우 고양이가 위로 올라가 버린다는 거다.

## About Process Layout

운영체제를 수강할 경우 process 단원에서 배우게 되는 내용이다.

<center><img src="source/static/img/231014-processLayout.png"/></center>

프로세스는 위와 같이 구성되어 있으며, c를 기반으로 한 언어인 cpp로 프로그래밍할 경우 실제 메모리 기반으로 돌아가기 때문에 메모리 관리를 주의하며 코드를 작성해야 한다.

## l-value, r-value의 동작

```c
int a = 3
```
-----

Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nam imperdiet urna eu dolor placerat varius. Vivamus eros augue, consequat id scelerisque nec, fringilla in est. Proin pellentesque malesuada mauris, quis aliquam augue vestibulum ac. Vestibulum ut feugiat nibh. Sed faucibus felis purus, sed convallis leo dictum vehicula. 

Nam maximus tempor feugiat. Mauris tristique imperdiet nulla id egestas. Proin eget lobortis magna. Duis consectetur nibh at elit viverra congue. Ut eu turpis enim. Suspendisse laoreet, diam sed consequat sodales, felis dolor accumsan justo, nec scelerisque mi sem quis dolor. Etiam ornare venenatis massa, a suscipit ex. Ut quis lectus id nibh mattis rutrum. Nunc vel cursus eros, at blandit mi. Vivamus ac posuere libero.