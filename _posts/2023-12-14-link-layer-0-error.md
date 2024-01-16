---
layout: post
categories: [Network]
title: Link layer - error detection/correction
author: tngtied
date: 2023-12-14
---

Transport layer의 datagram은 frame으로 encapsulate되어 link를 통해 노드 간에 전송된다. link layer은 host들마다 존재하는 NIC(network interface card)에 구현된다. host의 system bus에 연결되어 있으며, hardware, software, filmware의 집합이다.

Link layer는 아래와 같은 기능을 한다.

- framing, link access:
  데이터그램을 frame으로 encapsulate하고, header를 더한다. 만약 shared medium일 시 trailer channel 또한 더한다. 헤더의 MAC주소를 통해 source, destination을 식별한다.
- reliable delivery between adjacent nodes
  bit-error rate가 적을 때 가능하다.
- flow control:
  sending, retrieving node 간에 속도를 조절한다.
- error detection:
  signal attenuation, noise로 인해 발생하는 에러를 확인하고, retransmission을 신호하거나 drop한다.
- error correction:
  bit error를 감지하고 수정한 뒤 재전송한다.
- half-duplex and full-duplex:
  half-duplex의 경우 양방향 통신이 가능하지만 동시 통신은 불가능하다.

---

# Error Detection

## parity checking

error detection은 Parity checking을 통해 가능하다. single bit parity의 경우 detect만 가능하고 two-dimensional bit parity의 경우 correction까지 가능하다. 각각의 row와 column 모두의 parity bit을 확인하기 때문이다.

## cyclic redundancy checking

d 길이의 D data bits와 r+1 길이의 G bit pattern이 주어졌을 때, G mod 2로 divisible한 <\D, R>을 계산한다. 이 때 <\D, R>은 D\*2^r XOR R 이다.

---

# multiple access protocol

link엔 두 가지 종류가 존재한다.

- point-to-point
  point-to-point link between Ethernet switch, host
  PPP for dial-up access
- broadcast (shared wire or medium)
  old-fashioned Ethernet
  upstream HFC in cable-based access network

이 중 broadcast channel에서, node 간의 2 이상의 통신이 발생해서 node가 신호들을 동시에 받게 될 경우 collision이 발생한다. 이를 위한 프로토콜이 multiple access protocol이다.
해당 프로토콜은 distributed algorithm을 사용해 (통신을 발송할 시점을 결정하는 등의) channel이 어떻게 공유될지를 정한다. channel sharing에 대한 통신은 channel 자체를 통해야 한다.

## 이론

R bps 의 MAC(multiple access channel)이 주어졌을 때

1. 1 노드가 전송을 원할 때 rate R로 send한다.
2. N개의 노드가 전송을 원할 때, 매 전송은 R/M rate으로 전송된다.
3. fully decentralized:
   trasmission을 제어하는 노드가 존재하지 않는다. clock, slot에 대한 synchronization이 존재하지 않는다.
4. simple

## taxonomy

3 가지 종류가 존재한다.

### channel partitioning

채널을 slot, frequency, code와 같은 보다 작은 부분들로 나누고 노드에 배타적 사용권을 부여한다.

- TDMA (time division multiple access)
  채널이 round마다 사용된다. packet transmission time의 길이의, 고정된 time slot을 매 station이 라운드마다 부여받는다. 사용되지 않는 slot은 idle된다.
- FDMA (frequency division multiple access)
  매 station에 고정된 frequency band가 부여된다. 사용되지 않는 frequency band의 transmission time은 idle이 된다.

### random access

채널을 나누지 않고 충돌을 발생시키고 recover한다. 전송해야 할 패킷이 있을 때 full channel, rate R로 전송한다.

- Slotted ALOHA
  - 가정
    - 모든 프레임이 동일한 사이즈를 가진다.
    - time이 동일한 크기의 time slot들로 나누어져 있다.
    - time slot의 시작 지점에서만 전송할 수 있다.
    - 2개 이상의 노드들이 한 time slot에 전송할 때 충돌이 발생하며, 모든 노드가 충돌을 감지한다.
  - 실행
    - 노드가 새 frame을 얻을 시, 다음 타임 슬롯에 전송한다.
    - 충돌이 없을 시 다음 슬롯에 새 프레임을 전송할 수 있다.
    - 충돌이 발생할 시 이후의 슬롯마다 p의 확률로 성공할 때까지 재전송한다.
  - 장점
    - 개개의 노드가 연속적으로 full rate으로 채널을 사용할 수 있다
    - 탈중앙화되어있다: node 안에서만 time slot 사용을 관리하면 된다.
    - 간단하다
  - 단점
    - 충돌, slot 낭비
    - idle slot
    - 패킷 전송 이전에 collision을 감지해야 함
    - clock synchronization
  - 효율
    - max efficiency: 1/e = .37
  - Pure ALOHA
    - unslotted, collision probability increases, max efficiency .18
- CSMA
  - Simple CSMA
    - listen before transmit
    - if channel sensed idle: transmit entire frame
    - if channel sensed busy: differ transmission
    - _wireless_
  - CSMA/CD (CSMA with collision detection)
    - 충돌은 빠르게 감지된다
    - 충돌이 감지될 때 전송은 취소되며 채널 낭비를 줄인다.
    - 충돌 감지는 유선에서 쉬우며 무선에서 어렵다
    - CS(carrier sensing)에서도 충돌은 일어난다 : 방금 시작된 transmission을 Propagation delay로 인해 알지 못할 수 있다.
  - _Ethernet_ CSMA/CD algorithm
    1. NIC가 데이터그램을 전달받으면 frame을 형성한다.
    2. 채널을 감지한다
       - 만약 idle일 시 전송한다
       - 만약 busy일시 idle이 될 때까지 기다렸다 전송한다.
    3. 전송 중, 다른 전송을 감지할 경우 전송을 중단하고 jam 신호를 보낸다.
    4. NIC는 binary backoff에 들어선다.
       - m번째 충돌 이후, range(0, 2m) 사이의 k를 고른다. K·512 bit 시간을 기다렸다가 2. 으로 돌아간다.
       - collision이 길어질수록 binary backoff는 길어진다.
  - efficiency
    - propagation delay가 줄 수록 efficiency는 1에 가까워진다.
    - max-size frame의 trnasmission time이 늘어날수록 efficiency가 1에 가까워진다.

### “taking turns”

노드들이 차례를 기다려서 채널을 사용한다. 이 때 노드의 상황에 따라 노드의 차례에 사용되는 시간이 조정된다.
_bluethooth, DFFI, tocken ring_

- polling
  마스터 노드가 다른 노드들로 하여금 차례대로 전송하도록 초대한다. 보다 단순한 장치들에서 사용된다.
  우려할 점들은 다음과 같다: polling overhead, latency, single point of failure (master)
- tocken passing
  control tocken은 node들간에 순서대로 이동한다.
  우려할 점은 다음과 같다: tocken overhead, latency, single point of failure.

## cable access network

Single CMTS(cable modem termination system)에서 (Internet frames, TV channels, control)을 *downstream*으로 전송될 때 multiple FDM channel이 사용된다.
random multiple access를 통해, certain *upstream channel*에 모든 유저가 참여한다. 다른 경우 TDM에 참여한다.
