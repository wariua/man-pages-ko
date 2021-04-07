## NAME

sched_setscheduler, sched_getscheduler - 스케줄링 정책/매개변수 설정하고 얻기

## SYNOPSIS

```c
#include <sched.h>

int sched_setscheduler(pid_t pid, int policy,
                       const struct sched_param *param);

int sched_getscheduler(pid_t pid);
```

## DESCRIPTION

`sched_setscheduler()` 시스템 호출은 `pid`로 지정한 ID의 스레드를 위한 스케줄링 정책과 매개변수 모두를 설정한다. `pid`가 0이면 호출 스레드의 스케줄링 정책과 매개변수를 설정하게 된다.

스케줄링 매개변수를 `param` 인자로 지정하는데, 이는 다음 형태의 구조체에 대한 포인터이다.

```c
struct sched_param {
    ...
    int sched_priority;
    ...
};
```

현재 구현에서 이 구조체는 `sched_priority` 필드 하나만 담고 있다. `param`의 해석 방식은 선택한 정책에 따라 달라진다.

`policy`에 지정할 수 있는 값으로 현재 리눅스에서는 다음의 "일반" (즉 비실시간) 스케줄링 정책들을 지원한다.

<dl>
<dt><code>SCHED_OTHER</code></dt>
<dd>표준 라운드 로빈 시공유 정책.</dd>
<dt><code>SCHED_BATCH</code></dt>
<dd>"배치" 방식 프로세스 실행</dd>
<dt><code>SCHED_IDLE</code></dt>
<dd><em>아주</em> 낮은 우선순위의 배경 작업 실행</dd>
</dl>

위 정책들 각각에서 `param->sched_priority`가 0이어야 한다.

실행 가능 스레드들 가운데 실행할 것을 선택하는 방식을 정밀하게 제어해야 하는 특수한 시간 제약적 응용들을 위해 다양한 "실시간" 정책들도 지원한다. 프로세스에서 언제 이 정책들을 사용할 수 있는지에 대한 규칙은 <tt>[[sched(7)]]</tt>를 보라. `policy`에 지정할 수 있는 실시간 정책들은 다음과 같다.

<dl>
<dt><code>SCHED_FIFO</code></dt>
<dd>선입선출 정책</dd>
<dt><code>SCHED_RR</code></dt>
<dd>라운드 로빈 정책</dd>
</dl>

위 정책들 각각에서 `param->sched_priority`가 스레드의 스케줄링 우선순위를 지정한다. 이 값은 지정한 `policy`로 <tt>[[sched_get_priority_min(2)]]</tt>과 <tt>[[sched_get_priority_max(2)]]</tt>를 호출해서 반환받은 범위 내의 수이다. 리눅스에서 이 시스템 호출들은 각각 1과 99를 반환한다.

리눅스 2.6.32부터 `sched_setscheduler()`를 호출할 때 `policy`에 `SCHED_RESET_ON_FORK` 플래그를 OR 할 수 있다. 이 플래그를 포함시키면 <tt>[[fork(2)]]</tt>로 생성된 자식이 특권적 스케줄링 정책을 물려받지 않는다. 자세한 내용은 <tt>[[sched(7)]]</tt>를 보라.

`sched_getscheduler()`는 `pid`가 나타내는 스레드의 현재 스케줄링 정책을 반환한다. `pid`가 0이면 호출 스레드의 정책을 가져오게 된다.

## RETURN VALUE

성공 시 `sched_setscheduler()`는 0을 반환한다. 성공 시 `sched_getscheduler()`는 스레드의 정책(음수 아닌 정수)을 반환한다. 오류 시 두 호출 모두 -1을 반환하며 `errno`를 적절히 설정한다.

## ERRORS

