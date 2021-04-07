## NAME

sched_rr_get_interval - 지정한 프로세스의 `SCHED_RR` 시간 얻기

## SYNOPSIS

```c
#include <sched.h>

int sched_rr_get_interval(pid_t pid, struct timespec *tp);
```

## DESCRIPTION

`sched_rr_get_interval()`은 `tp`가 가리키는 `timespec` 구조체에 `pid`가 나타내는 프로세스의 라운드 로빈 단위 시간(quantum)을 써넣는다. 지정한 프로세스가 `SCHED_RR` 스케줄링 정책 하에서 돌고 있어야 한다.

`timespec` 구조체는 다음과 같은 형태이다.

```c
struct timespec {
    time_t tv_sec;    /* 초 */
    long   tv_nsec;   /* 나노초 */
};
```

`pid`가 0이면 호출 프로세스의 단위 시간을 `*tp`에 써넣는다.

## RETURN VALUE

성공 시 `sched_rr_get_interval()`은 0을 반환한다. 오류 시 -1을 반환하며 `errno`를 적절히 설정한다.

## ERRORS

<dl>
<dt><code>EFAULT</code></dt>
<dd>사용자 공간으로 정보를 복사하는 중의 문제.</dd>
<dt><code>EINVAL</code></dt>
<dd>유효하지 않은 pid.</dd>
<dt><code>ENOSYS</code></dt>
<dd>시스템 호출이 아직 구현되어 있지 않음. (꽤 오래된 커널에서만)</dd>
<dt><code>ESRCH</code></dt>
<dd>ID가 <code>pid</code>인 프로세스를 찾을 수 없음.</dd>
</dl>

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## NOTES

`sched_rr_get_interval()`을 사용할 수 있는 POSIX 시스템에는 `<unistd.h>`에 `_POSIX_PRIORITY_SCHEDULING`이 정의되어 있다.

### 리눅스 참고 사항

POSIX에서는 라운드 로빈 단위 시간의 크기를 제어하기 위한 어떤 메커니즘도 명세하고 있지 않다. 오래된 리눅스 커널에서는 이를 위한 (이식성 없는) 방법을 제공한다. 프로세스의 나이스 값(<tt>[[setpriority(2)]]</tt>) 참고)을 조정하여 단위 시간을 조정할 수 있다. 음수인 (즉 우선순위 높은) 나이스 값을 부여하면 시간이 길어지고 양수인 (즉 우선순위 낮은) 나이스 값을 부여하면 시간이 짧아진다. 기본 단위 시간은 0.1초이며 나이스 값 변경으로 단위 시간에 영향을 줄 수 있는 정도는 커널 버전에 따라 좀 다르다. 단위 시간 조정을 위한 이 방법은 리눅스 2.6.24부터 제거되었다.

리눅스 3.9에서 `SCHED_RR` 단위 시간 조정을 (그리고 조회를) 위한 새로운 메커니즘이 추가되었다. `/proc/sys/kernel/sched_rr_timeslice_ms` 파일이 밀리초 단위로 단위 시간을 노출하며 그 기본값은 100이다. 이 파일에 0을 써넣으면 단위 시간을 기본값으로 초기화 한다.

## SEE ALSO

<tt>[[sched(7)]]</tt>

----

2017-09-15
