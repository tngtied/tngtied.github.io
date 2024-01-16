---
layout: post
categories: [Network]
title: Link layer - MPLS, Datacenter Networks
author: tngtied
date: 2023-12-15
---

# MPLS

Ethernet header와 ethernet frame, IP datagram 사이에 MPLS(Multiprotocol label switching) header를 넣고 MPLS-capable router를 사용함으로써 아래와 같은 효과들이 가능하다.

- fixed length identifier을 사용한 더욱 빠른 lookup
- VC(virtual circuit)와 유사한 동작
  MPLS 헤더는 아래와 같은 구조를 지닌다.
  > label(20) \| Exp(3) \| S(1) \| TTL(5)

## MPLS capable routers

label-switched router로, IP주소가 아닌 label value에 의거해 패킷을 포워딩한다. 기존의 라우터와는 flexibility의 면에서 차이를 갖는다. traffic engineering과 link fail시 pre-computed backup path를 사용한다는 점에서 그러하다.

## MPLS versus IP paths

- IP routing: 도착지만을 기반으로 routing한다
- MPLS routing: 출발지와 도착지 모두를 고려하여 routing한다. generalized forwarding의 기법도 차용하며, fast reroute가 가능하다.

## MPLS signaling

IS-IS link state flooding protocol을 사용한다.
RSVP-TE 시그널 프로토콜을 사용하여 MPLS 포워딩을 downstream router에 설정한다.

---

# Datacenter Network

데이터센터 네트워크에는 host의 scale과 closley coupled, in close proximity라는 특성 때문에 아래와 같은 챌린지가 발생한다.

- 다수의 클라이언트들과 클라이언트 각각의 어플리케이션
- 안정성
- 부하의 밸런싱, 네트워킹, 데이터 보틀넥, avoiding 처리

## 구조

데이터센터 네트워크는 아래와 같은 위계를 갖는다.

- border router: 데이터센터 바깥으로 연결한다.
- Tier-1 switches
- Tier-2 switches
- (TOR)Top of Rack switch: rack 마다 하나로, 40-100Gbps Ethernet을 서비스한다.
- Server racks: 20-40 server blades - hosts

## multipath

구조 상 각각의 인접한 위계끼리 전부 연결되어 있는 것을 의미한다. rack간의 throughput을 늘리며, redundancy를 통해 reliability를 늘린다.

## Application-layer Routing

load balancer가 존재해 아래와 같은 기능을 함으로써 데이터센터 내부 구조를 바깥으로부터 가린다.

- 외부의 클라이언트 요청을 받는다
- workload를 데이터센터 내부로 향한다.
- 결과를 외부 클라이언트에 전달한다.
