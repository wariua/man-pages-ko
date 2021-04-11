## NAME

sigwaitinfo, sigtimedwait, rt_sigtimedwait - 동기적으로 대기 시그널 기다리기

## SYNOPSIS

```c
#include <signal.h>

int sigwaitinfo(const sigset_t *restrict set,
                siginfo_t *restrict info);
int sigtimedwait(const sigset_t *restrict set,
                siginfo_t *restrict info,
                const struct timespec *restrict timeout);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`sigwaitinfo()`, `sigtimedwait()`:
:   `_POSIX_C_SOURCE >= 199309L`

## DESCRIPTION

`sigwaitinfo()`는 `set`의 시그널들 중 하나가 미처리 상태일 때까지 호출 스레드의 실행을 중지한다. (`set`의 시그널들 중 하나가 이미 호출 스레드에게 미처리 상태이면 `sigwaitinfo()`가 즉시 반환하게 된다.)

`sigwaitinfo()`는 미처리 시그널 집합에서 그 시그널을 제거하고서 시그널 번호를 함수 결과로 반환한다. `info` 인자가 NULL이 아니라면 그 버퍼를 이용해 시그널에 대한 정보를 담은 `siginfo_t` 타입 구조체(<tt>[[sigaction(2)]]</tt> 참고)를 반환한다.

`set`에 있는 시그널 여러 개가 호출자에게 미처리 상태인 경우 `sigwaitinfo()`가 가져오는 시그널은 일반적인 순서 규칙에 따라 정해진다. 더 자세한 내용은 <tt>[[signal(7)]]</tt>을 보라.

`sigtimedwait()`은 `sigwaitinfo()`와 정확히 같은 식으로 동작하되 추가로 `timeout` 인자가 있어서 시그널을 기다리며 스레드를 중지해 둘 시간을 지정한다. (이 시간을 시스템 클럭 해상도에 따라 올림 하게 되며 커널 스케줄링 지연도 있기 때문에 그 시간을 약간 넘길 수도 있다.) 이 인자는 다음 타입이다.

```c
struct timespec {
    time_t   tv_sec;        /* 초 */
    long     tv_nsec;       /* 나노초 */
};
```

이 구조체의 두 필드 모두를 0으로 지정하면 검사를 수행하는 것이다. 즉, 호출자에게 미처리였던 시그널에 대한 정보를 가지고, 또는 `set`의 어느 시그널도 미처리가 아니었으면 오류와 함께 `sigtimedwait()`이 즉시 반환한다.

## RETURN VALUE

성공 시 `sigwaitinfo()`와 `sigtimedwait()` 모두 시그널 번호를 (즉 0보다 큰 값을) 반환한다. 실패 시 두 호출 모두 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EAGAIN`
:   `sigtimedwait()`에 지정한 `timeout` 기간 내에 `set`의 어떤 시그널도 미처리 상태가 되지 않았다.

`EINTR`
:   시그널 핸들러에 의해 기다리기가 중단되었다. <tt>[[signal(7)]]</tt> 참고. (이 핸들러는 `set`에 있는 것 외의 시그널에 대한 것이다.)

`EINVAL`
:   `timeout`이 유효하지 않다.

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## NOTES

일반적 사용 방식에서는 호출 프로그램이 <tt>[[sigprocmask(2)]]</tt>를 미리 호출해서 `set` 안의 시그널들을 막으며 (그래서 이어지는 `sigwaitinfo()` 내지 `sigtimedwait()` 호출과의 사이에서 시그널이 미처리 상태가 되는 경우 기본 처리가 일어나지 않도록 하며), 그 시그널들에 대한 핸들러를 설정하지 않는다. 다중 스레드 프로그램에서는 `sigwaitinfo()` 내지 `sigtimedwait()`을 호출한 스레드 외의 스레드에서 기본 처리 방식에 따라 시그널이 처리되는 것을 막기 위해 모든 스레드에서 시그널을 막아야 한다.

어떤 스레드에 미처리 상태인 시그널들의 집합은 특별히 그 스레드에게 미처리인 시그널들과 프로세스 전체에게 미처리인 시그널들의 합집합이다 (<tt>[[signal(7)]]</tt> 참고).

`SIGKILL` 및 `SIGSTOP`을 기다리려는 시도는 조용히 무시된다.

한 프로세스의 여러 스레드가 `sigwaitinfo()` 내지 `sigtimedwait()`에서 같은 시그널(들)을 기다리며 블록 되어 있는 경우에 프로세스 전체에 대한 시그널이 미처리가 되면 그 스레드들 중 정확히 한 개가 실제로 시그널을 받게 된다. 어느 스레드가 시그널을 받는지는 정해져 있지 않다.

`sigwaitinfo()` 내지 `sigtimedwait()`은 비유효 메모리 주소 접근으로 인한 `SIGSEGV`나 산술 오류로 인한 `SIGFPE`처럼 동기적으로 생성된 시그널을 받는 데는 쓸 수 없다. 그런 시그널들은 시그널 핸들러를 통해서만 잡을 수 있다.

POSIX에서는 `sigtimedwait()`의 `timeout` 인자에서 NULL 값의 의미를 명세 안 된 것으로 남겨두어 이를 `sigwaitinfo()` 호출과 같은 의미로 하는 가능성을 허용하며, 실제로 리눅스에서 그렇게 한다.

### C 라이브러리/커널 차이

리눅스에서 `sigwaitinfo()`는 `sigtimedwait()` 위에 구현된 라이브러리 함수이다.

glibc의 `sigwaitinfo()` 및 `sigtimedwait()` 래퍼 함수에서는 NPTL 스레딩 구현 내부에서 쓰는 두 가지 실시간 시그널을 기다리려는 시도를 조용히 무시한다.

원래 리눅스 시스템 호출의 이름은 `sigtimedwait()`이었다. 하지만 리눅스 2.2에 실시간 시그널이 추가되면서 그 시스템 호출이 지원하던 고정 크기 32비트 `sigset_t` 타입이 더는 용도에 맞지 않게 되었다. 그에 따라 확장된 `sigset_t` 타입을 지원하기 위해 새로운 시스템 호출 `rt_sigtimedwait()`이 추가되었다. 새 시스템 호출에서 네 번째 인자로 `size_t sigsetsize`를 받는데, 이는 `set`의 시그널 집합의 바이트 단위 크기를 나타낸다. 현재는 이 인자가 `sizeof(sigset_t)` 값을 가져야 한다. (안 그러면 `EINVAL` 오류가 난다.) glibc의 `sigtimedwait()` 래퍼 함수에서 이런 세부 사항을 감추고 커널이 제공할 때 투명하게 `rt_sigtimedwait()`을 호출한다.

## SEE ALSO

<tt>[[kill(2)]]</tt>, <tt>[[sigaction(2)]]</tt>, <tt>[[signal(2)]]</tt>, <tt>[[signalfd(2)]]</tt>, <tt>[[sigpending(2)]]</tt>, <tt>[[sigprocmask(2)]]</tt>, <tt>[[sigqueue(3)]]</tt>, <tt>[[sigsetops(3)]]</tt>, <tt>[[sigwait(3)]]</tt>, <tt>[[signal(7)]]</tt>, <tt>[[time(7)]]</tt>

----

2021-03-22
