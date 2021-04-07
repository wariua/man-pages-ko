## NAME

bswap_16, bswap_32, bswap_64 - 바이트 순서 뒤집기

## SYNOPSIS

```c
#include <byteswap.h>

bswap_16(x);
bswap_32(x);
bswap_64(x);
```

## DESCRIPTION

이 매크로들은 각각의 2바이트, 4바이트, 8바이트 인자의 바이트 순서를 뒤집은 값을 반환한다.

## RETURN VALUE

이 매크로들은 인자의 바이트들을 뒤집은 값을 반환한다.

## ERRORS

이 매크로들은 항상 성공한다.

## CONFORMING TO

이 매크로들은 GNU 확장이다.

## EXAMPLE

아래 프로그램에서는 명령행 인자로 받은 8바이트 정수의 바이트들을 뒤집는다. 다음 셸 세션이 프로그램 사용 방식을 보여 준다.

```
$ ./a.out 0x0123456789abcdef
0x123456789abcdef ==> 0xefcdab8967452301
```

### 프로그램 소스

```c
#include <stdio.h>
#include <stdint.h>
#include <stdlib.h>
#include <inttypes.h>
#include <byteswap.h>

int
main(int argc, char *argv[])
{
    uint64_t x;

    if (argc != 2) {
        fprintf(stderr, "Usage: %s <num>\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    x = strtoul(argv[1], NULL, 0);
    printf("0x%" PRIx64 " ==> 0x%" PRIx64 "\n", x, bswap_64(x));

    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[byteorder(3)]]</tt>, <tt>[[endian(3)]]</tt>

----

2019-03-06
