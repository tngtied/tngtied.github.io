---
layout: post
categories: Network
title: Network layer Data Plane - IP
author: tngtied
date: 2023-12-09
---

# datagram format
<center><img src="/static/img/datagram-architecture.png" alt="Datagram Architecture" style="max-width:100%;"/></center>
데이터그램은 위와 같이 구성되어 있다.

-----
# IP fragmentation/reassembly
network link들은 max transfer size를 보유한다. 이를 초과하는 데이터그램은 fragment되며, 도착지에서만 재조립된다. order related한 fragmentation에서 각각의 조각들은 IP header bits로 identify된다.

-----

# addressing
IP 주소는 32-bit의 식별자로, host 혹은 router interface를 지칭한다. 여기서 interface란 host/router와 physical link간의 연결을 의미한다. host와 달리 라우터의 경우 대개 여러 개의 interface를 가진다.
## subnet
subnet이란 router를 통하지 않고도 서로 연결될 수 있는 인터페이스들을 의미한다. ip 주소는 subnet part와 host part로 구성되어 있다. 즉, 동일한 subnet의 interface들의 ip주소는 일정 부분 동일하고, 해당 부분을 subnet part라고 하며, 동일한 subnet 내의 interface들을 구분하는 것이 나머지 host part인 것이다. subnet part는 CIDR (Classless InterDomain Routing)이라고 불린다. 
이 때 subnet을 의미하는 Mask는 /n(24) 으로 표현되며, 상위의 n bit들이 CIDR임을 의미한다.
네트워크는 provider ISP address의 subnet part를 보유함으로써 subnet의 주소를 알 수 있다. 

## DHCP(Dynamic Host Configuration Protocol)
host가 IP 주소를 얻는 것은 두 가지 방법으로 가능하다. 첫 번째는 config file의 sysadmin으로부터 하드코드될 수 있다. 두 번째 방법이 DHCP이다. 동적으로 서버로부터, 네트워트에 참가할 때 ip 주소를 얻는 것이다.
이는 아래와 같은 특성을 지닌다.
* 사용 중인 주소를 갱신할 수 있다.
* 주소의 재사용을 가능하게 한다
* 모바일 유저에게 주소를 부여할 수 있다.

DHCP는 ip 주소 뿐 아니라 아래와 같은 것들을 제공할 수 있다.
* first-hop router의 주소
* DNS 서버의 이름과 ip 주소
* network mask

아래와 같은 과정을 거친다.
1. host broadcasts DHCP discover msg \[optional\]
해당 매시지는 UDP, IP, Ethernet으로 encapsulated된다. 이의 destination은 FFFFFFFFFFFF로 랜에 전송된다. 
2. DHCP server responds with DHCP offer msg \[optional\]
3. host requests IP address: DHCP request msg
4. DHCP server sends address: DHCP ack msg 
해당 msg는 client ip address, first-hop router ip address, DNS server name & IP address 를 담는다. 

-----

# network address translation (NAT)
하나의 IPv4 주소를 공유하는 지역 네트워크의 기기들은 NAT를 필요로 한다. 지역 네트워크로부터 전송되는 데이터그램은 하나의 주소, 각기 다른 source port number를 가진다. 지역 네트워크 안에서 기기들은 해당 네트워크에서만 사용되는 32-bit 형식의 private IP 주소를 가진다.

이렇게 했을 때 이점은 아래와 같다.
* ISP가 다수의 기기들을 서비스할 때 하나의 IP 주소만을 필요로 한다.
* 지역망 안에서 host의 주소를 변경할 때 외부에 알릴 필요가 없다
* ISP를 바꿀 때 지역망의 기기들의 주소를 전부 바꿀 필요가 없다.
* 지역망 내부의 기기들이 직접적으로 접근가능하지 않으므로 보안상 이점이 있다.
이러한 이점들로 인하여 가정과 기관의 네트워크 그리고 cellular network에서 주로 사용된다.

NAT 라우터는 아래와 같은 기능들을 구현해야 한다. 
* replace 
(source IP address, port #) to (NAT IP address, new port #) outgoing datagram
* remember
in NAT translation table (source IP address, port #)  to (NAT IP address, new port #) 
* replace 
(NAT IP address, new port #) to (source IP address, port #) incoming datagram stored in NAT table

NAT 라우터의 문제점은 아래와 같다.
* routers should only process up to layer 3
* IPv6 address shortage
* violates end-to-end argument 
예시: port # manipulated by NW layer
* NAT traversal
예시: client가 NAT 뒤의 서버와 연결되고 싶어할 경우

-----
# IPv6
IPv6의 목표는 32bit의 IP 주소를 완전히 담는 것이다. 또한 추가적으로 40byte의 고정된 헤더 길이를 통해 processing/forwarding 속도를 높이고 different NW layer "flow"를 다루기 위해서 고안되었다.

구조는 아래와 같다.
<center><img src="/static/img/IPv6-datagram.png" alt="IPv6 Datagram Architecture" style="max-width:100%;"/></center>
IPv4와 비교했을 때 checksum, fragment/reassembly, option이 없다. 옵션의 경우 대신에 upper layer, next header protocol에서 추가 가능하다.

## Tunneling
네트워크는 IPv4와 IPv6 라우터의 집합으로 구성되어 있다. 이 때 IPv6 데이터그램이 IPv4 라우터에서 처리되어야 하는 경우가 발생한다. 그 경우, IPv4 데이터그램 내부의 payload로서 담겨지는 tunneling과정을 거친다.
tunneling은 logical view에서 IPv6/v4(router A) 와 IPv6/v4(router B) 간의 터널을 거치는 것으로 표현된다. 터널을 거치는 과정에서, IPv6 데이터그램을 캡슐화한 IPv4 데이터그램은 기존의 src와 dest가 아닌, A를 src로 그리고 B를 dest로 가진다. 기존의 src와 dest는 페이로드 내부에 보존된다.