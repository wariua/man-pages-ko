## NAME

timer_create - POSIX 프로세스별 타이머 만들기

## SYNOPSIS

```c
#include <signal.h>
#include <time.h>

int timer_create(clockid_t clockid, struct sigevent *restrict sevp,
                 timer_t *restrict timerid);
```

`-lrt`로 링크.

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`timer_create()`:
:   `_POSIX_C_SOURCE >= 199309L`

## DESCRIPTION

`timer_create()`는 새로운 프로세스별 간격 타이머를 만든다. 새 타이머의 ID가 `timerid`가 가리키는 버퍼로 반환된다. `timerid`는 널 아닌 포인터여야 한다. 이 ID는 그 타이머가 삭제될 때까지 프로세스 내에서 유일하다. 새로운 타이머는 처음에 해제 상태이다.

`clockid` 인자는 새 타이머가 시간 측정에 사용할 클럭을 나타낸다. 다음 값들 중 하나를 지정할 수 있다.

`CLOCK_REALTIME`
:   설정 가능하며 시스템 전역인 실제 시간 클럭.

`CLOCK_MONOTONIC`
:   설정 불가능하며 시스템 구동 후 바뀌지 않는 과거 불특정 시점으로부터의 시간을 측정하는 단조 증가 클럭.

`CLOCK_PROCESS_CPUTIME_ID` (리눅스 2.6.12부터)
:   호출 프로세스가 (그 안의 모든 스레드들이) 소모한 (사용자 및 시스템) CPU 시간을 측정하는 클럭.

`CLOCK_THREAD_CPUTIME_ID` (리눅스 2.6.12부터)
:   호출 스레드가 소모한 (사용자 및 시스템) CPU 시간을 측정하는 클럭.

`CLOCK_BOOTTIME` (리눅스 2.6.39부터)
:   `CLOCK_MONOTONIC`처럼 단조 증가하는 클럭이다. 하지만 `CLOCK_MONOTONIC` 클럭에서 시스템이 절전 대기 상태인 시간을 측정하지 않는 반면 `CLOCK_BOOTTIME` 클럭에서는 시스템이 절전 대기 상태인 시간을 포함한다. 절전 대기를 인식할 필요가 있는 응용들에 유용하다. 그런 응용들에 `CLOCK_REALTIME`은 적합하지 않은데, 그 클럭은 시스템 클럭의 불연속적 변화에 영향을 받기 때문이다.

`CLOCK_REALTIME_ALARM` (리눅스 3.0부터)
:   이 클럭은 `CLOCK_REALTIME`과 비슷하되 시스템이 절전 대기 상태이면 깨우게 된다. 이 클럭에 대해 타이머를 설정하기 위해선 호출자가 `CAP_WAKE_ALARM` 역능을 가지고 있어야 한다.

`CLOCK_BOOTTIME_ALARM` (리눅스 3.0부터)
:   이 클럭은 `CLOCK_BOOTTIME`과 비슷하되 시스템이 절전 대기 상태이면 깨우게 된다. 이 클럭에 대해 타이머를 설정하기 위해선 호출자가 `CAP_WAKE_ALARM` 역능을 가지고 있어야 한다.

`CLOCK_TAI` (리눅스 3.10부터)
:   벽시계 시간에서 파생되었고 윤초를 무시하는 시스템 전역 클럭.

위 클럭들에 대한 좀 더 자세한 내용은 <tt>[[clock_getres(2)]]</tt>를 보라.

위 값들뿐만 아니라 <tt>[[clock_getcpuclockid(3)]]</tt> 내지 <tt>[[pthread_getcpuclockid(3)]]</tt> 호출이 반환한 `clockid`로도 `clockid`를 지정할 수 있다.

`sevp` 인자는 타이머가 만료될 때 호출자가 알림 받는 방법을 나타내는 `sigevent` 구조체를 가리킨다. 이 구조체의 정의와 일반적 세부 내용은 <tt>[[sigevent(7)]]</tt>를 보라.

`sevp.sigev_notify` 필드가 다음 값을 가질 수 있다.

