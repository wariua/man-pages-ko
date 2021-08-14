## NAME

strchr, strrchr, strchrnul - 문자열에서 문자 찾기

## SYNOPSIS

```c
#include <string.h>

char *strchr(const char *s, int c);
char *strrchr(const char *s, int c);

#define _GNU_SOURCE         /* feature_test_macros(7) 참고 */
#include <string.h>

char *strchrnul(const char *s, int c);
```

## DESCRIPTION

`strchr()` 함수는 문자열 `s`에서 문자 `c`의 첫 번째 위치에 대한 포인터를 반환한다.

`strrchr()` 함수는 문자열 `s`에서 문자 `c`의 마지막 위치에 대한 포인터를 반환한다.

`strchrnul()` 함수는 `strchr()`과 같되, `s`에서 `c`를 찾을 수 없는 경우에 NULL이 아니라 `s` 끝의 널 바이트에 대한 포인터를 반환한다.

여기서 "문자"는 "바이트"를 뜻한다. 확장 문자나 다중 바이트 문자에는 이 함수들이 동작하지 않는다.

## RETURN VALUE

glibc 버전 2.1.1에서 `strchrnul()`이 처음 등장했다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `strchr()`, `strrchr()`, `strchrnul()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

`strchr()`, `strrchr()`: POSIX.1-2001, POSIX.1-2008, C89, C99, SVr4, 4.3BSD.

`strchrnul()`은 GNU 확장이다.

## SEE ALSO

<tt>[[index(3)]]</tt>, <tt>[[memchr(3)]]</tt>, <tt>[[rindex(3)]]</tt>, <tt>[[string(3)]]</tt>, <tt>[[strlen(3)]]</tt>, <tt>[[strpbrk(3)]]</tt>, <tt>[[strsep(3)]]</tt>, <tt>[[strspn(3)]]</tt>, <tt>[[strstr(3)]]</tt>, <tt>[[strtok(3)]]</tt>, `wcschr(3)`, `wcsrchr(3)`

----

2021-03-22
