## NAME

ip-netns - 프로세스 네트워크 네임스페이스 관리

## SYNOPSIS

<pre>
<strong>ip</strong> [ <em>OPTIONS</em> ] <strong>netns</strong> { <em>COMMAND</em> | <strong>help</strong> }

<strong>ip netns</strong> [ <strong>list</strong> ]

<strong>ip netns add</strong> <em>NETNSNAME</em>

<strong>ip</strong> [<strong>-all</strong>] <strong>netns del</strong> [ <em>NETNSNAME</em> ]

<strong>ip netns set</strong> <em>NETNSNAME NETNSID</em>

<strong>ip netns identify</strong> [ <em>PID</em> ]

<strong>ip netns pids</strong> <em>NETNSNAME</em>

<strong>ip</strong> [<strong>-all</strong>] <strong>netns exec</strong> [ <em>NETNSNAME</em> ] <em>command</em>...

<strong>ip netns monitor</strong>

<strong>ip netns list-id</strong>
</pre>

## DESCRIPTION

네트워크 네임스페이스는 논리적으로 네트워크 스택의 또 다른 사본이다. 자체 라우트와 방화벽 규칙, 네트워크 장치가 있다.

기본적으로 프로세스는 부모의 네트워크 네임스페이스를 물려받는다. 처음에는 모든 프로세스가 init 프로세스에서 온 기본 네트워크 네임스페이스를 같이 공유한다.

관행 상 기명(named) 네트워크 네임스페이스란 `/var/run/netns/NAME`에 있는 열 수 있는 객체이다. `/var/run/netns/NAME`을 열어서 나온 파일 디스크립터가 지정한 네트워크 네임스페이스를 가리킨다. 파일 디스크립터를 열어 두면 네트워크 네임스페이스가 계속 살아 있게 된다. 이 파일 디스크립터를 <tt>[[setns(2)]]</tt> 시스템 호출에 사용해서 태스크에 연계된 네트워크 네임스페이스를 바꿀 수 있다.

네트워크 네임스페이스를 인식하는 응용에서는 전역 네트워크 설정을 `/etc/netns/NAME/`에서 먼저 찾아본 다음 `/etc/`에서 찾는 게 관행이다. 예를 들어 vpn을 격리시키기 위한 어느 네트워크 네임스페이스에 다른 버전의 `/etc/resolv.conf`를 쓰고 싶다면 이름을 `/etc/netns/myvpn/resolv.conf`라고 하게 될 것이다.

네트워크 네임스페이스 비인식 응용을 위해 `ip netns exec`가 이런 설정 및 파일 관행 처리를 자동화해 준다. 마운트 네임스페이스를 만들어서 네트워크 네임스페이스별 설정 파일 모두를 `/etc` 내의 전통적 위치로 결속(bind) 마운트 한다.

`ip netns list` - 모든 기명 네트워크 네임스페이스 표시하기
:   이 명령은 `/var/run/netns` 내의 네트워크 네임스페이스를 모두 표시한다.

`ip netns add NAME` - 새 기명 네트워크 네임스페이스 생성하기
:   이 명령은 `/var/run/netns/`에서 NAME이 사용 가능하면 새 네트워크 네임스페이스를 만들고 NAME을 부여한다.

`ip [-all] netns delete [ NAME ]` - 네트워크 네임스페이스(들)의 이름 삭제하기
:   `/var/run/netns`에 NAME이 있으면 언마운트 하고 마운트 지점을 제거한다. 이 명령이 그 네트워크 네임스페이스의 마지막 사용자이면 네트워크 네임스페이스가 해제되고 모든 물리적 장치들이 기본 네임스페이스로 옮겨진다. 그렇지 않으면 더 이상 사용자가 없을 때까지 네트워크 네임스페이스가 유지된다. 마운트 지점이 다른 마운트 네임스페이스에서 사용 중이면 `ip netns delete`가 실패할 수 있다.

    `-all` 옵션을 지정하면 모든 네트워크 네임스페이스 이름이 제거된다.

    물리적 장치를 어떤 netns로 옮기고서 동작 프로세스가 있는 채로 netns를 삭제하면 그 장치를 놓치게 될 수 있다.

        $ ip netns add net0
        $ ip link set dev eth0 netns net0
        $ ip netns exec net0 SOME_PROCESS_IN_BACKGROUND
        $ ip netns del net0

    이렇게 하면 SOME_PROCESS_IN_BACKGROUND가 종료하거나 죽은 후에만 eth0가 기본 netns에 나타나게 된다. 이를 막으려면 net0를 삭제하기 전에 그 netns에서 도는 프로세스들을 죽이는 게 좋다.

        $ ip netns pids net0 | xargs kill
        $ ip netns del net0

