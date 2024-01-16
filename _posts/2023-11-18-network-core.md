---
layout: post
categories: [Network]
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
반대로, 패킷 스위칭은 패킷화를 이용한다. 기본적으로 패킷 스위칭은 greedy하기 때문에 데이터를 패킷 단위로 쪼개 full link capacity로 전송하므로, 서킷 스위칭에서 일어날 수 있는 리소스 낭비가 일어나지 않는다. 또한, 미리 리소스를 예약하지 않기 때문에 다른 사용자들에게 transmission capacity를 공유해줄 수 있다. 더욱 간단하며, 효율적이고, 비용이 적게 든다. Router Scheduling의 granuality가 적어진다.
그러나 패킷 스위칭은 excessive congestion이 발생할 가능성이 있다. 이를 방지하기 위해 RED(Random Early Drop)과 admission control with "virtual circuit" approach와 같은 기법이 사용된다.
이에 대해서는 아래 항목에서 하술하겠다.

## Connection Oriented versus Connectionless

Circuit switching은 connection oriented이며, packet switching은 connectionless한 기법이다. 이는 해당 데이터가 전송되는 경로가 지정되어 있는지, 동적으로 라우팅되는지와도 연관되어 있다. connectionless한 패킷 스위칭은 여러 경로를 통해 데이터를 나누어 전송할 수 있기 때문에, Packetizing이라는 기법을 사용한다. 이러한 기법은 partial failure의 가능성을 높이지만, 그 failure로부터의 recovery의 비용을 줄인다.
중요한 점은, 여기서의 연결성은 network layer의 연결성이라는 점이다. network layer에서 connectionless한 패킷 스위칭도, transport layer에서 가상 서킷, 상술한 virtual circuit을 이용하여 connection oriented하게 만들 수 있다.

# End to End Delay

<center><img src="/static/img/network-core-img1.png" alt="Process Layout" style="max-width:100%;"/></center>

Source와 Destination 간의 end to end delay는 다음과 같이 계산된다.

```
Transmission delays + propagation delay + buffering delay of router(depends on how the queue is designed) = whole delay
```

- Transmission delays = bits per packet / link rate to the router

## Routers

라우터는 어플리케이션의 메시지를 패킷으로 분해해 transmission rate R의 속도로 전송한다. 이 때 Transmission rate R은 아래와 같은 상관관계를 가진다.

```
Tranmission rate == link capacity == link bandwidth
```

## Access Network

shared access인지, dedicated line인지에 따라 두 가지로 나뉜다.

### shared access: CMTRS(Cable Modem Termination System)

Network of cables attaches homes to ISP routers의 구조로 이루어진다. 종류는 두 가지가 있다.

- FDM(frequency division multiplexing) - diff channels in diff frequency
- HFC(hybrid fiber coax) - asymmetric downstream/upstream rate

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

- WLANS (wireless local area nws): within/around building
- wide area cellular access nws: mobile, cellular nw operator

### Physical Media

여기서 사용되는 Physical Media에도 두 가지 종류가 있다.

#### Guided Media

solid media like copper, fiber, coax와 같은 물질적인 것을 의미한다.

- TP(twisted copper)
- coaxial cable: 양방향 전달이 가능하다
- fiber optic cable(glass): high speed low error rate

### Unguided Media

비물질적인 media로, 자유롭게 propagate가능하다는 잔점이 있다. 예시로는 라디오가 있다.

- electromagnetic spectrum
- broadcast
- “half duplex”
- environment affects the connection: reflection, obstruction, interference
- usage: terrestrial microwave, WIFI(wireless lan), wide-area(celluar), Satellite
