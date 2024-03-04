---
layout: post
categories: [Network]
title: End to End Delay
author: tngtied
date: 2023-11-18
---

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
