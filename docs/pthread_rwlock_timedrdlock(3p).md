## PROLOG

이 매뉴얼 페이지는 POSIX 프로그래머 매뉴얼의 일부이다. 이 인터페이스의 리눅스 구현에 차이가 있을 수 있으며 (상세한 리눅스 동작 방식은 해당 리눅스 매뉴얼 페이지 참고) 리눅스에서 이 인터페이스가 구현되어 있지 않을 수도 있다.

## NAME

pthread_rwlock_timedrdlock - 읽기-쓰기 락을 읽기용으로 잠그기

## SYNOPSIS

```c
#include <pthread.h>
#include <time.h>

int pthread_rwlock_timedrdlock(pthread_rwlock_t *restrict rwlock,
    const struct timespec *restrict abstime);
```

## DESCRIPTION

`pthread_rwlock_timedrdlock()` 함수는 `pthread_rwlock_rdlock()` 함수에서처럼 `rwlock`이 가리키는 락에 읽기 락을 적용한다. 하지만 다른 스레드가 락을 푸는 걸 기다리지 않고는 락을 획득할 수 없는 경우에 지정한 타임아웃 만료 시 그 대기를 끝낸다. 타임아웃 기준 클럭으로 측정해서 `abstime`으로 지정한 절대 시간이 지날 때 (즉 그 클럭의 값이 `abstime`과 같거나 그보다 커질 때), 또는 `abstime`으로 지정한 절대 시간이 호출 시점에 이미 지나 있는 경우에 타임아웃이 만료된다.

타임아웃은 `CLOCK_REALTIME` 클럭을 기준으로 한다. 타임아웃의 해상도는 `CLOCK_REALTIME` 클럭의 해상도이다. `<time.h>` 헤더에 `timespec` 데이터 타입이 정의되어 있다. 락을 즉시 획득할 수 있으면 어떤 경우에도 타임아웃 때문에 함수가 실패하지 않는다. 락을 즉시 획득할 수 있다면 `abstime` 매개변수의 유효성을 검사할 필요가 없다.

`pthread_rwlock_timedrdlock()` 호출을 통해 읽기-쓰기 락에 블록 중인 스레드에게 시그널 핸들러 실행을 유발하는 시그널이 전달되는 경우 시그널 핸들러에서 반환한 스레드가 중단된 적 없는 것처럼 다시 락에 대기한다.

호출이 이뤄지는 시점에 호출 쓰레드가 `rwlock`에 쓰기 락을 잡고 있다면 교착이 일어날 수도 있다. 초기화 안 된 읽기-쓰기 락으로 이 함수를 호출하는 경우의 결과는 규정되어 있지 않다.

## RETURN VALUE

`pthread_rwlock_timedrdlock()` 함수는 `rwlock`이 가리키는 읽기-쓰기 락 객체에서 읽기용 락을 획득한 경우 0을 반환한다. 아니면 오류를 나타내는 오류 번호를 반환한다.

## ERRORS

다음 경우에 `pthread_rwlock_timedrdlock()` 함수가 실패한다.

<dl>
<dt><code>ETIMEDOUT</code></dt>
<dd>지정한 타임아웃 만료 전에 락을 획득할 수 없었다.</dd>
</dl>

다음 경우에 `pthread_rwlock_timedrdlock()` 함수가 실패할 수도 있다.

<dl>
<dt><code>EAGAIN</code></dt>
<dd>락에 대한 읽기 락 최대 횟수를 초과했기 때문에 읽기 락을 획득할 수 없다.</dd>
<dt><code>EDEADLK</code></dt>
<dd>교착 조건을 탐지했거나 호출 스레드가 이미 <code>rwlock</code>에 쓰기 락을 잡고 있다.</dd>
<dt><code>EINVAL</code></dt>
<dd><code>abstime</code>의 나노초 값이 0보다 작거나 10억 이상이다.</dd>
</dl>

이 함수는 오류 코드 `[EINTR]`을 반환하지 않는다.

<em>이하는 규범적이지 않은 내용이다.</em>

## EXAMPLES

없음.

## APPLICATION USAGE

이 함수를 쓰는 응용에서 POSIX.1-2008 Base Definitions 권의 <em>3.287절 Priority Inversion</em>에서 논의하는 우선순위 역전을 겪을 수 있다.

## RATIONALE

`pthread_rwlock_timedrdlock()`의 `rwlock` 인자로 지정한 값이 초기화 된 읽기-쓰기 락 객체를 가리키고 있지 않음을 구현에서 감지하는 경우 함수를 실패 처리하고 `[EINVAL]` 오류를 보고하기를 권장한다.

## FUTURE DIRECTIONS

없음.

## SEE ALSO

<tt>[[pthread_rwlock_destroy()|pthread_rwlock_destroy(3p)]]</tt>, <tt>[[pthread_rwlock_rdlock()|pthread_rwlock_rdlock(3p)]]</tt>, <tt>[[pthread_rwlock_timedwrlock()|pthread_rwlock_timedwrlock(3p)]]</tt>, <tt>[[pthread_rwlock_trywrlock()|pthread_rwlock_trywrlock(3p)]]</tt>, <tt>[[pthread_rwlock_unlock()|pthread_rwlock_unlock(3p)]]</tt>

POSIX.1-2008 Base Definitions 권, <em>3.287절 Priority Inversion</em>, <em>4.11절 Memory Synchronization</em>, `<pthread.h>`, `<time.h>`

## COPYRIGHT

Portions of this text are reprinted and reproduced in electronic form from IEEE Std 1003.1, 2013 Edition, Standard for Information Technology -- Portable Operating System Interface (POSIX), The Open Group Base Specifications Issue 7, Copyright (C) 2013 by the Institute of Electrical and Electronics Engineers, Inc and The Open Group. (This is POSIX.1-2008 with the 2013 Technical Corrigendum 1 applied.) In the event of any discrepancy between this version and the original IEEE and The Open Group Standard, the original IEEE and The Open Group Standard is the referee document. The original Standard can be obtained online at http://www.unix.org/online.html .

Any typographical or formatting errors that appear in this page are most likely to have been introduced during the conversion of the source files to man page format. To report such errors, see https://www.kernel.org/doc/man-pages/reporting_bugs.html .

----

2013
