## NAME

pthread_sigqueue - 스레드에게 시그널과 데이터 큐잉 하기

## SYNOPSIS

```c
#include <signal.h>
#include <pthread.h>

int pthread_sigqueue(pthread_t thread, int sig,
                     const union sigval value);
```

`-pthread`로 컴파일 및 링크.

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`pthread_sigqueue()`:
:   `_GNU_SOURCE`

## DESCRIPTION

`pthread_sigqueue()` 함수는 <tt>[[sigqueue(3)]]</tt>와 비슷한 일을 수행하되 프로세스에게 시그널을 보내는 게 아니라 호출 스레드와 같은 프로세스 내의 스레드에게 시그널을 보낸다.

`thread` 인자는 호출자와 같은 프로세스 안에 있는 스레드의 ID이다. `sig` 인자는 보낼 시그널을 나타낸다. `value` 인자는 시그널에 동반시킬 데이터를 나타낸다. 자세한 내용은 <tt>[[sigqueue(3)]]</tt>를 보라.

## RETURN VALUE

성공 시 `pthread_sigqueue()`는 0을 반환한다. 오류 시 오류 번호를 반환한다.

## ERRORS

`EAGAIN`
:   큐에 넣을 수 있는 시그널 개수 한계에 도달했다. (자세한 내용은 <tt>[[signal(7)]]</tt>을 보라.)

`EINVAL`
:   `sig`가 유효하지 않다.

`ENOSYS`
:   이 시스템에서 `pthread_sigqueue()`를 지원하지 않는다.

`ESRCH`
:   `thread`가 유효하지 않다.

## VERSIONS

glibc 2.11에서 `pthread_sigqueue()` 함수가 처음 등장했다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `pthread_sigqueue()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

이 함수는 GNU 확장이다.

## NOTES

glibc의 `pthread_sigqueue()` 구현은 NPTL 스레딩 구현 내부에서 쓰는 실시간 시그널들 중 하나를 보내려고 시도하면 오류(`EINVAL`)를 내놓는다. 자세한 내용은 <tt>[[nptl(7)]]</tt>을 보라.

## SEE ALSO

<tt>[[rt_tgsigqueueinfo(2)]]</tt>, <tt>[[sigaction(2)]]</tt>, <tt>[[pthread_sigmask(3)]]</tt>, <tt>[[sigqueue(3)]]</tt>, <tt>[[sigwait(3)]]</tt>, <tt>[[pthreads(7)]]</tt>, <tt>[[signal(7)]]</tt>

----

2021-03-22
