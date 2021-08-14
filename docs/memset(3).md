## NAME

memset - 똑같은 바이트로 메모리 채우기

## SYNOPSIS

```c
#include <string.h>

void *memset(void *s, int c, size_t n);
```

## DESCRIPTION

`memset()` 함수는 `s`가 가리키는 메모리 구역의 처음 `n` 바이트를 바이트 `c`로 똑같이 채운다.

## RETURN VALUE

`memset()` 함수는 메모리 구역 `s`의 포인터를 반환한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `memset()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, C89, C99, SVr4, 4.3BSD.

## SEE ALSO

<tt>[[bstring(3)]]</tt>, <tt>[[bzero(3)]]</tt>, <tt>[[swab(3)]]</tt>, `wmemset(3)`

----

2021-03-22
