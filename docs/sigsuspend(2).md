## NAME

sigsuspend, rt_sigsuspend - 시그널 기다리기

## SYNOPSIS

```c
#include <signal.h>

int sigsuspend(const sigset_t *mask);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`sigsuspend()`:
:   `_POSIX_C_SOURCE`

## DESCRIPTION

`sigsuspend()`는 일시적으로 호출 스레드의 시그널 마스크를 `mask`로 준 마스크로 교체하고서 처리 동작이 시그널 핸들러 호출이나 프로세스 종료인 시그널이 전달될 때까지 스레드 실행을 중지한다.

시그널이 프로세스를 종료시키는 경우에는 `sigsuspend()`가 반환하지 않는다. 시그널을 잡는 경우에는 시그널 핸들러 반환 후에 `sigsuspend()`가 반환하며, 시그널 마스크가 `sigsuspend()` 호출 전 상태로 복원된다.

`SIGKILL`이나 `SIGSTOP`을 막는 것은 불가능하다. `mask`에 이 시그널들을 지정해도 스레드의 시그널 마스크에 아무 영향을 주지 않는다.

## RETURN VALUE

`sigsuspend()`는 항상 -1을 반환하며 오류(보통은 `EINTR`)를 나타내도록 `errno`를 설정한다.

## ERRORS

`EFAULT`
:   `mask`가 프로세스 주소 공간의 유효한 일부가 아닌 메모리를 가리키고 있다.

`EINTR`
:   시그널에 의해 호출이 중단되었다. <tt>[[signal(7)]]</tt> 참고.

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## NOTES

보통은 중요한 코드 구간 실행 동안의 시그널 전달을 막기 위해 <tt>[[sigprocmask(2)]]</tt>와 결합해서 `sigsuspend()`를 사용한다. 먼저 호출자가 <tt>[[sigprocmask(2)]]</tt>로 시그널을 막는다. 중요한 코드가 끝났을 때 <tt>[[sigprocmask(2)]]</tt>가 (`oldset` 인자로) 반환했던 시그널 마스크로 `sigsuspend()`를 호출해서 시그널을 기다린다.

시그널 집합 조작에 대한 자세한 내용은 <tt>[[sigsetops(3)]]</tt>를 보라.

### C 라이브러리/커널 차이

원래 리눅스 시스템 호출의 이름은 `sigsuspend()`였다. 하지만 리눅스 2.2에 실시간 시그널이 추가되면서 그 시스템 호출이 지원하던 고정 크기 32비트 `sigset_t` 타입이 더는 용도에 맞지 않게 되었다. 그에 따라 확장된 `sigset_t` 타입을 지원하기 위해 새로운 시스템 호출 `rt_sigsuspend()`가 추가되었다. 새 시스템 호출에서 두 번째 인자로 `size_t sigsetsize`를 받는데, 이는 `mask`의 시그널 집합의 바이트 단위 크기를 나타낸다. 현재는 이 인자가 `sizeof(sigset_t)` 값을 가져야 한다. (안 그러면 `EINVAL` 오류가 난다.) glibc의 `sigsuspend()` 래퍼 함수에서 이런 세부 사항을 감추고 커널이 제공할 때 투명하게 `rt_sigsuspend()`를 호출한다.

## SEE ALSO

<tt>[[kill(2)]]</tt>, <tt>[[pause(2)]]</tt>, <tt>[[sigaction(2)]]</tt>, <tt>[[signal(2)]]</tt>, <tt>[[sigprocmask(2)]]</tt>, <tt>[[sigwaitinfo(2)]]</tt>, <tt>[[sigsetops(3)]]</tt>, <tt>[[sigwait(3)]]</tt>, <tt>[[signal(7)]]</tt>

----

2021-03-22
