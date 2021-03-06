## PROLOG

이 매뉴얼 페이지는 POSIX 프로그래머 매뉴얼의 일부이다. 이 인터페이스의 리눅스 구현에 차이가 있을 수 있으며 (상세한 리눅스 동작 방식은 해당 리눅스 매뉴얼 페이지 참고) 리눅스에서 이 인터페이스가 구현되어 있지 않을 수도 있다.

## NAME

pthread_cond_timedwait, pthread_cond_wait - 조건 기다리기

## SYNOPSIS

```c
#include <pthread.h>

int pthread_cond_timedwait(pthread_cond_t *restrict cond,
    pthread_mutex_t *restrict mutex,
    const struct timespec *restrict abstime);
int pthread_cond_wait(pthread_cond_t *restrict cond,
    pthread_mutex_t *restrict mutex);
```

## DESCRIPTION

`pthread_cond_timedwait()` 및 `pthread_cond_wait()` 함수는 조건 변수에 블록 한다. 응용에서는 호출 스레드가 `mutex`를 잠근 상태로 이 함수들을 호출하도록 해야 한다. 안 그러면 (`PTHREAD_MUTEX_ERRORCHECK` 및 견고 뮤텍스에서) 오류나 (다른 뮤텍스에서) 규정 안 된 동작이 일어난다.

이 함수들은 원자적으로 `mutex`를 놓고 호출 스레드가 조건 변수 `cond`에 블록 하게 만든다. 여기서 원자적이라는 것은 "다른 스레드가 뮤텍스에 이어 조건 변수에 접근하는 것에 대해 원자적"이라는 뜻이다. 즉 블록 예정 스레드가 뮤텍스를 놓은 후 다른 쓰레드가 그 뮤텍스를 획득할 수 있는 경우에 그 스레드가 이어서 `pthread_cond_broadcast()`나 `pthread_cond_signal()`을 호출하면 블록 예정 스레드가 블록 한 후에 그 호출이 이뤄진 것처럼 동작하게 된다.

성공 반환 시 뮤텍스가 잠겨 있으며 호출 스레드가 그 뮤텍스를 소유하고 있게 된다. `mutex`가 견고 뮤텍스인데 소유자가 락을 잡은 채 종료했고 그 상태가 복구 가능하면 함수는 오류 코드를 반환하더라도 뮤텍스를 획득하게 된다.

조건 변수를 사용할 때는 항상 각 조건 대기 관련 공유 변수들이 포함된 불리언 술어가 있으며 스레드가 진행해야 하는 경우에는 그 술어가 참이다. 한편 `pthread_cond_timedwait()` 내지 `pthread_cond_wait()` 함수에서 거짓으로 깨어나는 경우가 발생할 수도 있다. `pthread_cond_timedwait()` 내지 `pthread_cond_wait()`에서 반환하는 것에는 그 술어의 값에 대해 어떤 함의도 없으므로 반환 시 술어를 재평가 해야 한다.

스레드가 `pthread_cond_timedwait()`이나 `pthread_cond_wait()`에 특정 뮤텍스를 지정하고서 조건 변수에 대기할 때 그 뮤텍스와 조건 변수 사이에는 동적 결속이 형성되며 그 조건 변수에 적어도 한 스레드가 블록 되어 있는 동안은 그 결속이 유지된다. 그 시간 동안 어느 스레드라도 다른 뮤텍스를 이용해 그 조건 변수에 대기하려 시도하는 결과는 규정되어 있지 않다. 대기 중인 스레드 모두가 (`pthread_cond_broadcast()` 동작 등에 의해) 블록이 풀리고 나면 그 조건 변수에 다음 대기 동작이 이뤄질 때 그 대기 동작에 지정된 뮤텍스로 동적 결속이 새로 형성된다. 스레드가 조건 변수 대기에서 풀리는 시점과 호출자에게 반환하거나 취소 정리를 시작하는 시점 사이에 조건 변수와 뮤텍스 간 동적 결속이 제거되거나 교체될 수 있기는 하지만 블록에서 풀린 스레드는 항상 반환 중인 조건 대기 동작 호출에 지정됐던 뮤텍스를 재획득 한다.

