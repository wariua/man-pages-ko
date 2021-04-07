## PROLOG

이 매뉴얼 페이지는 POSIX 프로그래머 매뉴얼의 일부이다. 이 인터페이스의 리눅스 구현에 차이가 있을 수 있으며 (상세한 리눅스 동작 방식은 해당 리눅스 매뉴얼 페이지 참고) 리눅스에서 이 인터페이스가 구현되어 있지 않을 수도 있다.

## NAME

pthread_cond_destroy, pthread_cond_init - 조건 변수 파기 및 초기화

## SYNOPSIS

```c
#include <pthread.h>

int pthread_cond_destroy(pthread_cond_t *cond);
int pthread_cond_init(pthread_cond_t *restrict cond,
    const pthread_condattr_t *restrict attr);
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;
```

## DESCRIPTION

`pthread_cond_destroy()` 함수는 `cond`가 나타내는 조건 변수를 파기한다. 그 객체를 초기화 안 된 상태로 만드는 효과가 있다. 구현 시 `pthread_cond_destroy()`에서 `cond`가 가리키는 객체를 비유효 값으로 설정하도록 할 수도 있다. 파기된 조건 변수 객체를 `pthread_cond_init()`으로 다시 초기화 할 수 있다. 파기된 객체를 그 외 방식으로 참조하는 결과는 규정되어 있지 않다.

현재 어떤 스레드도 블록 되어 있지 않은 초기화 된 조건 변수를 파기하는 것은 안전하다. 현재 다른 스레드가 블록 되어 있는 조건 변수를 파기하려는 시도의 결과는 규정되어 있지 않다.

`pthread_cond_init()` 함수는 `cond`가 가리키는 조건 변수를 `attr`이 가리키는 속성들로 초기화 한다. `attr`이 NULL이면 기본 조건 변수 속성들을 사용한다. 즉 기본 조건 변수 속성 객체의 주소를 전달하는 것과 효과가 같다. 초기화 성공 시 조건 변수는 초기화 된 상태가 된다.

동기화 수행에는 `cond` 자체만 사용할 수 있다. `cond`의 사본을 `pthread_cond_wait()`, `pthread_cond_timedwait()`, `pthread_cond_signal()`, `pthread_cond_broadcast()`, `pthread_cond_destroy()` 호출에서 참조하는 결과는 규정되어 있지 않다.

이미 초기화 된 조건 변수를 초기화 하려는 시도의 결과는 규정되어 있지 않다.

기본 조건 변수 속성들이 적합한 경우에는 매크로 `PTHREAD_COND_INITIALIZER`를 써서 조건 변수를 초기화 할 수 있다. 매개변수 `attr`을 NULL로 지정해 `pthread_cond_init()`을 호출하는 동적 초기화와 효과가 동등하되 오류 검사를 수행하지 않는다는 점이 다르다.

`pthread_cond_destroy()`의 `cond` 인자로 지정한 값이 초기화 된 조건 변수를 가리키고 있지 않은 경우의 동작 방식은 규정되어 있지 않다.

`pthread_cond_init()`의 `attr` 인자로 지정한 값이 초기화 된 조건 변수 속성 객체를 가리키고 있지 않은 경우의 동작 방식은 규정되어 있지 않다.

## RETURN VALUE

성공 시 `pthread_cond_destroy()`와 `pthread_cond_init()` 함수는 0을 반환한다. 아니면 오류를 나타내는 오류 번호를 반환한다.

## ERRORS

다음 경우에 `pthread_cond_init()` 함수가 실패한다.

<dl>
<dt><code>EAGAIN</code></dt>
<dd>조건 변수를 새로 초기화 하는 데 필요한 (메모리 외의) 자원이 시스템에 부족하다.</dd>
<dt><code>ENOMEM</code></dt>
<dd>조건 변수를 초기화 하기에 충분한 메모리가 없다.</dd>
</dl>

이 함수들은 오류 코드 `[EINTR]`을 반환하지 않는다.

