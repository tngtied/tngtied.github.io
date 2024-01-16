---
layout: post
categories: [Network]
title: Network layer Data Plane  - Router
author: tngtied
date: 2023-12-08
---

<center><img src="/static/img/generic-router-architecture.png" alt="Generic Router Architecture" style="max-width:100%;"/></center>

router의 기본적인 구조는 위와 같다.

## line termination (physical layer): bit-level reception

## lookup, forwarding, queueing

포워딩의 방식은 아래와 같다.

1. destination-based forwarding: destination ip based
2. generalized forwarding: based on any set of header field values

## decentralized switching

match plus action: header 값에 따라 input port memory에 있는 output port forwarding table을 살핀다.

---

# lookup, forwarding, queueing

### input port queueing

router마다 존재하는 Input port의 구조는 아래와 같다.

<center><img src="/static/img/router-input-port.png" alt="Generic Router Architecture" style="max-width:100%;"/></center>

만약 switch fabric이 input port들의 합보다 느릴 경우 input queuing이 발생할 수 있다. 이 경우 HOL(Head of Line) blocking이 발생할 수 있다.

### output port queueing

router마다 존재하는 output port의 구조는 아래와 같다.

<center><img src="/static/img/router-output-port.png" alt="Generic Router Architecture" style="max-width:100%;"/></center>

link transmission rate보다 fabric으로부터의 데이터그램 전송이 빠를 경우 **buffering**이 발생할 수 있다. 이 때 버퍼가 부족할 경우, **drop policy**가 필요하다. 즉, 데이터그램이 congestion때문에 유실될 수 있다.
여기서 **Scheduling discipline**이 필요하다. performance와 network neutrality를 고려한 priority scheduling이 이루어진다.

#### buffer management

버퍼는 두 가지 방식으로 운영된다.

1. drop
   buffer가 꽉 찼을 때 드롭할 패킷을 결정하는 데에 사용할 방침으로는 tail drop, priority 두 가지가 있다.
2. marking
   congestion을 시그널하기 위해 어떤 패킷을 마킹(ECN, RED)할지를 결정한다

#### packet scheduling

다음으로 전송된 패킷을 결정하는 방침이다. 이에는 4가지 방식이 있다.

1. first come, first served (FCFS, FIFO)
2. priority
   queue에 들어오는 패킷을 header field에 따라 분류하고, highest priority 순서대로 전송한다.
3. round robin(RR)
   queue에 들어오는 패킷을 header field에 따라 각각의 큐들에 분류하고, 큐들을 서버가 순회하며 각각의 분류마다 온전한 패킷을 하나씩 전송한다.
4. weighted fair queueing
   일반화된 RR으로, 분류(클래스)마다의 가중치만큼 전송한다. 그리 할 경우 클래스마다 최소한의 bandwidth가 보장된다.

### destination based forwarding - longest prefix matching

forwarding table의 destination address range가 겹칠 경우, destination address와 prefix가 가장 많이 겹치는 link interface를 사용한다.
이는 주로 ternary content addressable memories (TCAMs)를 사용하여 이루어진다.

- content addressable: present address to TCAM: retrieve address in one clock cycle, regardless of table size
- Cisco Catalyst: ~1M routing table entries in TCAM

---

# Switching Fabrics

input link로부터 적절한 output link로 ㅔacket을 전달하는 것을 의미한다. switching rate은 input/output line rate들로 주로 계산된다.
방식은 아래와 같은 3 가지가 존재한다.

## memory-based switching

전통적인 방식으로, system cpu의 통제 하에 packet을 메모리로 카피하고 스위칭한다. memory bandwidth만큼의 속도를 지닌다. 데이터그램은 2개의 버스를 지나게 된다.

<center><img src="/static/img/switching-memory.png" alt="switching via memory Architecture" style="max-width:100%;"/></center>

## bus-based switching

input memory에서 output memory로 공유되는 bus를 통해 데이터그램이 이동한다. bus bandwidth만큼의 속도를 가진다.  
그 예시로는 32 Gbps bus, Cisco 5600가 있다.

<center><img src="/static/img/switching-bus.png" alt="switching via Bus Architecture" style="max-width:100%;"/></center>

## interconnected network based switching

**multistage switch**, 즉 보다 작은 switch들의 multiple stage를 사용한다. datagram을 정해진 길이의 cell로 나누어 fabric을 통과시킴으로써 병렬 처리(**exploit paralleism**)한다. 다중의 스위칭 plane을 병렬적으로 사용하며 speed up, scale up 한다.
그 예시로는 Cisco CRS router가 있다. 8 개의 스위칭 plane, plane마다 3 stage의 interconnected network를 사용한다. 100’s Tbps 까지의 switching capacity를 지닌다.

<center><img src="/static/img/switching-interconnected-nw.png" alt="switching via interconnected network Architecture" style="max-width:100%;"/></center>
