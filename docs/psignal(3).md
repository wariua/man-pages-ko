## NAME

psignal, psiginfo - 시그널 메시지 찍기

## SYNOPSIS

```c
#include <signal.h>

void psignal(int sig, const char *s);
void psiginfo(const siginfo_t *pinfo, const char *s);

extern const char *const sys_siglist[];
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`psignal()`:
:   glibc 2.19부터:
    :   `_DEFAULT_SOURCE`

    glibc 2.19 및 이전:
    :   `_BSD_SOURCE || _SVID_SOURCE`

`psiginfo()`:
:   `_POSIX_C_SOURCE >= 200809L`

`sys_siglist`:
:   glibc 2.19부터:
    :   `_DEFAULT_SOURCE`

    glibc 2.19 및 이전:
    :   `_BSD_SOURCE`

## DESCRIPTION

`psignal()` 함수는 문자열 `s`, 콜론, 공백, 시그널 번호 `sig`를 설명하는 문자열, 마지막 개행으로 이뤄진 메시지를 `stderr`에 표시한다. 문자열 `s`가 NULL이거나 비어 있으면 콜론과 공백을 생략한다. `sig`가 유효하지 않으면 모르는 시그널이라는 메시지가 표시된다.

`psiginfo()` 함수는 `psignal()`과 비슷하되 `pinfo`가 기술하는 시그널에 대한 정보를 표시한다. `pinfo`는 유효한 `siginfo_t` 구조체를 가리켜야 한다. 시그널 설명에 더해서 `psiginfo()`는 시그널이 어디서 온 것인지에 대한 정보와 기타 시그널 관련 정보(가령 하드웨어 생성 시그널에서 관련 메모리 주소, `SIGCHLD`에서 자식 프로세스 ID, <tt>[[kill(2)]]</tt>이나 <tt>[[sigqueue(3)]]</tt>로 보낸 시그널에서 송신자의 사용자 ID와 프로세스 ID)를 표시한다.

배열 `sys_siglist`는 시그널 설명 문자열들을 담고 있으며 시그널 번호가 인덱스다.

## RETURN VALUE

`psignal()` 및 `psiginfo()` 함수는 아무 값도 반환하지 않는다.

## VERSIONS

glibc 버전 2.10에서 `psiginfo()` 함수가 추가되었다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `psignal()`, `psiginfo()` | 스레드 안전성 | MT-Safe locale |

## CONFORMING TO

POSIX.1-2008, 4.3BSD.

## BUGS

glibc 버전 2.12까지에서 `psiginfo()`에 다음 버그가 있었다.

* 어떤 경우에 마지막 개행을 찍지 않는다.

* 실시간 시그널에 대한 추가 세부 정보를 표시하지 않는다.

## SEE ALSO

<tt>[[sigaction(2)]]</tt>, <tt>[[perror(3)]]</tt>, <tt>[[strsignal(3)]]</tt>, <tt>[[signal(7)]]</tt>

----

2017-09-15
