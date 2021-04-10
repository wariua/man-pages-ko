## NAME

sigwait - 시그널 기다리기

## SYNOPSIS

```c
#include <signal.h>

int sigwait(const sigset_t *set, int *sig);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`sigwait()`:
:   glibc 2.26부터:
    :   `_POSIX_C_SOURCE >= 199506L`

    glibc 2.25 및 이전:
    :   `_POSIX_C_SOURCE`

## DESCRIPTION

`sigwait()` 함수는 시그널 집합 `set`에 지정한 시그널들 중 하나가 미처리 상태가 될 때까지 호출 스레드의 실행을 중지한다. 함수가 그 시그널을 받아들이고 (미처리 시그널 목록에서 그 시그널을 제거하고) `sig`에 시그널 번호를 반환한다.

`sigwait()`의 동작은 다음을 제외하면 <tt>[[sigwaitinfo(2)]]</tt>와 같다.

* `sigwait()`은 시그널을 기술하는 `siginfo_t` 구조체가 아니라 시그널 번호만 반환한다.

* 두 함수의 반환 값이 다르다.

## RETURN VALUE

성공 시 `sigwait()`은 0을 반환한다. 오류 시 (ERRORS에 나열된) 양수 오류 번호를 반환한다.

## ERRORS

`EINVAL`
:   `set`이 유효하지 않은 시그널 번호를 담고 있다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값
| --- | --- | --- |
| `sigwait()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## NOTES

`sigwait()`은 <tt>[[sigtimedwait(2)]]</tt>을 이용해 구현되어 있다.

glibc의 `sigwait()` 구현에서는 NPTL 스레딩 구현 내부에서 쓰는 두 가지 실시간 시그널을 기다리려는 시도를 조용히 무시한다.

## EXAMPLE

<tt>[[pthread_sigmask(3)]]</tt> 참고.

## SEE ALSO

<tt>[[sigaction(2)]]</tt>, <tt>[[signalfd(2)]]</tt>, <tt>[[sigpending(2)]]</tt>, <tt>[[sigsuspend(2)]]</tt>, <tt>[[sigwaitinfo(2)]]</tt>, <tt>[[sigsetops(3)]]</tt>, <tt>[[signal(7)]]</tt>

----

2017-07-13