`SIGEV_NONE`
:   타이머가 만료될 때 비동기적으로 알리지 않는다. <tt>[[timer_gettime(2)]]</tt>을 이용해 타이머의 진행을 확인한다.

`SIGEV_SIGNAL`
:   타이머 만료 시 프로세스에게 시그널 `sigev_signo`를 생성한다. 일반적 세부 내용은 <tt>[[sigevent(7)]]</tt>를 보라. `siginfo_t` 구조체의 `si_code` 필드가 `SI_TIMER`로 설정된다. 어느 시점이든 어떤 타이머에 대해 프로세스로의 큐에 최대 한 개 시그널이 들어간다. 더 자세한 내용은 <tt>[[timer_getoverrun(2)]]</tt>을 보라.

`SIGEV_THREAD`
:   타이머 만료 시 새 스레드의 시작 함수인 것처럼 `sigev_notify_function`을 호출한다. 자세한 내용은 <tt>[[sigevent(7)]]</tt>를 보라.

`SIGEV_THREAD_ID` (리눅스 한정)
:   `SIGEV_SIGNAL`과 같되 `sigev_notify_thread_id`로 준 ID를 가진 스레드가 시그널의 대상이다. 호출자와 같은 프로세스 내의 스레드여야 한다. `sigev_notify_thread_id` 필드에서는 커널 스레드 ID를, 즉 <tt>[[clone(2)]]</tt>이나 <tt>[[gettid(2)]]</tt>가 반환한 값을 지정한다. 이 플래그는 스레드 라이브러리에서의 사용만을 위한 것이다.

`sevp`를 NULL로 지정하는 것은 `sigev_notify`가 `SIGEV_SIGNAL`이고 `sigev_signo`가 `SIGALRM`, `sigev_value.sival_int`가 타이머 ID인 `sigevent` 구조체의 포인터를 지정하는 것과 동등하다.

## RETURN VALUE

성공 시 `timer_create()`은 0을 반환하며 새 타이머의 ID가 `*timerid`에 들어간다. 실패 시 -1을 반환하며 `errno`를 설정하여 오류를 나타낸다.

## ERRORS

`EAGAIN`
:   커널 내 타이머 구조체 할당 중의 일시적 오류.

`EINVAL`
:   클럭 ID나 `sigev_notify`, `sigev_signo`, `sigev_notify_thread_id`가 유효하지 않다.

`ENOMEM`
:   메모리를 할당할 수 없다.

`ENOTSUP`
:   커널에서 이 `clockid`에 대한 타이머 만들기를 지원하지 않는다.

`EPERM`
:   `clockid`가 `CLOCK_REALTIME_ALARM` 또는 `CLOCK_BOOTTIME_ALARM`이었지만 호출자가 `CAP_WAKE_ALARM` 역능을 가지고 있지 않다.

## VERSIONS

리눅스 2.6부터 이 시스템 호출이 사용 가능하다.

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## NOTES

프로그램에서 `timer_create()`를 이용해 여러 개의 간격 타이머를 만들 수 있다.

<tt>[[fork(2)]]</tt>의 자식이 타이머를 물려받지 않으며 <tt>[[execve(2)]]</tt> 중에 타이머가 해제 및 삭제된다.

`timer_create()`를 이용해 만든 각 타이머마다 커널이 "큐에 들어간 실시간 시그널"을 미리 할당한다. 따라서 `RLIMIT_SIGPENDING` 자원 제한에 의해 타이머 개수가 제한된다 (<tt>[[setrlimit(2)]]</tt> 참고).

`timer_create()`로 만든 타이머를 보통 "POSIX (간격) 타이머"라고 한다. POSIX 타이머 API는 다음 인터페이스들로 이뤄져 있다.

* `timer_create()`: 타이머 만들기.

* <tt>[[timer_settime(2)]]</tt>: 타이머를 장전(시작)하거나 해제(정지)하기.

* <tt>[[timer_gettime(2)]]</tt>: 타이머 다음 만료까지 남은 시간과 타이머 간격 설정 가져오기.

