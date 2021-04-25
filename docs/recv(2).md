## NAME

recv, recvfrom, recvmsg - 소켓에서 메시지 받기

## SYNOPSIS

```c
#include <sys/socket.h>

ssize_t recv(int sockfd, void *buf, size_t len, int flags);
ssize_t recvfrom(int sockfd, void *restrict buf, size_t len, int flags,
                 struct sockaddr *restrict src_addr,
                 socklen_t *restrict addrlen);
ssize_t recvmsg(int sockfd, struct msghdr *msg, int flags);
```

## DESCRIPTION

`recv()`, `recvfrom()`, `recvmsg()` 호출을 이용해 소켓에서 메시지를 받는다. 무연결 소켓과 연결 지향 소켓 모두에서 데이터를 받을 수 있다. 이 페이지에서는 먼저 세 가지 시스템 호출의 공통 기능을 기술한 다음 호출들 간의 차이점을 기술한다.

`recv()`와 `read(2)`의 유일한 차이는 `flags`의 존재 여부다. `flags` 인자가 0이면 `recv()`는 일반적으로 `read(2)`와 동등하다. (하지만 NOTES 참고.) 또한 다음 호출은

```c
recv(sockfd, buf, len, flags);
```

다음과 동등하다.

```c
recvfrom(sockfd, buf, len, flags, NULL, NULL);
```

세 호출 모두 성공 완료 시 메시지 길이를 반환한다. 메시지가 너무 길어서 제공 버퍼에 들어가지 않는 경우 메시지를 받는 소켓의 종류에 따라선 넘치는 바이트들을 버릴 수도 있다.

수신 호출들은 소켓에 메시지가 없으면 메시지가 도착하기를 기다린다. 단 소켓이 논블로킹이면 (<tt>[[fcntl(2)]]</tt> 참고) -1 값을 반환하고 `errno`를 `EAGAIN`이나 `EWOULDBLOCK`으로 설정한다. 수신 호출들은 보통 요청 받은 양 전체를 수신할 때까지 기다리기보다는 사용 가능한 데이터를 최대 요청 크기만큼 바로 반환한다.

응용에서 <tt>[[select(2)]]</tt>, <tt>[[poll(2)]]</tt>, <tt>[[epoll(7)]]</tt>을 이용해 소켓에 데이터가 더 도착했는지 알아낼 수 있다.

### `flags` 인자

`flags` 인자는 다음 값들을 1개 이상 OR 해서 구성한다.

`MSG_CMSG_CLOEXEC` (`recvmsg()` 전용, 리눅스 2.6.23부터)
:   (<tt>[[unix(7)]]</tt>에서 기술하는) `SCM_RIGHTS` 동작으로 유닉스 도메인 파일 디스크립터를 통해 수신하는 파일 디스크립터에 exec에서 닫기 플래그를 설정한다. <tt>[[open(2)]]</tt>의 `O_CLOEXEC` 플래그와 같은 이유로 이 플래그가 유용하다.

`MSG_DONTWAIT` (리눅스 2.2부터)
:   논블로킹 동작을 켠다. 동작이 블록 되려는 경우 호출이 `EAGAIN`이나 `EWOULDBLOCK`으로 실패한다. (<tt>[[fcntl(2)]]</tt> `F_SETFL` 동작을 통해) `O_NONBLOCK` 플래그를 설정한 경우와 비슷한 동작 방식인데, `MSG_DONTWAIT`이 호출별 옵션인 반면 `O_NONBLOCK`은 열린 파일 기술 항목에 대한 설정이어서 (<tt>[[open(2)]]</tt> 참고) 호출 프로세스 내 모든 스레드뿐 아니라 같은 열린 파일 기술 항목을 가리키는 파일 디스크립터를 가진 다른 프로세스에도 영향을 끼치게 된다.

