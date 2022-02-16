## NAME

memmove - 메모리 영역 복사하기

## SYNOPSIS

```c
#include <string.h>

void *memmove(void *dest, const void *src, size_t n);
```

## DESCRIPTION

`memmove()` 함수는 메모리 영역 `src`에서 메모리 영역 `dest`로 `n`개 바이트를 복사한다. 메모리 영역이 겹칠 수 있다. 마치 `src`의 바이트들을 `src`나 `dest`와 겹치지 않는 임시 배열로 복사한 다음 그 임시 배열을 `dest`로 복사하는 것처럼 복사가 이뤄진다.

## RETURN VALUE

`memmove()` 함수는 `dest` 포인터를 반환한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `memmove()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, C89, C99, SVr4, 4.3BSD.

## SEE ALSO

<tt>[[bcopy(3)]]</tt>, <tt>[[bstring(3)]]</tt>, <tt>[[memccpy(3)]]</tt>, <tt>[[memcpy(3)]]</tt>, <tt>[[strcpy(3)]]</tt>, <tt>[[strncpy(3)]]</tt>, `wmemmove(3)`

----

2021-03-22
