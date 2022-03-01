## NAME

strspn, strcspn - 선두 부분열 길이 얻기

## SYNOPSIS

```c
#include <string.h>

size_t strspn(const char *s, const char *accept);
size_t strcspn(const char *s, const char *reject);
```

## DESCRIPTION

`strspn()` 함수는 `accept`에 있는 바이트들로만 이뤄진 `s` 시작 부분의 (바이트 단위) 길이를 계산한다.

`strcspn()` 함수는 `reject`에 없는 바이트들로만 이뤄진 `s` 시작 부분의 길이를 계산한다.

## RETURN VALUE

`strspn()` 함수는 `accept`에 있는 바이트들로만 이뤄진 `s` 시작 부분의 바이트 수를 반환한다.

`strcspn()` 함수는 `reject`에 없는 바이트들로만 이뤄진 `s` 시작 부분의 바이트 수를 반환한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `strspn()`, `strcspn()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, C89, C99, SVr4, 4.3BSD.

## SEE ALSO

<tt>[[index(3)]]</tt>, <tt>[[memchr(3)]]</tt>, <tt>[[rindex(3)]]</tt>, <tt>[[strchr(3)]]</tt>, <tt>[[string(3)]]</tt>, `strpbrk(3)`, <tt>[[strsep(3)]]</tt>, <tt>[[strstr(3)]]</tt>, <tt>[[strtok(3)]]</tt>, `wcscspn(3)`, `wcspn(3)`

----

2021-03-22
