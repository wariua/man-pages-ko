## NAME

rtime - 원격 머신에게서 시간 얻기

## SYNOPSIS

```c
#include <rpc/auth_des.h>

int rtime(struct sockaddr_in *addrp, struct rpc_timeval *timep,
          struct rpc_timeval *timeout);
```

## DESCRIPTION

이 함수는 RFC 868에서 설명하는 시간 서버 프로토콜을 이용해 원격 머신에게서 시간을 얻는다.

시간 서버 프로토콜에서는 UTC 1900년 1월 1일 00:00:00 기준 초 수로 시간을 준다. 이 함수에서 적당한 상수를 빼서 에포크, 즉 1970-01-01 00:00:00 +0000 (UTC) 기준의 초 수로 바꾼다.

`timeout`이 NULL이 아니면 udp/time 소켓(37번 포트)을 쓴다. 아니면 tcp/time 소켓(37번 포트)을 쓴다.

## RETURN VALUE

성공 시 0을 반환하며 얻은 32비트 시간 값을 `timep->tv_sec`에 저장한다. 오류인 경우 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

기반 함수들(<tt>[[sendto(2)]]</tt>, <tt>[[poll(2)]]</tt>, <tt>[[recvfrom(2)]]</tt>, `connect(2)`, `read(2)`)의 모든 오류들이 발생할 수 있다. 추가로,

`EIO`
:   반환된 바이트 수가 4가 아니다.

`ETIMEDOUT`
:   `timeout`에 지정된 대기 시간이 만료됐다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `rtime()` | 스레드 안전성 | MT-Safe |

## NOTES

IPv4만 지원한다.

일부 `in.timed` 버전들에선 TCP만 지원한다. 예시 프로그램에서 `use_tcp`를 1로 설정해 보라.

## BUGS

glibc 2.2.5 및 이전에서 `rtime()`이 64비트 머신에서 올바로 동작하지 않는다.

## EXAMPLES

이 예시를 위해선 37번 포트가 열려 있어야 한다. `/etc/inetd.conf`에서 time 항목이 주석 처리돼 있지 않은지 확인해 볼 수 있다.

프로그램이 "linux"라는 컴퓨터로 연결한다. "localhost"를 쓰는 방법은 동작하지 않는다. 결과는 컴퓨터 "linux"의 지역 시간이다.

```c
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>
#include <string.h>
#include <time.h>
#include <rpc/auth_des.h>
#include <netdb.h>

static int use_tcp = 0;
static char *servername = "linux";

int
main(void)
{
    struct sockaddr_in name;
    struct rpc_timeval time1 = {0,0};
    struct rpc_timeval timeout = {1,0};
    struct hostent *hent;
    int ret;

    memset(&name, 0, sizeof(name));
    sethostent(1);
    hent = gethostbyname(servername);
    memcpy(&name.sin_addr, hent->h_addr, hent->h_length);

    ret = rtime(&name, &time1, use_tcp ? NULL : &timeout);
    if (ret < 0)
        perror("rtime error");
    else {
        time_t t = time1.tv_sec;
        printf("%s\n", ctime(&t));
    }

    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

`ntpdate(1)`, `inetd(8)`

----

2021-03-22
