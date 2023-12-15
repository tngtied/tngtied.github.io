---
layout: post
categories: Network
title: Link layer - LANs
author: tngtied
date: 2023-12-15
---
MAC 주소는 프레임을 하나의 인터페이스에서 다른 인터페이스로 옮기기 위해 지역적으로 사용된다. 여기서 지역적이란, 동일한 subnet 내부 IP-addressing의 의미에서이다. 4 bit짜리의 hexadecimal 12자리로 이루어져 있으며 총 48bit이다. MAC주소는 IEE에 의해 부여되며, 생산자가 MAC 주소의 일부를 산다.
MAC 주소는 LAN에 있어서 유동성을 띤다. 하나의 LAN에서 다른 LAN으로 이동할 때 고유하게 유지되며, 이 점에서 LAN에 의존적인 IP 주소와 구분할 수 있다.

# ARP(address resolution protocol)
매 IP 노드는 IP 주소마다 MAC 주소를 매핑한 ARP 테이블을 가진다. TTL은 대체로 20분이다. 

## routing inside a subnet
만약 도착지의 MAC 주소를 모르는 채로 전송하고자 한다면, 전송자 노드는 도착지 IP주소를 담은 ARP 쿼리를 브로드캐스트(도착지: FF-FF-FF-FF-FF-FF)한다. 이를 수신한 노드들 중 해당 MAC주소를 보유한 노드가 답신한다. 쿼리의 답신을 받은 전송자 노드는 해당 매핑을 ARP 테이블에 기록한다. 

## routing to another subnet
서브넷 1에 속한 노드 A가 서브넷 2에 속한 노드 B에게 라우터 R을 통해 전송하고자 할 때, A는 출발지 A, 도착지 B의 IP 데이터그램을 생성하고, 이를 출발지 A, 도착지 R의 MAC 주소로 하는 link layer 프레임으로 감싼다. R에 도착한 프레임은 decapsulate되며, 새로이 출발지 R, 도착지 B MAC 주소로 하는 프레임으로 encapsulate하여 전송한다.

-----
# Ethernet
시장에서 지배적으로 사용되는 LAN 기술로, 간단하고 저렴하며, 10 Mbps – 400 Gbps 를 커버 가능하며, 하나의 cheap으로 여러 속도를 낼 수 있다는 특징이 있다. 

## 종류
* bus: 과거에 사용되었으며, 모든 노드가 하나의 collision domain에 속한다.
* switched: 현재에 사용되며, 활성화된 link layer는 2개의 switch를 중심으로 하며 separate ethernet protocol을 운영하므로 충돌하지 않는다. 

## frame
ethernet frame은 다음과 같은 구조를 지닌다. 
> preamble | dest. address | source address | type | data(payload) | CRC
* preamble
    - sender와 receiver의 clock rate을 동기화하기 위해 사용된다. 
    - 7 bytes of 10101010 followed by one byte of 10101011
* addresses 
    - 6 byte로 이루어진 MAC 주소이다.
    - adapter가 일치하는 destination/broadcast(ARP 패킷) 주소를 받았을 떄, 데이터 프레임을 network layer protocol로 넘긴다.
    - 일치하지 않을 경우, 프레임을 폐기한다.
* type
    - 보다 높은 layer protocol을 의미한다. (예시: IP)
    - receiver를 demultiplex up 하기 위해 사용된다.
* CRC (cyclic redundancy check at receiver)
    - error 감지: frame 드랍

## 특징
* connectionless: NIC 간 handshaking 과정이 없다. 
* unreliable
    - NIC 수신 시 ACK 혹은 NACK을 전송하지 않는다.
    - dropped frame은 higher layer RDT를 사용할 때에야만이 복구된다.
* unslotted CSMA/CD with binary backoff
----- 
# switches
* Ethernet frame을 store, MAC 주소를 검사하고 **선택적으로** 하나 이상의 outgoing 링크로 forward한다. 
* **transparent**: 호스트들은 switch의 존재를 고려하지 않는다.
* **plug-and-play, self-learning**: switch는 configure될 필요가 없다.

## multiple simultaneous transmissions
* 호스트들은 스위치에 직접적으로 연결되어 있다. 
* 스위치는 패킷들을 버퍼링한다.
* Full duplex: 매 incoming link에 대해 ethernet protocol이 사용되므로 충돌이 발생하지 않는다.
* 매 link는 각자의 collision domain을 가진다. 
* Switching: 각기 다른 링크를 사용하는, 스위치를 거치는 통신들은 동시에 이루어질 수 있다. 

## switching forward table
매 스위치는 switch table을 가진다. switch table의 entry는 호스트, 인터페이스마다의 MAC 주소이다. 
이는 self learning하다. 스위치는 frame이 수신될 때마다 인터페이스마다의 호스트를 스위치 테이블에 기록한다. 자세한 과정은 아래와 같다.
1. 프레임이 수신된다.
2. 출발지의 incoming link, MAC 주소를 기록한다
3. 도착지를 switch table에서 찾는다
4. if 도착지를 찾았을 경우에
    - if 목적지가 프레임이 도착한 세그먼트에 있을 경우 프레임을 드랍한다
    - else table entry의 목적지로 프레임을 포워딩한다
5. else flood: 출발지의 interface를 제외한 모든 interface로 포워딩한다. 

## Switches vs. routers
공통점은 아래와 같다.
* store-and-forward의 방식으로 이루어진다.
* 포워딩 테이블을 가진다.
차이점은 아래와 같다.
* router는 network-layer device인 반면, switch는 link-layer device이다.
* router는 routing algorithm을 사용해 table을 계산하는 반면, switch는 flooding을 통해 forwarding table을 얻는다.
* router는 IP 주소를 사용하는 반면, switch는 MAC 주소를 사용한다. 

-----
# VLANs
LAN이 scale해져서 Point of attachment를 변경할 경우 efficiency, security, privacy, efficiency 문제가 발생한다. 또한, 스위치에 대한 logical attachment와 physical attachemnt를 달리 하고 싶을 경우 문제가 발생할 수 있다. 이를 위한 해결책으로 제시된 것이 VLAN이다.
VLAN이란 다수의 virtual LAN을 하나의 LAN에서 운영하는 것이다. 물리적으로는 하나인 LAN이 logical하게는 다수로 기능할 수 있다. 

## Port-based VLANs
* traffic isolation: VLAN을 포트 혹은 종단점의 MAC 주소로 정의할 수 있다. 
* dynamic membership: 포트들이 동적으로 VLAN에 부여될 수 있다. 
* forwarding between VLANS: 라우팅을 통해, 별개의 스위치들 간에 그러하듯이 이루어질 수 있다. 
* trunk port: 프레임을 여러 개의 물리적인 스위치들에 분산되어 있는 VLAN들로 이동시킬 수 있다. 이를테면 VLAN A가 switch A, B에 걸쳐져 있을 때, 하나의 VLAN안에서 프레임이 조작되도록 A와 B를 연결해 주는 것이 trunk port 이다. trunk port간에 통신할 때 부가적인 프레임 헤더들은 제거된다. 

## VLAN frame

> preamble | dest. address | *2-byte Tag Protocol Identifier* | *Tag Control Information* | source address | type | data(payload) | CRC
