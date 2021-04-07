## PROLOG

이 매뉴얼 페이지는 POSIX 프로그래머 매뉴얼의 일부이다. 이 인터페이스의 리눅스 구현에 차이가 있을 수 있으며 (상세한 리눅스 동작 방식은 해당 리눅스 매뉴얼 페이지 참고) 리눅스에서 이 인터페이스가 구현되어 있지 않을 수도 있다.

## NAME

pthread_cond_broadcast, pthread_cond_signal - 조건을 동보하거나 알리기

## SYNOPSIS

```c
#include <pthread.h>

int pthread_cond_broadcast(pthread_cond_t *cond);
int pthread_cond_signal(pthread_cond_t *cond);
```

## DESCRIPTION

이 함수들은 조건 변수에 블록 되어 있는 스레드를 풀어 준다.

`pthread_cond_broadcast()` 함수는 지정한 조건 변수 `cond`에 현재 블록 되어 있는 모든 스레드를 풀어 준다.

`pthread_cond_signal()` 함수는 지정한 조건 변수 `cond`에 블록 되어 있는 스레드들 중 최소 하나를 (그런 스레드가 있다면) 풀어 준다.

조건 변수에 여러 스레드가 블록 되어 있으면 스케줄링 정책에 따라 스레드의 블록이 풀리는 순서가 정해진다. `pthread_cond_broadcast()` 내지 `pthread_cond_signal()`로 인해 블록이 풀리는 각 스레드가 `pthread_cond_wait()` 내지 `pthread_cond_timedwait()` 호출에서 반환할 때 그 스레드가 `pthread_cond_wait()` 내지 `pthread_cond_timedwait()` 호출 시 사용했던 뮤텍스를 소유하게 된다. 블록에서 풀리는 스레드(들)은 (적용 가능하다면) 스케줄링 정책에 따라서 마치 각각이 `pthread_mutex_lock()`을 호출한 것처럼 뮤텍스를 두고 경쟁한다.

`pthread_cond_wait()`이나 `pthread_cond_timedwait()`을 호출한 스레드가 대기 동안 조건 변수에 연계시켜 둔 뮤텍스를 현재 소유하고 있는지와 상관없이 스레드에서 `pthread_cond_broadcast()`나 `pthread_cond_signal()` 함수를 호출할 수 있다. 하지만 예측 가능한 스케줄링 동작이 필요한 경우에는 `pthread_cond_broadcast()`나 `pthread_cond_signal()`을 호출하는 스레드에서 그 뮤텍스를 잠그게 된다.

현재 `cond`에 블록 되어 있는 스레드가 없으면 `pthread_cond_broadcast()` 및 `pthread_cond_signal()` 함수의 효과가 없다.

`pthread_cond_broadcast()`나 `pthread_cond_signal()`의 `cond` 인자로 지정한 값이 초기화 된 조건 변수를 가리키고 있지 않은 경우의 동작 방식은 규정되어 있지 않다.

## RETURN VALUE

성공 시 `pthread_cond_broadcast()`와 `pthread_cond_signal()` 함수는 0을 반환한다. 아니면 오류를 나타내는 오류 번호를 반환한다.

## ERRORS

이 함수들은 오류 코드 `[EINTR]`을 반환하지 않는다.

<em>이하는 규범적이지 않은 내용이다.</em>

## EXAMPLES

없음.

## APPLICATION USAGE

여러 스레드가 자기 작업을 진행할 수 있는 방식으로 공유 변수 상태가 바뀌었을 때는 항상 `pthread_cond_broadcast()` 함수를 사용한다. 단일 생산자/다중 소비자 문제 한 가지를 생각해 보자. 생산자는 리스트에 여러 항목을 집어넣을 수 있고 소비자는 한번에 한 항목씩 접근한다. 생산자가 `pthread_cond_broadcast()` 함수를 호출하여 대기 중일 수 있는 모든 소비자에게 알림을 보낼 수 있으며, 그러면 다중 프로세서 상에서 응용이 높은 스루풋을 얻게 된다. 또한 `pthread_cond_broadcast()`는 읽기-쓰기 락 구현을 쉽게 만들어 준다. 쓰기 쪽에서 락을 놓을 때 대기 중인 읽기 쪽 모두를 깨우려면 `pthread_cond_broadcast()` 함수가 필요하다. 마지막으로 2단계 커밋 알고리듬에서 이 동보 함수를 사용해서 임박한 트랜잭션 커밋의 모든 클라이언트에게 알림을 보낼 수 있다.

비동기적으로 호출되는 시그널 핸들러에서 `pthread_cond_signal()` 함수를 사용하는 것은 안전하지 않다. 설령 안전하더라도 `pthread_cond_wait()`의 불리언 검사와의 사이에 효율적으로 제거할 수 없는 경쟁이 있을 것이다.

