## NAME

bcopy - 바이트 열 복사하기

## SYNOPSIS

```c
#include <strings.h>

void bcopy(const void *src, void *dest, size_t n);
```

## DESCRIPTION

`bcopy()` 함수는 `src`에서 `dest`로 `n`개 바이트를 복사한다. 두 영역이 겹치더라도 결과가 올바르다.

## RETURN VALUE

없음.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `bcopy()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

4.3BSD. 이 함수는 제거 예정이다. (POSIX.1-2001에서 LEGACY로 표시됨.) 새 프로그램에서는 <tt>[[memcpy(3)]]</tt>나 <tt>[[memmove(3)]]</tt>를 사용하라. <tt>[[memcpy(3)]]</tt> 및 <tt>[[memmove(3)]]</tt>의 처음 두 인자 순서가 반대라는 점에 유의하라. POSIX.1-2008에서 `bcopy()` 명세를 제거하였다.

## SEE ALSO

<tt>[[bstring(3)]]</tt>, <tt>[[memccpy(3)]]</tt>, <tt>[[memcpy(3)]]</tt>, <tt>[[memmove(3)]]</tt>, <tt>[[strcpy(3)]]</tt>, <tt>[[strncpy(3)]]</tt>

----

2021-03-22