<em>이하는 규범적이지 않은 내용이다.</em>

## EXAMPLES

조건 변수에 블록 되어 있는 스레드가 모두 깨어난 직후에 조건 변수를 파기할 수 있다. 예를 들어 다음 코드를 보자.

```c
struct list {
    pthread_mutex_t lm;
    ...
}

struct elt {
    key k;
    int busy;
    pthread_cond_t notbusy;
    ...
}

/* 리스트 항목을 찾아서 예약 */
struct elt *
list_find(struct list *lp, key k)
{
    struct elt *ep;

    pthread_mutex_lock(&lp->lm);
    while ((ep = find_elt(l, k) != NULL) && ep->busy)
        pthread_cond_wait(&ep->notbusy, &lp->lm);
    if (ep != NULL)
        ep->busy = 1;
    pthread_mutex_unlock(&lp->lm);
    return(ep);
}

delete_elt(struct list *lp, struct elt *ep)
{
    pthread_mutex_lock(&lp->lm);
    assert(ep->busy);
    ... 목록에서 ep 제거 ...
    ep->busy = 0;  /* 편집증 */
(A) pthread_cond_broadcast(&ep->notbusy);
    pthread_mutex_unlock(&lp->lm);
(B) pthread_cond_destroy(&ep->notbusy);
    free(ep);
}
```

이 예에서, (A 행) 조건 변수에 대기 중인 스레드를 모두 깨운 직후에 (B 행) 조건 변수와 그 리스트 항목을 해제할 수 있다. 뮤텍스와 코드 때문에 삭제할 항목을 다른 스레드가 건드릴 수 없기 때문이다.

## APPLICATION USAGE

없음.

## RATIONALE

`pthread_cond_destroy()`의 `cond` 인자로 지정한 값이 초기화 된 조건 변수를 가리키고 있지 않음을 구현에서 감지하는 경우 함수를 실패 처리하고 `[EINVAL]` 오류를 보고하기를 권장한다.

`pthread_cond_destroy()`나 `pthread_cond_init()`의 `cond` 인자로 지정한 값을 다른 스레드가 사용 중임을 (가령 `pthread_cond_wait()` 호출 내에 있음을), 또는 `pthread_cond_init()`의 `cond` 인자로 지정한 값이 이미 초기화 된 조건 변수를 가리키고 있음을 구현에서 감지하는 경우 함수를 실패 처리하고 `[EBUSY]` 오류를 보고하기를 권장한다.

`pthread_cond_init()`의 `attr` 인자로 지정한 값이 초기화 된 조건 변수 속성 객체를 가리키고 있지 않음을 구현에서 감지하는 경우 함수를 실패 처리하고 `[EINVAL]` 오류를 보고하기를 권장한다.

`pthread_mutex_destroy()`도 참고하라.

## FUTURE DIRECTIONS

없음.

## SEE ALSO

[[pthread_cond_broadcast()|pthread_cond_broadcast(3p)]], [[pthread_cond_timedwait()|pthread_cond_timedwait(3p)]], [[pthread_mutex_destroy()|pthread_mutex_destroy(3p)]]

POSIX.1-2008 Base Definitions 권, `<pthread.h>`

## COPYRIGHT

Portions of this text are reprinted and reproduced in electronic form from IEEE Std 1003.1, 2013 Edition, Standard for Information Technology -- Portable Operating System Interface (POSIX), The Open Group Base Specifications Issue 7, Copyright (C) 2013 by the Institute of Electrical and Electronics Engineers, Inc and The Open Group. (This is POSIX.1-2008 with the 2013 Technical Corrigendum 1 applied.) In the event of any discrepancy between this version and the original IEEE and The Open Group Standard, the original IEEE and The Open Group Standard is the referee document. The original Standard can be obtained online at http://www.unix.org/online.html .

Any typographical or formatting errors that appear in this page are most likely to have been introduced during the conversion of the source files to man page format. To report such errors, see https://www.kernel.org/doc/man-pages/reporting_bugs.html .

----

2013
