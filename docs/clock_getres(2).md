## NAME

clock_getres, clock_gettime, clock_settime - 클럭 및 시간 함수들

## SYNOPSIS

```c
#include <time.h>

int clock_getres(clockid_t clk_id, struct timespec *res);

int clock_gettime(clockid_t clk_id, struct timespec *tp);
 
int clock_settime(clockid_t clk_id, const struct timespec *tp);
```

`-lrt`로 링크 (glibc 버전 2.17 전에서만).

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`clock_getres()`, `clock_gettime()`, `clock_settime()`:
:   `_POSIX_C_SOURCE >= 199309L`

## DESCRIPTION

`clock_getres()` 함수는 지정한 클럭 `clk_id`의 해상도(정밀도)를 알아내고, `res`가 NULL이 아니면 `res`가 가리키는 `struct timespec`에 그 해상도를 저장한다. 클럭의 해상도는 구현체에 따라 달라지며 특정 프로세스가 설정할 수 있는 것이 아니다. `clock_settime()`의 인자 `tp`가 가리키는 시간 값이 `res`의 배수가 아니면 배수가 되게 잘라낸다.

함수 `clock_gettime()` 및 `clock_settime()`은 지정한 클럭 `clk_id`의 시간을 가져오거나 설정한다.

`res` 및 `tp` 인자는 `<time.h>`에 있는 `timespec` 구조체이다.

```c
struct timespec {
    time_t   tv_sec;        /* 초 */
    long     tv_nsec;       /* 나노초 */
};
```

`clk_id` 인자는 동작을 수행할 구체적 클럭의 식별자이다. 클럭은 시스템 전역이어서 모든 프로세스에게 보일 수도 있고 프로세스별로 있어서 한 프로세스 내에서만 시간을 측정할 수도 있다.

모든 구현체는 시스템 전역 실제 시간 클럭을 제공하며 `CLOCK_REALTIME`으로 이를 나타낸다. 이 클럭의 시간은 에포크 이후 지난 초와 나노초를 나타낸다. 이 클럭의 시간이 바뀔 때 상대적 시간의 타이머는 영향을 받지 않지만 절대적 시점에 대한 타이머는 영향을 받는다.

더 많은 클럭들이 구현되어 있을 수 있다. 해당하는 시간 값을 해석하는 방식과 타이머에 대한 영향은 명세되어 있지 않다.

충분히 최신인 glibc 및 리눅스 커널에서는 다음 클럭들을 지원한다.

`CLOCK_REALTIME`
:   실제 시간을 (즉 벽시계 시간을) 재는 시스템 전역 클럭. 이 클럭을 설정하려면 적절한 특권이 필요하다. 이 클럭은 시스템 시간의 불연속적 도약(가령 시스템 관리자가 수동으로 클럭을 변경하는 경우)과 <tt>[[adjtime(3)]]</tt> 및 NTP가 수행하는 점진적 조정에 영향을 받는다.

`CLOCK_REALTIME_COARSE` (리눅스 2.6.32부터. 리눅스 전용.)
:   `CLOCK_REALTIME`의 더 빠르지만 덜 정확한 버전. 아주 빠르되 정밀하지는 않은 타임스탬프가 필요할 때 쓰면 된다. 아키텍처별 지원이 필요하며, <tt>[[vdso(7)]]</tt> 내에서 이 플래그에 대한 아키텍처 지원도 필요할 것이다.

`CLOCK_MONOTONIC`
:   "어떤 규정돼 있지 않은 시점"(POSIX의 서술)부터 단조 증가하는 시간을 나타내며 설정할 수 없는 클럭. 리눅스에서는 시스템이 부팅 하고 동작한 초 수에 해당한다.

    `CLOCK_MONOTONIC` 클럭은 시스템 시간의 불연속적 도약(가령 시스템 관리자가 수동으로 클럭을 변경하는 경우)에는 영향을 받지 않지만 <tt>[[adjtime(3)]]</tt> 및 NTP가 수행하는 점진적 조정에는 영향을 받는다. 시스템이 절전 대기 상태인 시간은 포함하지 않는다.

