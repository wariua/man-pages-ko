## NAME

send, sendto, sendmsg - 소켓으로 메시지 보내기

## SYNOPSIS

```c
#include <sys/types.h>
#include <sys/socket.h>

ssize_t send(int sockfd, const void *buf, size_t len, int flags);

ssize_t sendto(int sockfd, const void *buf, size_t len, int flags,
               const struct sockaddr *dest_addr, socklen_t addrlen);

ssize_t sendmsg(int sockfd, const struct msghdr *msg, int flags);
```

## DESCRIPTION

시스템 호출 `send()`, `sendto()`, `sendmsg()`를 이용해 다른 소켓으로 메시지를 전송한다.

`send()` 호출은 소켓이 *연결* 상태일 때만 (그래서 대상 수신자가 알려진 경우에만) 쓸 수 있다. `send()`와 `write(2)`의 유일한 차이는 `flags`의 존재 여부다. `flags` 인자가 0이면 `send()`는 `write(2)`와 동등하다. 또한 다음 호출은

```c
send(sockfd, buf, len, flags);
```

다음과 동등하다.

```c
sendto(sockfd, buf, len, flags, NULL, 0);
```

`sockfd` 인자는 송신 소켓의 파일 디스크립터이다.

`sendto()`를 연결 모드 (`SOCK_STREAM`, `SOCK_SEQPACKET`) 소켓에 쓰면 `dest_addr` 및 `addrlen` 인자를 무시하며 (두 인자가 NULL과 0이 아니면 `EISCONN`을 반환할 수도 있음), 소켓이 실제로 연결돼 있지 않은 경우 `ENOTCONN` 오류를 반환한다. 그 외 경우에는 `dest_addr`이 대상의 주소이고 `addrlen`이 그 크기를 나타낸다. `sendmsg()`에서는 `msg.msg_name`이 대상의 주소이고 `msg.msg_namelen`이 그 크기를 나타낸다.

`send()`와 `sendto()`에서는 `buf`에 메시지가 있으며 길이가 `len`이다. `sendmsg()`에서는 `msg.msg_iov` 배열의 항목들이 메시지를 가리킨다. `sendmsg()` 호출로는 (제어 정보라고도 하는) 보조 데이터를 보내는 것도 가능하다.

메시지가 너무 길어서 하위 프로토콜을 원자적으로 통과할 수 없으면 `EMSGSIZE`를 반환하며 메시지는 전송되지 않는다.

`send()`에서는 어떤 전달 실패 표시도 암묵적이지 않다. 로컬에서 오류를 탐지하면 반환 값 -1로 나타낸다.

메시지가 소켓의 송신 버퍼에 들어가지 않으면 보통은 `send()`가 블록 된다. 하지만 소켓을 논블로킹 I/O 모드로 만들어 뒀으면 `EAGAIN`이나 `EWOULDBLOCK` 오류로 실패하게 된다. <tt>[[select(2)]]</tt> 호출을 이용해 데이터를 더 보내는 게 가능한지 알아낼 수 있다.

### `flags` 인자

`flags` 인자는 다음 플래그들을 0개 이상 비트 OR 해서 구성한다.

<dl>
<dt><code>MSG_CONFIRM</code> (리눅스 2.3.15부터)</dt>
<dd>어떤 진척이 있었다고, 즉 상대에게서 성공적으로 응답을 받았다고 링크 계층에게 알린다. 알려 주지 않으면 링크 계층에서는 주기적으로 (가령 유니캐스트 ARP를 통해) 이웃을 재조사하게 된다. <code>SOCK_DGRAM</code> 및 <code>SOCK_RAW</code> 소켓에서만 유효하며 현재 IPv4 및 IPv6에만 구현돼 있다. 자세한 내용은 <tt>[[arp(7)]]</tt>를 보라.</dd>

<dt><code>MSG_DONTROUTE</code></dt>
<dd>게이트웨이를 이용해 패킷을 보내지 않고 직접 연결된 망의 호스트로만 보낸다. 일반적으로 진단 프로그램이나 라우팅 프로그램에서만 쓴다. 라우팅을 하는 프로토콜 족에만 규정돼 있다. 가령 패킷 소켓은 아니다.</dd>

<dt><code>MSG_DONTWAIT</code> (리눅스 2.2부터)</dt>
<dd>논블로킹 동작을 켠다. 동작이 블록 되려는 경우 <code>EAGAIN</code>이나 <code>EWOULDBLOCK</code>을 반환한다. (<tt>[[fcntl(2)]]</tt> <code>F_SETFL</code> 동작을 통해) <code>O_NONBLOCK</code> 플래그를 설정한 경우와 비슷한 동작 방식인데, <code>MSG_DONTWAIT</code>이 호출별 옵션인 반면 <code>O_NONBLOCK</code>은 열린 파일 기술 항목에 대한 설정이어서 (<tt>[[open(2)]]</tt> 참고) 호출 프로세스 내 모든 스레드뿐 아니라 같은 열린 파일 기술 항목을 가리키는 파일 디스크립터를 가진 다른 프로세스에도 영향을 끼치게 된다.</dd>

