## NAME

gettimeofday, settimeofday - 시간 얻기/설정하기

## SYNOPSIS

```c
#include <sys/time.h>

int gettimeofday(struct timeval *restrict tv,
                 struct timezone *restrict tz);

int settimeofday(const struct timeval *tv,
                 const struct timezone *tz);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`settimeofday()`:
:   glibc 2.19부터:
    :   `_DEFAULT_SOURCE`

    glibc 2.19 및 이전:
    :   `_BSD_SOURCE`

## DESCRIPTION

`gettimeofday()` 및 `settimeofday()` 함수는 시간에 더해서 시간대를 얻거나 설정할 수 있다.

`tv` 인자는 (`<sys/time.h>`에 있는) `struct timeval`이며,

```c
struct timeval {
    time_t      tv_sec;     /* 초 */
    suseconds_t tv_usec;    /* 마이크로초 */
};
```

에포크(<tt>[[time(2)]]</tt> 참고) 이후의 초 및 마이크로초 수를 나타낸다.

`tz` 인자는 `struct timezone`이다.

```c
struct timezone {
    int tz_minuteswest;     /* 그리니치 서쪽으로 몇 분 */
    int tz_dsttime;         /* DST 수정 종류 */
};
```

`tv`나 `tz` 중 어느 쪽이라도 NULL이면 해당 구조체를 설정 내지 반환하지 않는다. (하지만 `tv`가 NULL이면 컴파일 경고가 나게 된다.)

`timezone` 구조체 사용은 구식화되었다. 보통은 `tz` 인자를 NULL로 지정하는 게 좋다. (아래 NOTES 참고.)

리눅스에는 `settimeofday()` 시스템 호출과 관련해 좀 독특한 "클럭 왜곡" 동작 방식이 있다. (부팅 후) 최초 호출에서 `tz` 인자가 NULL이 아니고 `tz` 인자가 NULL이면서 `tz_minuteswest` 필드가 0이 아닐 때의 동작이다. (이때 `tz_dsttime`은 0으로 하는 게 좋다.) 이런 경우에는 CMOS 클럭이 지역 시간이고, 거기에 그만큼 시간을 더해야 UTC 시스템 시간을 얻을 수 있다고 친다. 이 기능을 쓰지 않는 게 좋다는 데는 두말할 나위가 없다.

## RETURN VALUE

`gettimeofday()`와 `settimeofday()`는 성공 시 0을 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EFAULT`
:   `tv`나 `tz` 중 하나가 접근 가능 주소 공간 밖을 가리킨다.

`EINVAL`
:   (`settimeofday()`): `timezone`이 유효하지 않다.

`EINVAL`
:   (`settimeofday()`): `tv.tv_sec`이 음수거나 `tv.tv_usec`이 [0..999,999] 범위 밖이다.

`EINVAL` (리눅스 4.3부터)
:   (`settimeofday()`): 시간을 `CLOCK_MONOTONIC` 클럭의 현재 값보다 작은 값으로 설정하려 했다. (<tt>[[clock_gettime(2)]]</tt> 참고.)

`EPERM`
:   호출 프로세스에게 `settimeofday()` 호출을 위한 충분한 특권이 없다. 리눅스에서는 `CAP_SYS_TIME` 역능이 필요하다.

## CONFORMING TO

SVr4, 4.3BSD. POSIX.1-2001에서는 `gettimeofday()`는 기술하지만 `settimeofday()`는 기술하고 있지 않다. POSIX.1-2008에서는 `gettimeofday()`를 구식으로 표시하고 있으며 대신 <tt>[[clock_gettime(2)]]</tt> 사용을 권장한다.

## NOTES

`gettimeofday()`가 반환하는 시간은 (시스템 관리자가 시스템 시간을 수동으로 바꾸는 경우 같은) 불연속적 시스템 시간 도약의 영향을 받는다. 단조 증가하는 클럭이 필요하다면 <tt>[[clock_gettime(2)]]</tt>을 보라.

`timeval` 구조체를 다루기 위한 매크로들을 <tt>[[timeradd(3)]]</tt>에서 설명한다.

전통적으로 `struct timeval`의 필드들은 `long` 타입이었다.

### C 라이브러리/커널 차이

일부 아키텍처에서는 <tt>[[vdso(7)]]</tt>에서 `gettimeofday()` 구현을 제공한다.

### `tz_dsttime` 필드

리눅스 아닌 커널 상에서 glibc를 쓰는 경우에는 현재 시간대에서 한 번이라도 일광 절약 규칙을 적용한 적이 있거나 적용할 예정이면 `gettimeofday()`에서 `struct timezone`의 `tz_dsttime` 필드를 0 아닌 값으로 설정하게 된다. 즉 현재 시간대에 대해 <tt>[[daylight(3)]]</tt>의 의미를 정확히 반영한다. 리눅스 상에서 glibc를 쓰는 경우에는 `settimeofday()`나 `gettimeofday()`에서 `struct timezone`의 `tz_dsttime` 필드 설정을 이용한 적이 한 번도 없다. 따라서 다음 내용은 순수하게 역사적 관심사일 뿐이다.

오래된 시스템들에서 `tz_dsttime` 필드에는 그 해 어느 시기에 일광 절약 시간이 시행되는지 나타내는 심볼 상수(아래에 값들이 있음)가 담긴다. (참고: 이 값은 한해 내내 고정돼 있다. 즉 DST가 시행 중임을 나타내는 게 아니라 알고리즘을 선정할 뿐이다.) 다음 일광 절약 시간 알고리즘들이 정의돼 있다.

```c
DST_NONE     /* DST 비시행 */
DST_USA      /* 미국 방식 DST */
DST_AUST     /* 오스트레일리아 방식 DST */
DST_WET      /* 서유럽 DST */
DST_MET      /* 중유럽 DST */
DST_EET      /* 동유럽 DST */
DST_CAN      /* 캐나다 */
DST_GB       /* 영국 */
DST_RUM      /* 루마니아 */
DST_TUR      /* 터키 */
DST_AUSTALT  /* 오스트레일리아 1986년 변경 방식 */
```

당연하게도 일광 절약 시간 시행 시기를 나라마다 하나씩의 단순한 알고리즘으로 나타낼 수는 없다는 게 드러났다. 실제로 그 시기는 예측 불가능한 정치적 판단에 따라 정해진다. 그래서 이렇게 시간대를 나타내는 방식은 버려졌다.

## SEE ALSO

`date(1)`, <tt>[[adjtimex(2)]]</tt>, <tt>[[clock_gettime(2)]]</tt>, <tt>[[time(2)]]</tt>, <tt>[[ctime(3)]]</tt>, <tt>[[ftime(3)]]</tt>, <tt>[[timeradd(3)]]</tt>, <tt>[[capabilities(7)]]</tt>, <tt>[[time(7)]]</tt>, <tt>[[vdso(7)]]</tt>, `hwclock(8)`

----

2021-03-22
