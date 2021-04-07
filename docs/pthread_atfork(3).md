## NAME

pthread_atfork - 분기 핸들러 등록하기

## SYNOPSIS

```c
#include <pthread.h>

int pthread_atfork(void (*prepare)(void), void (*parent)(void),
                   void (*child)(void));
```

`-pthread`로 링크.

## DESCRIPTION

`pthread_atfork()` 함수는 이 스레드에서 <tt>[[fork(2)]]</tt>를 호출할 때 실행될 분기 핸들러들을 등록한다. <tt>[[fork(2)]]</tt>를 호출하는 스레드의 문맥에서 핸들러들이 실행된다.

세 가지 종류의 핸들러를 등록할 수 있다.

* `prepare`는 <tt>[[fork(2)]]</tt> 처리가 시작되기 전에 실행되는 핸들러를 나타낸다.

* `parent`는 <tt>[[fork(2)]]</tt> 처리가 끝난 후 부모 프로세스에서 실행되는 핸들러를 나타낸다.

* `child`는 <tt>[[fork(2)]]</tt> 처리가 끝난 후 자식 프로세스에서 실행되는 핸들러를 나타낸다.

해당 <tt>[[fork(2)]]</tt> 처리 단계에 핸들러가 필요치 않으면 세 인자 중 어느 것이든 NULL일 수 있다.

## RETURN VALUE

성공 시 `pthread_atfork()`는 0을 반환한다. 오류 시 오류 번호를 반환한다. 한 스레드가 `pthread_atfork()`를 여러 번 호출하여 각 단계에 핸들러를 여러 개 등록할 수도 있다. 각 단계의 핸들러들은 정해진 순서대로 호출되는데, `prepare` 핸들러는 등록 역순으로 호출되고 `parent` 및 `child` 핸들러는 등록한 순서대로 호출된다.

## ERRORS

<dl>
<dt><code>ENOMEM</code></dt>
<dd>핸들러 항목을 기록하기 위한 메모리를 할당할 수 없다.</dd>
</dl>

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## NOTES

다중 스레드 프로세스에서 <tt>[[fork(2)]]</tt>를 호출하면 호출 스레드만 자식 프로세스에 복제된다. `pthread_atfork()`의 원래 의도는 호출 스레드가 일관성 있는 상태로 돌아갈 수 있게 하는 것이었다. 예를 들어 자식으로 복제되는 사용자 공간 메모리에서 보이는 어떤 뮤텍스를 <tt>[[fork(2)]]</tt> 호출 시점에 어떤 스레드가 잠근 상태일 수 있다. 잠근 스레드가 자식으로 복제되지 않으므로 그 뮤텍스는 절대 풀리지 않을 것이다. `pthread_atfork()`의 의도는 응용(또는 라이브러리)에서 뮤텍스 및 기타 프로세스 및 스레드 상태를 일관성 있는 상태로 복원시킬 수 있는 메커니즘을 제공하는 것이었다. 하지만 현실에서는 일반적으로 그 작업이 너무 어려워서 실행 가능하지 않다.

다중 스레드 프로세스에서 <tt>[[fork(2)]]</tt>가 자식에서 반환한 후에 <tt>[[execve(2)]]</tt>를 호출해서 새 프로그램을 실행하거나 할 때까지 자식에서는 비동기 시그널 안전 함수(<tt>[[signal-safety(7)]]</tt> 참고)만 호출해야 한다.

POSIX.1에서는 `pthread_atfork()`가 `EINTR` 오류로 실패하지 않는다고 명세한다.

## SEE ALSO

<tt>[[fork(2)]]</tt>, <tt>[[atexit(3)]]</tt>, <tt>[[pthreads(7)]]</tt>

----

2017-09-15
