## NAME

sendmmsg - 소켓으로 여러 메시지 보내기

## SYNOPSIS

```c
#define _GNU_SOURCE         /* feature_test_macros(7) 참고 */
#include <sys/socket.h>

int sendmmsg(int sockfd, struct mmsghdr *msgvec, unsigned int vlen,
             int flags);
```

## DESCRIPTION

`sendmmsg()` 시스템 호출은 <tt>[[sendmsg(2)]]</tt>를 확장한 것이다. 호출자가 시스템 호출 한 번으로 소켓에서 여러 메시지를 전송할 수 있게 해 준다. (일부 응용에서 성능상 이점이 있다.)

`sockfd` 인자는 데이터를 송신할 소켓의 파일 디스크립터이다.

`msgvec` 인자는 `mmsghdr` 구조체 배열에 대한 포인터이다. 이 배열의 크기를 `vlen`에 지정한다.

`mmsghdr` 구조체는 `<sys/socket.h>`에 다음처럼 정의돼 있다.

```c
struct mmsghdr {
    struct msghdr msg_hdr;  /* 메시지 헤더 */
    unsigned int  msg_len;  /* 전송한 바이트 수 */
};
```

`msg_hdr` 필드는 <tt>[[sendmsg(2)]]</tt>에서 기술하는 `msghdr` 구조체이다. `msg_len` 필드를 이용해 `msg_hdr`의 메시지에서 보낸 바이트 수를 반환한다. (즉 따로 <tt>[[sendmsg(2)]]</tt>를 호출했을 때의 반환 값과 같다.)

`flags` 인자는 OR 한 플래그들을 담는다. 플래그들은 <tt>[[sendmsg(2)]]</tt>에서와 같다.

블로킹 `sendmmsg()` 호출은 `vlen`개 메시지를 송신할 때까지 블록 한다. 논블로킹 호출은 가능한 많은 메시지를 (`vlen`으로 지정한 한계까지) 보내고서 즉시 반환한다.

`sendmmsg()`에서 반환 시 `msgvec`의 연속된 항목들의 `msg_len` 필드가 대응하는 `msg_hdr`에서의 전송 바이트 수를 담도록 갱신되어 있다. 호출의 반환 값은 갱신된 `msgvec` 항목 개수를 나타낸다.

## RETURN VALUE

성공 시 `sendmmsg()`는 `msgvec`에서 보낸 메시지 개수를 반환한다. 그 값이 `vlen`보다 작은 경우 호출자가 `sendmmsg()`를 추가로 호출해서 나머지 메시지들을 보내려 할 수 있다.

오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

오류들이 <tt>[[sendmsg(2)]]</tt>에서와 같다. 어떤 데이터그램도 보낼 수 없었던 경우에만 오류를 반환한다. BUGS도 참고.

## VERSIONS

리눅스 3.0에서 `sendmmsg()` 시스템 호출이 추가되었다. glibc 버전 2.14에서 지원이 추가되었다.

## CONFORMING TO

`sendmmsg()`는 리눅스 전용이다.

## NOTES

`vlen`에 지정할 수 있는 값의 한도가 `UIO_MAXIOV`(1024)이다.

## BUGS

최소 한 개 메시지를 보낸 후에 오류가 발생하면 호출이 성공하고 보낸 메시지 수를 반환한다. 오류 코드는 유실된다. 호출자가 실패한 첫 번째 메시지부터 전송을 재시도할 수 있지만 그때 설령 오류가 발생한다고 해도 그게 이전 호출에서 유실된 것과 같은 오류라는 보장이 없다.

## EXAMPLES

아래 예에서는 `sendmmsg()`를 사용해 시스템 호출 한 번으로 UDP 데이터그램 두 개에 `onetwo`와 `three`를 보낸다. 첫 번째 데이터그램의 내용은 두 버퍼에서 온 것이다.

```c
#define _GNU_SOURCE
#include <netinet/ip.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/types.h>
#include <sys/socket.h>

int
main(void)
{
    int sockfd;
    struct sockaddr_in addr;
    struct mmsghdr msg[2];
    struct iovec msg1[2], msg2;
    int retval;

    sockfd = socket(AF_INET, SOCK_DGRAM, 0);
    if (sockfd == -1) {
        perror("socket()");
        exit(EXIT_FAILURE);
    }

    addr.sin_family = AF_INET;
    addr.sin_addr.s_addr = htonl(INADDR_LOOPBACK);
    addr.sin_port = htons(1234);
    if (connect(sockfd, (struct sockaddr *) &addr, sizeof(addr)) == -1) {
        perror("connect()");
        exit(EXIT_FAILURE);
    }

    memset(msg1, 0, sizeof(msg1));
    msg1[0].iov_base = "one";
    msg1[0].iov_len = 3;
    msg1[1].iov_base = "two";
    msg1[1].iov_len = 3;

    memset(&msg2, 0, sizeof(msg2));
    msg2.iov_base = "three";
    msg2.iov_len = 5;

    memset(msg, 0, sizeof(msg));
    msg[0].msg_hdr.msg_iov = msg1;
    msg[0].msg_hdr.msg_iovlen = 2;

    msg[1].msg_hdr.msg_iov = &msg2;
    msg[1].msg_hdr.msg_iovlen = 1;

    retval = sendmmsg(sockfd, msg, 2, 0);
    if (retval == -1)
        perror("sendmmsg()");
    else
        printf("%d messages sent\n", retval);

    exit(0);
}
```

## SEE ALSO

<tt>[[recvmmsg(2)]]</tt>, <tt>[[sendmsg(2)]]</tt>, <tt>[[socket(2)]]</tt>, <tt>[[socket(7)]]</tt>

----

2020-06-09