조건 대기는 (시한이 있든 없든) 취소점이다. 스레드의 취소 가능성 유형이 `PTHREAD_CANCEL_DEFERRED`로 설정되어 있을 때 조건 대기 중 취소 요청에 대응 시 첫 번째 취소 정리 핸들러 호출 전에 뮤텍스를 (사실상) 재획득하는 부대 효과가 있다. 스레드가 블록에서 풀려서 `pthread_cond_timedwait()` 내지 `pthread_cond_wait()` 호출에서 반환하는 지점까지 실행했다가 그 지점에서 취소 요청을 알아채고서 `pthread_cond_timedwait()` 내지 `pthread_cond_wait()` 호출자에게 반환하는 대신 취소 정리 핸들러 호출을 포함한 스레드 취소 활동을 시작하는 것 같은 효과이다.

`pthread_cond_timedwait()` 내지 `pthread_cond_wait()` 호출 내에서 블록 되어 있는 중에 취소되어 블록이 해제된 스레드는 조건 변수에 블록 된 다른 스레드가 있다면 동시에 그 조건 변수로 향해 있을 수 있는 조건 신호를 소모하지 않는다.

`pthread_cond_timedwait()` 함수는 `pthread_cond_wait()`과 동등하다. 단, 조건 `cond`에 대한 신호나 동보가 있기 전에 `abstime`으로 지정한 절대 시간이 지날 때(즉 시스템 시간이 `abstime`과 같거나 그보다 커질 때), 또는 `abstime`으로 지정한 절대 시간이 호출 시점에 이미 지나 있는 경우에 오류를 반환한다. 그런 타임아웃이 일어날 때에도 `pthread_cond_timedwait()`은 `mutex`가 가리키는 뮤텍스를 놓았다가 재획득하며, 동시에 그 조건 변수로 향해 있을 수 있는 조건 신호를 소모할 수도 있다.

조건 변수에는 클럭 속성이 있으며 `abstime` 인자로 지정하는 시간 측정에 사용할 클럭을 나타낸다. `pthread_cond_timedwait()` 함수도 취소점이다.

조건 변수에 대기 중인 스레드에게 시그널이 전달되는 경우 시그널 핸들러에서 반환한 스레드가 중단된 적 없는 것처럼 다시 조건 변수에 대기하거나, 불요 wakeup 때문에 0을 반환하게 된다.

이 함수들의 `cond`나 `mutex` 인자로 지정한 값이 각각 초기화 된 조건 변수 객체와 초기화 된 뮤텍스 객체를 가리키고 있지 않은 경우의 동작 방식은 규정되어 있지 않다.

## RETURN VALUE

`[ETIMEDOUT]` 경우를 제외하고 이 오류 검사들은 모두 함수 처리 시작 직후에 수행된 것처럼 동작하며 효과상 `mutex`로 지정한 뮤텍스나 `cond`로 지정한 조건 변수의 상태 변경 전에 오류 반환을 유발한다.

성공 완료시 0 값을 반환한다. 아니면 오류를 나타내는 오류 번호를 반환한다.

## ERRORS

다음 경우에 이 함수들이 실패한다.

`ENOTRECOVERABLE`
:   뮤텍스가 보호하는 상태가 복구 가능하지 않다.

`EOWNERDEAD`
:   뮤텍스가 견고 뮤텍스이며 이전 소유자 스레드를 포함한 프로세스가 뮤텍스 락을 잡은 채로 종료했다. 호출 스레드가 뮤텍스 락을 획득하게 되며 상태를 정상으로 만드는 것은 새 소유자의 몫이다.

`EPERM`
:   뮤텍스 유형이 `PTHREAD_MUTEX_ERRORCHECK`이거나 뮤텍스가 견고 뮤텍스이며 현재 스레드가 그 뮤텍스를 소유하고 있지 않다.

다음 경우에 `pthread_cond_timedwait()` 함수가 실패한다.

`ETIMEDOUT`
:   `pthread_cond_timedwait()`의 `abstime`으로 지정한 시간이 지났다.

`EINVAL`
:   `abstime`에 지정한 나노초 값이 0보다 작거나 10억 이상이다.

