## NAME

strsignal, sigdescr_np, sigabbrev_np, sys_siglist - 시그널을 설명하는 문자열 반환

## SYNOPSIS

```c
#include <string.h>

char *strsignal(int sig);
char *sigdescr_np(int sig);
char *sigabbrev_np(int sig);

extern const char * const sys_siglist[];
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`sigabbrev_np()`, `sigdescr_np()`:
:   `_GNU_SOURCE`

`strsignal()`:
:   glibc 2.10부터 2.31까지:
    :   `_POSIX_C_SOURCE >= 200809L`

    glibc 2.10 전:
    :   `_GNU_SOURCE`

`sys_siglist`:
:   glibc 2.19부터:
    :   `_DEFAULT_SOURCE`

    glibc 2.19 및 이전:
    :   `_BSD_SOURCE`

## DESCRIPTION

`strsignal()` 함수는 `sig` 인자로 준 시그널 번호를 설명하는 문자열을 반환한다. 그 문자열은 다음 `strsignal()` 호출 전까지만 사용할 수 있다. `strsignal()`이 반환하는 문자열은 현재 로캘의 `LC_MESSAGES` 카테고리에 따라 지역화되어 있다.

`sigdescr_np()` 함수는 `sig` 인자로 준 시그널 번호를 설명하는 문자열을 반환한다. `strsignal()`과 달리 이 문자열은 현재 로캘의 영향을 받지 않는다.

`sigabbrev_np()` 함수는 시그널 `sig`의 축약 이름을 반환한다. 예를 들어 `SIGINT` 값을 받아서 문자열 "INT"를 반환한다.

(제거 예정인) 배열 `sys_siglist`는 시그널 설명 문자열들을 담고 있으며 시그널 번호가 인덱스다. 이 배열 대신 `strsignal()`이나 `sigdescr_np()` 함수를 쓰는 게 좋다. VERSIONS도 확인해 보라.

## RETURN VALUE

`strsignal()` 함수는 적절한 설명 문자열을 반환한다. 시그널 번호가 유효하지 않으면 알 수 없는 시그널이라는 메시지를 반환한다. 일부 시스템(리눅스는 아님)에서는 유효하지 않은 시그널 번호에 대해 NULL을 반환할 수도 있다.

`sigdescr_np()` 및 `sigabbrev_np()` 함수는 적절한 설명 문자열을 반환한다. 반환된 문자열은 정적으로 할당되어 있어서 프로그램 수명 내내 유효하다. 유효하지 않은 시그널 번호에 대해 NULL을 반환한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `strsignal()` | 스레드 안전성 | MT-Unsafe race:strsignal locale |
| `sigdescr_np()`, `sigabbrev_np()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2008. 솔라리스 및 BSD 계열에 있음.

`sigdescr_np()`와 `sigabbrev_np()`는 GNU 확장이다.

`sys_siglist`는 비표준이지만 다른 여러 시스템에 존재한다.

## NOTES

`sigdescr_np()`와 `sigabbrev_np()`는 스레드 안전이고 비동기 시그널 안전이다.

## SEE ALSO

<tt>[[psignal(3)]]</tt>, <tt>[[strerror(3)]]</tt>

----

2021-03-22
