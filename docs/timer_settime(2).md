## NAME

timer_settime, timer_gettime - POSIX 프로세스별 타이머 장전/해제 및 상태 가져오기

## SYNOPSIS

```c
#include <time.h>

int timer_settime(timer_t timerid, int flags,
                  const struct itimerspec *restrict new_value,
                  struct itimerspec *restrict old_value);
int timer_gettime(timer_t timerid, struct itimerspec *curr_value);
```

`-lrt`로 링크.

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`timer_settime()`, `timer_gettime()`:
:   `_POSIX_C_SOURCE >= 199309L`

## DESCRIPTION

`timer_settime()`은 `timerid`가 나타내는 타이머를 장전하거나 해제한다. `new_value` 인자는 타이머의 새 최초 값과 새 간격을 나타내는 `itimerspec` 구조체에 대한 포인터이다. `itimerspec` 구조체는 다음과 같이 정의되어 있다.

```c
struct timespec {
    time_t tv_sec;                /* 초 */
    long   tv_nsec;               /* 나노초 */
};

struct itimerspec {
    struct timespec it_interval;  /* 타이머 간격 */
    struct timespec it_value;     /* 최초 만료 */
};
```

`itimerspec` 구조체의 하위 구조 각각은 `timespec` 구조체여서 시간 값을 초와 나노초로 지정할 수 있다. <tt>[[timer_create(2)]]</tt>로 타이머를 만들 때 지정한 클럭에 따라 이 시간 값들을 측정한다.

`new_value->it_value`가 0 아닌 값이면 (즉 한 하위 필드라도 0이 아니면) `timer_settime()`은 그 시간에 최초 만료되도록 설정하여 타이머를 장전(시작)한다. (타이머가 이미 장전되어 있으면 이전 설정을 덮어 쓴다.) `new_value->it_value`가 0 값이면 (즉 두 하위 필드 모두 0이면) 타이머를 해제한다.

`new_value->it_interval` 필드는 초와 나노초로 타이머의 주기를 나타낸다. 이 필드가 0이 아니면 장전된 타이머가 만료될 때마다 `new_value->it_interval`에 지정한 값으로 타이머를 재장전한다. `new_value->it_interval`이 0 값이면 `it_value`로 지정한 시간에 한 번만 타이머가 만료된다.

기본적으로 `new_value->it_value`로 지정하는 최초 만료 시간은 호출 시점에 타이머 클럭의 현재 시간을 기준으로 상대적으로 해석한다. `flags`에 `TIMER_ABSTIME`을 지정해서 바꿀 수 있는데, 그렇게 하면 `new_value->it_value`를 타이머 클럭에서 측정한 절댓값으로 해석한다. 즉 클럭 값이 `new_value->it_value`로 지정한 값에 도달할 때 타이머가 만료된다. 지정한 절대 시간이 이미 지났으면 타이머가 즉시 발화되며 초과 횟수(<tt>[[timer_getoverrun(2)]]</tt> 참고)가 그에 맞게 설정된다.

`CLOCK_REALTIME` 클럭 기반의 절댓값 타이머가 장전되어 있는 동안 그 클럭의 값을 조정하는 경우에는 타이머 만료가 적절히 조정된다. `CLOCK_REALTIME` 클럭 기반의 상댓값 타이머에는 클럭 조정이 영향을 주지 않는다.

`old_value`가 NULL이 아니면 그 버퍼를 이용해 타이머의 이전 간격(`old_value->it_interval`)과 타이머가 다음으로 만료될 때까지 남았던 시간(`old_value->it_value`)을 반환한다.

`timer_gettime()`은 `timerid`로 지정한 타이머에 대해 다음 만료까지의 시간과 간격을 `curr_value`가 가리키는 버퍼로 반환한다. 다음 타이머 만료까지 남은 시간을 `curr_value->it_value`로 반환하는데, 타이머를 장전할 때 `TIMER_ABSTIME` 플래그를 썼는지와 상관없이 항상 상대적인 값이다. `curr_value->it_value`로 반환된 값이 0이면 타이머가 현재 해제되어 있는 것이다. 타이머 간격은 `curr_value->it_interval`로 반환한다. `curr_value->it_interval`로 반환한 값이 0이면 "단발성" 타이머인 것이다.

## RETURN VALUE

성공 시 `timer_settime()` 및 `timer_gettime()`은 0을 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

이 함수들은 다음 오류로 실패할 수 있다.

`EFAULT`
:   `new_value`나 `old_value`, `curr_value`가 유효한 포인터가 아니다.

`EINVAL`
:   `timerid`가 유효하지 않다.

`timer_settime()`은 다음 오류로 실패할 수도 있다.

`EINVAL`
:   `new_value.it_value`가 음수이다. 또는 `new_value.it_value.tv_nsec`이 음수이거나 999,999,999보다 크다.

## VERSIONS

리눅스 2.6부터 이 시스템 호출들이 사용 가능하다.

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## EXAMPLES

<tt>[[timer_create(2)]]</tt> 참고.

## SEE ALSO

<tt>[[timer_create(2)]]</tt>, <tt>[[timer_getoverrun(2)]]</tt>, <tt>[[time(7)]]</tt>

----

2021-03-22
