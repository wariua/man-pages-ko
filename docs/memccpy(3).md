## NAME

memccpy - 메모리 영역 복사하기

## SYNOPSIS

```c
#include <string.h>

void *memccpy(void *restrict dest, const void *restrict src,
              int c, size_t n);
```

## DESCRIPTION

`memccpy()` 함수는 메모리 영역 `src`에서 메모리 영역 `dest`로 최대 `n` 개 바이트를 복사하되, 문자 `c`를 발견하면 중단한다.

두 메모리 영역이 겹치는 경우 결과는 규정되어 있지 않다.

## RETURN VALUE

`memccpy()` 함수는 `dest`에서 `c` 다음의 문자에 대한 포인터를 반환한다. `src`의 처음 `n` 개 바이트에서 `c`를 찾지 못한 경우에는 NULL을 반환한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `memccpy()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, SVr4, 4.3BSD.

## SEE ALSO

<tt>[[bcopy(3)]]</tt>, <tt>[[bstring(3)]]</tt>, <tt>[[memcpy(3)]]</tt>, <tt>[[memmove(3)]]</tt>, <tt>[[strcpy(3)]]</tt>, <tt>[[strncpy(3)]]</tt>

----

2021-03-22
