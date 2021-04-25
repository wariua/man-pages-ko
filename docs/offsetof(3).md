## NAME

offsetof - 구조체 멤버의 내부 위치

## SYNOPSIS

```c
#include <stddef.h>

size_t offsetof(type, member);
```

## DESCRIPTION

매크로 `offsetof()`는 구조체 `type`의 시작점 기준 `member` 필드의 위치를 반환한다.

이 매크로가 유용한 이유는 구조체를 이루는 필드들의 크기가 구현에 따라 다를 수 있으며 컴파일러가 필드들 사이에 패딩 바이트들을 다르게 삽입할 수도 있기 때문이다. 그렇기 때문에 한 요소의 오프셋이 반드시 앞쪽 요소들의 크기의 합으로 정해지지 않는다.

`member`가 바이트 경계에 정렬되어 있지 않으면 (즉 비트 필드이면) 컴파일러 오류가 발생하게 된다.

## RETURN VALUE

`offsetof()`는 주어진 `type` 내에서 주어진 `member`의 바이트 단위 오프셋을 반환한다.

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, C89, C99.

## EXAMPLES

리눅스/i386 시스템에서 `gcc(1)` 기본 옵션으로 컴파일 할 때 아래 프로그램이 다음 출력을 내놓는다.

```text
$ ./a.out
offsets: i=0; c=4; d=8 a=16
sizeof(struct s)=16
```

### 프로그램 소스

```c
#include <stddef.h>
#include <stdio.h>
#include <stdlib.h>

int
main(void)
{
    struct s {
        int i;
        char c;
        double d;
        char a[];
    };

    /* 컴파일러에 따라 출력이 다름 */

    printf("offsets: i=%zu; c=%zu; d=%zu a=%zu\n",
            offsetof(struct s, i), offsetof(struct s, c),
            offsetof(struct s, d), offsetof(struct s, a));
    printf("sizeof(struct s)=%zu\n", sizeof(struct s));

    exit(EXIT_SUCCESS);
}
```

----

2020-11-01
