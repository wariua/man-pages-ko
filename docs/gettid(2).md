## NAME

gettid - 스레드 식별자 얻기

## SYNOPSIS

```c
#include <sys/types.h>

pid_t gettid(void);
```

## DESCRIPTION

`gettid()`는 호출자의 스레드 ID(TID)를 반환한다. 단일 스레드인 프로세스에서 스레드 ID는 프로세스 ID(<tt>[[getpid(2)]]</tt>가 반환하는 PID)와 같다. 다중 스레드인 프로세스에서 모든 스레드는 PID가 같지만 각각 유일한 TID를 가진다. 더 자세한 내용은 <tt>[[clone(2)]]</tt>의 `CLONE_THREAD` 논의를 보라.

## RETURN VALUE

성공 시 호출 스레드의 스레드 ID를 반환한다.

## ERRORS

이 호출은 항상 성공이다.

## VERSIONS

리눅스 커널 2.4.11에서 `gettid()` 시스템 호출이 처음 등장했다. glibc 2.30에서 라이브러리 지원이 추가되었다. (그 전 glibc 버전에서는 이 시스템 호출의 래퍼를 제공하지 않아서 <tt>[[syscall(2)]]</tt>을 써야 했다.)

## CONFORMING TO

`gettid()`는 리눅스 전용이므로 이식성이 있어야 하는 프로그램에서는 사용하지 말아야 한다.

## NOTES

이 호출이 반환하는 스레드 ID는 POSIX 스레드 ID와 (즉 <tt>[[pthread_self(3)]]</tt>가 반환하는 불투명한 값과) 같은 것이 아니다.

`CLONE_THREAD` 플래그를 지정하지 않은 <tt>[[clone(2)]]</tt> 호출로 생성한 새 스레드 그룹에서 (또는 그와 동등하게, <tt>[[fork(2)]]</tt>로 생성한 새 프로세스에서) 새 프로세스는 스레드 그룹 리더이며 그 스레드 그룹 ID가 (즉 <tt>[[getpid(2)]]</tt>가 반환하는 값이) 그 스레드 ID와 (즉 `gettid()`가 반환하는 값과) 같다.

## SEE ALSO

<tt>[[capget(2)]]</tt>, <tt>[[clone(2)]]</tt>, <tt>[[fcntl(2)]]</tt>, <tt>[[fork(2)]]</tt>, <tt>[[getpid(2)]]</tt>, <tt>[[get_robust_list(2)]]</tt>, <tt>[[ioprio_set(2)]]</tt>, <tt>[[perm_event_open(2)]]</tt>, <tt>[[sched_setaffinity(2)]]</tt>, <tt>[[sched_setparam(2)]]</tt>, <tt>[[sched_setscheduler(2)]]</tt>, <tt>[[tgkill(2)]]</tt>, <tt>[[timer_create(2)]]</tt>

----

2019-03-06
