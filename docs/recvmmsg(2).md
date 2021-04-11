## NAME

recvmmsg - 소켓에서 여러 메시지 받기

## SYNOPSIS

```c
#define _GNU_SOURCE         /* feature_test_macros(7) 참고 */
#include <sys/socket.h>

int recvmmsg(int sockfd, struct mmsghdr *msgvec, unsigned int vlen,
             int flags, struct timespec *timeout);
```

## DESCRIPTION

`recvmmsg()` 시스템 호출은 <tt>[[recvmsg(2)]]</tt>를 확장한 것이다. 호출자가 시스템 호출 한 번으로 소켓에서 여러 메시지를 수신할 수 있게 해 준다. (일부 응용에서 성능상 이점이 있다.) 또 <tt>[[recvmsg(2)]]</tt>를 확장해서 수신 동작의 타임아웃을 지원한다.

`sockfd` 인자는 데이터를 수신할 소켓의 파일 디스크립터이다.

`msgvec` 인자는 `mmsghdr` 구조체 배열에 대한 포인터이다. 이 배열의 크기를 `vlen`에 지정한다.

`mmsghdr` 구조체는 `<sys/socket.h>`에 다음처럼 정의돼 있다.

```c
struct mmsghdr {
    struct msghdr msg_hdr;  /* 메시지 헤더 */
    unsigned int  msg_len;  /* 수신한 바이트 수 */
};
```

`msg_hdr` 필드는 <tt>[[recvmsg(2)]]</tt>에서 기술하는 `msghdr` 구조체이다. `msg_len` 필드는 그 항목의 메시지의 반환 바이트 수이다. 이 필드의 값은 그 헤더에 대해 따로 <tt>[[recvmsg(2)]]</tt>를 호출했을 때의 반환 값과 같다.

`flags` 인자는 OR 한 플래그들을 담는다. <tt>[[recvmsg(2)]]</tt>에 기록된 것들에 더해 다음 플래그가 있다.

`MSG_WAITFORONE` (리눅스 2.6.34부터)
:   첫 번째 메시지를 수신한 후에 `MSG_DONTWAIT`을 켠다.

`timeout` 인자는 `struct timespec`(<tt>[[clock_gettime(2)]]</tt> 참고)을 가리키며 수신 동작의 타임아웃(초 더하기 나노초)을 지정한다. (하지만 BUGS 참고.) (이 시간을 시스템 클럭 해상도에 따라 올림 하게 되며 커널 스케줄링 지연도 있기 때문에 그 블록 시간을 약간 넘길 수도 있다.) `timeout`이 NULL이면 동작이 영원히 블록 한다.

블로킹 `recvmmsg()` 호출은 `vlen` 개 메시지를 수신하거나 타임아웃이 만료될 때까지 블록 한다. 논블로킹 호출은 가능한 많은 메시지를 (`vlen`으로 지정한 한계까지) 읽어 들이고서 즉시 반환한다.

`recvmmsg()`에서 반환 시 `msgvec`의 연속된 항목들이 각 수신 메시지에 대한 정보를 담도록 갱신되어 있다. `msg_len`은 수신 메시지의 크기를 담는다. 그리고 `msg_hdr`의 하위 필드들이 <tt>[[recvmsg(2)]]</tt>에서 기술하는 대로 갱신되어 있다. 호출의 반환 값은 갱신된 `msgvec` 항목 개수를 나타낸다.

## RETURN VALUE

성공 시 `recvmmsg()`는 `msgvec`에 받은 메시지 개수를 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

오류들이 <tt>[[recvmsg(2)]]</tt>에서와 같다. 추가로 다음 오류가 발생할 수 있다.

`EINVAL`
:   `timeout`이 유효하지 않다.

BUGS도 참고.

## VERSIONS

리눅스 2.6.33에서 `recvmmsg()` 시스템 호출이 추가되었다. glibc 버전 2.12에서 지원이 추가되었다.

## CONFORMING TO

`recvmmsg()`는 리눅스 전용이다.

## BUGS

`timeout` 인자가 의도 대로 동작하지 않는다. 각 데이터그램 수신 후에만 타임아웃을 확인하며, 따라서 타임아웃 만료 전에 `vlen-1` 개까지 데이터그램을 수신하고서 이후 더는 데이터그램을 수신하지 않으면 호출이 영원히 블록 하게 된다.

최소 한 개 메시지를 받은 후에 오류가 발생하면 호출이 성공하고 받은 메시지 수를 반환한다. 오류 코드는 이어지는 `recvmmsg()`에서 반환되도록 돼 있다. 하지만 현재 구현에서는 소켓에서의 관련 없는 네트워크 이벤트(가령 ICMP 패킷 입력) 때문에 그 사이에 오류 코드가 덮어 써질 수 있다.

## EXAMPLE

다음 프로그램에서는 `recvmmsg()`를 사용해 소켓에서 여러 메시지를 받아서 여러 버퍼에 저장한다. 모든 버퍼가 차거나 지정한 타임아웃이 만료된 경우에 호출이 반환한다.

다음 명령은 난수를 담은 UDP 데이터그램을 주기적으로 생성한다.

```text
$ while true; do echo $RANDOM > /dev/udp/127.0.0.1/1234;
      sleep 0.25; done
```

그 데이터그램들을 예시 응용이 읽어 들여서 다음과 같이 출력한다.

```text
$ ./a.out
5 messages received
1 11782
2 11345
3 304
4 13514
5 28421
```

### 프로그램 소스

```c
#define _GNU_SOURCE
#include <netinet/ip.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>

int
main(void)
{
#define VLEN 10
#define BUFSIZE 200
#define TIMEOUT 1
    int sockfd, retval, i;
    struct sockaddr_in addr;
    struct mmsghdr msgs[VLEN];
    struct iovec iovecs[VLEN];
    char bufs[VLEN][BUFSIZE+1];
    struct timespec timeout;

    sockfd = socket(AF_INET, SOCK_DGRAM, 0);
    if (sockfd == -1) {
        perror("socket()");
        exit(EXIT_FAILURE);
    }

    addr.sin_family = AF_INET;
    addr.sin_addr.s_addr = htonl(INADDR_LOOPBACK);
    addr.sin_port = htons(1234);
    if (bind(sockfd, (struct sockaddr *) &addr, sizeof(addr)) == -1) {
        perror("bind()");
        exit(EXIT_FAILURE);
    }

    memset(msgs, 0, sizeof(msgs));
    for (i = 0; i < VLEN; i++) {
        iovecs[i].iov_base         = bufs[i];
        iovecs[i].iov_len          = BUFSIZE;
        msgs[i].msg_hdr.msg_iov    = &iovecs[i];
        msgs[i].msg_hdr.msg_iovlen = 1;
    }

    timeout.tv_sec = TIMEOUT;
    timeout.tv_nsec = 0;

    retval = recvmmsg(sockfd, msgs, VLEN, 0, &timeout);
    if (retval == -1) {
        perror("recvmmsg()");
        exit(EXIT_FAILURE);
    }

    printf("%d messages received\n", retval);
    for (i = 0; i < retval; i++) {
        bufs[i][msgs[i].msg_len] = 0;
        printf("%d %s", i+1, bufs[i]);
    }
    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[clock_gettime(2)]]</tt>, <tt>[[recvmsg(2)]]</tt>, <tt>[[sendmmsg(2)]]</tt>, <tt>[[sendmsg(2)]]</tt>, <tt>[[socket(2)]]</tt>, <tt>[[socket(7)]]</tt>

----

2019-03-06
