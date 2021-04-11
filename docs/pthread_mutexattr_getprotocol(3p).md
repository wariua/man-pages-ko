## PROLOG

이 매뉴얼 페이지는 POSIX 프로그래머 매뉴얼의 일부이다. 이 인터페이스의 리눅스 구현에 차이가 있을 수 있으며 (상세한 리눅스 동작 방식은 해당 리눅스 매뉴얼 페이지 참고) 리눅스에서 이 인터페이스가 구현되어 있지 않을 수도 있다.

## NAME

pthread_mutexattr_getprotocol, pthread_mutexattr_setprotocol - 뮤텍스 속성 객체의 프로토콜 속성 얻기 및 설정하기 (**실시간 스레드**)

## SYNOPSIS

```c
#include <pthread.h>

int pthread_mutexattr_getprotocol(const pthread_mutexattr_t
    *restrict attr, int *restrict protocol);
int pthread_mutexattr_setprotocol(pthread_mutexattr_t *attr,
    int protocol);
```

## DESCRIPTION

`pthread_mutexattr_getprotocol()` 및 `pthread_mutexattr_setprotocol()` 함수는 앞서 `pthread_mutexattr_init()` 함수로 생성한 `attr`이 가리키는 뮤텍스 속성 객체의 프로토콜 속성을 얻고 설정한다.

`protocol` 속성은 뮤텍스 이용 시 따를 프로토콜을 규정한다. `protocol`의 값이 다음 중 하나일 수 있다.

* `PTHREAD_PRIO_INHERIT`
* `PTHREAD_PRIO_NONE`
* `PTHREAD_PRIO_PROTECT`

`<pthread.h>` 헤더에 이 값들이 정의되어 있다. 속성의 기본값은 `PTHREAD_PRIO_NONE`이다.

스레드가 `protocol` 속성이 `PTHREAD_PRIO_NONE`인 뮤텍스를 가지고 있을 때는 그 뮤텍스 소유가 우선순위와 스케줄링에 영향을 끼치지 않는다.

스레드가 `protocol` 속성이 `PTHREAD_PRIO_INHERIT`인 견고 뮤텍스를 하나 이상 가지고 있어서 더 높은 우선순위의 스레드 실행을 막고 있을 때는 그 스레드가 가지고 있고 이 프로토콜로 초기화 된 견고 뮤텍스들에 대기 중인 최고 우선순위 스레드의 우선순위와 자기 우선순위 중 높은 쪽으로 실행된다.

스레드가 `protocol` 속성이 `PTHREAD_PRIO_INHERIT`인 비견고 뮤텍스를 하나 이상 가지고 있어서 더 높은 우선순위의 스레드 실행을 막고 있을 때는 그 스레드가 가지고 있고 이 프로토콜로 초기화 된 비견고 뮤텍스들에 대기 중인 최고 우선순위 스레드의 우선순위와 자기 우선순위 중 높은 쪽으로 실행된다.

스레드가 `PTHREAD_PRIO_PROTECT` 프로토콜로 초기화 된 견고 뮤텍스를 하나 이상 가지고 있을 때는 그 스레드가 가지고 있고 이 속성으로 초기화 된 모든 견고 뮤텍스들의 우선순위 상한 최댓값과 자기 우선순위 중 높은 쪽으로 실행된다. 그 견고 뮤텍스들에 다른 스레드 실행이 막혀 있는지 여부는 상관없다.

스레드가 `PTHREAD_PRIO_PROTECT` 프로토콜로 초기화 된 비견고 뮤텍스를 하나 이상 가지고 있을 때는 그 스레드가 가지고 있고 이 속성으로 초기화 된 모든 비견고 뮤텍스들의 우선순위 상한 최댓값과 자기 우선순위 중 높은 쪽으로 실행된다. 그 비견고 뮤텍스들에 다른 스레드 실행이 막혀 있는지 여부는 상관없다.

스레드가 `PTHREAD_PRIO_INHERIT`이나 `PTHREAD_PRIO_PROTECT` 프로토콜 속성으로 초기화 된 뮤텍스를 잡고 있는 동안은 `sched_setparam()` 호출 등에 의해 원래 우선순위가 바뀌는 경우에 그 우선순위의 스케줄링 큐 꼬리로 옮겨지지 않는다. 마찬가지로 스레드가 `PTHREAD_PRIO_INHERIT`이나 `PTHREAD_PRIO_PROTECT` 프로토콜 속성으로 초기화 된 뮤텍스를 풀 때 원래 우선순위가 바뀌는 경우에 그 우선순위의 스케줄링 큐 꼬리로 옮겨지지 않는다.

