## PROLOG

이 매뉴얼 페이지는 POSIX 프로그래머 매뉴얼의 일부이다. 이 인터페이스의 리눅스 구현에 차이가 있을 수 있으며 (상세한 리눅스 동작 방식은 해당 리눅스 매뉴얼 페이지 참고) 리눅스에서 이 인터페이스가 구현되어 있지 않을 수도 있다.

## NAME

pthread_mutex_destroy, pthread_mutex_init - 뮤텍스 파기 및 초기화

## SYNOPSIS

```c
#include <pthread.h>

int pthread_mutex_destroy(pthread_mutex_t *mutex);
int pthread_mutex_init(pthread_mutex_t *restrict mutex,
    const pthread_mutexattr_t *restrict attr);
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
```

## DESCRIPTION

`pthread_mutex_destroy()` 함수는 `mutex`가 가리키는 뮤텍스 객체를 파기한다. 그 뮤텍스 객체를 초기화 안 된 상태로 만드는 효과가 있다. 구현 시 `pthread_mutex_destroy()`에서 `mutex`가 가리키는 객체를 비유효 값으로 설정하도록 할 수도 있다.

파기된 뮤텍스 객체를 `pthread_mutex_init()`으로 다시 초기화 할 수 있다. 파기된 객체를 그 외 방식으로 참조하는 결과는 규정되어 있지 않다.

잠겨 있지 않은 초기화 된 뮤텍스를 파기하는 것은 안전하다. 잠겨 있는 뮤텍스나 다른 스레드에서 참조하는 (가령 `pthread_cond_timedwait()`이나 `pthread_cond_wait()`에 쓰이고 있는) 뮤텍스를 파기하려는 시도의 결과는 규정되어 있지 않다.

`pthread_mutex_init()` 함수는 `mutex`가 가리키는 뮤텍스를 `attr`이 가리키는 속성들로 초기화 한다. `attr`이 NULL이면 기본 뮤텍스 속성들을 사용한다. 즉 기본 뮤텍스 속성 객체의 주소를 전달하는 것과 효과가 같다. 초기화 성공 시 뮤텍스는 초기화 되고 풀려 있는 상태가 된다.

`mutex` 자체만 동기화 수행에 사용할 수 있다. `mutex`의 사본을 `pthread_mutex_lock()`, `pthread_mutex_trylock()`, `pthread_mutex_unlock()`, `pthread_mutex_destroy()` 호출에서 참조하는 결과는 규정되어 있지 않다.

이미 초기화 된 뮤텍스를 초기화 하려는 시도의 결과는 규정되어 있지 않다.

기본 뮤텍스 속성들이 적합한 경우에는 매크로 `PTHREAD_MUTEX_INITIALIZER`를 써서 뮤텍스를 초기화 할 수 있다. 매개변수 `attr`을 NULL로 지정해 `pthread_mutex_init()`을 호출하는 동적 초기화와 효과가 동등하되 오류 검사를 수행하지 않는다는 점이 다르다.

`pthread_mutex_destroy()`의 `mutex` 인자로 지정한 값이 초기화 된 뮤텍스를 가리키고 있지 않은 경우의 동작 방식은 규정되어 있지 않다.

`pthread_mutex_init()`의 `attr` 인자로 지정한 값이 초기화 된 뮤텍스 속성 객체를 가리키고 있지 않은 경우의 동작 방식은 규정되어 있지 않다.

## RETURN VALUE

성공 시 `pthread_mutex_destroy()`와 `pthread_mutex_init()` 함수는 0을 반환한다. 아니면 오류를 나타내는 오류 번호를 반환한다.

## ERRORS

다음 경우에 `pthread_mutex_init()` 함수가 실패한다.

`EAGAIN`
:   뮤텍스를 새로 초기화 하는 데 필요한 (메모리 외의) 자원이 시스템에 부족하다.

`ENOMEM`
:   뮤텍스를 초기화 하기에 충분한 메모리가 없다.

`EPERM`
:   호출자에게 동작을 수행하기 위한 특권이 없다.

다음 경우에 `pthread_mutex_init()` 함수가 실패할 수도 있다.

`EINVAL`
:  `attr`이 가리키는 속성 객체에 견고 뮤텍스 속성이 설정되어 있으면서 프로세스 공유 속성이 설정되어 있지 않다.

이 함수들은 오류 코드 `[EINTR]`을 반환하지 않는다.

*이하는 규범적이지 않은 내용이다.*

## EXAMPLES

없음.

## APPLICATION USAGE

없음.

## RATIONALE

`pthread_mutex_destroy()`의 `mutex` 인자로 지정한 값이 초기화 된 뮤텍스를 가리키고 있지 않음을 구현에서 감지하는 경우 함수를 실패 처리하고 `[EINVAL]` 오류를 보고하기를 권장한다.