다음 경우에 이 함수들이 실패할 수도 있다.

`EOWNERDEAD`
:   뮤텍스가 견고 뮤텍스이며 이전 소유자 스레드가 뮤텍스 락을 잡은 채로 종료했다. 호출 스레드가 뮤텍스 락을 획득하게 되며 상태를 정상으로 만드는 것은 새 소유자의 몫이다.

이 함수는 오류 코드 `[EINTR]`을 반환하지 않는다.

*이하는 규범적이지 않은 내용이다.*

## EXAMPLES

없음.

## APPLICATION USAGE

0 아닌 반환 값이 오류라고 가정했던 응용에서 견고 뮤텍스를 쓰려면 변경이 필요할 것이다. 현재 비일관인 상태를 보호하는 뮤텍스를 스레드가 얻을 때 `[EOWNERDEAD]`가 유효 반환이기 때문이다. 그런 오류가 발생할 가능성을 배제하기에 오류 반환을 검사하지 않은 응용에서는 견고 뮤텍스를 사용하지 말아야 한다. 응용에서 일반 뮤텍스와 견고 뮤텍스 모두를 사용할 수 있어야 한다면 모든 반환 값의 오류 조건을 검사하고 필요시 적절한 동작을 취해야 한다.

## RATIONALE

`pthread_cond_timedwait()`이나 `pthread_cond_wait()`의 `cond` 인자로 지정한 값이 초기화 된 조건 변수를 가지기코 있지 않음을, 또는 `pthread_cond_timedwait()`이나 `pthread_cond_wait()`의 `mutex` 인자로 지정한 값이 초기화 된 뮤텍스 객체를 가리키고 있지 않음을 구현에서 감지하는 경우 함수를 실패 처리하고 `[EINVAL]` 오류를 보고하기를 권장한다.

### 조건 대기 동작 방식

`pthread_cond_wait()`과 `pthread_cond_timedwait()`이 오류 없이 반환할 때에도 연관 술어가 거짓일 수 있다는 점에 매우 유의해야 한다. 마찬가지로 `pthread_cond_timedwait()`이 타임아웃 오류로 반환할 때에 타임아웃 만료와 술부 상태 변경 간의 불가피한 경쟁 때문에 연관 술어가 참일 수 있다.

반환 시에 그 스레드가 신호를 다루는 것을 기다리고 있는 또 다른 스레드가 있는지를, 그리고 없다면 신호가 유실된 것인지를 확신할 수 없기 때문에 응용에서 술어를 다시 검사할 필요가 있다. 술어를 검사하는 부담을 응용이 진다.

일부 구현에서는 다중 프로세서 시스템의 다른 프로세서에서 조건 변수에 동시에 신호를 보내는 경우에 때로 여러 스레드가 깨어나게 될 수도 있다.

일반적으로 조건 대기에서 반환할 때마다 스레드에서 그 조건 대기와 연관된 술어를 재평가하여 안전하게 진행할 수 있는지, 아니면 다시 대기하거나 타임아웃을 지정해야 하는지 판단해야 한다. 대기에서 반환하는 것에는 연관 술어가 참이나 거짓 중 어느 한쪽이라는 함의가 없다.

따라서 술어를 검사하는 "while 루프" 등가물로 조건 대기를 감싸기를 권장한다.

### 시한 대기 동작 방식

타임아웃 매개변수 지정에 절대 시간 척도를 택한 데는 두 가지 이유가 있다. 첫째로, 절대 시간 지정 함수 상에서 상대 시간 척도를 구현하는 것은 쉽지만 상대 타임아웃 지정 함수 상에서 절대 타임아웃을 지정하는 데는 경쟁 조건이 엮인다. 예를 들어 `clock_gettime()`이 현재 시간을 반환하고 `cond_relative_timed_wait()`이 상대 타임아웃을 쓴다고 하자.

```c
clock_gettime(CLOCK_REALTIME, &now);
reltime = sleep_til_this_absolute_time -now;
cond_relative_timed_wait(c, m, &reltime);
```

첫 번째 문과 마지막 문 사이에서 스레드가 선점되면 스레드가 너무 오래 블록 한다. 하지만 절대 타임아웃을 쓰면 문제가 안 된다. 또한 절대 타임아웃은 조건 대기를 감싸는 루프 등에서 사용 시 다시 계산할 필요가 없다.

