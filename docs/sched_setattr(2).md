## NAME

sched_setattr, sched_getattr - 스케줄링 정책과 속성 설정하고 얻기

## SYNOPSIS

```c
#include <sched.h>

int sched_setattr(pid_t pid, struct sched_attr *attr,
                  unsigned int flags);
int sched_getattr(pid_t pid, struct sched_attr *attr,
                  unsigned int size, unsigned int flags);
```

*주의*: 이 시스템 호출에 대한 glibc 래퍼가 없다. NOTES 참고.

## DESCRIPTION

### `sched_setattr()`

`sched_setattr()` 시스템 호출은 `pid`로 지정한 ID의 스레드를 위한 스케줄링 정책과 연관 속성들을 설정한다. `pid`가 0이면 호출 스레드의 스케줄링 정책과 속성을 설정하게 된다.

`policy`에 지정할 수 있는 값으로 현재 리눅스에서는 다음의 "일반" (즉 비실시간) 스케줄링 정책들을 지원한다.

`SCHED_OTHER`
:   표준 라운드 로빈 시공유 정책

`SCHED_BATCH`
:   "배치" 방식 프로세스 실행

`SCHED_IDLE`
:   *아주* 낮은 우선순위의 배경 작업 실행

실행 가능 스레드들 가운데 실행할 것을 선택하는 방식을 정밀하게 제어해야 하는 특수한 시간 제약적 응용들을 위해 다양한 "실시간" 정책들도 지원한다. 프로세스에서 언제 이 정책들을 사용할 수 있는지에 대한 규칙은 <tt>[[sched(7)]]</tt>를 보라. `policy`에 지정할 수 있는 실시간 정책들은 다음과 같다.

`SCHED_FIFO`
:   선입선출 정책

`SCHED_RR`
:   라운드 로빈 정책

리눅스에서는 다음 정책도 지원한다.

`SCHED_DEADLINE`
:   마감 스케줄링 정책. 자세한 내용은 <tt>[[sched(7)]]</tt> 참고.

`attr` 인자는 지정한 스레드를 위한 새 스케줄링 정책과 속성들을 규정하는 구조체에 대한 포인터이다. 이 구조체는 다음과 같은 형태이다.

```c
struct sched_attr {
    u32 size;              /* 이 구조체의 크기 */
    u32 sched_policy;      /* 정책 (SCHED_*) */
    u64 sched_flags;       /* 플래그 */
    s32 sched_nice;        /* 나이스 값 (SCHED_OTHER,
                              SCHED_BATCH) */
    u32 sched_priority;    /* 고정 우선순위 (SCHED_FIFO,
                              SCHED_RR) */
    /* 나머지 필드들은 SCHED_DEADLINE 용 */
    u64 sched_runtime;
    u64 sched_deadline;
    u64 sched_period;
};
```

`sched_attr` 구조체의 필드들은 다음과 같다.

`size`
:   이 필드는 `sizeof(struct sched_attr)`처럼 해서 구조체의 바이트 단위 크기로 설정해야 한다. 제공한 구조체가 커널 구조체보다 작으면 추가 필드들이 '0'인 것으로 상정한다. 제공한 구조체가 커널 구조체보다 크면 커널에서는 모든 추가 필드들이 0인지 검사한다. 0이 아니면 `sched_setattr()`이 `E2BIG` 오류로 실패하며 커널 구조체의 크기를 담도록 `size`를 갱신한다.

    사용자 공간 `sched_attr` 구조체의 크기가 커널 구조체 크기와 일치하지 않을 때의 위 동작 방식을 통해 향후 인터페이스 확장이 가능하다. 너무 큰 구조체를 전달하는 이상한 응용이 있을 때 향후에 커널 `sched_attr` 구조체의 크기가 커지더라도 문제가 생기지 않게 된다. 또한 향후에 더 큰 사용자 공간 `sched_attr` 구조체를 알고 있는 응용에서 자기가 지금 그 큰 구조체를 지원하지 않는 오래된 커널 상에서 동작 중인지 여부를 판단할 수 있다.

`sched_policy`
:   이 필드는 위에 나열한 `SCHED_*` 값들 중 하나로 스케줄링 정책을 나타낸다.