`pthread_mutex_destroy()`나 `pthread_mutex_init()`의 `mutex` 인자로 지정한 값이 잠겨 있는 뮤텍스나 다른 스레드에서 참조하는 (가령 `pthread_cond_timedwait()`이나 `pthread_cond_wait()`에 사용 중인) 뮤텍스를 참조하고 있음을, 또는 `pthread_mutex_init()`의 `mutex` 인자로 지정한 값이 이미 초기화 된 뮤텍스를 가리키고 있음을 구현에서 감지하는 경우 함수를 실패 처리하고 `[EBUSY]` 오류를 보고하기를 권장한다.

`pthread_mutex_init()`의 `attr` 인자로 지정한 값이 초기화 된 뮤텍스 속성 객체를 가리키고 있지 않음을 구현에서 감지하는 경우 함수를 실패 처리하고 `[EINVAL]` 오류를 보고하기를 권장한다.

### 여러 가능한 구현

POSIX.1-2008의 이 권에선 여러 대안적인 뮤텍스 구현들을 지원한다. 구현에서 락을 `pthread_mutex_t` 타입 객체에 바로 저장할 수도 있고, 아니면 락을 힙에 저장하고 뮤텍스 객체에는 포인터나 핸들, 고유 ID만 저장할 수도 있다. 어느 한 구현 방식에 장점이 있을 수도 있고 특정 하드웨어 구성에서 어느 한쪽만 가능할 수도 있다. 이 선택에 영향을 받지 않는 이식성 있는 코드를 작성할 수 있도록 하기 위해 POSIX.1-2008의 이 권에서는 이 타입에 할당 내지 동일 연산을 정의하지 않으며, "초기화"라는 용어를 사용해서 락이 실제 뮤텍스 객체 내에 있을 수도 있다는 (더 제한적인) 개념을 강조한다.

이는 뮤텍스 내지 조건 변수 타입의 과잉 명세를 방지하며 타입 불투명성의 동기가 된다.

`pthread_mutex_destroy()`에서 뮤텍스에 부적법한 값을 저장하도록 구현하는 것이 가능하되 필수는 아니다. 그렇게 하는 것이 이미 파기된 뮤텍스를 잠그려고 (또는 달리 참조하려고) 하는 오류 있는 프로그램을 감지하는 데 도움이 될 수도 있다.

### 오류 검사와 지원 성능 간의 타협

발생 가능한 많은 오류 조건들이 구현에서의 감지가 필수가 아닌데, 구현에서 구체적인 응용 및 실행 환경의 필요에 따라 성능과 오류 검사 수준 간에 타협을 할 수 있도록 하기 위해서이다. 일반적으로 시스템에 의한 (메모리 불충분 같은) 조건들은 감지가 필수이지만 잘못 코딩 된 응용에 의한 (사용 중인 뮤텍스가 삭제되는 것을 막기 위한 적절한 동기화에 실패하는 것 같은) 조건들은 결과가 규정되어 있지 않은 것으로 명세한다.

그래서 넓은 범위의 구현이 가능하다. 예를 들어 응용 디버깅을 위한 구현에서는 오류 검사를 모두 구현할 수 있을 것이고, 증명 가능하게 올바른 응용 하나를 임베디드 컴퓨터의 아주 엄격한 성능 제약 하에서 돌리는 구현에서는 최소한의 검사만 구현할 수 있을 것이다. 또한 컴파일러에서 제공하는 옵션과 비슷하게 두 가지 버전(전체 검사를 하는 느린 버전과 제한된 검사를 하는 빠른 버전)으로 구현을 제공할 수도 있을 것이다. 이런 선택을 막는 것은 사용자에게 해가 될 것이다.

"동작이 규정되어 있지 않음"을 잘못된 (잘못 코딩 된) 응용에서 할 수 있는 동작들에만 조심스럽게 적용하고 자원 사용 불가 오류들은 필수로 정의함으로써 POSIX.1-2008의 이 권은 올바른 프로그램이라면 절대 하지 않을 많은 것들을 검사하는 추가 오버헤드를 모든 구현에 강제하지 않으면서도 완전한 준수 응용이 모든 구현들에 걸쳐 이식 가능하도록 보장한다. 동작이 규정되어 있지 않을 때는 그 오류를 감지하는 구현에서 어떤 오류 번호의 반환도 명세하지 않는다. 동작이 규정되어 있지 않다는 것은 *무슨 일이든* 일어날 수 있다는 것이고, 임의 값(유효하되 상이할 수도 있는 오류 번호)을 반환하는 것도 포함된다. 하지만 응용 개발 중 문제를 진단할 때 오류 번호가 응용 개발자에게 도움이 될 수도 있으므로 구현에서 오류 조건을 감지하는 경우 특정 오류 번호를 반환하는 게 좋다는 권고를 이 절에서 한다.