<dt><code>MSG_EOR</code> (리눅스 2.2부터)</dt>
<dd>(<code>SOCK_SEQPACKET</code> 타입 소켓처럼 레코드 개념을 지원하는 경우에) 레코드를 끝낸다.</dd>

<dt><code>MSG_MORE</code> (리눅스 2.4.4부터)</dt>
<dd>

호출자가 보내려는 데이터가 더 있다. TCP 소켓에서 이 플래그를 사용해서 <code>TCP_CORK</code> 소켓 옵션(<tt>[[tcp(7)]]</tt> 참고)과 같은 효과를 얻는데 이 플래그는 호출별로 설정할 수 있다는 점이 다르다.

리눅스 2.6부터는 UDP 소켓에서도 이 플래그를 지원한다. 이 플래그를 설정한 호출로 보낸 데이터 모두를 커널에서 데이터그램 한 개로 만들게 하며 이 플래그를 지정하지 않은 호출을 수행해야 그 데이터그램이 전송된다. (<tt>[[udp(7)]]</tt>에서 설명하는 <code>UDP_CORK</code> 소켓 옵션도 참고.)
</dd>

<dt><code>MSG_NOSIGNAL</code> (리눅스 2.2부터)</dt>
<dd>스트림 지향 소켓에서 상대가 연결을 닫은 경우에 <code>SIGPIPE</code> 시그널을 발생시키지 않는다. <code>EPIPE</code> 오류는 여전히 반환한다. <tt>[[sigaction(2)]]</tt>으로 <code>SIGPIPE</code>를 무시하는 것과 동작이 비슷한데, <code>MSG_NOSIGNAL</code>은 호출별 기능이고 <code>SIGPIPE</code> 무시하기는 프로세스 속성을 설정하여 프로세스 내 모든 스레드에 영향을 준다.</dd>

<dt><code>MSG_OOB</code></dt>
<dd>지원하는 (가령 <code>SOCK_STREAM</code> 타입) 소켓인 경우에 <em>대역외</em> 데이터를 보낸다. 하위 프로토콜에서도 <em>대역외</em> 데이터를 지원해야 한다.</dd>
</dl>

### `sendmsg()`

`sendmsg()`에서 사용하는 `msghdr` 구조체의 정의는 다음과 같다.

```c
struct msghdr {
    void         *msg_name;       /* 주소, 선택적 */
    socklen_t     msg_namelen;    /* 주소 크기 */
    struct iovec *msg_iov;        /* 스캐터/개더 배열 */
    size_t        msg_iovlen;     /* msg_iov의 항목 개수 */
    void         *msg_control;    /* 보조 데이터, 아래 참고 */
    size_t        msg_controllen; /* 보조 데이터 버퍼 길이 */
    int           msg_flags;      /* 플래그 (사용하지 않음) */
};
```

연결 안 된 소켓에서 `msg_name` 필드를 이용해 데이터그램의 대상 주소를 지정한다. 이 필드는 주소를 담은 버퍼를 가리키며 `msg_namelen` 필드는 그 주소의 크기로 설정해야 한다. 연결된 소켓에서는 이 필드들을 각각 NULL과 0으로 지정해야 한다.

`msg_iov` 및 `msg_iovlen` 필드는 <tt>[[writev(2)]]</tt>에서처럼 스캐터-개더 위치들을 지정한다.

`msg_control` 및 `msg_controllen` 멤버를 이용해 제어 정보를 보낼 수 있다. 커널에서 처리할 수 있는 소켓별 제어 버퍼 최대 길이를 `/proc/sys/net/core/optmem_max` 값이 제한한다. <tt>[[socket(7)]]</tt> 참고.

`msg_flags` 필드는 무시한다.

## RETURN VALUE

성공 시 이 호출들은 보낸 바이트 수를 반환한다. 오류 시 -1을 반환하며 `errno`를 적절히 설정한다.

## ERRORS

소켓 계층에서 생성하는 몇 가지 표준 오류가 있다. 추가로 하위 프로토콜 모듈에서 다른 오류를 생성하여 반환할 수도 있다. 프로토콜별 매뉴얼 페이지를 참고하라.

<dl>
<dt><code>EACCES</code></dt>
<dd>

(경로명으로 식별하는 유닉스 도메인 소켓에서) 대상 소켓 파일에서 쓰기 권한이 거부되었다. 또는 경로 선두부의 한 디렉터리에서 탐색 권한이 거부되었다. (<tt>[[path_resolution(7)]]</tt> 참고.)

