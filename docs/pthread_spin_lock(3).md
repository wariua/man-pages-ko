## NAME

pthread_spin_lock, pthread_spin_trylock, pthread_spin_unlock - 스핀락 잠그기 및 풀기

## SYNOPSIS

```c
#include <pthread.h>

int pthread_spin_lock(pthread_spinlock_t *lock);
int pthread_spin_trylock(pthread_spinlock_t *lock);
int pthread_spin_unlock(pthread_spinlock_t *lock);
```

`-pthread`로 컴파일 및 링크.

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`pthread_spin_lock()`, `pthread_spin_trylock()`:
:   `_POSIX_C_SOURCE >= 200112L`

## DESCRIPTION

`pthread_spin_lock()` 함수는 `lock`이 가리키는 스핀락을 잠근다. 스핀락이 현재 잠겨 있지 않으면 호출 스레드가 즉시 락을 획득한다. 스핀락이 현재 다른 스레드에 의해 잠겨 있으면 호출 스레드가 맴돌면서 락을 검사하며, 락이 사용 가능해지는 시점에 호출 스레드가 락을 획득한다.

호출자가 이미 잡고 있는 락이나 <tt>[[pthread_spin_init(3)]]</tt>으로 초기화 하지 않은 락에 `pthread_spin_lock()`을 호출하는 결과는 규정되어 있지 않다.

`pthread_spin_trylock()` 함수는 `pthread_spin_lock()`과 비슷하되 `lock`이 가리키는 스핀락이 현재 잠겨 있으면 맴돌기를 하지 않고 호출이 즉시 `EBUSY` 오류로 반환한다.

`pthread_spin_unlock()` 함수는 `lock`이 가리키는 스핀락을 푼다. 그 락에 맴돌고 있는 스레드가 있으면 그 중 한 스레드가 락을 획득하게 된다.

호출자가 잡고 있지 않은 락에 `pthread_spin_unlock()`을 호출하는 결과는 규정되어 있지 않다.

## RETURN VALUE

성공 시 이 함수들은 0을 반환한다. 실패 시 오류 번호를 반환한다.

## ERRORS

`pthread_spin_lock()`이 다음 오류로 실패할 수 있다.

`EDEADLOCK`
:   시스템이 교착 조건을 탐지했다.

`pthread_spin_trylock()`이 다음 오류로 실패할 수 있다.

`EBUSY`
:   스핀락이 현재 다른 스레드에 의해 잠겨 있다.

## VERSIONS

glibc 버전 2.2에서 이 함수들이 처음 등장했다.

## CONFORMING TO

POSIX.1-2001.

## NOTES

이 페이지에서 기술하는 함수들을 초기화 되어 있지 않은 락에 적용하는 결과는 규정되어 있지 않다.

<tt>[[pthread_spin_init(3)]]</tt>의 NOTES를 주의 깊게 읽어 보라.

## SEE ALSO

<tt>[[pthread_spin_destroy(3)]]</tt>, <tt>[[pthread_spin_init(3)]]</tt>, <tt>[[pthreads(7)]]</tt>

----

2017-09-30