### 제한을 정의하지 않은 이유

뮤텍스 및 조건 변수의 최대 개수에 대한 심볼을 정의하는 것을 고려하였으나 이 객체들의 수가 동적으로 바뀔 수도 있기 때문에 거부하였다. 게다가 많은 구현에서는 이 객체들을 응용 메모리에 두고, 그래서 명시적 최댓값이 없다.

### 뮤텍스와 조건 변수의 정적 초기화

정적 할당 동기화 객체를 정적으로 초기화 할 수 있으면 내부에 정적 동기화 변수를 가진 모듈에서 런타임 초기화 검사와 오버헤드를 피할 수 있다. 또한 자가 초기화 모듈을 작성하는 것이 간단해진다. C 라이브러리들에 그런 모듈이 흔한데, 여러 이유 때문에 설계 과정에서 명시적인 모듈 초기화 함수 호출을 요구하는 대신 자가 초기화를 택한다. 정적 초기화 사용 예시가 아래에 있다.

정적 초기화 없이는 자가 초기화 루틴 `foo()`가 다음과 같은 모양일 것이다.

```c
static pthread_once_t foo_once = PTHREAD_ONCE_INIT;
static pthread_mutex_t foo_mutex;

void foo_init()
{
    pthread_mutex_init(&foo_mutex, NULL);
}

void foo()
{
    pthread_once(&foo_once, foo_init);
    pthread_mutex_lock(&foo_mutex);
    /* 작업 수행 */
    pthread_mutex_unlock(&foo_mutex);
}
```

정적 초기화를 쓰면 같은 루틴을 다음과 같이 작성할 수 있다.

```c
static pthread_mutex_t foo_mutex = PTHREAD_MUTEX_INITIALIZER;

void foo()
{
    pthread_mutex_lock(&foo_mutex);
    /* 작업 수행 */
    pthread_mutex_unlock(&foo_mutex);
}
```

정적 초기화는 `pthread_once()` 내의 초기화 검사와 `pthread_mutex_lock()` 내지 `pthread_mutex_unlock()`에 전달할 주소를 알아내기 위한 `&foo_mutex` 페치의 필요성을 모두 없애 준다.

따라서 정적 객체 초기화를 위해 작성하는 C 코드가 모든 시스템에서 단순해지며, 동기화 객체(전체)를 응용 메모리에 저장할 수 있는 여러 시스템에서 빨라진다.

하지만 뮤텍스를 특수한 메모리에 할당해야 하는 머신에서는 락킹 성능 문제가 제기될 가능성이 높다. 사실 그런 머신에서는 뮤텍스와 어쩌면 조건 변수까지에 실제 하드웨어 락에 대한 포인터가 들어가게 해야 한다. 그런 머신에서 정적 초기화가 동작하려면 `pthread_mutex_lock()`에서 실제 락에 대한 포인터가 할당되었는지 여부도 검사해야 한다. 할당되지 않았다면 `pthread_mutex_lock()`에서 사용 전에 초기화를 해야 한다. 프로그램 적재 때 그런 자원 예약이 이뤄질 수 있으며, 따라서 뮤텍스 잠그기와 조건 변수 대기에 초기화 완료 실패를 나타내는 반환 코드를 추가하지 않았다.

`pthread_mutex_lock()`의 이런 런타임 검사가 일견 가욋일처럼 보일 것이다. 포인터가 초기화 되었는지 알아보려면 추가 검사가 필요하다. 실제로 대부분 머신에서는 포인터를 가져오고, 0인지 검사하고, 이미 초기화 되었으면 포인터를 사용하는 식으로 이를 구현할 것이다. 그 검사가 가욋일을 더하는 것으로 보일 수도 있겠지만 레지스터를 검사하는 데 드는 추가 노력은 일반적으로 무시할 만하다. 실제 추가로 이뤄지는 메모리 참조가 전혀 없기 때문이다. 점점 많은 머신들에서 캐시를 제공하면서 진짜 비싼 것은 실행 인스트럭션이 아니라 메모리 참조이다.

또는 머신 아키텍처에 따라선 가장 중요한 경우인 초기화 *후* 이뤄지는 락 작업에서 오버헤드를 *완전히* 없애는 것이 종종 가능하다. 이를 위해 더 많은 오버헤드를 덜 빈번한 작업인 초기화로 옮긴다. 외부 뮤텍스 할당은 실제 락을 알아내기 위해 주소를 따라가야 한다는 의미이기도 하며, 그래서 널리 적용할 수 있는 기법 하나는 정적 초기화 때 그 주소로 가짜 값을, 구체적으로는 머신 폴트가 일어나게 하는 주소를 저장하는 것이다. 그런 뮤텍스를 잠그려는 첫 시도에서 그런 폴트가 일어나면 유효성 검사를 한 다음 실제 락의 올바른 주소를 채울 수 있다. 그러면 이후 락 작업에서 "폴트" 하지 않으므로 어떤 추가 오버헤드도 발생하지 않는다. 이는 락 획득 성능에 부정적 영향을 주지 않으면서 정적 초기화를 지원하기 위해 쓸 수 있는 한 기법일 뿐이다. 머신에 매우 의존적인 다른 기법들도 있을 것이다.