운용자가 시스템 클럭을 불연속적으로 전진시키는 경우에 구현에서는 조정 시점에 만료되는 시한 대기를 실제 그 시간을 거친 것처럼 처리할 것으로 기대한다.

### 취소와 조건 대기

조건 대기는 시한이 있든 없는 취소점이다. 즉 함수 `pthread_cond_wait()` 내지 `pthread_cond_timedwait()`가 미처리 (또는 동시의) 취소 요청을 인식할 수 있는 지점이다. 이렇게 한 이유는 그 지점에서 무한 대기가 가능하기 때문이다. 기다리는 이벤트가 무엇이든 간에 프로그램이 완벽하게 올바르다 해도 이벤트가 절대 발생하지 않을 수 있을 것이다. 예를 들어 기다리고 있는 어떤 입력 데이터가 절대 전송되지 않을 수도 있다. 조건 대기를 취소점으로 만들면 스레드가 어떤 무한 대기에 갇힐 수도 있을 때에도 스레드를 취소하고 취소 정리 핸들러를 수행하게 할 수 있게 된다.

스레드가 조건 변수에 블록 되어 있을 때 취소 요청에 대응 시 취소 정리 핸들러 호출 전에 뮤텍스를 재획득하는 부대 효과가 있다. 이렇게 하는 것은 취소 정리 핸들러가 조건 대기 함수 호출 전후의 임계 코드와 같은 상태에서 실행되도록 하기 위해서이다. Ada나 C++처럼 취소를 언어 예외로 사상하기로 할 수 있는 언어들에서 POSIX 스레드를 이용할 때에도 이 규칙이 요구된다. 이 규칙은 임계 구역을 지키는 각 예외 핸들러가 정확히 임계 구역 내 어디서 예외가 제기되었든 관련 뮤텍스가 이미 잠겨 있다는 점에 언제나 안전하게 의지할 수 있도록 해 준다. 이 규칙이 없다면 락과 관련해 예외 핸들러에서 따를 수 있을 정해진 규칙이 없을 것이고, 그래서 코딩이 매우 번거로워질 것이다.

그래서 대기 중 취소가 전달될 때의 락 상태와 관련해 *뭔가* 서술이 있어야 하므로 응용 코딩을 가급적 편리하고 오류가 적게 만드는 방식으로 규정을 선택하였다.

한 스레드가 조건 변수에 블록 되어 있는 중 취소 요청에 대응할 때 구현에서는 그 조건 변수에 대기 중인 다른 스레드가 있는 경우 그 조건 변수를 향한 조건 신호를 그 스레드가 소모하지 않도록 해야 한다. 이 규칙을 명시하는 것은 이 두 가지 독립적 요청(스레드에 대한 요청과 조건 변수에 대한 요청)을 독립적으로 처리하지 않으면 발생할 수 있을 교착 조건을 피하기 위해서이다.

### 뮤텍스와 조건 변수의 성능

인스트럭션 몇 개만으로 뮤텍스를 잠글 것으로 기대한다. 이를 거의 자동으로 현실화하는 것은 실행 구역이 길게 직렬화 되는 (그래서 총 실효 병렬성이 감소하는) 것을 피하려는 프로그래머들의 욕구이다.

뮤텍스와 조건 변수를 쓸 때는 뮤텍스를 잠그고 공유 데이터에 접근한 다음 뮤텍스를 푸는 것이 일반적인 경우가 되도록 한다. 조건 변수에 대기하는 것은 상대적으로 드문 상황이어야 한다. 예를 들어 읽기-쓰기 락을 구현할 때 읽기 락을 획득하는 코드에서는 보통 (뮤텍스 하에서) 읽기 스레드 카운트를 올리고 반환하기만 하면 된다. 이미 쓰기 쪽이 있을 때에만 호출 스레드가 실제로 조건 변수에 대기하게 된다. 따라서 동기화 동작의 효율성을 결정짓는 것은 조건 변수가 아니라 뮤텍스 잠그기/풀기의 비용이다. 참고로 일반적인 경우에서는 문맥 전환이 없다.

