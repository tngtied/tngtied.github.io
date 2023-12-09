---
layout: post
categories: Network
title: Network layer Control Plane - routing protocol
author: tngtied
date: 2023-12-10
---
routing protocol은 sending host로부터 receiving host까지의 path를 결정한다. 이 때의 path란 경로상의 sequence of routers를 의미한다. 해당 path가 efficient하고 fast하고 least congested하도록 결정해야 한다. 

routing alogorithm은 2가지 척도로 분류된다.
* global <-> decentralized
첫 번째 척도는 global한지 decentralized한지이다.
global한 경우 모든 라우터가 topology, link cost 정보를 가진다. “link state” algorithm이 사용된다.
decentralized의 경우 주변 라우터와의 link cost만을 가지고 주변과 iterative하게 통신한다. 이 때 “distance vector” algorithms이 사용된다.
* static <-> dynamic
static한 알고리즘의 경우 route는 천천히 바뀐다. 동적일 경우, 보다 빠르게 변하며 주기적으로 혹은 link cost 변경에 따라 update된다.

대표적으로 link state 알고리즘과 distance vector 알고리즘이 존재한다. 이는 3가지 척도로 비교 가능하다.
* message complexity 
LS의 경우 n개의 라우터가 존재한다면 O(n^2)개의 메시지가 전송된다.
DV의 경우 분산되어 있으므로 추정하기 어렵다.
* speed of convergence
LS의 경우 O(n^2)개의 알고리즘과 O(n^2)개의 메시지가 발생하므로 oscillation이 발생할 수 있다.
DV의 경우 추정하기 어려우며, routing loop, count-to-infinity가 발생할 수 있다.
* robustness (router malfunctions, compromisation)
LS의 경우 incorrect link cost를 notify할 수 있다. 또한, 매 라우터가 자신의 table만을 안다.
DV의 경우 incorrect path cost를 notify하며 black-holing을 유발할 수 있다. 또한, 매 라우터가 이웃 노드에게 알리기 때문에 에러가 네트워크를 통해 전파될 수 있다.

 

-----
# link state
link-state routing algorithm의 경우 iterative한 다익스트라를 사용한다. “link state broadcast” 를 통해 centralized함을 유지하며, 모든 노드가 동일한 정보를 가진다. pseudo code는 다음과 같다.

~~~
Initialization: 
    N' = {u}
    /* compute least cost path from u to all other nodes */
    for all nodes v 
        if v adjacent to u
        /* u initially knows direct-path-cost only to  direct neighbors */
            then D(v) = cu,v
        /* but may not be minimum cost!*/
        else D(v) = ∞ 
Loop: 
    find w not in N' such that D(w) is a minimum 
    add w to N' 
    update D(v) for all v adjacent to w and not in N' : 
        D(v) = min ( D(v),  D(w) + cw,v  ) 
    /* new least-path-cost to v is either old least-cost-path to v or known least-cost-path to w plus direct-cost from w to v */ 
until all nodes in N' 
~~~

해당 알고리즘의 time complexity는 O(n^2)이다. n iteration에, n(n+1)/2 comparison을 필요로 하기 때문이다. 
message complexity의 경우, 매 router가 link state information을 모든 n개의 라우터에게 **broadcast**해야 한다. 즉 overall message complexity는 O(n^2)이다.
link costs가 traffic volume에 의존할 경우, route oscillations이 발생가능하다.

# distance vector
벨만 포드 알고리즘을 사용한다. 매번 매 노드가 distance vector estimate을 주변 노드에게 전송한다. 전송받을 때마다, 노드는 BF 공식을 사용해 DV를 업데이트하거나 하지 않는다. 업데이트될 경우, 또 다시 주변 노드들에게 알린다. 
이는 iterative 하며 asynchronous하다. 즉, 매 local iteration은 local link cost change 혹은 DV update message from neighbour 둘 중 하나로 인해 발생한다.
또한, distributed 하며 self stoping하다.
아래는 수도코드이다.
~~~
Let Dx(y): cost of least-cost path from x to y.
Then:
   Dx(y) = minv { cx,v + Dv(y) }
~~~

distance vector에서 Link cost change에 의한 notify는 “bad news travels slow”하다. 즉, 링크의 cost가 높아질 경우, 서로 무한하게 cost를 업데이트하게 되는 문제가 발생할 수 있다.