따라서 외부 뮤텍스 할당을 하는 머신에서는 묵시적 초기화 모듈에 대한 락킹 오버헤드가 비슷하고 뮤텍스 할당을 완전히 자체에 하는 머신에서는 향상된다. 따라서 자체 경우는 훨씬 빨라지고 외부 경우는 크게 나쁘지 않다.

그런 머신들에서 락킹 성능 문제 외에 제기되는 우려가 있는데, 스레드들이 정적 할당 뮤텍스 초기화를 마무리하려 할 때 초기화 락을 두고 경쟁하면서 직렬화 될 수 있다는 점이다. (그 마무리는 보통 내부 락을 잡고, 어떤 구조체를 할당하고, 그 구조체에 대한 포인터를 뮤텍스에 저장하고, 내부 락을 놓는 식으로 이뤄진다.) 일단 많은 구현에서는 뮤텍스 주소로 해싱을 해서 그런 직렬화를 줄일 것이다. 그리고 그런 직렬화는 제한된 횟수로만 일어날 수 있다. 정확히는 최대로도 정적 할당 동기화 객체 수만큼만 발생할 수 있다. 동적 할당 객체는 여전히 `pthread_mutex_init()`이나 `pthread_cond_init()`을 통해 초기화 될 것이다.

마지막으로, 어떤 구현 상에서 외부 할당을 위한 위의 최적화 기법 어느 것도 응용에게 충분한 성능을 내지 못한다면 응용에서 동기화 객체 모두를 어느 구현에서도 지원하는 `pthread_*_init()` 함수로 명시적으로 초기화 해서 정적 초기화를 아예 피할 수 있다. 또한 구현에서는 각 방식의 장단점을 문서화 하고 그 특정 구현에서 어느 초기화 기법이 더 효율적인지 조언할 수 있다.

### 뮤텍스 파기

잠금을 푼 직후에 뮤텍스를 파기할 수 있다. 예를 들어 다음 코드를 보자.

```c
struct obj {
    pthread_mutex_t om;
    int refcnt;
};

obj_done(struct obj *op)
{
    pthread_mutex_lock(&op->om);
    if (--op->refcnt == 0) {
        pthread_mutex_unlock(&op->om);
(A)     pthread_mutex_destroy(&op->om);
(B)     free(op);
    } else
(C)     pthread_mutex_unlock(&op->om);
}
```

여기서 `obj`에는 참조 카운트가 있고 객체 참조가 내려갈 때마다 `obj_done()`이 호출된다. 구현에서는 객체 잠금을 푼 (C 행) 직후에 객체를 파기 및 해제하고 어쩌면 맵 해제까지 할 수 (가령 A 및 B 행) 있도록 해야 한다.

### 견고 뮤텍스

구현에서는 프로세스 공유 속성이 `PTHREAD_PROCESS_SHARED`로 설정된 뮤텍스에 대해 견고 뮤텍스를 제공해야 한다. 구현에서 프로세스 공유 속성이 `PTHREAD_PROCESS_PRIVATE`으로 설정되어 있을 때 견고 뮤텍스를 제공할 수 있되 필수는 아니다.

## FUTURE DIRECTIONS

없음.

## SEE ALSO

<tt>[[pthread_mutex_getprioceiling()]]</tt>, <tt>[[pthread_mutexattr_getrobust()]]</tt>, <tt>[[pthread_mutex_lock()]]</tt>, <tt>[[pthread_mutex_timedlock()]]</tt>, <tt>[[pthread_mutexattr_getpshared()]]</tt>

POSIX.1-2008 Base Definitions 권, `<pthread.h>`

## COPYRIGHT

Portions of this text are reprinted and reproduced in electronic form from IEEE Std 1003.1, 2013 Edition, Standard for Information Technology -- Portable Operating System Interface (POSIX), The Open Group Base Specifications Issue 7, Copyright (C) 2013 by the Institute of Electrical and Electronics Engineers, Inc and The Open Group. (This is POSIX.1-2008 with the 2013 Technical Corrigendum 1 applied.) In the event of any discrepancy between this version and the original IEEE and The Open Group Standard, the original IEEE and The Open Group Standard is the referee document. The original Standard can be obtained online at <http://www.unix.org/online.html>.

Any typographical or formatting errors that appear in this page are most likely to have been introduced during the conversion of the source files to man page format. To report such errors, see <https://www.kernel.org/doc/man-pages/reporting_bugs.html>.

----

2013
