## PROLOG

이 매뉴얼 페이지는 POSIX 프로그래머 매뉴얼의 일부이다. 이 인터페이스의 리눅스 구현에 차이가 있을 수 있으며 (상세한 리눅스 동작 방식은 해당 리눅스 매뉴얼 페이지 참고) 리눅스에서 이 인터페이스가 구현되어 있지 않을 수도 있다.

## NAME

pthread_mutex_getprioceiling, pthread_mutex_setprioceiling - 뮤텍스의 우선순위 상한 얻기 및 설정하기 (**실시간 스레드**)

## SYNOPSIS

```c
#include <pthread.h>

int pthread_mutex_getprioceiling(const pthread_mutex_t *restrict mutex,
    int *restrict prioceiling);
int pthread_mutex_setprioceiling(pthread_mutex_t *restrict mutex,
    int prioceiling, int *restrict old_ceiling);
```

## DESCRIPTION

`pthread_mutex_getprioceiling()` 함수는 뮤텍스의 현재 우선순위 상한을 반환한다.

`pthread_mutex_setprioceiling()` 함수는 `pthread_mutex_lock()` 호출에 의한 것처럼 뮤텍스를 잠그려고 하되 뮤텍스를 잠그는 과정에서 우선순위 보호 프로토콜을 준수할 필요가 없다. 뮤텍스를 획득하고 나면 뮤텍스의 우선순위 상한을 바꾸고서 `pthread_mutex_unlock()` 호출에 의한 것처럼 뮤텍스를 놓는다. 변경에 성공한 경우 이전 우선순위 상한 값을 `old_ceiling`으로 반환한다.

`pthread_mutex_setprioceiling()` 함수가 실패하는 경우 뮤텍스 우선순위 상한이 바뀌지 않는다.

## RETURN VALUE

성공 시 `pthread_mutex_getprioceiling()` 및 `pthread_mutex_setprioceiling()` 함수는 0을 반환한다. 아니면 오류를 나타내는 오류 번호를 반환한다.

## ERRORS

다음 경우에 이 함수들이 실패한다.

`EINVAL`
:   `mutex`의 프로토콜 속성이 `PTHREAD_PRIO_NONE`이다.

`EPERM`
:   구현에서 작업 수행에 적절한 특권을 요구하며 호출자가 적절한 특권을 가지고 있지 않다.

다음 경우에 `pthread_mutex_setprioceiling()` 함수가 실패한다.

`EAGAIN`
:   `mutex`의 재귀 락 최대 횟수를 초과했기 때문에 뮤텍스를 획득할 수 없다.

`EDEADLK`
:   뮤텍스 유형이 `PTHREAD_MUTEX_ERRORCHECK`이며 현재 스레드가 이미 그 뮤텍스를 소유하고 있다.

`EINVAL`
:   프로토콜 속성 값을 `PTHREAD_PRIO_PROTECT`로 해서 뮤텍스를 생성했으며 호출 스레드의 우선순위가 뮤텍스의 현재 우선순위 상한보다 높으며 구현의 뮤텍스 잠그기 과정에서 우선순위 보호 프로토콜을 준수한다.

`ENOTRECOVERABLE`
:   뮤텍스가 견고 뮤텍스이며 뮤텍스가 보호하는 상태가 복구 가능하지 않다.

`EOWNERDEAD`
:   뮤텍스가 견고 뮤텍스이며 이전 소유자 스레드를 포함한 프로세스가 뮤텍스 락을 잡은 채로 종료했다. 호출 스레드가 뮤텍스 락을 획득하게 되며 상태를 정상으로 만드는 것은 새 소유자의 몫이다. (`pthread_mutex_lock()` 참고.)

다음 경우에 `pthread_mutex_setprioceiling()` 함수가 실패할 수도 있다.

`EDEADLK`
:   교착 조건을 탐지했다.

`EINVAL`
:   `prioceiling`으로 요청한 우선순위가 범위를 벗어난다.

`EOWNERDEAD`
:   뮤텍스가 견고 뮤텍스이며 이전 소유자 스레드가 뮤텍스 락을 잡은 채로 종료했다. 호출 스레드가 뮤텍스 락을 획득하게 되며 상태를 정상으로 만드는 것은 새 소유자의 몫이다. (`pthread_mutex_lock()` 참고.)

이 함수들은 오류 코드 `[EINTR]`을 반환하지 않는다.

*이하는 규범적이지 않은 내용이다.*

## EXAMPLES

없음.

## APPLICATION USAGE

없음.

## RATIONALE

없음.

## FUTURE DIRECTIONS

없음.

## SEE ALSO

<tt>[[pthread_mutex_destroy()]]</tt>, <tt>[[pthread_mutex_lock()]]</tt>, <tt>[[pthread_mutex_timedlock()]]</tt>

POSIX.1-2008 Base Definitions 권, `<pthread.h>`

## COPYRIGHT

Portions of this text are reprinted and reproduced in electronic form from IEEE Std 1003.1, 2013 Edition, Standard for Information Technology -- Portable Operating System Interface (POSIX), The Open Group Base Specifications Issue 7, Copyright (C) 2013 by the Institute of Electrical and Electronics Engineers, Inc and The Open Group. (This is POSIX.1-2008 with the 2013 Technical Corrigendum 1 applied.) In the event of any discrepancy between this version and the original IEEE and The Open Group Standard, the original IEEE and The Open Group Standard is the referee document. The original Standard can be obtained online at <http://www.unix.org/online.html>.

Any typographical or formatting errors that appear in this page are most likely to have been introduced during the conversion of the source files to man page format. To report such errors, see <https://www.kernel.org/doc/man-pages/reporting_bugs.html>.

----

2013
