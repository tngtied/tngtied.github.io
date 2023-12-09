---
layout: post
categories: Network
title: Network layer Control Plane - SDN
author: tngtied
date: 2023-12-10
---
SDN(Software defined networking) control plane은 logically centralized돼있다. remote controller가 compute하고 각 router에 forwarding table을 설치한다. 중앙화는 network management에 있어서 router misconfiguration을 줄이고, traffic flow의 유연성을 늘린다. 또한 text based forwarding은 라우터를 programmable하게 만든다. 
distributed programming의 경우, distributed algorithm이 각각의 라우터에 있기 때문에 table을 연산하기 힘든 것이다. 또한 load balancing 혹은 traffic route들의 변경을 가능하게 한다.

# 구조
SDN의 구조는 다음과 같다. 
<center><img src="/static/img/sdn-structure.png" alt="SDN structure" style="max-width:100%;"/></center>

## Data-plane switch
generalized data plane forwarding을 하는 스위치들로 구성되어 있다. flow table은 controller supervision 하에 연산되고 설치된다. table-based switch control을 위한, OpenFlow와 같은 API가 존재한다. Controller와 통신하는 Protocol이 존재한다.

## SDN controller
SDN은 network operating system으로, network state information을 보유한다. 또한, network control application과 northbound API를 통해 통신하며, network switch와도 southbound API를 통해 통신한다. distributed system으로 구현되어 performance, scalability, fault-tolerance, robustness를 달성한다.
SDN controller는 다음과 같이 구성되어 있다. 
<center><img src="/static/img/sdn-controller.png" alt="SDN controller structure" style="max-width:100%;"/></center>

* interface layer: network control applications을 위한 abstractions API이다.
* network-wide state management: 분산된 데이터베이스에 네트워크 링크, 스위치, 서비스들의 상태를 저장한 것이다
* communation: SDN controller와 controlled switch간의 상호작용이 이루어진다.
## Network-control apps
routing, access control, load balance와 같은 control function으로 기능하며, 이 때 SDN controller의 API가 제공하는 lower level services를 사용한다. 해당 어플리케이션들은 routing vendor나 SDN controller와 독립적으로 서드파티에 제공될 수 있다.

-----

# OpenFlow
OpenFlow는 controller와 switch간에 사용되며, TCP의 형태로 이루어진다. 종류로는 세 가지가 있다. 
## controller-to-switch
해당 종류는 또 다시 네 가지로 세분화된다. 
* features: controller가 switch의 세부 사항을 쿼리하고 스위치가 답한다.
* configure: controller가 switch configuration parameter를 query하거나 설정한다. 
* modify-state: OpenFlow table에 flow entry를 add, delete, modify한다.
* packet-out: controller가 특정한 패킷을 특정한 스위치 포트 밖으로 전송한다. 
## switch to controller(asynchronous)
해당 종류는 또 다시 세 가지로 세분화된다.
* packet-in: 패킷을 컨트롤러에 전달한다.
* flow-removed: flow table entry가 스위치에서 삭제됐음을 전달한다.
* port status: port 상태의 변화를 전달한다.
## symmetric (misc.)

## 응용
Openflow interface를 응용한 controller는 OpenDaylight (ODL)과 ONOS 가 있다. 