`CLOCK_MONOTONIC_COARSE` (리눅스 2.6.32부터. 리눅스 전용.)
:   `CLOCK_MONOTONIC`의 더 빠르지만 덜 정확한 버전. 아주 빠르되 정밀하지는 않은 타임스탬프가 필요할 때 쓰면 된다. 아키텍처별 지원이 필요하며, <tt>[[vdso(7)]]</tt> 내에서 이 플래그에 대한 아키텍처 지원도 필요할 것이다.

`CLOCK_MONOTONIC_RAW` (리눅스 2.6.28부터. 리눅스 전용.)
:   `CLOCK_MONOTONIC`과 유사하되 NTP 조정이나 <tt>[[adjtime(3)]]</tt>이 수행하는 점진적 조정의 영향을 받지 않는 하드웨어 기반 시간에 대한 접근을 제공한다. 시스템이 절전 대기 상태인 시간은 포함하지 않는다.

`CLOCK_BOOTTIME` (리눅스 2.6.39부터. 리눅스 전용.)
:   `CLOCK_MONOTONIC`과 동일하되 시스템이 절전 대기 상태인 시간도 포함한다. <tt>[[settimeofday(2)]]</tt> 등을 이용해 시간을 바꾸면 불연속적일 수도 있는 `CLOCK_REALTIME`의 복잡함을 응용에서 다룰 필요 없이 절전 대기를 인식하는 단조 증가 클럭을 얻을 수 있다.

`CLOCK_PROCESS_CPUTIME_ID` (리눅스 2.6.12부터.)
:   프로세스별 CPU 시간 클럭. (프로세스 내 모든 스레드들이 소모한 CPU 시간을 측정함.)

`CLOCK_THREAD_CPUTIME_ID` (리눅스 2.6.12부터.)
:   스레드 한정 CPU 시간 클럭.

## RETURN VALUE

`clock_gettime()`, `clock_settime()`, `clock_getres()`는 성공 시 0을 반환하고 실패 시 -1을 반환한다. (그리고 실패한 경우 `errno`를 적절히 설정한다.)

## ERRORS

`EFAULT`
:   `tp`가 접근 가능한 주소 공간 밖을 가리키고 있다.

`EINVAL`
:   지정한 `clk_id`를 이 시스템에서 지원하지 않는다.

`EINVAL` (리눅스 4.3부터)
:   `clk_id`를 `CLOCK_REALTIME`으로 한 `clock_settime()` 호출에서 시간을 `CLOCK_MONOTINIC` 클럭 현재 값보다 작은 값으로 설정하려 했다.

`EPERM`
:   `clock_settime()`에서 표시 클럭을 설정할 권한을 가지고 있지 않다.

## VERSIONS

