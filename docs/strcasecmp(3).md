## NAME

strcasecmp, strncasecmp - 대소문자 무시하고 두 문자열 비교하기

## SYNOPSIS

```c
#include <strings.h>

int strcasecmp(const char *s1, const char *s2);
int strncasecmp(const char *s1, const char *s2, size_t n);
```

## DESCRIPTION

`strcasecmp()` 함수는 문자의 대소문자를 무시하면서 문자열 `s1`과 `s2`를 바이트 단위로 비교한다. `s1`이 `s2`보다 작거나, 일치하거나, 큰 경우에 0보다 작거나, 같거나, 큰 정수를 반환한다.

`strncasecmp()` 함수는 이와 비슷하되, `s1`과 `s2`의 `n` 바이트까지만 비교한다.

## RETURN VALUE

`strcasecmp()`와 `strncasecmp()` 함수는 `s1`이 `s2`보다 작거나, 일치하거나, 큰 경우에 0보다 작거나, 같거나, 큰 정수를 반환한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `strcasecmp()`, `strncasecmp()` | 스레드 안전성 | MT-Safe locale |

## CONFORMING TO

4.4BSD, POSIX.1-2001, POSIX.1-2008.

## NOTES

`strcasecmp()`와 `strncasecmp()` 함수는 4.4BSD에서 처음 등장했으며, `<string.h>`에 선언돼 있었다. 그래서 과거의 호환성 문제 때문에 `_DEFAULT_SOURCE` (또는 glibc 2.19까지에선 `_BSD_SOURCE`) 기능 확인 매크로가 정의돼 있을 때 glibc의 `<string.h>` 헤더 파일에도 이 함수들이 선언돼 있다.

POSIX.1-2008 표준에선 이 함수들에 대해 다음을 언급하고 있다.

> 사용하는 로캘의 `LC_CTYPE` 부문이 POSIX 로캘에서 온 것일 때 이 함수들은 문자열들을 소문자로 변환하고서 바이트 비교를 수행한 것처럼 동작한다. 그렇지 않을 때의 결과는 명세돼 있지 않다.

## SEE ALSO

<tt>[[bcmp(3)]]</tt>, <tt>[[memcmp(3)]]</tt>, <tt>[[strcmp(3)]]</tt>, <tt>[[strcoll(3)]]</tt>, <tt>[[string(3)]]</tt>, <tt>[[strncmp(3)]]</tt>, `wcscasecmp(3)`, `wcsncasecmp(3)`

----

2021-03-22
