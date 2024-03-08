---
layout: post
categories: [Network]
title: 1. Network Edge
author: tngtied
date: 2023-11-18
---

# 인터넷이란?

인터넷은 크게 두 가지 관점으로 바라볼 수 있다. 첫 번째로, 구성요소의 관점이다. 인터넷은 컴퓨팅 디바이스, 네트워킹 어플리케이션과 같은 호스트들과 이들 간에 오가는 packet들을 포워딩하는 패킷 스위치, communication link, 그리고 네트워크들로 이루어져있다. 인터넷은 네트워크들의 네트워크로, interconnected ISP들이며, 이들이 통신하는 protocol들로 이루어져있다.
서비스의 관점에서 본다면 인터넷은 어플리케이션에게 서비스를, 그리고 분산된 어플리케이션에게 프로그래밍 인터페이스를 제공하는 인프라구조이다.

# 프로토콜이란?

네트워크 내 개체들 간에 송신, 수신되는 메시지의 포맷과 순서 그리고 메시지 전송에 있어서 취해지는 액션을 정의한다.

# Edges

network edge들은 호스트와 서버들이다. 이들은 residential/institutional access network, mobile wireless networks(wifi)를 통해 edge router와 연결된다.

## Routers

라우터는 어플리케이션의 메시지를 패킷으로 분해해 transmission rate R의 속도로 전송한다. 이 때 Transmission rate R은 아래와 같은 상관관계를 가진다.

```
Tranmission rate == link capacity == link bandwidth
```

## Access Network

해당 디바이스가 네트워크에 접근하는 방식이 shared access인지, dedicated line인지에 따라 두 가지로 나뉜다.

### shared access: CMTRS(Cable Modem Termination System)

Network of cables attaches homes to ISP routers의 구조로 이루어진다. 종류는 두 가지가 있다.

- FDM(frequency division multiplexing) - diff channels in diff frequency
- HFC(hybrid fiber coax) - asymmetric downstream/upstream rate

### dedicated line: DSL (Digital Subscriber Line)

기존에 존재하는 telephone line을 통해 central office DSLAM(DSL Access Multiplexer)로 접근한다. DSLAM은 해당 통신을 ISP로 전달한다.

Access 요청을 하는 주체에 따라 나누어 보자면 아래와 같다.

#### Home

```
( Wifi(Wireless Access Point) + router(NAT) + cable || DSL modem )-( headend || central office )
```

#### Enterprise Network

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