* <tt>[[timer_getoverrun(2)]]</tt>: 최근 타이머 만료에 대해 초과 횟수 반환하기.

* <tt>[[timer_delete(2)]]</tt>: 타이머를 해제하고 삭제하기.

리눅스 3.10부터는 `/proc/[pid]/timers` 파일을 이용해 PID가 `pid`인 프로세스의 POSIX 타이머들을 나열할 수 있다. 더 많은 정보는 <tt>[[proc(5)]]</tt>을 보라.

리눅스 4.10부터는 POSIX 타이머 지원이 구성 가능 옵션이며 기본적으로 켜져 있다. `CONFIG_POSIX_TIMERS` 옵션을 통해 커널 지원을 끌 수 있다.

### C 라이브러리/커널 차이

POSIX 타이머 API의 구현 일부를 glibc에서 제공한다. 구체적으로 다음과 같다.

* `SIGEV_THREAD`의 기능성 상당 부분이 커널이 아니라 glibc에 구현되어 있다. (알림을 다루는 데 관련된 스레드가 C 라이브러리의 POSIX 스레드 구현에서 관리해야 하는 스레드이므로 그럴 수밖에 없다.) 프로세스로 알림이 전달되는 것은 스레드를 통해서이지만 내부적으로 NPTL 구현에서는 `sigev_notify` 값을 `SIGEV_THREAD_ID`로 하고 구현에서 예약해 둔 실시간 시그널을 이용한다 (<tt>[[nptl(7)]]</tt> 참고).

* `evp`가 NULL인 기본 경우 구현을 glibc 내에서 다룬다. 적절히 채운 `sigevent` 구조체로 기반 시스템 호출을 부른다.

* 사용자 공간에 주고받는 타이머 ID들을 glibc에서 관리한다. 이 ID들을 커널에서 사용하는 타이머 ID들로 사상한다.

리눅스 2.6에서 POSIX 타이머 시스템 호출이 처음 등장했다. 그 전에 glibc에서 POSIX 스레드를 이용해 불완전한 사용자 공간 구현을 (`CLOCK_REALTIME` 타이머만) 제공하였으며 glibc 버전 2.17 전의 구현에서는 2.6 전 리눅스 커널을 돌리는 시스템에서 이 기법에 의지한다.

## EXAMPLES

아래 프로그램은 두 가지 인자를 받는다. 초 단위의 sleep 주기와 나노초 단위의 타이머 빈도이다. 프로그램이 타이머에 사용하는 시그널의 핸들러를 설정하고, 그 시그널을 차단하고, 주어진 빈도로 만료되는 타이머를 만들어서 장전하고, 지정한 초 동안 잠들고, 마지막으로 타이머 시그널 차단을 푼다. 프로그램이 잠들었던 동안 타이머가 최소 한 번은 만료되었다고 하면 시그널 핸들러가 호출될 것이고 핸들러에서 타이머 알림에 대한 몇 가지 정보를 표시한다. 시그널 핸들러가 한 번 호출된 후에는 프로그램이 종료된다.

다음 실행 예에서 프로그램은 100나노초 빈도로 타이머를 만든 다음 1초 동안 잠든다. 시그널 차단이 풀려서 전달되는 시점까지 천만 번 정도의 초과가 있었다.

```text
$ ./a.out 1 100
Establishing handler for signal 34
Blocking signal 34
timer ID is 0x804c008
Sleeping for 1 seconds
Unblocking signal 34
Caught signal 34
    sival_ptr = 0xbfb174f4;     *sival_ptr = 0x804c008
    overrun count = 10004886
```

### 프로그램 소스