`sched_flags`
:   이 필드는 스케줄링 동작 방식을 제어하는 다음 플래그를 0개 이상 OR 해서 담는다.

    `SCHED_FLAG_RESET_ON_FORK`
    :   <tt>[[fork(2)]]</tt>로 생성된 자식이 특권적 스케줄링 정책을 물려받지 않는다. 자세한 내용은 <tt>[[sched(7)]]</tt> 참고.

    `SCHED_FLAG_RECLAIM` (리눅스 4.13부터)
    :   이 플래그는 `SCHED_DEADLINE` 스레드가 다른 실시간 스레드에서 안 쓴 대역폭을 가져다 쓸 수 있도록 한다.

    `SCHED_FLAG_DL_OVERRUN` (리눅스 4.16부터)
    :   이 플래그는 `SCHED_DEADLINE` 스레드가 실행 시간을 초과했을 때 응용에서 알림을 받을 수 있도록 한다. (예를 들면) 덜 세밀한 실행 시간 계산이나 부정확한 매개변수 할당 때문에 그렇게 초과하는 경우가 생길 수 있다. 알림은 `SIGXCPU` 시그널 형태이고 초과 때마다 생성된다.
 
        그 `SIGXCPU` 시그널은 스레드가 아니라 *프로세스*를 향한다. (<tt>[[signal(7)]]</tt> 참고.) 이는 버그라고 볼 수도 있다. `sched_setattr()`은 스레드별 속성을 설정하는 데 쓰이고 있다. 그런데 실행 시간이 초과한 스레드가 아닌 프로세스 내 스레드로 프로세스 지향 시그널이 전달되면 응용에서는 어느 스레드에서 초과가 일어났는지 알 방법이 없다.

`sched_nice`
:   이 필드는 `sched_policy`를 `SCHED_OTHER`나 `SCHED_BATCH`로 지정할 때 설정할 나이스 값을 나타낸다. 나이스 값은 -20(높은 우선순위)에서 +19(낮은 우선순위)까지 범위의 수이다. <tt>[[sched(7)]]</tt> 참고.

`sched_priority`
:   이 필드는 `sched_policy`를 `SCHED_FIFO`나 `SCHED_RR`로 지정할 때 설정할 고정 우선순위를 나타낸다. 이 정책들에서 허용하는 우선순위 범위를 <tt>[[sched_get_priority_min(2)]]</tt>과 <tt>[[sched_get_priority_max(2)]]</tt>를 이용해 알아낼 수 있다. 다른 정책에서는 이 필드를 0으로 지정해야 한다.

`sched_runtime`
:   이 필드는 마감 스케줄링의 "런타임" 매개변수를 나타낸다. 나노초 단위로 나타낸 값이다. 이 필드와 다음의 두 필드는 `SCHED_DEADLINE` 스케줄링에서만 쓴다. 더 자세한 내용은 <tt>[[sched(7)]]</tt>를 보라.

`sched_deadline`
:   이 필드는 마감 스케줄링의 "마감" 매개변수를 나타낸다. 나노초 단위로 나타낸 값이다.

`sched_period`
:   이 필드는 마감 스케줄링의 "주기" 매개변수를 나타낸다. 나노초 단위로 나타낸 값이다.

`flags` 인자는 향후 인터페이스 확장을 위한 것이다. 현재 구현에서는 0으로 지정해야 한다.

### `sched_getattr()`

`sched_getattr()` 시스템 호출은 `pid`로 지정한 ID의 스레드를 위한 스케줄링 정책과 관련 속성들을 가져온다. `pid`가 0이면 호출 스레드의 스케줄링 정책과 속성을 가져오게 된다.

`size` 인자는 사용자 공간에 알려져 있는 `sched_attr` 구조체의 크기로 설정해야 한다. 그 값이 적어도 최초 공개된 `sched_attr` 구조체의 크기만큼은 되어야 하며, 그렇지 않으면 `EINVAL` 오류로 호출이 실패한다.

얻어 낸 스케줄링 속성들이 `attr`이 가리키는 `sched_attr` 구조체의 필드들로 간다. 그리고 커널에서 `attr.size`를 자기 `sched_attr` 구조체의 크기로 설정한다.

호출자가 제공한 `attr` 버퍼가 커널의 `sched_attr` 구조체보다 크면 사용자 공간 구조체의 추가 바이트들을 건드리지 않는다. 호출자가 제공한 구조체가 커널의 `sched_attr` 구조체보다 작으면 제공된 공간 밖에 저장되었을 값들을 제외하고 조용히 반환한다. `sched_setattr()`에서처럼 이런 동작 방식을 통해 향후 인터페이스 확장이 가능하다.

`flags` 인자는 향후 인터페이스 확장을 위한 것이다. 현재 구현에서는 0으로 지정해야 한다.

