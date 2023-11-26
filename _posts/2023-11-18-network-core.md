---
layout: post
categories: Network
title: 2. Network Core
author: tngtied
date: 2023-11-18
---

네트워크의 코어란, interconnected routers(switch of switches)이다. 즉, 네트워크의 네트워크인 것이다.

네트워크의 코어를 이해하기 위해서는 패킷 스위칭과 서킷 스위칭의 차이를 알아야 한다. 
# Packet Switching versus Circuit Switching
이들의 가장 핵심적인 차이점은 Resource Reservation과 Connection orientation이다. 

## Resource Reservation
Circuit Switching은 Network Admission Control을 가지고 있기 때문에 만약 traffic pattern이 안정적이고 예측가능하다면, circuit switching이 갖는 이점이 훨씬 크다. 
반대로, 패킷 스위칭은 패킷화를 이용한다. 기본적으로 패킷 스위칭은 greedy하기 때문에 서킷 스위칭에서 일어날 수 있는 리소스 낭비가 일어나지 않는다. 또한, 미리 리소스를 예약하지 않기 때문에 다른 사용자들에게 transmission capacity를 공유해줄 수 있다. 더욱 간단하며, 효율적이고, 비용이 적게 든다. Router Scheduling의 granuality가 적어진다. 
그러나 패킷 스위칭은 excessive congestion이 발생할 가능성이 있다. 이를 방지하기 위해 RED(Random Early Drop)과 admission control with "virtual circuit" approach와 같은 기법이 사용된다. 
이에 대해서는 아래 항목에서 하술하겠다.  

## Connection Oriented versus Connectionless
Circuit switching은 connection oriented이며, packet switching은 connectionless한 기법이다. 

## Routers
라우터는 어플리케이션의 메시지를 패킷으로 분해해 transmission rate R의 속도로 전송한다. 이 때 Transmission rate R은 아래와 같은 상관관계를 가진다.
```
Tranmission rate == link capacity == link bandwidth
```

## Access Network 
shared access인지, dedicated line인지에 따라 두 가지로 나뉜다.

### shared access: CMTRS(Cable Modem Termination System)
Network of cables attaches homes to ISP routers의 구조로 이루어진다. 종류는 두 가지가 있다.

* FDM(frequency division multiplexing) - diff channels in diff frequency
* HFC(hybrid fiber coax) - asymmetric downstream/upstream rate

### dedicated line: DSL (Digital Subscriber Line)
기존에 존재하는 telephone line을 통해 central office DSLAM(DSL Access Multiplexer)로 접근한다. DSLAM은 해당 통신을 ISP로 전달한다.

Access 요청을 하는 주체에 따라 나누어 보자면 아래와 같다. 
### Home
```
( Wifi(Wireless Access Point) + router(NAT) + cable || DSL modem )-( headend || central office )
```
### Enterprise Network
```
( mix of wired/wireless link )-( mix of switches/routers )
```

앞에서 자주 나온 Wifi는 무엇일까?
### Wireless
end system과 router를 "access point"를 통해 연결한다. 이에는 두 가지 방식이 존재한다. 
* WLANS (wireless local area nws): within/around building
* wide area cellular access nws: mobile, cellular nw operator

### Physical Media
여기서 사용되는 Physical Media에도 두 가지 종류가 있다. 
#### Guided Media
solid media like copper, fiber, coax와 같은 물질적인 것을 의미한다.
* TP(twisted copper)
* coaxial cable: 양방향 전달이 가능하다
* fiber optic cable(glass): high speed low error rate

### Unguided Media
비물질적인 media로, 자유롭게 propagate가능하다는 잔점이 있다. 예시로는 라디오가 있다. 
* electromagnetic spectrum
* broadcast
* “half duplex”
* environment affects the connection: reflection, obstruction, interference
* usage: terrestrial microwave, WIFI(wireless lan), wide-area(celluar), Satellite

