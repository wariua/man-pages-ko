## NAME

veth - 가상 이더넷 장치

## DESCRIPTION

**veth** 장치는 가상 이더넷 장치이다. 네트워크 네임스페이스 간 터널 역할을 하며 다른 네임스페이스의 물리적 네트워크 장치로 가는 다리가 된다. 물론 단독 네트워크 장치로 사용할 수도 있다.

**veth** 장치는 언제나 서로 연결된 쌍으로 생겨난다. 다음 명령으로 쌍을 만들 수 있다.

```
# ip link add <p1-name> type veth peer name <p2-name>
```

`p1-name`과 `p2-name`은 연결된 두 종점에 부여할 이름이다.

쌍의 한 장치로 전송한 패킷이 다른 장치에서 즉시 수신된다. 어느 한쪽 장치가 내려가면 그 쌍의 링크 상태가 내려간다.

**veth** 장치 쌍이 유용한 점은 커널의 네트워크 요소들을 흥미로운 방식으로 결합시킬 수 있다는 것이다. 특히 흥미로운 사용 방식은 **veth** 쌍의 한쪽 끝을 한 네트워크 네임스페이스에 두고 다른 끝을 다른 네트워크 네임스페이스에 두어서 네트워크 네임스페이스 간 통신을 할 수 있게 하는 것이다. 이를 위해 먼저 위와 같이 **veth** 장치를 생성하고서 쌍의 한 쪽을 다른 네임스페이스로 옮긴다.

```
# ip link set <p2-name> netns <p2-namespace>
```

`ethtool(8)`을 이용하면 다음 같은 명령으로 **veth** 네트워크 인터페이스의 짝을 찾을 수 있다.

```
# ip link add ve_A type veth peer name ve_B   # veth 쌍 생성
# ethtool -S ve_A         # 짝의 인터페이스 번호 알아내기
NIC statistics:
     peer_ifindex: 16
# ip link | grep '^16:'   # 인터페이스 검색
16: ve_B@ve_A: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc ...
```

## SEE ALSO

<tt>[[clone(2)]]</tt>, <tt>[[network_namespaces(7)]]</tt>, `ip(8)`, `ip-link(8)`, <tt>[[ip-netns(8)]]</tt>

----

2018-02-02
