## NAME

NetEm - 네트워크 에뮬레이터

## SYNOPSIS

<pre>
<strong>tc qdisc</strong> ... <strong>dev</strong> <em>DEVICE</em> <strong>add netem</strong> <em>OPTIONS</em>

<em>OPTIONS</em> := [ <em>LIMIT</em> ]  [ <em>DELAY</em> ] [ <em>LOSS</em> ] [ <em>CORRUPT</em> ] [ <em>DUPLICATION</em> ] [ <em>REORDERING</em> ] [ <em>RATE</em> ]

<em>LIMIT</em> := <strong>limit</strong> <em>packets</em>

<em>DELAY</em> := <strong>delay</strong> <em>TIME</em> [ <em>JITTER</em> [ <em>CORRELATION</em> ]]
       [ <strong>distribution</strong> { <strong>uniform</strong> | <strong>normal</strong> | <strong>pareto</strong> | <strong>paretonormal</strong> } ]

<em>LOSS</em> := <strong>loss</strong> { <strong>random</strong> <em>PERCENT</em> [ <em>CORRELATION</em> ] |
               <strong>state</strong> <em>p13</em> [ <em>p31</em> [ <em>p32</em> [ <em>p23</em> [ <em>p14</em> ]]]] |
               <strong>gemodel</strong> <em>p</em> [ <em>r</em> [ <em>1-h</em> [ <em>1-k</em> ]]] } [ <strong>ecn</strong> ]

<em>CORRUPT</em> := <strong>corrupt</strong> <em>PERCENT</em> [ <em>CORRELATION</em> ]

<em>DUPLICATION</em> := <strong>duplicate</strong> <em>PERCENT</em> [ <em>CORRELATION</em> ]

<em>REORDERING</em> := <strong>reorder</strong> <em>PERCENT</em> [ <em>CORRELATION</em> ] [ <strong>gap</strong> <em>DISTANCE</em> ]

<em>RATE</em> := <strong>rate</strong> <em>RATE</em> [ <em>PACKETOVERHEAD</em> [ <em>CELLSIZE</em> [ <em>CELLOVERHEAD</em> ]]]
</pre>

## DESCRIPTION

NetEm은 특정 네트워크 인터페이스로 나가는 패킷들에 지연, 패킷 유실, 복제, 기타 다른 특성들을 더할 수 있는 리눅스 트래픽 제어 기능의 확장이다. NetEm은 리눅스 커널에 이미 있는 서비스 품질(QoS) 및 차등화 서비스(diffserv) 기능을 이용해 만들어졌다.

## netem 옵션

netem에는 다음 옵션이 있다.

### limit

설정한 옵션들의 효력을 다음으로 오는 지정한 수의 패킷들로 한정한다.

### delay

지정한 네트워크 인터페이스로 나가는 패킷에 지정한 지연을 더해 준다. 선택적 매개변수들을 통해 지연 변동과 상관도를 줄 수 있다. 지연 및 지터 값은 ms 단위로 나타내고 상관도는 퍼센트이다.

### destribution

지연 분포를 선택할 수 있다. 지정하지 않은 경우 기본은 정규 분포이다. 추가 매개변수들을 이용하면 동일 경로에서 겹치는 트래픽 흐름들에 따라 네트워크에 가변적인 지연이 생겨서 여러 지연 피크와 긴 꼬리를 유발하는 상황을 감안할 수 있다.

### loss random

지정한 네트워크 인터페이스로 나가는 패킷들에 독립적인 유실 확률을 더해 준다. 상관도를 주는 것도 가능하긴 하지만 잘못된 동작 방식 때문에 현재 제거 예정 상태다.

### loss state

입력 매개변수들을 전이 확률로 해서 4상 마르코프(Markov) 체인에 따라 패킷 유실을 더해 준다. 매개변수 p13은 필수이며 이것만 쓰면 베르누이(Bernoulli) 모델에 해당한다. 선택적 매개변수들을 통해 모델을 2상(p31), 3상(p23, p32), 4상(p14)으로 확장할 수 있다. 상태 1은 성공 수신에 해당하며, 상태 4는 독립적 유실, 상태 3은 버스트 유실, 상태 2는 버스트 내 성공 수신에 해당한다.

### loss gemodel

길버트-엘리엇(Gilbert-Elliot) 유실 모델 내지 그 특수 사례(길버트, 단순 길버트, 베르누이)에 따라 패킷 유실을 더해 준다. 베르누이 모델을 쓰려는 경우 필요한 매개변수는 p뿐이고 다른 매개변수들은 기본값인 r=1-p, 1-h=1, 1-k=0으로 설정된다. 단순 길버트(Simple Gilbert) 모델에 필요한 매개변수는 2개(p와 r)인 반면 길버트 모델에는 매개변수 3개(p, r, 1-h)가 필요하고 길버트-엘리엇 모델에는 4개(p, r, 1-h, 1-k)가 필요하다. 알다시피 p와 r은 나쁜 상태와 좋은 상태 간 전이 확률이며, 1-h는 나쁜 상태에서의 유실 확률, 1-k는 좋은 상태에서의 유실 확률이다.

