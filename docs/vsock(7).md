## NAME

vsock - 리눅스 VSOCK 주소 패밀리

## SYNOPSIS

```c
#include <sys/socket.h>
#include <linux/vm_sockets.h>

stream_socket = socket(AF_VSOCK, SOCK_STREAM, 0);
datagram_socket = socket(AF_VSOCK, SOCK_DGRAM, 0);
```

## DESCRIPTION

VSOCK 주소 패밀리는 가상 머신과 그 머신이 도는 호스트 간의 통신을 쉽게 만들어 준다. 가상 머신 망 구성과 독립적인 통신 채널이 필요한 게스트 에이전트와 하이퍼바이저 서비스에서 이 주소 패밀리를 사용한다.

유효한 소켓 타입은 `SOCK_STREAM`과 `SOCK_DGRAM`이다. `SOCK_STREAM`은 전달과 순서 유지가 보장되는 연결 지향 바이트 스트림을 제공한다. `SOCK_DGRAM`은 최선형 전달 및 최선형 순서 유지 방식의 무연결 데이터그램 패킷 서비스를 제공한다. 이 소켓 타입들을 사용할 수 있는지 여부는 기반 하이퍼바이저에 따라 달라진다.

다음과 같이 새 소켓을 만든다.

```c
socket(AF_VSOCK, socket_type, 0);
```

프로세스에서 연결을 수립하고 싶으면 어떤 목적 소켓 주소로 `connect(2)`를 호출한다. 소켓이 포트에 결속되어 있지 않으면 자동으로 빈 포트에 결속된다.

프로세스에서 `bind(2)`로 소켓 주소에 결속하고서 `listen(2)`을 호출하여 들어오는 연결에 귀를 기울일 수 있다.

<tt>[[send(2)]]</tt> 내지 `write(2)` 계열 시스템 호출을 이용해 데이터를 송신하고 <tt>[[recv(2)]]</tt> 내지 `read(2)` 계열 시스템 호출을 이용해 데이터를 수신한다.

### 주소 형식

32비트 문맥 식별자(Context Identifier, CID)와 32비트 포트 번호의 조합으로 소켓 주소를 정의한다. CID는 출발지나 목적지를 나타내는데, 가상 머신이나 호스트 중 한쪽이다. 포트 번호는 동일 머신 상에서 돌고 있는 여러 서비스들을 구별해 준다.

```c
struct sockaddr_vm {
	sa_family_t    svm_family      /* 주소 패밀리: AF_VSOCK */
	unsigned short svm_reserved1;
	unsigned int   svm_port;       /* 포트 번호, 호스트 바이트 순서 */
	unsigned int   svm_cid;        /* 주소, 호스트 바이트 순서 */
};
```

`svm_family`는 항상 `AF_VSOCK`으로 설정한다. `svm_reserved1`은 항상 0으로 설정한다. `svn_port`는 포트 번호를 호스트 바이트 순서로 담는다. 1024 아래의 포트 번호를 *특권 포트*라고 한다. `CAP_NET_BIND_SERVICE` 역능을 가진 프로세스만 그 포트 번호에 `bind(2)` 할 수 있다.

특수 주소들이 여러 가지 있다. `VMADDR_CID_ANY`(-1U)는 아무 주소에 결속하라는 뜻이다. `VMADDR_CID_HYPERVISOR`(0)는 하이퍼바이저에 내장된 서비스를 위해 예약된 것이다. `VMADDR_CID_RESERVED`(1)는 사용해서는 안 된다. `VMADDR_CID_HOST`(2)는 호스트의 잘 알려진 주소이다.

특수 상수 `VMADDR_PORT_ANY`(-1U)는 아무 포트 번호에 결속하라는 뜻이다.

### 라이브 마이그레이션

소켓이 가상 머신 라이브 마이그레이션의 영향을 받는다. 가상 머신이 새 호스트로 이동할 때 연결되어 있던 `SOCK_STREAM` 소켓들이 끊어진다. 이렇게 되면 응용에서 재연결을 해야 한다.

이전 CID를 새 호스트에서 사용할 수 없다면 라이브 마이그레이션을 거치면서 로컬 CID가 바뀔 수도 있다. 결속된 소켓들이 자동으로 새 CID로 갱신된다.

### ioctl

`IOCTL_VM_SOCKETS_GET_LOCAL_CID`
:   로컬 머신의 CID를 얻는다. 인자는 `unsigned int` 포인터이다.

        ioctl(socket, IOCTL_VM_SOCKETS_GET_LOCAL_CID, &cid);

    결속 시 `IOCTL_VM_SOCKETS_GET_LOCAL_CID`로 로컬 CID를 얻을 필요 없이 `VMADDR_CID_ANY`를 사용할 수 있다.

## ERRORS

`EACCES`
:   `CAP_NET_BIND_SERVICE` 역능이 없어서 특권 포트에 결속할 수 없다.

`EADDRINUSE`
:   이미 사용 중인 포트에 결속할 수 없다.

`EADDRNOTAVAIL`
:   결속할 빈 포트를 찾을 수 없거나 로컬 아닌 CID에 결속할 수 없다.

`EINVAL`
:   유효하지 않은 매개변수. 이미 결속된 소켓을 결속하려 한 경우, 유효하지 않은 `sockaddr_vm` 구조체를 제공한 경우, 기타 입력 검증 오류.

`ENOPROTOOPT`
:   `setsockopt(2)`나 `getsockopt(2)`에 유효하지 않은 소켓 옵션 사용.

`ENOTCONN`
:   연결 안 된 소켓이어서 동작을 수행할 수 없음.

`EOPNOTSUPP`
:   동작을 지원하지 않음. `MSG_OOB` 플래그가 <tt>[[send(2)]]</tt> 계열 시스템 호출에 구현되어 있지 않거나 `MSG_PEEK`이 <tt>[[recv(2)]]</tt> 계열 시스템 호출에 구현되어 있지 않음.

`EPROTONOSUPPORT`
:   유효하지 않은 소켓 프로토콜 번호. 프로토콜이 항상 0이어야 한다.

`ESOCKTNOSUPPORT`
:   <tt>[[socket(2)]]</tt>에서 지원하지 않는 소켓 타입. `SOCK_STREAM`과 `SOCK_DGRAM`만 유효하다.

## VERSIONS

리눅스 3.9부터 VMware(VMCI) 지원을 이용 가능하다. 리눅스 4.8부터 KVM(virtio)을 지원한다. 리눅스 4.14부터 Hyper-V를 지원한다.

## SEE ALSO

`bind(2)`, `connect(2)`, `listen(2)`, <tt>[[recv(2)]]</tt>, <tt>[[send(2)]]</tt>, <tt>[[socket(2)]]</tt>, <tt>[[capabilities(7)]]</tt>

----

2017-11-30
