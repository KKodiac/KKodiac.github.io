---
layout: post
title: Redis Pipeline
date: 2022-09-14 05:38:00 +0900
description: Redis 에서 적용되는 Pipelining # Add post description (optional)
img: redis.png # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Redis, Pipeline, TCP]
---

# Pipelining in Redis

Redis에서 사용되는 pipelining 을 알아보기 전, pipelining이 무엇인지 우선 알아봅시다.

일반적인 TCP 통신은 아래 사진과 같이 Client-Server 핸드쉐이킹이 필요합니다. 연결을 기반으로 한 통신 이라서 그렇죠. Client 에서 server 로 갔다가 다시 오는 시간을 RTT(Round Trip Time) 이라고 합니다. 즉 1 RTT 란 **(1)client 에서 server 로 요청**을 보내고 **(2)요청을 server 에서 처리**후 **(3)해당 값을 다시 반환**하는 전체 과정에서 걸리는 시간이라는 거죠. 

![1200px-Tcp-handshake.svg.png]({{site.baseurl}}/assets/img/1200px-Tcp-handshake.svg.png)

여기서 지연시간에 포함 되는 자원은 두가지가 생깁니다. (1), (3) 을 처리하기 위해서는 [1]네트워크의 속도가 중요하겠죠.  그리고 (2), 즉 서버에서 해당 요청을 처리하는 과정에서는 서버 시스템의 kernel 레벨에서 사용하게되는 자원이 필요하게 됩니다. 서버에서 클라이언트에서 보낸 요청을 읽고, 다시 클라이언트로 보내는 과정에서 요청을 쓰게 됩니다. 이것을 **socket I/O** 라고 하는데, 시스템 함수(**syscall**) *read()* 와 *write()* 이 호출됩니다. 이러한 [2]I/O 작업은 Context Switch 로 인해 오버헤드 비용이 생길 수 밖에 없습니다.

![socket_network.png]({{site.baseurl}}/assets/img/socket_network.png)

> 따라서 이 두가지 **네트워크 사용과 I/O 작업**을 최대한 줄이는 방안이 속도 개선에 도움을 줄 것입니다.
> 

**HTTP Pipelining** 은 위 오버헤드를 줄이는 방식으로 아래 그림과 같은 방식으로 작동합니다. 클라이언트에서 서버로 요청하는 것을 한번에 여러 작업을 요청하면서 N 작업만큼 N RTT 가 걸리던 것을 N 작업 당  1 RTT 로 바꾸었습니다. 

![intro.png]({{site.baseurl}}/assets/img/pipeline.png)

Redis 에서 응용하는 pipelining 도 별 다를 것은 없습니다. Redis 서버로 요청하는 여러 작업을 한번에 받아 동일하게 처리됩니다. 하지만 추가적으로 엄청 많은 요청을 처리 시 Redis 에서는 배치(batch) 로 묶어 요청을 하는 것을 권장합니다. 이유는 한번의 요청으로 많은 양의 작업을 받을 시 서버 측은 이 모든 작업을 메모리를 할당하여 큐로 저장하는데, 매우 많은 양의 작업을 보낼 시 서버 메모리를 너무 많이 잡아 먹을 수 있기 때문이라고 합니다.