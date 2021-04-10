## NAME

htobe16, htole16, be16toh, le16toh, htobe32, htole32, be32toh, le32toh, htobe64, htole64, be64toh, le64toh - 호스트 바이트 순서와 빅/리틀 엔디언 바이트 순서 간에 값 변환하기

## SYNOPSIS

```c
#include <endian.h>

uint16_t htobe16(uint16_t host_16bits);
uint16_t htole16(uint16_t host_16bits);
uint16_t be16toh(uint16_t big_endian_16bits);
uint16_t le16toh(uint16_t little_endian_16bits);

uint32_t htobe32(uint32_t host_32bits);
uint32_t htole32(uint32_t host_32bits);
uint32_t be32toh(uint32_t big_endian_32bits);
uint32_t le32toh(uint32_t little_endian_32bits);

uint64_t htobe64(uint64_t host_64bits);
uint64_t htole64(uint64_t host_64bits);
uint64_t be64toh(uint64_t big_endian_64bits);
uint64_t le64toh(uint64_t little_endian_64bits);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`htobe16()`, `htole16()`, `be16toh()`, `le16toh()`, `htobe32()`, `htole32()`, `be32toh()`, `le32toh()`, `htobe64()`, `htole64()`, `be64toh()`, `le64toh()`:
:   glibc 2.19부터:
    :   `_DEFAULT_SOURCE`

    glibc 2.19까지:
    :   `_BSD_SOURCE`

## DESCRIPTION

이 함수들은 현재 CPU("호스트")에서 쓰는 바이트 순서와 리틀 엔디언 및 빅 엔디언 바이트 순서 사이에서 정수 값의 바이트 인코딩을 변환한다.

함수 이름의 숫자 *nn*은 함수에서 다루는 정수의 크기를 나타내며, 16, 32, 64비트 중 하나이다.

이름이 "htobe*nn*" 형태인 함수는 호스트 바이트 순서에서 빅 엔디언 순서로 변환한다.

이름이 "htole*nn*" 형태인 함수는 호스트 바이트 순서에서 리틀 엔디언 순서로 변환한다.

이름이 "be*nn*toh" 형태인 함수는 빅 엔디언 순서에서 호스트 바이트 순서로 변환한다.

이름이 "le*nn*toh" 형태인 함수는 리틀 엔디언 순서에서 호스트 바이트 순서로 변환한다.

## VERSIONS

glibc 버전 2.9에서 이 함수들이 추가되었다.

## CONFORMING TO

이 함수들은 비표준이다. 비슷한 함수들이 BSD들에 존재하는데 거기서 필요한 헤더 파일은 `<endian.h>`가 아니라 `<sys/endian.h>`이다. 안타깝게도 NetBSD, FreeBSD, glibc에서는 *nn* 부분이 항상 함수 이름 끝에 오는 OpenBSD의 원래 명명 관행을 따르지 않는다. (그래서 예를 들어 OpenBSD "betoh32"의 등가물이 NetBSD, FreeBSD, glibc에서는 "be32toh"이다.)

## NOTES

이 함수들은 더 오래된 <tt>[[byteorder(3)]]</tt> 계열 함수들과 비슷하다. 예를 들어 `be32toh()`는 `ntohl()`과 같다.

<tt>[[byteorder(3)]]</tt> 함수들의 장점은 표준 함수라서 모든 유닉스 시스템에서 사용 가능하다는 점이다. 하지만 TCP/IP 맥락에서 쓰려고 설계한 것이기 때문에 이 페이지에 기술된 64비트 형태와 리틀 엔디언 형태가 빠져 있다.

## EXAMPLE

아래 프로그램에서는 정수를 호스트 바이트 순서에서 리틀 엔디언 및 빅 엔디언 바이트 순서로 변환한 결과를 보여 준다. 호스트 바이트 순서가 리틀 엔디언 아니면 빅 엔디언이므로 변환 중 한쪽만 효과가 있게 된다. 이 프로그램을 x86-32 같은 리틀 엔디언 시스템에서 돌리면 다음을 보게 된다.

```
$ ./a.out
x.u32 = 0x44332211
htole32(x.u32) = 0x44332211
htobe32(x.u32) = 0x11223344
```

### 프로그램 소스

```c
#include <endian.h>
#include <stdint.h>
#include <stdio.h>
#include <stdlib.h>

int
main(int argc, char *argv[])
{
    union {
        uint32_t u32;
        uint8_t arr[4];
    } x;

    x.arr[0] = 0x11;    /* 최하위 주소 바이트 */
    x.arr[1] = 0x22;
    x.arr[2] = 0x33;
    x.arr[3] = 0x44;    /* 최상위 주소 바이트 */

    printf("x.u32 = 0x%x\n", x.u32);
    printf("htole32(x.u32) = 0x%x\n", htole32(x.u32));
    printf("htobe32(x.u32) = 0x%x\n", htobe32(x.u32));

    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[bswap(3)]]</tt>, <tt>[[byteorder(3)]]</tt>

----

2019-03-06
