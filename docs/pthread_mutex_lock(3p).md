## PROLOG

이 매뉴얼 페이지는 POSIX 프로그래머 매뉴얼의 일부이다. 이 인터페이스의 리눅스 구현에 차이가 있을 수 있으며 (상세한 리눅스 동작 방식은 해당 리눅스 매뉴얼 페이지 참고) 리눅스에서 이 인터페이스가 구현되어 있지 않을 수도 있다.

## NAME

pthread_mutex_lock, pthread_mutex_trylock, pthread_mutex_unlock - 뮤텍스 잠그고 풀기

## SYNOPSIS

```c
#include <pthread.h>

int pthread_mutex_lock(pthread_mutex_t *mutex);
int pthread_mutex_trylock(pthread_mutex_t *mutex);
int pthread_mutex_unlock(pthread_mutex_t *mutex);
```

## DESCRIPTION

`pthread_mutex_lock()` 호출은 `mutex`가 가리키는 뮤텍스 객체를 잠그고 0이나 `[EOWNERDEAD]`를 반환한다. 뮤텍스가 이미 다른 스레드에 의해 잠겨 있으면 뮤텍스가 사용 가능해질 때까지 호출 스레드가 블록 한다. 이 동작이 반환할 때 `mutex`가 가리키는 뮤텍스 객체는 호출 스레드를 소유자로 해서 잠긴 상태이다. 이미 잠긴 뮤텍스를 스레드에서 다시 잠그려고 하면 `pthread_mutex_lock()`이 다음 표의 *다시 잠그기* 열에 나온 대로 동작한다. 잠근 적이 없거나 잠금을 푼 뮤텍스를 스레드에서 풀려고 하면 `pthread_mutex_unlock()`이 다음 표의 *소유자 아닐 때 풀기* 열에 나온 대로 동작한다.

| 뮤텍스 유형 | 견고성    | 다시 잠그기         | 소유자 아닐 때 풀기 |
| ----------- | --------- | ------------------- | ------------------- |
| NORMAL      | 비견고    | 교착                | 비규정 동작         |
| NORMAL      | 견고      | 교착                | 오류 반환           |
| ERRORCHECK  | 어느 한쪽 | 오류 반환           | 오류 반환           |
| RECURSIVE   | 어느 한쪽 | 재귀 (아래 참고)    | 오류 반환           |
| DEFAULT     | 비견고    | 비규정 동작&dagger; | 비규정 동작&dagger; |
| DEFAULT     | 견고      | 비규정 동작&dagger; | 오류 반환           |

&dagger; 뮤텍스 유형이 `PTHREAD_MUTEX_DEFAULT`인 경우 `pthread_mutex_lock()`의 동작 방식이 위 표에 기술된 다른 표준 뮤텍스 유형들 중 하나에 대응할 수도 있다. 그 셋 중 하나에 대응하지 않으면 &dagger; 표시가 된 경우에서 동작 방식이 규정되어 있지 않다.

표에 재귀적 동작이라고 된 경우에는 뮤텍스에 락 카운트 개념을 유지한다. 스레드에서 성공적으로 뮤텍스를 처음 획득할 때 락 카운트를 1로 설정한다. 스레드에서 그 뮤텍스를 다시 잠글 때마다 락 카운트가 1씩 올라간다. 스레드에서 뮤텍스를 풀 때마다 락 카운트가 1씩 내려간다. 락 카운트가 0에 도달하면 다른 스레드에서 그 뮤텍스를 획득할 수 있게 된다.

`pthread_mutex_trylock()` 함수는 `pthread_mutex_lock()`과 동등하되 `mutex`가 가리키는 뮤텍스 객체가 현재 (현 스레드 포함 어느 스레드에 의해서든) 잠겨 있으면 호출이 즉시 반환한다. 뮤텍스 유형이 `PTHREAD_MUTEX_RECURSIVE`이고 뮤텍스를 현재 호출 스레드가 소유하고 있으면 뮤텍스 카운트가 1만큼 올라가고 `pthread_mutex_trylock()` 함수가 즉시 성공을 반환한다.

