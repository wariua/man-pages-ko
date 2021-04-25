## NAME

pthread_kill_other_threads_np - 프로세스 내의 다른 모든 스레드들 종료시키기

## SYNOPSIS

```c
#include <pthread.h>

void pthread_kill_other_threads_np(void);
```

## DESCRIPTION

`pthread_kill_other_threads_np()`는 LinuxThreads 스레딩 구현에서만 효력이 있다. 그 구현에서 이 함수를 호출하면 호출 스레드를 제외한 응용 내 모든 스레드들이 즉시 종료된다. 종료되는 스레드들의 취소 상태와 취소 유형이 무시되며 그 스레드에서 정리 핸들러들이 호출되지 않는다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `pthread_kill_other_threads_np()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

이 함수들은 비표준 GNU 확장이다. 그래서 이름 뒤에 "\_np"(nonportable: 이식성 없음)가 붙어 있다.

## NOTES

`pthread_kill_other_threads_np()`는 스레드가 <tt>[[execve(2)]]</tt>나 유사 함수를 호출하기 바로 전에 호출하기 위한 것이다. 이 함수는 (POSIX.1-2001에 요구하는 대로) <tt>[[execve(2)]]</tt> 과정에서 응용의 다른 스레드들이 자동으로 종료되지 않는 구식 LinuxThreads 구현의 한계에 대처하기 위해 설계된 것이다.

NPTL 스레딩 구현에는 `pthread_kill_other_threads_np()`가 존재하기는 하지만 아무것도 하지 않는다. (구현이 <tt>[[execve(2)]]</tt> 과정에서 제대로 동작하므로 아무것도 할 필요가 없다.)

## SEE ALSO

<tt>[[execve(2)]]</tt>, <tt>[[pthread_cancel(3)]]</tt>, <tt>[[pthread_setcancelstate(3)]]</tt>, <tt>[[pthread_etcanceltype(3)]]</tt>, <tt>[[pthreads(7)]]</tt>

----

2021-03-22
