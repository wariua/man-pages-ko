## NAME

clock_nanosleep - 클럭 지정이 가능한 고해상도 sleep

## SYNOPSIS

```c
#include <time.h>

int clock_nanosleep(clockid_t clockid, int flags,
                    const struct timespec *request,
                    struct timespec *remain);
```

`-lrt`로 링크 (glibc 버전 2.17 전에서만).

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`clock_nanosleep()`:
:   `_POSIX_C_SOURCE >= 200112L`

## DESCRIPTION

<tt>[[nanosleep(2)]]</tt>과 마찬가지로 `clock_nanosleep()`을 통해 호출 스레드가 나노초 정밀도로 지정한 시간 동안 잠들 수 있다. 차이점은 수면 시간을 측정하는 클럭을 호출자가 선택할 수 있다는 점과 수면 시간을 절댓값이나 상댓값으로 지정할 수 있다는 점이다.

이 호출로 주거나 돌려받는 시간 값들은 다음과 같이 정의된 `timespec` 구조체로 나타낸다.

```c
struct timespec {
    time_t   tv_sec;        /* 초 */
    long     tv_nsec;       /* 나노초 ([0 .. 999999999] */
};
```

`clockid` 인자는 잠드는 시간을 측정할 클럭을 나타낸다. 다음 값들 중 하나를 이 인자에 쓸 수 있다.

`CLOCK_REALTIME`
:   설정 가능한 시스템 전역 실제 시간 클럭.

`CLOCK_TAI` (리눅스 3.10부터)
:   벽시계 시간에서 파생되었고 윤초를 무시하는 시스템 전역 클럭.

`CLOCK_MONOTONIC`
:   시스템 시동 후 바뀌지 않는 과거 어떤 불특정 시점부터의 시간을 측정하는 설정 불가능한 단조 증가 클럭.

`CLOCK_BOOTTIME` (리눅스 2.6.39부터)
:   `CLOCK_MONOTONIC`과 동일하되 시스템이 절전 대기 상태인 시간도 포함한다.

`CLOCK_PROCESS_CPUTIME_ID` (리눅스 2.6.12부터.)
:   프로세스의 모든 스레드가 소모한 CPU 시간을 측정하는 설정 가능한 프로세스별 클럭.

이 클럭들에 대한 더 자세한 내용은 <tt>[[clock_getres(2)]]</tt>를 보라. 또한 <tt>[[clock_getcpuclockid(3)]]</tt> 및 <tt>[[pthread_getcpuclockid(3)]]</tt>가 반환한 CPU 클럭 ID도 `clockid`에 줄 수 있다.

`flags`가 0이면 `request`에 지정한 값을 `clockid`에 지정한 클럭의 현재 값에 대한 상대적 시간으로 해석한다.

`flags`가 `TIMER_ABSTIME`이면 `request`를 `clockid` 클럭으로 측정한 절대 시간으로 해석한다. `request`가 클럭의 현재 값보다 작거나 같으면 호출 스레드가 멈추지 않고 `clock_nanosleep()`이 즉시 반환한다.

`clock_nanosleep()`은 적어도 `request`에 지정한 시간이 지날 때까지, 또는 핸들러 호출을 유발하거나 프로세스를 종료시키는 시그널이 전달될 때까지 호출 스레드의 실행을 멈춘다.

호출이 시그널 핸들러에 의해 중단되는 경우에는 `clock_nanosleep()`이 `EINTR` 오류로 실패한다. 더불어 `remain`이 NULL이 아니고 `flags`가 `TIMER_ABSTIME`이 아니었으면 남은 수면 시간을 `remain`으로 반환한다. 그러면 이 값으로 다시 `clock_nanosleep()`을 호출해서 (상대적) 수면을 끝마칠 수 있다.

## RETURN VALUE

요청 시간 동안 성공적으로 잠든 경우 `clock_nanosleep()`은 0을 반환한다. 호출이 시그널 핸들러에 의해 중단되거나 오류를 만난 경우에는 ERRORS에 나열된 양수 오류 번호들 중 하나를 반환한다.

## ERRORS

`EFAULT`
:   지정한 `request`나 `remain`이 유효하지 않은 주소이다.

`EINTR`
:   시그널 핸들러에 의해 잠들기가 중단되었다. <tt>[[signal(7)]]</tt> 참고.

`EINVAL`
:   `tv_nsec` 필드의 값이 0에서 999999999까지 범위 안이 아니거나 `tv_sec`이 음수이다.

`EINVAL`
:   `clockid`가 유효하지 않다. (`CLOCK_THREAD_CPUTIME_ID`는 `clockid`에 가능한 값이 아니다.)

`ENOTSUP`
:   이 `clockid`로 잠드는 것을 커널에서 지원하지 않는다.

## VERSIONS

리눅스 2.6에서 `clock_nanosleep()` 시스템 호출이 처음 등장했다. glibc 버전 2.1부터 지원을 쓸 수 있다.

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## NOTES

`request`에 지정한 시간이 기반 클럭 정밀도(<tt>[[time(7)]]</tt> 참고)의 정수배가 아니면 다음 배수로 시간을 올림한다. 또한 잠들기가 끝난 후에도 CPU에서 호출 스레드를 다시 실행할 수 있게 될 때까지 지연이 있을 수도 있다.

절대 타이머를 쓰면 <tt>[[nanosleep(2)]]</tt>에서 설명하는 늦춰지는 문제를 막는 데 도움이 된다. (상대적 잠들기가 반복적으로 시그널에 의해 중단돼서 재시작하려 하는 프로그램에서 그 문제가 심해진다.) 상대적 잠들기를 하면서 이 문제를 피하려면 원하는 클럭으로 <tt>[[clock_gettime(2)]]</tt>을 호출하고서 `TIMER_ABSTIME` 플래그로 `clock_nanosleep()`을 호출하면 된다.

`clock_nanosleep()`은 <tt>[[sigaction(2)]]</tt> `SA_RESTART` 플래그를 쓰더라도 시그널 핸들러에 의해 중단된 후 절대 재시작되지 않는다.

`flags`가 `TIMER_ABSTIME`일 때는 `remain` 인자를 안 쓰며 필요도 없다. (절대 시간 잠들기는 같은 `request` 인자를 써서 재시작할 수 있다.)

POSIX.1에서는 `clock_nanosleep()`이 시그널 처리 방식이나 시그널 마스크에 어떤 영향도 끼치지 않는다고 명세한다.

POSIX.1에서는 <tt>[[clock_settime(2)]]</tt>을 통해 `CLOCK_REALTIME`의 값을 바꾼 후에는 절대적 `clock_nanosleep()`에 블록된 스레드가 깨어날 시점을 새 클럭 값으로 정해야 한다고 명세하고 있다. 새 클럭 값이 수면 시간 끝을 넘어간다면 `clock_nanosleep()` 호출이 즉시 반환된다.

POSIX.1에서는 <tt>[[clock_settime(2)]]</tt>을 통해 `CLOCK_REALTIME`의 값을 바꾸는 것이 상대적 `clock_nanosleep()`에 블록된 스레드에 어떤 영향도 끼치지 않아야 한다고 명세하고 있다.

## SEE ALSO

<tt>[[clock_getres(2)]]</tt>, <tt>[[nanosleep(2)]]</tt>, <tt>[[restart_syscall(2)]]</tt>, <tt>[[timer_create(2)]]</tt>, <tt>[[sleep(3)]]</tt>, <tt>[[usleep(3)]]</tt>, <tt>[[time(7)]]</tt>

----

2021-03-22