`pthread_mutex_unlock()` 함수는 `mutex`가 가리키는 뮤텍스 객체를 놓는다. 뮤텍스를 놓는 방식은 뮤텍스 유형 속성에 따라 다르다. `pthread_mutex_unlock()`가 호출되어 뮤텍스가 사용 가능해질 때 `mutex`가 가리키는 뮤텍스 객체에 블록 된 스레드들이 있으면 스케줄링 정책에 따라 그 뮤텍스를 획득할 스레드가 정해진다.

(`PTHREAD_MUTEX_RECURSIVE` 뮤텍스인 경우에는 카운트가 0에 도달하여 호출 스레드가 그 뮤텍스에 더는 락을 잡고 있지 않을 때 뮤텍스가 사용 가능하게 된다.)

뮤텍스에 대기 중인 스레드에게 시그널이 전달되는 경우 시그널 핸들러에서 반환한 스레드가 중단된 적 없는 것처럼 다시 뮤텍스에 대기한다.

`mutex`가 견고 뮤텍스이고 뮤텍스 락을 잡고 있는 동안 소유 스레드가 포함된 프로세스가 종료했으면 `pthread_mutex_lock()` 호출이 오류 값 `[EOWNERDEAD]`를 반환한다. `mutex`가 견고 뮤텍스이고 뮤텍스 락을 잡고 있는 동안 소유 스레드가 종료했으면 소유 스레드가 속한 프로세스가 종료하지 않았더라도 `pthread_mutex_lock()`이 오류 값 `[EOWNERDEAD]`를 반환할 수 있다. 이 경우 그 스레드에 의해 뮤텍스가 잠기지만 뮤텍스가 보호하는 상태가 비일관이라고 표시된다. 재사용을 위해선 응용에서 그 상태를 정상으로 만들어야 하며, 완료됐을 때 `pthread_mutex_consistent()`를 호출하면 된다. 응용에서 상태를 복구할 수 없으면 `pthread_mutex_consistent()` 호출 없이 뮤텍스를 풀어야 하며, 그러면 그 뮤텍스는 영구히 사용 불가능한 것으로 표시된다.

`mutex`가 초기화 된 뮤텍스 객체를 가리키고 있지 않은 경우 `pthread_mutex_lock()`, `pthread_mutex_trylock()`, `pthread_mutex_unlock()`의 동작 방식은 규정되어 있지 않다.

## RETURN VALUE

성공 시 `pthread_mutex_lock()`, `pthread_mutex_trylock()`, `pthread_mutex_unlock()` 함수는 0을 반환한다. 아니면 오류를 나타내는 오류 번호를 반환한다.

## ERRORS

다음 경우에 `pthread_mutex_lock()` 및 `pthread_mutex_trylock()` 함수가 실패한다.

`EAGAIN`
:   `mutex`의 재귀 락 최대 횟수를 초과했기 때문에 뮤텍스를 획득할 수 없다.

`EINVAL`
:   프로토콜 속성 값을 `PTHREAD_PRIO_PROTECT`로 해서 `mutex`를 생성했으며 호출 스레드의 우선순위가 뮤텍스의 현재 우선순위 상한보다 높다.

`ENOTRECOVERABLE`
:   뮤텍스가 보호하는 상태가 복구 가능하지 않다.

`EOWNERDEAD`
:   뮤텍스가 견고 뮤텍스이며 이전 소유자 스레드를 포함한 프로세스가 뮤텍스 락을 잡은 채로 종료했다. 호출 스레드가 뮤텍스 락을 획득하게 되며 상태를 정상으로 만드는 것은 새 소유자의 몫이다.

다음 경우에 `pthread_mutex_lock()` 함수가 실패한다.

`EDEADLK`
:   뮤텍스 유형이 `PTHREAD_MUTEX_ERRORCHECK`이며 현재 스레드가 이미 그 뮤텍스를 소유하고 있다.

다음 경우에 `pthread_mutex_trylock()` 함수가 실패한다.

`EBUSY`
:   `mutex`가 이미 잠겨 있어서 획득할 수 없다.

다음 경우에 `pthread_mutex_unlock()` 함수가 실패한다.

`EPERM`
:   뮤텍스 유형이 `PTHREAD_MUTEX_ERRORCHECK`나 `PTHREAD_MUTEX_RECURSIVE`이거나 뮤텍스가 견고 뮤텍스이며 현재 스레드가 그 뮤텍스를 소유하고 있지 않다.

