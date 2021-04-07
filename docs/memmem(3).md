## NAME

memmem - 부분열 위치 찾기

## SYNOPSIS

```c
#define _GNU_SOURCE         /* feature_test_macros(7) 참고 */
#include <string.h>

void *memmem(const void *haystack, size_t haystacklen,
             const void *needle, size_t needlelen);
```

## DESCRIPTION

`memmem()` 함수는 길이 `haystacklen`인 메모리 영역 `haystack`에서 처음 등장하는 길이 `needlelen`인 부분열 `needle`의 시작점을 찾는다.

## RETURN VALUE

`memmem()` 함수는 그 부분열 시작점에 대한 포인터를 반환한다. 부분열을 찾지 못하면 NULL을 반환한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값
| --- | --- | --- |
| `memmem()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

이 함수는 POSIX.1에 명세되어 있지 않지만 다른 여러 시스템들에 존재한다.

## BUGS

glibc 2.0에서는 `needle`이 비어 있으면 `memmem()`이 `haystack`의 마지막 바이트에 대한 포인터를 반환한다. glibc 2.1에서 고쳐졌다.

## SEE ALSO

<tt>[[bstring(3)]]</tt>, `strstr(3)`

----

2017-03-13