한 스레드가 서로 다른 프로토콜들로 초기화 된 여러 뮤텍스를 동시에 가지고 있으면 그 프로토콜들 각각으로 얻었을 우선순위들 중 최댓값으로 실행된다.

스레드가 `pthread_mutex_lock()` 호출을 하는데 뮤텍스가 `PTHREAD_PRIO_INHERIT` 값의 프로토콜 속성으로 초기화 되었을 때에, 그 뮤텍스를 다른 스레드가 가지고 있어서 호출 스레드가 실행이 막히는 경우에 그 소유자 스레드가 뮤텍스를 가지고 있는 동안은 호출 스레드의 우선순위 수준을 물려받는다. 구현에서 실행 우선순위를 부여받은 우선순위와 물려받은 우선순위들 중 최댓값으로 변경한다. 또한 그 소유자 스레드가 `PTHREAD_PRIO_INHERIT` 값의 `protocol` 속성을 가진 또 다른 뮤텍스에서 실행이 막히게 되는 경우 동일한 우선순위 상속 효과가 재귀적 방식으로 그 다른 소유자 스레드에게 전파된다.

`pthread_mutexattr_getprotocol()` 내지 `pthread_mutexattr_setprotocol()`의 `attr` 인자로 지정한 값이 초기화 된 뮤텍스 속성 객체를 가리키고 있지 않은 경우의 동작 방식은 규정되어 있지 않다.

## RETURN VALUE

성공 완료 시 `pthread_mutexattr_getprotocol()` 및 `pthread_mutexattr_setprotocol()` 함수는 0을 반환한다. 아니면 오류를 나타내는 오류 번호를 반환한다.

## ERRORS

다음 경우에 `pthread_mutexattr_setprotocol()` 함수가 실패한다.

`ENOTSUP`
:   `protocol`로 지정한 값이 지원하지 않는 값이다.

다음 경우에 `pthread_mutexattr_getprotocol()` 및 `pthread_mutexattr_setprotocol()` 함수가 실패할 수도 있다.

`EINVAL`
:   `protocol`로 지정한 값이 유효하지 않다.

`EPERM`
:   호출자가 동작을 수행하기 위한 특권을 가지고 있지 않다.

이 함수들은 오류 코드 `[EINTR]`을 반환하지 않는다.

*이하는 규범적이지 않은 내용이다.*

## EXAMPLES

없음.

## APPLICATION USAGE

없음.

## RATIONALE

`pthread_mutexattr_getprotocol()` 내지 `pthread_mutexattr_setprotocol()`의 `attr` 인자로 지정한 값이 초기화 된 뮤텍스 속성 객체를 가리키고 있지 않음을 구현에서 감지하는 경우 함수를 실패 처리하고 `[EINVAL]` 오류를 보고하기를 권장한다.

## FUTURE DIRECTIONS

없음.

## SEE ALSO

<tt>[[pthread_cond_destroy()]]</tt>, <tt>[[pthread_create()]]</tt>, <tt>[[pthread_mutex_destroy()]]</tt>

POSIX.1-2008 Base Definitions 권, `<pthread.h>`

## COPYRIGHT

Portions of this text are reprinted and reproduced in electronic form from IEEE Std 1003.1, 2013 Edition, Standard for Information Technology -- Portable Operating System Interface (POSIX), The Open Group Base Specifications Issue 7, Copyright (C) 2013 by the Institute of Electrical and Electronics Engineers, Inc and The Open Group. (This is POSIX.1-2008 with the 2013 Technical Corrigendum 1 applied.) In the event of any discrepancy between this version and the original IEEE and The Open Group Standard, the original IEEE and The Open Group Standard is the referee document. The original Standard can be obtained online at <http://www.unix.org/online.html>.

Any typographical or formatting errors that appear in this page are most likely to have been introduced during the conversion of the source files to man page format. To report such errors, see <https://www.kernel.org/doc/man-pages/reporting_bugs.html>.

----

2013