## RETURN VALUE

성공 시 `sched_setattr()`과 `sched_getattr()`은 0을 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`sched_getattr()`과 `sched_setattr()` 모두 다음 이유로 실패할 수 있다.

`EINVAL`
:   `attr`이 NULL이거나, `pid`가 음수이거나, `flags`가 0이 아니다.

`ESRCH`
:   ID가 `pid`인 스레드를 찾을 수 없다.

더불어 `sched_getattr()`이 다음 이유로 실패할 수 있다.

`E2BIG`
:   `size`와 `attr`로 지정한 버퍼가 너무 작다.

`EINVAL`
:   `size`가 유효하지 않다. 즉, `sched_attr` 구조체의 최초 버전(48바이트)보다 작거나 시스템 페이지 크기보다 크다.

더불어 `sched_setattr()`이 다음 이유로 실패할 수 있다.

`E2BIG`
:   `size`와 `attr`로 지정한 버퍼가 커널 구조체보다 크며 추가 바이트들 중 일부가 0이 아니다.

`EBUSY`
:   `SCHED_DEADLINE` 승인 통제 실패. <tt>[[sched(7)]]</tt> 참고.

`EINVAL`
:   `attr.sched_policy`가 알려진 정책이 아니거나, `attr.sched_flags`가 `SCHED_FLAG_RESET_ON_FORK` 외의 플래그를 담고 있거나, `attr.sched_priority`가 유효하지 않거나, `attr.sched_policy`가 `SCHED_DEADLINE`인데 `attr` 내의 마감 스케줄링 매개변수가 유효하지 않다.

`EPERM`
:   호출자가 적절한 특권을 가지고 있지 않다.

`EPERM`
:   `pid`로 지정한 스레드의 CPU 친화성 마스크에 시스템의 모든 CPU들이 포함돼 있지 않다. (<tt>[[sched_setaffinity(2)]]</tt> 참고.)

## VERSIONS

리눅스 3.14에서 이 시스템 호출들이 처음 등장했다.

## CONFORMING TO

이 시스템 호출들은 비표준 리눅스 확장이다.

## NOTES

glibc에서 이 시스템 호출의 래퍼를 제공하지 않는다. <tt>[[syscall(2)]]</tt>을 이용해 호출해야 한다.

`sched_setattr()`은 <tt>[[sched_setscheduler(2)]]</tt>, <tt>[[sched_setparam(2)]]</tt>, <tt>[[nice(2)]]</tt>, 그리고 (특정 사용자에게 속한 모든 프로세스나 특정 그룹의 모든 프로세스들의 우선순위를 설정하는 것을 제외하고) <tt>[[setpriority(2)]]</tt>의 상위집합 기능을 제공한다. 유사하게 `sched_getattr()`은 <tt>[[sched_getscheduler(2)]]</tt>, <tt>[[sched_getparam(2)]]</tt>, 그리고 (부분적으로) <tt>[[getpriority(2)]]</tt>의 상위집합 기능을 제공한다.

## BUGS

리눅스 버전 3.15까지에서 ERRORS 절에서 기술하는 경우에 `sched_setattr()`이 `E2BIG` 대신 `EFAULT` 오류로 실패했다.

리눅스 버전 5.3까지에서 커널 내 `sched_attr` 구조체가 사용자 공간에서 준 `size`보다 큰 경우 `sched_getattr()`이 `EFBIG` 오류로 실패했다.

## SEE ALSO

`chrt(1)`, <tt>[[nice(2)]]</tt>, <tt>[[sched_get_priority_max(2)]]</tt>, <tt>[[sched_get_priority_min(2)]]</tt>, <tt>[[sched_getaffinity(2)]]</tt>, <tt>[[sched_getparam(2)]]</tt>, <tt>[[sched_getscheduler(2)]]</tt>, <tt>[[sched_rr_get_interval(2)]]</tt>, <tt>[[sched_setaffinity(2)]]</tt>, <tt>[[sched_setparam(2)]]</tt>, <tt>[[sched_setscheduler(2)]]</tt>, <tt>[[sched_yield(2)]]</tt>, <tt>[[setpriority(2)]]</tt>, <tt>[[pthread_setschedparam(3)]]</tt>, <tt>[[pthread_setschedparam(3)]]</tt>, <tt>[[pthread_setschedprio(3)]]</tt>, <tt>[[capabilities(7)]]</tt>, <tt>[[cpuset(7)]]</tt>, <tt>[[sched(7)]]</tt>

----

2021-03-22
