## NAME

htonl, htons, ntohl, ntohs - 호스트 바이트 순서와 네트워크 바이트 순서 간에 값 변환하기

## SYNOPSIS

```c
#include <arpa/inet.h>

uint32_t htonl(uint32_t hostlong);
uint16_t htons(uint16_t hostshort);

uint32_t ntohl(uint32_t netlong);
uint16_t ntohs(uint16_t netshort);
```

## DESCRIPTION

`htonl()` 함수는 부호 없는 정수 `hostlong`을 호스트 바이트 순서에서 네트워크 바이트 순서로 변환한다.

`htons()` 함수는 부호 없는 단정수 `hostshort`을 호스트 바이트 순서에서 네트워크 바이트 순서로 변환한다.

`ntohl()` 함수는 부호 없는 정수 `netlong`을 네트워크 바이트 순서에서 호스트 바이트 순서로 변환한다.

`ntohs()` 함수는 부호 없는 단정수 `netshort`을 네트워크 바이트 순서에서 호스트 바이트 순서로 변환한다.

i386의 호스트 바이트 순서는 하위 바이트 우선이다. 반면 인터넷에서 쓰는 네트워크 바이트 순서는 상위 바이트 우선이다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `htonl()`, `htons()`, `ntohl()`, `ntohs()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

일부 시스템에서는 `<arpa/inet.h>` 대신 `<netinet/in.h>`을 포함시켜야 한다.

## SEE ALSO

<tt>[[bswap(3)]]</tt>, <tt>[[endian(3)]]</tt>, <tt>[[gethostbyname(3)]]</tt>, <tt>[[getservent(3)]]</tt>

----

2021-03-22