`MSG_ERRQUEUE` (리눅스 2.2부터)
:   이 플래그는 소켓 오류 큐에 있는 오류를 받아와야 함을 나타낸다. 보조 메시지로 오류가 전달되며 그 타입은 프로토콜에 따라 다르다. (IPv4에서는 `IP_RECVERR`.) 사용자가 충분한 크기의 버퍼를 제공해야 한다. 자세한 내용은 <tt>[[cmsg(3)]]</tt>와 <tt>[[ip(7)]]</tt>를 보라. 오류를 유발한 원인 패킷의 페이로드가 `msg_iovec`을 통해 정상 데이터처럼 전달된다. 오류를 유발한 데이터그램의 원래 목적 주소가 `msg_name`으로 제공된다.

    `sock_extended_err` 구조체로 오류를 제공한다.

        #define SO_EE_ORIGIN_NONE    0
        #define SO_EE_ORIGIN_LOCAL   1
        #define SO_EE_ORIGIN_ICMP    2
        #define SO_EE_ORIGIN_ICMP6   3

        struct sock_extended_err
        {
            uint32_t ee_errno;   /* 오류 번호 */
            uint8_t  ee_origin;  /* 오류가 어디서 왔는가 */
            uint8_t  ee_type;    /* 타입 */
            uint8_t  ee_code;    /* 코드 */
            uint8_t  ee_pad;     /* 패딩 */
            uint32_t ee_info;    /* 추가 정보 */
            uint32_t ee_data;    /* 기타 데이터 */
            /* 데이터가 더 올 수 있음 */
        };

        struct sockaddr *SO_EE_OFFENDER(struct sock_extended_err *);

    `ee_errno`는 큐에 있는 오류의 `errno` 번호를 담는다. `err_origin`은 오류가 발생한 곳을 나타내는 코드이다. 나머지는 프로토콜별 필드이다. 매크로 `SOCK_EE_OFFENDER`는 보조 메시지에 대한 포인터를 받아서 오류의 근원이 된 네트워크 객체의 주소에 대한 포인터를 반환한다. 그 주소를 알지 못하면 `sockaddr`의 `sa_family`가 `AF_UNSPEC`을 담으며 `sockaddr`의 나머지 필드들이 정의돼 있지 않다. 오류를 유발한 패킷의 페이로드는 정상 데이터처럼 전달된다.

    로컬 오류인 경우 주소가 전달되지 않는다. (`cmsghdr`의 `cmsg_len` 멤버로 확인할 수 있다.) 오류 수신에서는 `msghdr`에 `MSG_ERRQUEUE` 플래그가 설정돼 있다. 오류가 전달된 후에는 큐에 있는 다음 오류에 기반해서 미처리 소켓 오류가 재생성되어 다음 소켓 동작으로 전달된다.

`MSG_OOB`
:   이 플래그는 정상 데이터 스트림으로는 받게 되지 않을 대역외 데이터 수신을 요청한다. 어떤 프로토콜에서는 긴급 데이터를 정상 데이터 큐 선두에 집어넣으며, 그래서 그런 프로토콜에서는 이 플래그를 사용할 수 없다.

`MSG_PEEK`
:   이 플래그는 수신 동작이 수신 큐 처음의 데이터를 반환하되 큐에서 그 데이터를 제거하지 않게 한다. 따라서 이어지는 수신 호출이 같은 데이터를 반환하게 된다.

`MSG_TRUNC` (리눅스 2.2부터)
:   로우 (`AF_PACKET`), 인터넷 데이터그램 (리눅스 2.4.27/2.6.8부터), 넷링크 (리눅스 2.6.22부터), 유닉스 데이터그램 (리눅스 3.4부터) 소켓에서: 패킷 내지 데이터그램의 실제 길이가 전달한 버퍼보다 컸던 경우에도 그 실제 길이를 반환한다.

    인터넷 스트림 프로토콜에서의 사용에 대해선 <tt>[[tcp(7)]]</tt> 참고.

`MSG_WAITALL` (리눅스 2.2부터)
:   이 플래그는 요청 전체가 충족될 때까지 동작이 블록하기를 요청한다. 하지만 시그널을 잡거나, 오류 내지 연결 끊김이 발생하거나, 다음 수신 데이터가 다른 타입인 경우에는 여전히 요청보다 적은 데이터를 반환할 수 있다. 데이터그램 소켓에는 이 플래그가 효과가 없다.

### `recvfrom()`

`recvfrom()`은 수신한 메시지를 버퍼 `buf`에 집어넣는다. 버퍼의 크기를 호출자가 `len`에 지정해야 한다.

`src_addr`이 NULL이 아니며 기반 프로로콜에서 메시지의 출발 주소를 제공하는 경우 그 출발 주소를 `src_addr`이 가리키는 버퍼에 집어넣는다. 이 경우에 `addrlen`은 값이자 결과인 인자이다. 호출 전에는 `src_addr`에 연계된 버퍼의 크기로 초기화 해야 한다. 그리고 반환 시 출발 주소의 실제 크기를 담도록 `addrlen`이 갱신된다. 제공 버퍼가 너무 작으면 반환 주소가 잘리는데, 이 경우 `addrlen`이 호출에 줬던 것보다 큰 값을 반환하게 된다.