```c
#include <stdint.h>
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <signal.h>
#include <time.h>

#define CLOCKID CLOCK_REALTIME
#define SIG SIGRTMIN

#define errExit(msg)    do { perror(msg); exit(EXIT_FAILURE); \
                        } while (0)

static void
print_siginfo(siginfo_t *si)
{
    timer_t *tidp;
    int or;

    tidp = si->si_value.sival_ptr;

    printf("    sival_ptr = %p; ", si->si_value.sival_ptr);
    printf("    *sival_ptr = %#jx\n", (uintmax_t) *tidp);

    or = timer_getoverrun(*tidp);
    if (or == -1)
        errExit("timer_getoverrun");
    else
        printf("    overrun count = %d\n", or);
}

static void
handler(int sig, siginfo_t *si, void *uc)
{
    /* 주의: 시그널 핸들러에서 printf()를 호출하는 것은 안전하지
       않다. (따라서 실제 사용하는 프로그램에서는 하지 말아야
       한다.) printf()가 비동기 시그널 안전이 아니기 때문이다.
       signal-safety(7) 참고. 그럼에도 불구하고 핸들러가 호출된
       것을 간단히 보여 주기 위해 여기에 printf()를 사용한다. */

    printf("Caught signal %d\n", sig);
    print_siginfo(si);
    signal(sig, SIG_IGN);
}

int
main(int argc, char *argv[])
{
    timer_t timerid;
    struct sigevent sev;
    struct itimerspec its;
    long long freq_nanosecs;
    sigset_t mask;
    struct sigaction sa;

    if (argc != 3) {
        fprintf(stderr, "Usage: %s <sleep-secs> <freq-nanosecs>\n",
                argv[0]);
        exit(EXIT_FAILURE);
    }

    /* 타이머 시그널 핸들러 설정하기. */

    printf("Establishing handler for signal %d\n", SIG);
    sa.sa_flags = SA_SIGINFO;
    sa.sa_sigaction = handler;
    sigemptyset(&sa.sa_mask);
    if (sigaction(SIG, &sa, NULL) == -1)
        errExit("sigaction");

    /* 잠시 타이머 시그널 차단하기. */

    printf("Blocking signal %d\n", SIG);
    sigemptyset(&mask);
    sigaddset(&mask, SIG);
    if (sigprocmask(SIG_SETMASK, &mask, NULL) == -1)
        errExit("sigprocmask");

    /* 타이머 생성하기. */

    sev.sigev_notify = SIGEV_SIGNAL;
    sev.sigev_signo = SIG;
    sev.sigev_value.sival_ptr = &timerid;
    if (timer_create(CLOCKID, &sev, &timerid) == -1)
        errExit("timer_create");

    printf("timer ID is %#jx\n", (uintmax_t) timerid);

    /* 타이머 시작하기. */

    freq_nanosecs = atoll(argv[2]);
    its.it_value.tv_sec = freq_nanosecs / 1000000000;
    its.it_value.tv_nsec = freq_nanosecs % 1000000000;
    its.it_interval.tv_sec = its.it_value.tv_sec;
    its.it_interval.tv_nsec = its.it_value.tv_nsec;

    if (timer_settime(timerid, 0, &its, NULL) == -1)
         errExit("timer_settime");

    /* 잠깐 눈 붙이기. 그 동안 타이머가 여러 번
       만료될 수 있다. */

    printf("Sleeping for %d seconds\n", atoi(argv[1]));
    sleep(atoi(argv[1]));

    /* 타이머 시그널 차단을 풀어서 타이머 알림이
       전달될 수 있게 하기. */

    printf("Unblocking signal %d\n", SIG);
    if (sigprocmask(SIG_UNBLOCK, &mask, NULL) == -1)
        errExit("sigprocmask");

    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[clock_gettime(2)]]</tt>, <tt>[[setitimer(2)]]</tt>, <tt>[[timer_delete(2)]]</tt>, <tt>[[timer_getoverrun(2)]]</tt>, <tt>[[timer_settime(2)]]</tt>, <tt>[[timerfd_create(2)]]</tt>, <tt>[[clock_getcpuclockid(3)]]</tt>, <tt>[[pthread_getcpuclockid(3)]]</tt>, <tt>[[pthreads(7)]]</tt>, <tt>[[sigevent(7)]]</tt>, <tt>[[signal(7)]]</tt>, <tt>[[time(7)]]</tt>

----

2021-03-22