<dl>
<dt><code>EINVAL</code></dt>
<dd>유효하지 않은 인자: <code>pid</code>가 음수이거나 <code>param</code>이 NULL이다.</dd>
<dt><code>EINVAL</code></dt>
<dd>(<code>sched_setscheduler()</code>) <code>policy</code>가 알려진 정책이 아니다.</dd>
<dt><code>EINVAL</code></dt>
<dd>(<code>sched_setscheduler()</code>) 지정한 <code>policy</code>에 대해 <code>param</code>이 말이 되지 않는다.</dd>
<dt><code>EPERM</code></dt>
<dd>호출 스레드가 적절한 특권을 가지고 있지 않다.</dd>
<dt><code>ESRCH</code></dt>
<dd>ID가 <code>pid</code>인 스레드를 찾을 수 없다.</dd>
</dl>

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008 (하지만 아래 BUGS 참고). `SCHED_BATCH` 및 `SCHED_IDLE` 정책은 리눅스 전용이다.

## NOTES

위의 "일반" 및 "실시간" 스케줄링 정책들 모두의 더 자세한 의미론을 <tt>[[sched(7)]]</tt> 매뉴얼 페이지에서 찾을 수 있다. 그 페이지에서는 <tt>[[sched_setattr(2)]]</tt>을 통해서만 설정 가능한 `SCHED_DEADLINE`이라는 정책도 추가로 기술한다.

`sched_setscheduler()`와 `sched_getscheduler()`를 사용할 수 있는 POSIX 시스템에는 `<unistd.h>`에 `_POSIX_PRIORITY_SCHEDULING`이 정의되어 있다.

POSIX.1에서는 비특권 스레드가 `sched_setscheduler()`를 호출하기 위해 필요한 권한들에 대해 상술하지 않으며, 그래서 시스템에 따라 세부 사항이 다르다. 예를 들어 솔라리스 7 매뉴얼 페이지에서는 호출자의 실제 사용자 ID나 실효 사용자 ID가 대상의 실제 사용자 ID나 저장된 set-user-ID와 일치해야 한다고 한다.

리눅스에서 스케줄링 정책과 매개변수는 사실 스레드별 속성이다. <tt>[[gettid(2)]]</tt> 호출이 반환한 값을 `pid` 인자로 전달할 수 있다. `pid`를 0으로 지정하면 호출 스레드의 속성에 대해 동작하게 되고 <tt>[[getpid(2)]]</tt> 호출이 반환한 값을 전달하면 스레드 그룹의 주 스레드의 속성들에 대해 동작하게 된다. (POSIX 스레드 API를 쓰고 있다면 `sched_*(2)` 시스템 호출 대신 <tt>[[pthread_setschedparam(3)]]</tt>, <tt>[[pthread_getschedparam(3)]]</tt>, <tt>[[pthread_setschedprio(3)]]</tt>을 사용하라.)

## BUGS

POSIX.1에서는 성공 시 `sched_setscheduler()`가 이전 스케줄링 정책을 반환해야 한다고 한다. 리눅스의 `sched_setscheduler()`는 이 요구 사항을 준수하지 않는데, 성공 시 항상 0을 반환한다.

## SEE ALSO

`chrt(1)`, <tt>[[nice(2)]]</tt>, <tt>[[sched_get_priority_max(2)]]</tt>, <tt>[[sched_get_priority_min(2)]]</tt>, <tt>[[sched_getaffinity(2)]]</tt>, <tt>[[sched_getattr(2)]]</tt>, <tt>sched_getparam(2)]]</tt>, <tt>[[sched_rr_get_interval(2)]]</tt>, <tt>[[sched_setaffinity(2)]]</tt>, <tt>[[sched_setattr(2)]]</tt>, <tt>[[sched_setparam(2)]]</tt>, <tt>[[sched_yield(2)]]</tt>, <tt>[[setpriority(2)]]</tt>, <tt>[[capabilities(7)]]</tt>, <tt>[[cpuset(7)]]</tt>, <tt>[[sched(7)]]</tt>

----

2017-09-15