따라서 뮤텍스와 조건 변수는 시그널 핸들러에서 도는 코드에서 신호를 주어 대기 중인 스레드를 해제하기에 적합하지 않다.

## RATIONALE

`pthread_cond_broadcast()`나 `pthread_cond_signal()`의 `cond` 인자로 지정한 값이 초기화 된 조건 변수를 가리키고 있지 않음을 구현에서 감지하는 경우 함수를 실패 처리하고 `[EINVAL]` 오류를 보고하기를 권장한다.

### 조건 신호가 여럿을 깨우기

`pthread_cond_signal()` 구현 시 다중 프로세서 상에서 조건 변수에 블록 되어 있는 스레드를 여러 개 풀어 주는 것을 피하기가 불가능할 수도 있다. 예를 들어 다음과 같은 `pthread_cond_wait()` 및 `pthread_cond_signal()` 부분 구현이 주어진 순서로 두 스레드에서 실행된다고 생각해 보자. 한 스레드가 조건 변수에서 대기하려 시도하고 다른 스레드는 동시에 `pthread_cond_signal()`을 실행하는데, 그때 세 번째 스레드가 이미 대기 중이다.

```
pthread_cond_wait(mutex, cond):
    value = cond->value; /* 1 */
    pthread_mutex_unlock(mutex); /* 2 */
    pthread_mutex_lock(cond->mutex); /* 10 */
    if (value == cond->value) { /* 11 */
        me->next_cond = cond->waiter;
        cond->waiter = me;
        pthread_mutex_unlock(cond->mutex);
        unable_to_run(me);
    } else
        pthread_mutex_unlock(cond->mutex); /* 12 */
    pthread_mutex_lock(mutex); /* 13 */

pthread_cond_signal(cond):
    pthread_mutex_lock(cond->mutex); /* 3 */
    cond->value++; /* 4 */
    if (cond->waiter) { /* 5 */
        sleeper = cond->waiter; /* 6 */
        cond->waiter = sleeper->next_cond; /* 7 */
        able_to_run(sleeper); /* 8 */
    }
    pthread_mutex_unlock(cond->mutex); /* 9 */
```

결과는 `pthread_cond_signal()` 호출 한 번에 의해 여러 스레드가 `pthread_cond_wait()` 내지 `pthread_cond_timedwait()` 호출에서 반환할 수 있다는 것이다. 이를 "거짓 깨어남(spurious wakeup)"라고 한다. 참고로 그렇게 깨어나는 스레드 수가 유한하므로 그 상황은 알아서 바로잡힌다. 예를 들어 위의 실행 순서 후에 다음으로 `pthread_cond_wait()`을 호출하는 스레드는 블록 한다.

이 문제를 해결할 수도 있겠지만 드물게만 발생하는 주변적 조건 때문에 효율성을 잃는 것이 수용 가능하지 않으며, 어차피 조건 변수에 연관된 술어를 검사해야 한다면 특히 그렇다. 이 문제를 고치는 것이 모든 상위 동기화 동작들의 기초가 되는 이 구성 요소의 동시성 수준을 불필요하게 감소시키게 될 것이다.

거짓 깨어남을 허용할 때의 추가 이득은 응용에서 조건 대기를 감싸는 술어 검사 루프 코드를 작성하도록 강제한다는 점이다. 이렇게 하면 응용의 어떤 다른 부분에 코딩 되어 있을 수 있는 동일 조건 변수에 대한 불필요한 조건 동보나 신호를 응용이 용인할 수 있게 되기도 한다. 그 결과 응용이 더 견고해진다. 그래서 POSIX.1-2008에서는 거짓 깨어나기가 발생할 수 있다고 명시적으로 적고 있다.

## FUTURE DIRECTIONS

없음.

## SEE ALSO

[[pthread_cond_destroy()|pthread_cond_destroy(3p)]], [[pthread_cond_timedwait()|pthread_cond_timedwait(3p)]]

POSIX.1-2008 Base Definitions 권, <em>4.11절 Memory Synchronization</em>, `<pthread.h>`

## COPYRIGHT

Portions of this text are reprinted and reproduced in electronic form from IEEE Std 1003.1, 2013 Edition, Standard for Information Technology -- Portable Operating System Interface (POSIX), The Open Group Base Specifications Issue 7, Copyright (C) 2013 by the Institute of Electrical and Electronics Engineers, Inc and The Open Group. (This is POSIX.1-2008 with the 2013 Technical Corrigendum 1 applied.) In the event of any discrepancy between this version and the original IEEE and The Open Group Standard, the original IEEE and The Open Group Standard is the referee document. The original Standard can be obtained online at http://www.unix.org/online.html .

Any typographical or formatting errors that appear in this page are most likely to have been introduced during the conversion of the source files to man page format. To report such errors, see https://www.kernel.org/doc/man-pages/reporting_bugs.html .

----

2013