호출자가 출발 주소에 관심이 없으면 `src_addr`과 `addrlen`을 NULL로 지정해야 한다.

### `recv()`

`recv()` 호출은 보통 *연결* 상태인 (`connect(2)` 참고) 소켓에서만 쓴다. 다음 호출과 동등하다.

```c
recvfrom(fd, buf, len, flags, NULL, 0);
```

### `recvmsg()`

`recvmsg()` 호출에서는 `msghdr` 구조체를 사용해 직접 제공하는 인자들의 수를 줄인다. 이 구조체는 `<sys/socket.h>`에 다음과 같이 정의되어 있다.

```c
struct iovec {                    /* 스캐터/개더 배열 항목 */
    void  *iov_base;              /* 시작 주소 */
    size_t iov_len;               /* 이동할 바이트 수 */
};

struct msghdr {
    void         *msg_name;       /* 주소, 선택적 */
    socklen_t     msg_namelen;    /* 주소 크기 */
    struct iovec *msg_iov;        /* 스캐터/개더 배열 */
    size_t        msg_iovlen;     /* msg_iov의 항목 개수 */
    void         *msg_control;    /* 보조 데이터, 아래 참고 */
    size_t        msg_controllen; /* 보조 데이터 버퍼 길이 */
    int           msg_flags;      /* 수신 메시지에 대한 플래그 */
};
```

`msg_name` 필드는 호출자 할당 버퍼를 가리키며 소켓이 연결돼 있지 않은 경우 출발 주소를 반환하는 데 이 버퍼를 쓴다. 호출 전에 호출자가 `msg_namelen`에 버퍼 크기를 설정해야 하며 성공 호출 반환 시 `msg_namelen`이 반환 주소의 길이를 담게 된다. 응용에서 출발 주소를 알 필요가 없으면 `msg_name`을 NULL로 지정하면 된다.

`msg_iov` 및 `msg_iovlen` 필드는 <tt>[[readv(2)]]</tt>에서처럼 스캐터-개더 위치들을 기술한다.

`msg_control` 필드는 여타 프로토콜 제어 관련 메시지나 여러 보조 데이터를 위한 `msg_controllen` 길이의 버퍼를 가리킨다. `recvmsg()` 호출 시 `msg_controllen`이 `msg_control`의 가용 버퍼 길이를 담고 있어야 한다. 성공 호출 반환 시 제어 메시지 열의 길이를 담게 된다.

메시지는 다음 형태이다.

```c
struct cmsghdr {
    size_t cmsg_len;    /* 데이터 바이트 개수. 헤더 포함.
                           (POSIX에서는 타입이 socklen_t) */
    int    cmsg_level;  /* 기원 프로토콜 */
    int    cmsg_type;   /* 프로토콜별 타입 */
/* 이어서:
    unsigned char cmsg_data[]; */
};
```

보조 데이터는 <tt>[[cmsg(3)]]</tt>에 정의된 매크로를 통해서만 접근해야 한다.

예를 들어 리눅스에서는 이 보조 데이터 메커니즘을 이용해 확장 오류나 IP 옵션을, 그리고 유닉스 도메인 소켓 상으로 파일 디스크립터를 전달한다. 다양한 소켓 도메인에서 보조 데이터를 이용하는 방식에 대한 자세한 설명은 <tt>[[unix(7)]]</tt>와 <tt>[[ip(7)]]</tt>를 보라.

`recvmsg()` 반환 시 `msghdr`의 `msg_flags` 필드가 설정된다. 여러 플래그가 담길 수 있다.

`MSG_EOR`
:   레코드 끝(end-of-record)을 나타낸다. 반환 데이터가 레코드를 끝마친다. (일반적으로 `SOCK_SEQPACKET` 타입 소켓에서 사용)

`MSG_TRUNC`
:   데이터그램이 제공 버퍼보다 커서 데이터그램 끝 부분을 버렸음을 나타낸다.

`MSG_CTRUNC`
:   보조 데이터 버퍼에 공간이 부족해서 일부 제어 데이터를 버렸음을 나타낸다.

`MSG_OOB`
:   긴급 내지 대역외 데이터를 수신했음을 나타낸다.

`MSG_ERRQUEUE`
:   데이터를 수신하지 않고 소켓 오류 큐에서 확장 오류를 받아 왔음을 나타낸다.

## RETURN VALUE