조건 대기의 효율성이 중요하지 않다는 말이 아니다. Ada 랑데부마다 최소 한 번의 문맥 전환이 있어야 하므로 조건 변수 대기의 효율성은 중요하다. 조건 변수에 대기하는 비용은 문맥 전환 최소 비용에 뮤텍스 풀고 잠그는 시간을 더한 것과 거의 같아야 한다.

### 뮤텍스와 조건 변수의 기능

뮤텍스 획득 및 해제를 조건 대기와 분리시키자는 제안이 있었다. 이를 거부한 이유는 그 동작의 결합된 특성이 실제로 실시간 구현을 용이하게 하기 때문이다. 그런 구현에서 호출자에게 투명한 방식으로 조건 변수와 뮤텍스 사이에서 높은 우선도의 스레드를 원자적으로 옮길 수 있다. 이는 추가적인 문맥 전환을 막아 주며 대기 스레드에 신호가 갈 때 더 결정론적으로 뮤텍스 획득이 이뤄지게 해 준다. 즉 공정성과 우선순위 문제를 스케줄링 규율로 직접 다룰 수 있다. 또한 현재의 조건 대기 동작이 기존 관행과 일치한다.

### 뮤텍스와 조건 변수의 스케줄링 동작

순서 규칙을 지정하여 스케줄링 정책에 간섭하려 하는 동기화 요소는 바람직하지 않다고 본다. 뮤텍스와 조건 변수에 대기 중인 스레드들 가운데 진행할 것을 선정하는 것은 어떤 고정된 순서(예를 들어 FIFO나 우선순위)가 아니라 스케줄링 정책이 좌우하는 어떤 순서로 이뤄진다. 즉 스케줄링 정책이 어느 스레드(들)이 깨어나서 진행할 수 있는지를 결정한다.

### 시한 조건 대기

`pthread_cond_timedwait()` 함수를 사용하면 응용에서 일정 시간 후에 특정 조건에 대기하는 것을 포기할 수 있다. 사용 예시는 다음과 같다.

```c
(void) pthread_mutex_lock(&t.mn);
    t.waiters++;
    clock_gettime(CLOCK_REALTIME, &ts);
    ts.tv_sec += 5;
    rc = 0;
    while (! mypredicate(&t) && rc == 0)
        rc = pthread_cond_timedwait(&t.cond, &t.mn, &ts);
    t.waiters--;
    if (rc == 0 || mypredicate(&t))
        setmystate(&t);
(void) pthread_mutex_unlock(&t.mn);
```

타임아웃 매개변수가 절댓값이므로 프로그램에서 진행을 막는 술어를 검사할 때마다 다시 계산할 필요가 없다. 타임아웃이 상댓값이었다면 호출 전에 매번 다시 계산해야 했을 것이다. 또한 술어가 참이 되거나 타임아웃이 만료되기 전의 조건 변수에 대한 가외 동보 내지 신호로 인한 거짓 깨어남 가능성을 그 코드에서 고려해야 했을 것이므로 특히 어려웠을 것이다.

## FUTURE DIRECTIONS

없음.

## SEE ALSO

<tt>[[pthread_cond_broadcast()]]</tt>

POSIX.1-2008 Base Definitions 권, *4.11절 Memory Synchronization*, `<pthread.h>`

## COPYRIGHT

Portions of this text are reprinted and reproduced in electronic form from IEEE Std 1003.1, 2013 Edition, Standard for Information Technology -- Portable Operating System Interface (POSIX), The Open Group Base Specifications Issue 7, Copyright (C) 2013 by the Institute of Electrical and Electronics Engineers, Inc and The Open Group. (This is POSIX.1-2008 with the 2013 Technical Corrigendum 1 applied.) In the event of any discrepancy between this version and the original IEEE and The Open Group Standard, the original IEEE and The Open Group Standard is the referee document. The original Standard can be obtained online at <http://www.unix.org/online.html>.

Any typographical or formatting errors that appear in this page are most likely to have been introduced during the conversion of the source files to man page format. To report such errors, see <https://www.kernel.org/doc/man-pages/reporting_bugs.html>.

----

2013