리눅스 2.6에서 이 시스템 호출들이 처음 등장했다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `clock_getres()`, `clock_gettime()`,<br>`clock_settime()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, SUSv2.

## AVAILABILITY

이 함수들이 사용 가능한 POSIX 시스템에는 `<unistd.h>`에 심볼 `_POSIX_TIMERS`가 0보다 큰 값으로 정의되어 있다. 심볼 `_POSIX_MONOTONIC_CLOCK`, `_POSIX_CPUTIME`, `_POSIX_THREAD_CPUTIME`은 `CLOCK_MONOTONIC`, `CLOCK_PROCESS_CPUTIME_ID`, `CLOCK_THREAD_CPUTIME_ID`가 사용 가능함을 나타낸다. (<tt>[[sysconf(3)]]</tt>도 참고.)

## NOTES

POSIX.1에서 다음과 같이 명세하고 있다.

> `clock_settime()`을 통해 `CLOCK_REALTIME` 클럭의 값을 설정하는 것이 `nanosleep()`을 포함해 이 클럭 기반의 상대적 시간 서비스를 기다리며 블록 되어 있는 스레드들에 어떤 영향도 끼치지 않아야 하며, 이 클럭 기반의 상대적 타이머의 만료에 대해서도 마찬가지이다. 따라서 이런 시간 서비스들은 그 클럭의 신규 내지 이전 값과 상관없이 요청했던 상대적 시간이 경과했을 때에 만료되어야 한다.

### C 라이브러리/커널 차이

일부 아키텍처에서는 `clock_gettime()` 구현을 <tt>[[vdso(7)]]</tt>로 제공한다.

### SMP 시스템 관련 역사적 기록

리눅스에 `CLOCK_PROCESS_CPUTIME_ID` 및 `CLOCK_THREAD_CPUTIME_ID`에 대한 커널 지원이 추가되기 전에 glibc에서는 여러 플랫폼들에서 CPU의 타이머 레지스터(i386의 TSC, Itanium의 AR.ITC)를 이용해 이 클럭들을 구현했다. 이런 레지스터들은 CPU마다 값이 다를 수도 있으며 그로 인해 프로세스가 다른 CPU로 이전하는 경우 이 클럭들이 **엉터리 결과**를 내놓을 수도 있다.

SMP 시스템에서 CPU들이 클럭 원천이 서로 다르면 각 CPU가 살짝 다른 진동수로 동작할 것이므로 그 타이머 레지스터들 사이에 연관성을 유지할 방법이 없다. 그런 경우에 `clock_getcpuclockid(0)`은 `ENOENT`를 반환하여 이런 상황을 나타낸다. 그럴 때는 프로세스가 특정 CPU에 머물러 있다고 확신할 수 있는 경우에만 두 클럭이 쓸모가 있게 된다.

SMP 시스템 내의 프로세서들이 모두 정확히 같은 때에 시작하지는 않기 때문에 보통은 타이머 레지스터들이 차이를 가지고 돈다. 어떤 아키텍처들에는 부팅 때 이 차이를 제한하려고 시도하는 코드가 들어있다. 하지만 그 코드가 차이를 정확하게 조정한다고 보장하지는 못한다. glibc에는 (리눅스 커널과 달리) 이런 차이를 다루기 위한 어떤 대비책도 담겨 있지 않다. 보통 이 차이는 작기 때문에 대부분의 경우에서는 그 효과를 무시할 수 있다.

glibc 2.4부터는 이 페이지에서 기술하는 시스템 호출들에 대한 래퍼 함수들이 `CLOCK_PROCESS_CPUTIME_ID` 및 `CLOCK_THREAD_CPUTIME_ID`의 커널 구현을 제공하는 시스템(즉 리눅스 2.6.12 및 이후)에서는 그 구현을 사용하여 앞서 언급한 문제들을 피한다.

## BUGS

POSIX.1-2001에 따르면 "적절한 특권"을 가진 프로세스가 `clock_settime()`을 이용해 `CLOCK_PROCESS_CPUTIME_ID` 및 `CLOCK_THREAD_CPUTIME_ID` 클럭을 설정할 수 있다. 리눅스에서 이 클럭들은 설정 가능하지 않다. (즉, 어떤 프로세스도 "적절한 특권"을 가지고 있지 않다.)

## SEE ALSO

`date(1)`, <tt>[[gettimeofday(2)]]</tt>, <tt>[[settimeofday(2)]]</tt>, <tt>[[time(2)]]</tt>, <tt>[[adjtime(3)]]</tt>, <tt>[[clock_getcpuclockid(3)]]</tt>, <tt>[[ctime(3)]]</tt>, <tt>[[ftime(3)]]</tt>, <tt>[[pthread_getcpuclockid(3)]]</tt>, <tt>[[sysconf(3)]]</tt>, <tt>[[time(7)]]</tt>, <tt>[[vdso(7)]]</tt>, <tt>[[hwclock(8)]]</tt>

----

2019-03-06
