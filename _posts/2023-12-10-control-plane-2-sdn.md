---
layout: post
categories: [Network]
title: Network layer Control Plane - SDN
author: tngtied
date: 2023-12-10
---

SDN(Software defined networking) control plane은 logically centralized돼있다. remote controller가 compute하고 각 router에 forwarding table을 설치한다. 중앙화는 network management에 있어서 router misconfiguration을 줄이고, traffic flow의 유연성을 늘린다. 또한 text based forwarding은 라우터를 programmable하게 만든다.
distributed programming의 경우, distributed algorithm이 각각의 라우터에 있기 때문에 table을 연산하기 힘든 것이다. 또한 load balancing 혹은 traffic route들의 변경을 가능하게 한다.

# 구조

SDN의 구조는 다음과 같다.

<center><img src="/static/img/sdn-structure.png" alt="SDN structure" style="max-width:100%;"/></center>

## Data-plane switch

generalized data plane forwarding을 하는 스위치들로 구성되어 있다. flow table은 controller supervision 하에 연산되고 설치된다. table-based switch control을 위한, OpenFlow와 같은 API가 존재한다. Controller와 통신하는 Protocol이 존재한다.

## SDN controller

SDN은 network operating system으로, network state information을 보유한다. 또한, network control application과 northbound API를 통해 통신하며, network switch와도 southbound API를 통해 통신한다. distributed system으로 구현되어 performance, scalability, fault-tolerance, robustness를 달성한다.
SDN controller는 다음과 같이 구성되어 있다.

<center><img src="/static/img/sdn-controller.png" alt="SDN controller structure" style="max-width:100%;"/></center>

- interface layer: network control applications을 위한 abstractions API이다.
- network-wide state management: 분산된 데이터베이스에 네트워크 링크, 스위치, 서비스들의 상태를 저장한 것이다
- communation: SDN controller와 controlled switch간의 상호작용이 이루어진다.

## Network-control apps

routing, access control, load balance와 같은 control function으로 기능하며, 이 때 SDN controller의 API가 제공하는 lower level services를 사용한다. 해당 어플리케이션들은 routing vendor나 SDN controller와 독립적으로 서드파티에 제공될 수 있다.

---

# OpenFlow

OpenFlow는 controller와 switch간에 사용되며, TCP의 형태로 이루어진다. 종류로는 세 가지가 있다.

## controller-to-switch

해당 종류는 또 다시 네 가지로 세분화된다.

- features: controller가 switch의 세부 사항을 쿼리하고 스위치가 답한다.
- configure: controller가 switch configuration parameter를 query하거나 설정한다.
- modify-state: OpenFlow table에 flow entry를 add, delete, modify한다.
- packet-out: controller가 특정한 패킷을 특정한 스위치 포트 밖으로 전송한다.

## switch to controller(asynchronous)

해당 종류는 또 다시 세 가지로 세분화된다.

- packet-in: 패킷을 컨트롤러에 전달한다.
- flow-removed: flow table entry가 스위치에서 삭제됐음을 전달한다.
- port status: port 상태의 변화를 전달한다.

## symmetric (misc.)

## 응용

## Openflow interface를 응용한 controller는 OpenDaylight (ODL)과 ONOS 가 있다.

# Network Management

아래 4 가지로 구성되어 있다.

- Managing server
  application, typically with network managers (humans) in the loop
- Network management protocol
  used by managing server to query, configure, manage device; used by devices to inform managing server of data, events.
- Managed device
  equipment with manageable, configurable hardware, software components
- Data
  device “state” configuration data, operational data, device statistics

## 방법

방법으로는 세 가지가 있다.

### CLI (Command Line Interface)

operator issues (types, scripts) direct to individual devices (e.g., vis ssh)

### SNMP/MIB

operator queries/sets devices data (MIB) using Simple Network Management Protocol (SNMP)
메시지 타입에 따라 네 가지의 기능을 한다.

- GetRequest, GetNextRequest, GetBulkRequest
  manager-to-agent, data instance, next data in list, block of data를 요청한다.
- SetRequest
  manager-to-agent, MIB value를 설정한다.
- Response
  Agent-to-manager, value, response를 응답한다.
- Trap
  Agent-to-manager, exceptional event를 알린다.

각각은 아래와 같이 구성된다.

<center><img src="/static/img/snmp-structure.png" alt="SNMP structure" style="max-width:100%;"/></center>

이는 선택적으로 MIB(Management Information Base)에서 처리될 수 있다. DDL은 SMI(structure of management information)으로 칭해진다.

### NETCONF/YANG

more abstract, network-wide, holistic emphasis on multi-device configuration management.

- YANG: data modeling language
- NETCONF: communicate YANG-compatible actions/data to/from/among remote devices

#### NETCONFIG

managing server와 managed NW device간에서 retrieve, set, modify, activate configurations을 한다.
atomic-commit action이 여러 디바이스에서 가능하며, operational data, statistic을 query할 수 있다. 해당 메시지는 XML로 인코드된다.
그 통신의 종류는 다음과 같다.

- `get-config`
  주어진 configuration의 부분 혹은 전체를 리퀘스트한다.
- `get`
  주어진 configuration state와 함께 operational state data의 부분 혹은 전체를 리퀘스트한다.
- `edit-config`
  특정한 configuration을 바꾼다. Managed device `rpc-reply` contains `ok` or `rpcerror` with rollback.
- `lock`, `unlock`
  managed device의 configuration datastore을 lock(unlock)한다.
- `create-subscription`, `notification`
  managed device로부터의 이벤트 notification을 구독하거나 허용한다.

#### YANG

데이터 모델링 언어로, NETCONFIG 데이터의 structure, syntax, semantic을 명시한다.
XML document의 형식을 하며, NETCONFIG configuration을 검증하기 위한 contraint를 표현할 수 있다.
