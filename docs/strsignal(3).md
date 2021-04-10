## NAME

strsignal - 시그널을 설명하는 문자열 반환

## SYNOPSIS

```c
#include <string.h>

char *strsignal(int sig);

extern const char * const sys_siglist[];
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`strsignal()`:
:   glibc 2.10부터:
    :   `_POSIX_C_SOURCE >= 200809L`

    glibc 2.10 전:
    :   `_GNU_SOURCE`

## DESCRIPTION

`strsignal()` 함수는 `sig` 인자로 준 시그널 번호를 설명하는 문자열을 반환한다. 그 문자열은 다음 `strsignal()` 호출 전까지만 사용할 수 있다.

배열 `sys_siglist`는 시그널 설명 문자열들을 담고 있으며 시그널 번호가 인덱스다. 가능하면 이 배열 대신 `strsignal()` 함수를 쓰는 게 좋다.

## RETURN VALUE

`strsignal()` 함수는 적절한 설명 문자열을 반환한다. 시그널 번호가 유효하지 않으면 알 수 없는 시그널이라는 메시지를 반환한다. 일부 시스템(리눅스는 아님)에서는 유효하지 않은 시그널 번호에 대해 NULL을 반환할 수도 있다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `strsignal()` | 스레드 안전성 | MT-Unsafe race:strsignal locale |

## CONFORMING TO

POSIX.1-2008. 솔라리스 및 BSD 계열에 있음.

## SEE ALSO

<tt>[[psignal(3)]]</tt>, <tt>[[strerror(3)]]</tt>

----

2017-09-15