`ip netns set NAME SETNSID` - 상대 네트워크 네임스페이스에 id 부여하기
:   이 명령은 상대 네트워크 네임스페이스에 id를 부여한다. 이 id는 현재 네트워크 네임스페이스에서만 유효하다. 커널이 몇 가지 넷링크 메시지에서 이 id를 사용한다. 커널에서 필요할 때 id가 부여되어 있지 않으면 커널이 자동으로 부여한다. 한번 부여된 후에는 바꾸는 것이 불가능하다.

`ip netns identify [PID]` - 프로세스의 네트워크 네임스페이스들 이름 알려주기
:   이 명령은 `/var/run/netns`를 뒤져서 지정한 프로세스의 네트워크 네임스페이스에 대한 모든 네트워크 네임스페이스 이름을 찾는다. PID를 지정하지 않으면 현재 프로세스를 쓴다.

`ip netns pids NAME` - 기명 네트워크 네임스페이스 내의 프로세스들 알려주기
:   이 명령은 proc을 뒤져서 주 네트워크 네임스페이스가 지정한 네트워크 네임스페이스인 모든 프로세스를 찾는다.

`ip [-all] netns exec [ NAME ] cmd ...` - 기명 네트워크 네임스페이스에서 `cmd` 실행하기
:   이 명령을 이용하면 네트워크 네임스페이스 비인식인 응용을 기본 네트워크 네임스페이스 아닌 어딘가에서 돌릴 수 있다. 지정한 네트워크 네임스페이스에 대한 모든 설정들이 전통적인 전역 위치에 보인다. 네트워크 네임스페이스와 결속 마운트를 이용해서 다른 프로세스에 영향을 끼치지 않고 네트워크 네임스페이스별 위치에서 기본 위치로 파일들을 옮긴다.

    `-all` 옵션을 지정하면 기명 네트워크 네임스페이스 각각에서 동기적으로 `cmd`를 실행한다. 일부에서 `cmd`가 실패해도 마저 실행한다. 각 `cmd` 실행마다 네트워크 네임스페이스 이름을 찍는다.

`ip netns monitor` - 네트워크 네임스페이스 이름 추가되거나 삭제되면 알려주기
:   이 명령은 네트워크 네임스페이스 이름 추가 및 삭제 이벤트를 감시해서 이벤트마다 한 줄씩 찍는다.

`ip netns list-id` - 네트워크 네임스페이스 id(nsid) 나열하기
:   네트워크 네임스페이스 id는 상대 네트워크 네임스페이스를 식별하는 데 쓰인다. 이 명령은 현재 네트워크 네임스페이스의 nsid를 표시하며 대응하는 (`/var/run/netns`에서 온) iproute2 netns 이름이 있으면 알려 준다.

## EXAMPLES

`ip netns list`
:   현재의 기명 네트워크 네임스페이스 목록 보기

`ip netns add vpn`
:   네트워크 네임스페이스 생성하고 이름을 vpn으로 하기

`ip netns exec vpn ip link set lo up`
:   vpn 네트워크 네임스페이스에서 루프백 인터페이스 올리기

## SEE ALSO

`ip(8)`

## AUTHOR

Eric W. Biederman이 맨 페이지 작성함.

----

2013년 1월 16일
