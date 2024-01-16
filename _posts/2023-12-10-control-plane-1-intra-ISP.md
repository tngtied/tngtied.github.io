---
layout: post
categories: [Network]
title: Network layer Control Plane - ISP
author: tngtied
date: 2023-12-10
---

앞서 살펴본 라우팅 알고리즘은 두 가지 이유로 인해 불가능하다.

- scale: router의 수는 매우 많다. 또한 routing table exchange로 인한 congestion이 발생할 수 있다.
- administrative autonomy: 인터넷은 네트워크의 네트워크로 구성되어 있으며, 매 네트워크는 각자의 방식으로 routing을 통제한다.

# Intra ISP

Scalable routing을 위해 Autonomous System(AS) 즉 domain이 사용된다. 이는 intra-AS(domain), inter-AS(domain)으로 나뉜다.
이를 구분하는 이유는 다음과 같다.

- policy
  inter-AS의 admin들은 traffic이 route되는 방법, 해당 네트워크에서 route하는 주체를 관리하고 싶어하므로 policy의 문제가 발생한다.
  inter-AS의 admin은 단일하며, policy의 문제가 없다.
- scale
  hierarchical routing은 table size와 update traffic을 줄인다.
- performance
  intra-AS는 성능에 집중할 수 있으며, inter-AS는 성능에 영향을 주는 policy를 수립한다.

## intra-AS

동일한 AS 내부에서 routing하는 것을 의미한다.
AS 내부의 모든 라우터가 동일한 intra-domain protocol을 사용한다.
다른 AS와 해당 AS를 연결하는 gateway router가 존재한다.

## inter-AS

AS들 간의 routing을 의미한다. 앞서 언급한 gateway가 inter-domain router과 intra-domain routing 둘 다를 수행하며, 둘 모두로 forwarding table을 구성한다. 이 때, 각 이웃한 AS들을 통해 도달가능한 destination을 알고 있어야만 하며, 도달가능함을 AS 내부의 모든 router에게 알려야 한다.
inter-AS routing의 예시로는 다음과 같은 것이 있다.

- RIP: Routing Information Protocol
- EIGRP: Enhanced Interior Gateway Routing Protocol
  위 두 routing은 DV를 기반으로 한다.
- OSPF: Open Shortest Path First [RFC 2328]
  해당 routing은 LS를 기반으로 한다.

---

# OSPF

publicly available하다. classic link-state 알고리즘을 사용하며, 모든 OSPF 메시지는 인증되어야만 한다.
매 라우터는 TCP/UDP가 아닌 IP를 사용하여 link state advertisement를 한다. link cost는 bandwidth와 delay와 같은 다양한 metric이 사용가능하다. 매 라우터는 다익스트라 알고리즘을 사용하며 full topology를 가진다.

2 level hierarchy를 지닌다. area border router들을 모두 종합하는 backbone이 존재하며, 여기서 area border router는 해당 area 내부의 거리 정보를 요약해 backbon에 전달한다. backbone area 내부의 boundary router는 AS의 gateway 역할을 한다.

---

# BGP(Border Gateway Protocol)

subnet이 외부 망에 존재와 자신이 도달 가능한 destination를 advertise하는 방법이다.
BGF는 각 AS에게 아래와 같은 것들을 제공한다.

- eBGP: subnet reachability를 이웃 AS들로부터 얻는다
- iBGP: 얻은 reachability를 AS-internal router에게 전달한다.

BGP를 전달받는 gateway는 path를 받아들일지 말지, 그리고 이웃한 AS에게 이를 홍보할지 말지를 결정하는 import policy를 각자 가진다.

BGP advertisement는 provider network, customer network를 구분한다. ISP들은 customer networks를 도착지 혹은 출발지로 하는 route만을 원하며, 다른 ISP와의 transit traffic을 전개하지 않는다.

## path attribute

BGP는 route, 즉 prefix와 attribute의 총합을 교환한다. 이 때 attribute는 2 가지가 있다.

- AS-PATH: prefix가 거쳐온 AS의 목록
- NEXT-HOP: 다음에 거칠 특정한 internal-AS router

## BGP messages

BGP는 BGP session, 즉 두 BGP router 간의 semi-permanant한 TCP 연결을 통한 BGP msg 교환을 통해 이루어진다. BGP는 path vector protocol로, BGP session에서 교환하는 것은 각각 다른 도착지 prefix에 대한 path들이다.

BGP message의 종류는 네 가지가 있다.

- OPEN: TCP connection (BGP session)을 개시한다.
- UPDATE: advertises new path (or withdraws old)
- KEEPALIVE: OPEN request를 ACK하거나, UPDATE가 없을 때 connection을 keep-alive 한다.
- NOTIFICATION: 이전 메시지의 에러를 알리거나, connection을 close 한다.

## route selection

router는 아래와 같은 요인들을 고려하여 destination AS로의 route를 결정한다.

- local preference value attribute: policy decision
- shortest AS-PATH
- closest NEXT-HOP router: hot potato routing
  **hot potato routing**, choose local gateway that has least intra-domain cost이 가능해진다.
- additional criteria
