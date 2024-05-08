---
layout: post
categories: [Network]
title: Packet Switching vs Circuit Switching
author: tngtied
date: 2023-11-18
---

네트워크의 코어란, interconnected routers(switch of switches)이다. 즉, 네트워크의 네트워크인 것이다.

네트워크의 코어를 이해하기 위해서는 패킷 스위칭과 서킷 스위칭의 차이를 알아야 한다.
패킷 스위칭은 기본적으로 어플리케이션 레이어의 메시지를 패킷의 단위로 나누어, full link capacity로 router들 사이의 링크를 통해 전송하는 것을 의미한다.
서킷 스위칭은 링크를 서킷 단위로 나누어, 매 전송마다 resource를 예약해 할당받아 사용하는 것이다.circuit 단위는 FDM(Frequency Division Multiplexing)과 TDM(Time Division Multiplexing)이 있을 수 있다.

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

# Routing과 Forwarding

라우팅과 포워딩은 2가지의 network core 기능이다. 포워딩은 local action으로, 도착한 패킷을 router의 input link로부터 forwarding table에 기록된 알맞은 output link로 옮기는 것을 의미한다. routing은 global action으로, 패킷의 출발지부터 도착지까지의 길을 정한다. 이는 routing algorithm을 통해 이루어진다.