이 호출들은 받은 바이트 수를 반환한다. 또는 오류 발생 시 -1을 반환한다. 오류 시에 오류를 나타내도록 `errno`를 설정한다.

스트림 소켓 상대가 질서 있는 닫기를 수행했을 때에는 반환 값이 0이 된다. (전통적인 "파일 끝" 반환 값이다.)

여러 (가령 유닉스 및 인터넷) 도메인의 데이터그램 소켓에서는 길이 0인 데이터그램을 허용한다. 그런 데이터그램을 받았을 때 반환 값은 0이다.

스트림 소켓에서 수신 요청한 바이트 수가 0인 경우에도 0 값을 반환할 수 있다.

## ERRORS

소켓 계층에서 생성하는 몇 가지 표준 오류가 있다. 추가로 하위 프로토콜 모듈에서 다른 오류를 생성하여 반환할 수도 있다. 프로토콜별 매뉴얼 페이지를 참고하라.

`EAGAIN` 또는 `EWOULDBLOCK`
:   소켓이 논블로킹으로 표시돼 있는데 수신 동작이 블록 되려 했거나, 수신 타임아웃이 설정돼 있는데 데이터 수신 전에 타임아웃이 만료됐다. POSIX.1에서는 이 경우에 어느 쪽 오류도 반환할 수 있다고 허용한다. 따라서 이식성이 있어야 하는 응용에서는 두 가능성을 모두 확인해야 한다.

`EBADF`
:   인자 `sockfd`가 유효하지 않은 파일 디스크립터이다.

`ECONNREFUSED`
:   원격 호스트가 네트워크 연결 허용을 거부했다. (보통은 요청한 서비스가 동작 중이지 않기 때문이다.)

`EFAULT`
:   수신 버퍼 포인터(들)이 프로세스의 주소 공간 밖을 가리키고 있다.

`EINTR`
:   사용 가능한 데이터가 있기 전에 시그널 전달에 의해 수신이 중단되었다. <tt>[[signal(7)]]</tt> 참고.

`EINVAL`
:   유효하지 않은 인자를 전달했다.

`ENOMEM`
:   `recvmsg()`를 위한 메모리를 할당할 수 없다.

`ENOTCONN`
:   소켓이 연결 지향 프로토콜에 연계돼 있는데 연결이 되지 않았다. (`connect(2)` 및 `accept(2)` 참고.)

`ENOTSOCK`
:   파일 디스크립터 `sockfd`가 소켓을 가리키고 있지 않다.

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, 4.4BSD (4.2BSD에서 이 인터페이스들이 처음 등장).

POSIX.1에서는 `MSG_OOB`, `MSG_PEEK`, `MSG_WAITALL` 플래그만 기술한다.

## NOTES

길이가 0인 데이터그램이 대기 중인 경우 `read(2)`와 `flags` 인자 0인 `recv()`의 동작이 다르다. 이 경우에 `read(2)`는 효력이 없지만 (데이터그램이 대기 중으로 남는다) `recv()`는 대기 중인 데이터그램을 소모한다.

`socklen_t` 타입은 POSIX에서 고안한 것이다. `accept(2)`도 참고.

POSIX.1에 따르면 `msghdr` 구조체의 `msg_controllen` 필드가 `socklen_t` 타입이어야 하고 `msg_iovlen` 필드가 `int` 타입이어야 하지만 glibc에서는 현재 둘 모두 `size_t` 타입으로 하고 있다.

호출 한 번에 여러 데이터그램을 수신할 수 있는 리눅스 전용 시스템 호출에 대한 내용은 <tt>[[recvmmsg(2)]]</tt>를 보라.

## EXAMPLES

<tt>[[getaddrinfo(3)]]</tt>에서 `recvfrom()` 사용례를 볼 수 있다.

## SEE ALSO

<tt>[[fcntl(2)]]</tt>, `getsockopt(2)`, `read(2)`, <tt>[[recvmmsg(2)]]</tt>, <tt>[[select(2)]]</tt>, <tt>[[shutdown(2)]]</tt>, <tt>[[socket(2)]]</tt>, <tt>[[cmsg(3)]]</tt>, <tt>[[sockatmark(3)]]</tt>, <tt>[[ip(7)]]</tt>, <tt>[[ipv6(7)]]</tt>, <tt>[[socket(7)]]</tt>, <tt>[[tcp(7)]]</tt>, <tt>[[udp(7)]]</tt>, <tt>[[unix(7)]]</tt>

----

2021-03-22