(UDP 소켓에서) 유니캐스트 주소인 것처럼 네트워크/브로드캐스트 주소로 보내려는 시도를 했다.
</dd>
<dt><code>EAGAIN</code> 또는 <code>EWOULDBLOCK</code></dt>
<dd>소켓이 논블로킹으로 표시돼 있는데 요청 동작이 블록 되려 했다. POSIX.1-2001에서는 이 경우에 어느 쪽 오류도 반환할 수 있다고 허용하며 이 상수들이 같은 값을 가져야 한다고 요구하지 않는다. 따라서 이식성이 있어야 하는 응용에서는 두 가능성을 모두 확인해야 한다.</dd>
<dt><code>EAGAIN</code></dt>
<dd>(인터넷 도메인 데이터그램 소켓) <code>sockfd</code>가 가리키는 소켓이 미리 주소에 결속되지 않아서 임시 포트에 결속하려 할 때 임시 포트 범위 내의 모든 포트 번호가 현재 사용 중임을 알게 됐다. <tt>[[ip(7)]]</tt>의 <code>/proc/sys/net/ipv4/ip_local_port_range</code> 설명 참고.</dd>
<dt><code>EALREADY</code></dt>
<dd>다른 빠른 열기(Fast Open)가 진행 중이다.</dd>
<dt><code>EBADF</code></dt>
<dd><code>sockfd</code>가 유효한 열린 파일 디스크립터가 아니다.</dd>
<dt><code>ECONNRESET</code></dt>
<dd>상대가 연결을 재설정했다.</dd>
<dt><code>EDESTADDRREQ</code></dt>
<dd>소켓이 연결 모드가 아니며 상대 주소가 설정돼 있지 않다.</dd>
<dt><code>EFAULT</code></dt>
<dd>인자로 유효하지 않은 사용자 공간 주소를 지정했다.</dd>
<dt><code>EINTR</code></dt>
<dd>데이터를 전송하기 전에 시그널이 발생했다. <tt>[[signal(7)]]</tt> 참고.</dd>
<dt><code>EINVAL</code></dt>
<dd>유효하지 않은 인자를 전달했다.</dd>
<dt><code>EISCONN</code></dt>
<dd>연결 모드 소켓이 이미 연결되어 있는데 수신자를 지정했다. (지금은 이 오류를 반환할 수도 있고 수신자 지정을 무시할 수도 있다.)</dd>
<dt><code>EMSGSIZE</code></dt>
<dd>메시지를 원자적으로 보내야 하는 소켓 타입인데 보낼 메시지의 크기 때문에 불가능하다.</dd>
<dt><code>ENOBUFS</code></dt>
<dd>네트워크 인터페이스의 출력 큐가 가득 찼다. 보통은 인터페이스가 송신을 멈췄다는 표시이지만 순간적인 혼잡 때문에 발생할 수도 있다. (정상적으로 리눅스에서는 발생하지 않는다. 장치 큐가 넘칠 때 패킷을 그냥 조용히 버린다.)</dd>
<dt><code>ENOMEM</code></dt>
<dd>사용 가능 메모리 없음.</dd>
<dt><code>ENOTCONN</code></dt>
<dd>소켓이 연결돼 있지 않으며 대상을 주지 않았다.</dd>
<dt><code>ENOTSOCK</code></dt>
<dd>파일 디스크립터 <code>sockfd</code>가 소켓을 가리키고 있지 않다.</dd>
<dt><code>EOPNOTSUPP</code></dt>
<dd><code>flags</code> 인자의 어떤 비트가 소켓 타입에 맞지 않다.</dd>
<dt><code>EPIPE</code></dt>
<dd>연결 지향 소켓에서 로컬 종단이 닫혔다. 이 경우에 <code>MSG_NOSIGNAL</code>을 설정하지 않았으면 프로세스가 <code>SIGPIPE</code>도 받게 된다.</dd>
</dl>

## CONFORMING TO

4.4BSD, SVr4, POSIX.1-2001. 4.2BSD에서 이 인터페이스들이 처음 등장했다.

POSIX.1-2001에서는 `MSG_OOB` 및 `MSG_EOR` 플래그만 기술한다. POSIX.1-2008에서 `MSG_NOSIGNAL` 명세를 추가했다. `MSG_CONFIRM` 플래그는 리눅스 확장이다.

## NOTES

POSIX.1-2001에 따르면 `msghdr` 구조체의 `msg_controllen` 필드가 `socklen_t` 타입이어야 하지만 glibc에서는 현재 `size_t` 타입으로 하고 있다.

호출 한 번에 여러 데이터그램을 전송할 수 있는 리눅스 전용 시스템 호출에 대한 내용은 <tt>[[sendmmsg(2)]]</tt>를 보라.

## BUGS

리눅스에서 `ENOTCONN` 대신 `EPIPE`를 반환할 수도 있다.

## EXAMPLE

<tt>[[getaddrinfo(3)]]</tt>에서 `sendto()` 사용례를 볼 수 있다.

## SEE ALSO

<tt>[[fcntl(2)]]</tt>, `getsockopt(2)`, <tt>[[recv(2)]]</tt>, <tt>[[select(2)]]</tt>, <tt>[[sendfile(2)]]</tt>, <tt>[[sendmmsg(2)]]</tt>, <tt>[[shutdown(2)]]</tt>, <tt>[[socket(2)]]</tt>, `write(2)`, <tt>[[cmsg(3)]]</tt>, <tt>[[ip(7)]]</tt>, <tt>[[ipv6(7)]]</tt>, <tt>[[socket(7)]]</tt>, <tt>[[tcp(7)]]</tt>, <tt>[[udp(7)]]</tt>, <tt>[[unix(7)]]</tt>

----

2017-09-15
