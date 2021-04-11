## NAME

sigpending, rt_sigpending - 미처리 시그널 조사하기

## SYNOPSIS

```c
#include <signal.h>

int sigpending(sigset_t *set);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`sigpending()`:
:   `_POSIX_C_SOURCE`

## DESCRIPTION

`sigpending()`은 호출 스레드로 전달되기를 기다리고 있는 시그널들(즉 차단 상태에서 발생한 시그널들)의 집합을 반환한다. 그 미처리 시그널들의 마스크를 `set`으로 반환한다.

## RETURN VALUE

`sigpending()`은 성공 시 0을 반환한다. 실패 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## NOTES

시그널 집합 조작에 대한 자세한 내용은 <tt>[[sigsetops(3)]]</tt>를 보라.

시그널이 블록 되어 있으면서 처리 방식이 "무시"이면 생성 시 미처리 시그널 마스크에 추가되지 *않는다*.

어떤 스레드에 미처리 상태인 시그널들의 집합은 특별히 그 스레드에게 미처리인 시그널들과 프로세스 전체에게 미처리인 시그널들의 합집합이다. <tt>[[signal(7)]]</tt> 참고.

<tt>[[fork(2)]]</tt>를 통해 생성된 자식은 처음에 빈 미처리 시그널 집합을 가지고 있다. <tt>[[execve(2)]]</tt>를 거치면서 미처리 시그널 집합이 보존된다.

### C 라이브러리/커널 차이

원래 리눅스 시스템 호출의 이름은 `sigpending()`이었다. 하지만 리눅스 2.2에 실시간 시그널이 추가되면서 그 시스템 호출이 지원하던 고정 크기 32비트 `sigset_t` 타입이 더는 용도에 맞지 않게 되었다. 그에 따라 확장된 `sigset_t` 타입을 지원하기 위해 새로운 시스템 호출 `rt_sigpending()`이 추가되었다. 새 시스템 호출에서 두 번째 인자로 `size_t sigsetsize`를 받는데, 이는 `set`의 시그널 집합의 바이트 단위 크기를 나타낸다. glibc의 `sigpending()` 래퍼 함수에서 이런 세부 사항을 감추고 커널이 제공할 때 투명하게 `rt_sigpending()`을 호출한다.

## BUGS

glibc 2.2.1까지 버전에서는 `sigpending()` 래퍼 함수에 버그가 있는데, 미처리 실시간 시그널에 대한 정보가 올바로 반환되지 않는다.

## SEE ALSO

<tt>[[kill(2)]]</tt>, <tt>[[sigaction(2)]]</tt>, <tt>[[signal(2)]]</tt>, <tt>[[sigprocmask(2)]]</tt>, <tt>[[sigsuspend(2)]]</tt>, <tt>[[sigsetops(3)]]</tt>, <tt>[[signal(7)]]</tt>

----

2021-03-22
