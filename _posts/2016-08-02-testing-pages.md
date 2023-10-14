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

## About Process Layout

운영체제 중 배우게 되는 내용이다.

<center><img src="source/static/img/231014-processLayout.png"/></center>



## H2 Lorem Ipsum
-----

Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nam imperdiet urna eu dolor placerat varius. Vivamus eros augue, consequat id scelerisque nec, fringilla in est. Proin pellentesque malesuada mauris, quis aliquam augue vestibulum ac. Vestibulum ut feugiat nibh. Sed faucibus felis purus, sed convallis leo dictum vehicula. 

Nam maximus tempor feugiat. Mauris tristique imperdiet nulla id egestas. Proin eget lobortis magna. Duis consectetur nibh at elit viverra congue. Ut eu turpis enim. Suspendisse laoreet, diam sed consequat sodales, felis dolor accumsan justo, nec scelerisque mi sem quis dolor. Etiam ornare venenatis massa, a suscipit ex. Ut quis lectus id nibh mattis rutrum. Nunc vel cursus eros, at blandit mi. Vivamus ac posuere libero.