다음 경우에 `pthread_mutex_lock()` 및 `pthread_mutex_trylock()` 함수가 실패할 수도 있다.

`EOWNERDEAD`
:   뮤텍스가 견고 뮤텍스이며 이전 소유자 스레드가 뮤텍스 락을 잡은 채로 종료했다. 호출 스레드가 뮤텍스 락을 획득하게 되며 상태를 정상으로 만드는 것은 새 소유자의 몫이다.

다음 경우에 `pthread_mutex_lock()` 함수가 실패할 수도 있다.

`EDEADLK`
:   교착 조건을 탐지했다.

이 함수들은 오류 코드 `[EINTR]`을 반환하지 않는다.

*이하는 규범적이지 않은 내용이다.*

## EXAMPLES

없음.

## APPLICATION USAGE

0 아닌 반환 값이 오류라고 가정했던 응용에서 견고 뮤텍스를 쓰려면 변경이 필요할 것이다. 현재 비일관인 상태를 보호하는 뮤텍스를 스레드가 얻을 때 `[EOWNERDEAD]`가 유효 반환이기 때문이다. 그런 오류가 발생할 가능성을 배제하기에 오류 반환을 검사하지 않은 응용에서는 견고 뮤텍스를 사용하지 말아야 한다. 응용에서 일반 뮤텍스와 견고 뮤텍스 모두를 사용할 수 있어야 한다면 모든 반환 값의 오류 조건을 검사하고 필요시 적절한 동작을 취해야 한다.

## RATIONALE

뮤텍스 객체는 다른 스레드 동기화 함수를 만들기 위한 저수준 요소로 쓰기 위한 것이다. 그러므로 뮤텍스 구현이 가급적 효율적이어야 하며 이는 인터페이스에서 사용 가능한 기능에 영향을 준다.

뮤텍스 함수들과 뮤텍스 속성의 구체적 기본 설정들에는 빠르고 자체 할당 방식인 뮤텍스 잠그기 및 해제 구현을 배제하지 않으려는 바람이 근거가 되었다.

대부분의 속성들은 스레드가 블록 하려 할 때에만 검사하면 되므로 속성 사용 때문에 (빈번한) 뮤텍스 잠그기 경우가 느려지지 않는다.

마찬가지로 뮤텍스 소유자의 스레드 ID를 추출할 수 있다면 좋을 수도 있겠지만 그러려면 뮤텍스를 잠글 때마다 현재 스레드 ID를 저장해야 할 것이고 이는 수용 불가능한 수준의 오버헤드를 유발할 수 있다. 비슷한 논의가 `mutex_tryunlock` 동작에도 적용된다.

확장 뮤텍스 유형들에 대한 추가 근거는 POSIX.1-2008 Rationale (Informative) 권의 *Threads Extensions* 절을 보라.

`mutex` 인자로 지정한 값이 초기화 된 뮤텍스 객체를 가리키고 있지 않음을 구현에서 감지하는 경우 함수를 실패 처리하고 `[EINVAL]` 오류를 보고하기를 권장한다.

## FUTURE DIRECTIONS

없음.

## SEE ALSO

<tt>[[pthread_mutex_consistent()]]</tt>, <tt>[[pthread_mutex_destroy()]]</tt>, <tt>[[pthread_mutex_timedlock()]]</tt>, <tt>[[pthread_mutexattr_getrobust()]]</tt>

POSIX.1-2008 Base Definitions 권, *4.11절 Memory Synchronization*, `<pthread.h>`

## COPYRIGHT

Portions of this text are reprinted and reproduced in electronic form from IEEE Std 1003.1, 2013 Edition, Standard for Information Technology -- Portable Operating System Interface (POSIX), The Open Group Base Specifications Issue 7, Copyright (C) 2013 by the Institute of Electrical and Electronics Engineers, Inc and The Open Group. (This is POSIX.1-2008 with the 2013 Technical Corrigendum 1 applied.) In the event of any discrepancy between this version and the original IEEE and The Open Group Standard, the original IEEE and The Open Group Standard is the referee document. The original Standard can be obtained online at <http://www.unix.org/online.html>.

Any typographical or formatting errors that appear in this page are most likely to have been introduced during the conversion of the source files to man page format. To report such errors, see <https://www.kernel.org/doc/man-pages/reporting_bugs.html>.

----

2013
