## NAME

pthread_kill - 스레드에게 시그널 보내기

## SYNOPSIS

```c
#include <signal.h>

int pthread_kill(pthread_t thread, int sig);
```

`-pthread`로 컴파일 및 링크.

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`pthread_kill()`:
:   `_POSIX_C_SOURCE >= 199506L || _XOPEN_SOURCE >= 500`

## DESCRIPTION

`pthread_kill()` 함수는 호출자와 같은 프로세스 안에 있는 스레드인 `thread`에게 시그널 `sig`를 보낸다. `thread`에게 비동기적으로 시그널이 간다.

`sig`가 0이면 어떤 시그널도 보내지 않되, 오류 검사는 마찬가지로 수행한다.

## RETURN VALUE

성공 시 `pthread_kill()`은 0을 반환한다. 오류 시 오류 번호를 반환하며 어떤 시그널도 전송되지 않는다.

## ERRORS

`EINVAL`
:   유효하지 않은 시그널을 지정하였다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `pthread_kill()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## NOTES

시그널 처리 방식은 프로세스 범위이다. 즉, 시그널 핸들러가 설치되어 있으면 스레드 `thread`에서 핸들러가 호출되지만 시그널 처리 방식이 "정지"나 "재개", "종료"이면 이 동작이 프로세스 전체에 영향을 끼치게 된다.

glibc의 `pthread_kill()` 구현은 NPTL 스레딩 구현 내부에서 쓰는 실시간 시그널들 중 하나를 보내려고 시도하면 오류(`EINVAL`)를 내놓는다. 자세한 내용은 <tt>[[nptl(7)]]</tt>을 보라.

POSIX.1-2008에서는 스레드 ID가 수명이 다한 후에 쓰이는 것을 구현체에서 탐지한 경우 `pthread_kill()`이 오류 `ESRCH`를 반환하기를 권장한다. glibc 구현에서는 유효하지 않은 스레드 ID를 탐지할 수 있는 경우에 이 오류를 반환한다. 하지만 POSIX에서는 또한 수명이 다한 스레드 ID를 사용하려는 시도가 규정되지 않은 동작을 만들어 내며 `pthread_kill()` 호출에 유효하지 않은 ID를 사용하려는 시도가 가령 세그멘테이션 오류를 일으킬 수 있다고 한다.

## SEE ALSO

<tt>[[kill(2)]]</tt>, <tt>[[sigaction(2)]]</tt>, <tt>[[sigpending(2)]]</tt>, <tt>[[pthread_self(3)]]</tt>, <tt>[[pthread_sigmask(3)]]</tt>, <tt>[[raise(3)]]</tt>, <tt>[[pthreads(7)]]</tt>, <tt>[[signal(7)]]</tt>

----

2017-09-15
