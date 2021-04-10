## NAME

nanosleep - 고해상도 sleep

## SYNOPSIS

```c
#include <time.h>

int nanosleep(const struct timespec *req, struct timespec *rem);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`nanosleep()`
:   `_POSIX_C_SOURCE >= 199309L`

## DESCRIPTION

`nanosleep()`은 적어도 `*req`에 지정한 시간이 지날 때까지, 또는 호출 스레드에서 핸들러 호출을 유발하거나 프로세스를 종료시키는 시그널이 전달될 때까지 호출 스레드의 실행을 멈춘다.

호출이 시그널 핸들러에 의해 중단되는 경우에는 `nanosleep()`이 -1을 반환하고 `errno`를 `EINTR`로 설정하며 `rem`이 NULL이 아니면 남은 시간을 `rem`이 가리키는 구조체에 써 넣는다. 그러면 `*rem`의 값으로 다시 `nanosleep()`을 호출해서 지정했던 중지 시간을 채울 수 있다. (하지만 NOTES 참고.)

`timespec` 구조체를 사용해 나노초 정밀도로 시간을 지정한다. 구조체가 다음과 같이 정의돼 있다.

```c
struct timespec {
    time_t tv_sec;        /* 초 */
    long   tv_nsec;       /* 나노초 */
};
```

나노초 필드의 값은 0에서 999999999 사이 범위여야 한다.

<tt>[[sleep(3)]]</tt> 및 <tt>[[usleep(3)]]</tt>과 비교할 때 `nanosleep()`에는 몇 가지 강점이 있다. 잠드는 시간을 높은 해상도로 지정할 수 있고, POSIX.1에서 이 함수가 시그널과 상호작용하지 않는다고 명확히 명세하고 있으며, 시그널 핸들러로 중단된 잠들기를 재개하는 일이 쉬워진다.

## RETURN VALUE

요청 시간 동안 성공적으로 잠든 경우 `nanosleep()`은 0을 반환한다. 호출이 시그널 핸들러에 의해 중단되거나 오류를 만난 경우에는 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EFAULT`
:   사용자 공간으로부터 정보를 복사하면서 문제 발생.

`EINTR`
:   스레드로 전달된 시그널에 의해 휴지가 중단되었다. (<tt>[[signal(7)]]</tt> 참고.) 남는 시간이 `*rem`에 기록되었으므로 스레드에서 바로 `nanosleep()`을 다시 호출해서 휴지를 이어갈 수 있다.

`EINVAL`
:   `tv_nsec` 필드의 값이 0에서 999999999까지 범위 안이 아니거나 `tv_sec`이 음수이다.

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## NOTES

`req`에 지정한 시간이 기반 클럭 정밀도(<tt>[[time(7)]]</tt> 참고)의 정수배가 아니면 다음 배수로 시간을 올림 한다. 또한 잠들기가 끝난 후에도 CPU에서 호출 스레드를 다시 실행할 수 있게 될 때까지 지연이 있을 수도 있다.

`nanosleep()`이 상대적 시간 동안 잠든다는 점이 시그널 때문에 반복해서 호출이 재시작되는 경우 문제가 될 수 있다. 호출 중단과 재시작 사이의 시간 때문에 잠들기가 최종적으로 끝나는 시점이 늦춰지게 되기 때문이다. <tt>[[clock_nanosleep(2)]]</tt>을 절대 시간 값으로 사용하면 이 문제를 피할 수 있다.

POSIX.1에서는 `nanosleep()`이 `CLOCK_REALTIME` 클럭에 따라 시간을 재야 한다고 명세한다. 하지만 리눅스에서는 `CLOCK_MONOTONIC` 클럭으로 시간을 잰다. 이게 중요치 않을 수도 있는 것이, POSIX.1의 <tt>[[clock_settime(2)]]</tt> 명세에서는 `CLOCK_REALTIME`의 불연속적 변경이 `nanosleep()`에 영향을 끼치지 말아야 한다고 말한다.

> <tt>[[clock_settime(2)]]</tt>을 통해 `CLOCK_REALTIME` 클럭의 값을 설정하는 것이 이 클럭을 기반으로 한 `nanosleep()` 함수를 포함한 상대 시간 서비스를 기다리며 블록 돼 있는 스레드들에 어떤 영향도 주지 않아야 한다. ... 따라서 그 시간 서비스들은 클럭의 새 값이나 이전 값과는 상관없이 요청받은 상대적 시간이 경과했을 때 만료해야 한다.

### 구식 동작 방식

(가령 어떤 시간 제약적 하드웨어를 제어하기 위해) 훨씬 더 정밀한 휴지가 필요한 응용을 지원하기 위해서 `SCHED_FIFO`나 `SCHED_RR` 같은 실시간 정책으로 스케줄링 된 스레드에서 `nanosleep()`을 호출한 경우 2밀리초까지의 휴지는 바쁜 대기를 통해 마이크로초 정밀도로 처리하였다. 이 특수한 확장 동작은 커널 2.5.39에서 제거되었으므로 리눅스 2.6.0 및 이후 커널에서는 이용할 수 없다.

## BUGS

시그널을 잡으며 `nanosleep()`을 이용하는 프로그램에서 아주 높은 빈도로 시그널을 받는 경우에는 스케줄링 지연과 커널의 수면 시간 및 반환할 `remain` 값 계산 오차 때문에 `nanosleep()` 호출 재시작이 이어지면서 `remain` 값이 천천히 *증가할* 수도 있다. 그런 문제를 피하려면 <tt>[[clock_nanosleep(2)]]</tt>을 `TIMER_ABSTIME` 플래그와 함께 사용해서 절대적 시점까지 잠들면 된다.

리눅스 2.4에서는 `nanosleep()`이 시그널(가령 `SIGTSTP`)에 의해 정지되면 `SIGCONT` 시그널로 스레드가 재개된 후에 호출이 `EINTR` 오류로 실패한다. 이후 시스템 호출을 재시작하는 경우에는 스레드가 정지된 상태에서 보낸 시간이 잠든 시간에 계산되지 *않는다*. 리눅스 2.6.0 및 이후 커널에는 이 문제가 고쳐져 있다.

## SEE ALSO

<tt>[[clock_nanosleep(2)]]</tt>, <tt>[[restart_syscall(2)]]</tt>, <tt>[[sched_setscheduler(2)]]</tt>, <tt>[[timer_create(2)]]</tt>, <tt>[[sleep(3)]]</tt>, <tt>[[usleep(3)]]</tt>, <tt>[[time(7)]]</tt>

----

2017-09-15
