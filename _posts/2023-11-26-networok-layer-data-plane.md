---
layout: post
categories: Network
title: NW layer - Data Plane
author: tngtied
date: 2023-11-26
---

Network Layer은 Application layer, Transport Layer 하위의 레이어로, 데이터그램을 sender로부터 receiver로 routing하는 기능을 갖고 있다. Network Layer은 IP, routing protocol로 구성되어 있다.

## Network Layer Function
여기서 주의해야 하는 점은 link는 단일 router 주변 scope로, router의 location에 의존적인 용어이지만, route(path)는 데이터그램의 source와 destination까지의 scope에서 사용되는 용어이다.

### Forwarding
패킷을 router의 input link로부터, 적절한 output link로 옮기는 것을 의미한다. 

#### Data Plane : Local(per-router)
per router function으로, datagram이 input port로부터 output port로 forward되는 방식을 정한다.

### Routing
routing algorithm을 통해 패킷이 source부터 destination까지 도달하는 데 거쳐야 할 route를 결정한다.

#### Control Plane : Network-side
datagram이 source host로부터 destination host까지 도달할 수 있는 end-end path를 정한다.
라우팅 알고리즘은 control plane 상에서 결정되며, 종류는 두 가지가 있다.
* traditional routing algorithm: 라우트를 라우터에서 결정한다. 
개개의 라우터에 존재하는 local forwarding table이 control plane에 존재하는 각각의 routing algorithm들과 연결되어 있다. 이러한 개개의 모든 라우터의 Individual routing algotirhm이 서로 상호작용한다. 
* software-defined network(SDN): 라우트를 (remote)server가 결정한다. remote controller가 router들의 f

-----

## Network Service Model





Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nam imperdiet urna eu dolor placerat varius. Vivamus eros augue, consequat id scelerisque nec, fringilla in est. Proin pellentesque malesuada mauris, quis aliquam augue vestibulum ac. Vestibulum ut feugiat nibh. Sed faucibus felis purus, sed convallis leo dictum vehicula.

## H2 Lorem Ipsum
-----

Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nam imperdiet urna eu dolor placerat varius. Vivamus eros augue, consequat id scelerisque nec, fringilla in est. Proin pellentesque malesuada mauris, quis aliquam augue vestibulum ac. Vestibulum ut feugiat nibh. Sed faucibus felis purus, sed convallis leo dictum vehicula. 

Nam maximus tempor feugiat. Mauris tristique imperdiet nulla id egestas. Proin eget lobortis magna. Duis consectetur nibh at elit viverra congue. Ut eu turpis enim. Suspendisse laoreet, diam sed consequat sodales, felis dolor accumsan justo, nec scelerisque mi sem quis dolor. Etiam ornare venenatis massa, a suscipit ex. Ut quis lectus id nibh mattis rutrum. Nunc vel cursus eros, at blandit mi. Vivamus ac posuere libero.



# Header 1

## Header 2

### Header 3

#### Header 4

##### Header 5

###### Header 6

Some example text, okay? It's normal, right? :)

There are many variations of passages of Lorem Ipsum available, but the majority have suffered alteration in some form, by injected humour, or randomised words which don't look even slightly believable. If you are going to use a passage of Lorem Ipsum, you need to be sure there isn't anything embarrassing hidden in the middle of text. All the Lorem Ipsum generators on the Internet tend to repeat predefined chunks as necessary, making this the first true generator on the Internet. It uses a dictionary of over 200 Latin words, combined with a handful of model sentence structures, to generate Lorem Ipsum which looks reasonable. The generated Lorem Ipsum is therefore always free from repetition, injected humour, or non-characteristic words etc.

**bold text**

*italic text*

> And this is a blockquote

~~~
And this is a source code in a block
Yeah, don't use highligth :D
~~~

But if you like `inline code`, it's fine.

And, of course, links: [lampiaosec].

# Lists

* Item x
* Item z

1. Item 01
2. Item 02

## Table of content

* toc
{:toc}