### ecn

선택적으로 이 옵션을 사용해 패킷을 버리는 대신 표시를 붙일 수 있다. 이걸 쓰려면 유실 모델을 사용해야 한다.

### corrupt

지정한 비율의 패킷들의 임의 위치에 오류를 만드는 무작위 노이즈를 흉내낼 수 있다. 적당한 매개변수를 통해 상관도를 주는 것도 가능하다.

### duplicate

이 옵션을 쓰면 패킷들이 큐에 들어가기 전에 지정한 비율만큼 복제된다. 적당한 매개변수를 통해 상관도를 주는 것도 가능하다.

### reorder

순서 바꾸기를 하려면 delay 옵션을 지정해야 한다. 두 가지 방식으로 이 옵션을 사용할 수 있다. ('delay 10ms' 옵션을 지정했다고 가정.)

```text
reorder 25% 50% gap 5
```

이 예에서는 처음 4개(gap - 1) 패킷을 10ms만큼 지연시키고 후속 패킷들을 0.25 확률로 (상관도 50%로) 즉시 보내든지 0.75 확률로 지연시킨다. 패킷 순서가 바뀌고 나면 이 과정을 재시작한다. 즉 그 다음 4개 패킷을 지연시키고 후속 패킷들을 순서 바뀜 확률에 따라 즉시 보내거나 지연시킨다. 패킷이 5개마다 하나씩 확실히 순서가 바뀌는 재연 가능한 패턴을 만들려면 순서 바뀜 확률을 100%로 하면 된다.

```text
reorder 25% 50%
```

이 예에서는 패킷 25%를 (연관도 50%로) 즉시 보내고 나머지는 10ms 지연시킨다.

### rate

패킷 크기에 기반해 패킷을 지연시킨다. 즉 TBF를 대신할 수 있다. 흔히 쓰는 단위로 속도를 지정할 수 있다 (가령 100kbit). 선택적인 `PACKETOVERHEAD` 옵션은 (바이트 단위로) 패킷당 오버헤드를 나타내며 음수일 수도 있다. 양수 값을 사용해 추가로 있는 링크 계층 헤더들의 효과를 흉내낼 수 있으며, 음수 값을 사용해 (가령 -14라고 해서) 이더넷 헤더를 가상으로 벗겨 내거나 링크 계층 헤더 압축 체계를 흉내낼 수 있다. 세 번째 매개변수(부호 없는 값)는 셀 크기를 나타낸다. 그 셀 크기를 통해 링크 계층 체계를 흉내낼 수 있다. 예를 들어 ATM에서는 페이로드 셀 크기가 48바이트고 셀 헤더가 5바이트다. 패킷이 50바이트짜리라면 ATM에서 셀을 두 개 써야 하고, 48바이트 페이로드 2개에 5바이트 헤더 2개를 포함하면 회선 상에서 106바이트를 소모한다. 마지막 옵션 값 `CELLOVERHEAD`를 사용해 셀당 오버헤드를 지정할 수 있는데, ATM 예에서는 5가 된다. `CELLOVERHEAD`가 음수일 수 있기는 하지만 음수 값은 조심해서 써야 한다.

참고로 속도 제한을 하는 데 여러 제약 요인들이 있다. 커널의 클럭 정밀도 때문에 지정한 수준으로 완전한 셰이핑을 하는 게 불가능하다. 인위적 패킷 압축(버스트) 때 드러나게 된다. 영향을 끼치는 또 다른 요소로 네트워크 어댑터 버퍼가 있는데 거기서 인위적 지연이 더해질 수도 있다.

## 제약

Netem의 잘 알려진 제약들은 타이머 정밀도와 관련돼 있다. 리눅스가 실시간 운영 체제가 아니기 때문이다.

## EXAMPLES

```text
tc qdisc add dev eth0 root netem rate 5kbit 20 100 5
```

eth0 장치로 나가는 모든 패킷을 패킷당 오버헤드 20바이트, 셀 크기 100바이트, 셀 오버헤드 5바이트로 해서 5kbit 속도로 지연시킨다.

## SOURCES

1. Hemminger S., "Network Emulation with NetEm", Open Source Development Lab, April 2005 (http://devresources.linux-foundation.org/shemminger/netem/LCA2005_paper.pdf)

2. Netem page from Linux foundation, (http://www.linuxfoundation.org/en/Net:Netem)

3. Salsano S., Ludovici F., Ordine A., "Definition of a general and intuitive loss model for packet networks and its implementation in the Netem module in the Linux kernel", available at http://netgroup.uniroma2.it/NetemCLG

## SEE ALSO

`tc(8)`, `tc-tbf(8)`

## AUTHOR

Netem은 리눅스 재단의 Stephen Hemminger가 작성했으며 NISTnet을 기반으로 한다. 이 맨 페이지는 Favio Ludovici <favio.ludovici at yahoo dot it>와 Hagen Paul Pfeifer <hagen@jauu.net>가 만들었다.

----

2011년 11월 25일
