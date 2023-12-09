---
layout: post
categories: Network
title: Network layer Data Plane - middlebox
author: tngtied
date: 2023-12-09
---

라우터마다 포워딩 테이블(flow table)을 지닌다. 이 때 도착한 패킷의 bits를 확인하고 포워딩하는 **match plus action** 이 이루어진다.
# Match+action
match plus action에는 두 가지 종류가 있다. 하나는 destination-based forwarding으로 destination의 IP 주소를 기반으로 포워딩하는 것이다. 또 다른 하나는 generalized forwarding이다. 
## generalized forwarding
generalized forwarding에서 사용되는 flow tabledms match와 action으로 구성되어 있다.
매치를 위해 사용되는 결정자는 링크 네트워크, transport 모든 레이어의 헤더의 다양한 부분들이 될 수 있다. 액션 부분에서는 매치된 패킷마다 forward/drop/copy/modify/log 할 수 있으며 이 때 modify의 경우 header field value를 수정하거나 encapsulate하여 컨트롤러로 포워딩할 수 있다.

해당 abstaraction은 router 뿐 아니라 여러 기기에서 사용될 수 있다.
* Router
match: longest destination IP prefix
action: forward out a link
* Switch
match: destination MAC address
action: forward or flood
* Firewall
match: IP addresses and TCP/UDP port numbers
action: permit or deny 
* NAT
match: IP address and port
action: rewrite address and port

priority: disambiguate overlapping patterns
counters: #bytes and #packets

-----
# Middlebox
RFC 3234에 따른 Middlebox의 정의는 다음과 같다. 
>any intermediary box performing functions apart from normal, standard functions of an IP router on the data path between a source host and destination host
그 예시로는 NAT, Firewalls, IDS, Load balancer, Caches, Application Specific(service providers, CDN)이 있다.

## middlebox functions
middlebox는 아래와 같은 것을 제공한다.
* whitebox hardware for Open API: local action들이 programmable해진다.
* SDN: locally centralized control and configuration
* NFV(network functions virtualization): programmable services over whitebox networking, computation, storage