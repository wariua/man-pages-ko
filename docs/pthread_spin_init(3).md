## NAME

pthread_spin_init, pthread_spin_destroy - 스핀락 초기화 및 파기

## SYNOPSIS

```c
#include <pthread.h>

int pthread_spin_init(pthread_spinlock_t *lock, int pshared);
int pthread_spin_destroy(pthread_spinlock_t *lock);
```

`-pthread`로 컴파일 및 링크.

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`pthread_spin_init()`, `pthread_spin_destroy()`:
:   `_POSIX_C_SOURCE >= 200112L`

## DESCRIPTION

*일반적 주의 사항*: 대부분의 프로그래머는 스핀락 대신 뮤텍스를 사용해야 한다. 스핀락은 주로 실시간 스케줄링 정책과 연관되어 쓸모가 있다. NOTES 참고.

`pthread_spin_init()` 함수는 `lock`이 가리키는 스핀락을 사용하기 위해 필요한 자원을 할당하고 그 락을 잠기지 않은 상태로 초기화 한다. `pshared` 인자는 다음 값들 중 하나여야 한다.

`PTHREAD_PROCESS_PRIVATE`
:   `pthread_spin_init()`을 호출하는 스레드와 같은 프로세스 내에 있는 스레드만 그 스핀락을 조작하게 되어 있다. (프로세스 간에 그 스핀락을 공유하려는 시도의 결과는 규정되어 있지 않다.)

`PTHREAD_PROCESS_SHARED`
:   락을 담은 메모리에 접근권이 있는 모든 프로세스의 모든 스레드가 그 스핀락을 조작할 수 있다. (즉 여러 프로세스가 공유하는 공유 메모리 객체 내에 락이 있을 수 있다.)

이미 초기화 된 스핀락에 `pthread_spin_init()`을 호출하는 결과는 규정되어 있지 않다.

`pthread_spin_destroy()` 함수는 앞서 초기화 한 스핀락을 파기하고 그 락을 위해 할당했던 자원을 해제한다. 앞서 초기화 하지 않은 스핀락을 파기하거나 다른 스레드가 락을 잡고 있는 동안 스핀락을 파기하는 결과는 규정되어 있지 않다.

스핀락을 파기하고 나서 `pthread_spin_init()`으로 다시 초기화 하는 것 외에 락에 어떤 작업이든 수행하는 결과는 규정되어 있지 않다.

`lock`이 가리키는 객체의 *사본*에 <tt>[[pthread_spin_lock(3)]]</tt>, <tt>[[pthread_spin_unlock(3)]]</tt>, `pthread_spin_destroy()` 같은 작업을 수행하는 결과는 규정되어 있지 않다.

## RETURN VALUE

성공 시 이 함수들은 0을 반환한다. 실패 시 오류 번호를 반환한다. `pthread_spin_init()`이 실패하는 경우 락이 초기화 되어 있지 않다.

## ERRORS

`pthread_spin_init()`이 다음 오류로 실패할 수 있다.

`EAGAIN`
:   시스템에 새 스핀락을 초기화 하기에 충분한 자원이 없다.

`ENOMEM`
:   스핀락을 초기화 하기에 충분한 메모리가 없다.

## VERSIONS

glibc 버전 2.2에서 이 함수들이 처음 등장했다.

## CONFORMING TO

POSIX.1-2001.

프로세스 공유 스핀락 지원은 POSIX 옵션이다. glibc 구현에서 그 옵션을 지원한다.

## NOTES

스핀락은 실시간 스케줄링 정책(`SCHED_FIFO`, 그리고 아마 `SCHED_RR`)과 함께 사용해야 한다. 스핀락을 `SCHED_OTHER` 같은 비결정적 스케줄링 정책과 함께 쓰는 것은 아마도 설계 실수일 것이다. 그런 정책 하에서 동작하는 스레드가 스핀락을 잡고 있는 동안 스케줄에서 제외되면 다시 스케줄 되어 락을 놓을 때까지 다른 스레드들이 락에 맴돌면서 시간을 낭비하게 된다.

스핀락을 사용하는 동안 스레드들이 교착 상황을 만들면 그 스레드들이 영원히 맴돌며 CPU 시간을 소모하게 된다.

사용자 공간 스핀락은 범용 락킹 해법으로 적용 가능하지 *않다*. 그 정의상 우선순위 역전과 제한 없는 맴돌기 시간을 유발하기 쉽다. 스핀락을 쓰려는 프로그래머는 코드뿐만 아니라 시스템 구성, 스레드 배치, 우선순위 부여 측면에서도 특별히 주의를 기울여야 한다.

## SEE ALSO

<tt>[[pthread_mutex_init(3)]]</tt>, <tt>[[pthread_mutex_lock(3)]]</tt>, <tt>[[pthread_spin_lock(3)]]</tt>, <tt>[[pthread_spin_unlock(3)]]</tt>, <tt>[[pthreads(7)]]</tt>

----

2017-09-30
