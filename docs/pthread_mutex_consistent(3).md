## NAME

pthread_mutex_consistent - 견고 뮤텍스를 일관적 상태로 만들기

## SYNOPSIS

```c
#include <pthread.h>

int pthread_mutex_consistent(pthread_mutex_t *mutex);
```

`-pthread`로 컴파일 및 링크.

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`pthread_mutex_consistent()`:
:   `_POSIX_C_SOURCE >= 200809L`

## DESCRIPTION

이 함수는 견고 뮤텍스가 비일관 상태이면 정상 상태로 만든다. 소유자가 뮤텍스를 잡은 채 종료하면 뮤텍스가 비일관 상태로 남을 수 있는데, 그 경우에 그 뮤텍스를 획득하는 다음 소유자는 `pthread_mutex_lock()` 호출에 성공하면서 `EOWNERDEAD` 반환 값으로 알림을 받게 된다.

## RETURN VALUE

성공 시 `pthread_mutex_consistent()`는 0을 반환한다. 아니면 오류 원인을 나타내는 양수 오류 번호를 반환한다.

## ERRORS

`EINVAL`
:   뮤텍스가 견고가 아니거나 비일관 상태가 아니다.

## VERSIONS

glibc 버전 2.12에서 `pthread_mutex_consistent()`가 추가되었다.

## CONFORMING TO

POSIX.1-2008.

## NOTES

`pthread_mutex_consistent()`는 뮤텍스가 보호하는 상태(공유 데이터)가 정상 상태로 복구되었고 이제 그 뮤텍스로 정상적인 동작을 수행할 수 있다는 것을 구현에게 알릴 뿐이다. `pthread_mutex_consistent()`를 호출하기 전에 공유 데이터를 정상 상태로 복구하는 것은 응용의 책임이다.

POSIX에 `pthread_mutex_consistent()`가 추가되기 전에 glibc에서는 `_GNU_SOURCE`가 정의된 경우 동등한 다음 비표준 함수를 정의하였다.

```c
int pthread_mutex_consistent_np(const pthread_mutex_t *mutex);
```

이 GNU 전용 API는 glibc 2.4에서 처음 등장했으며, 현재는 구식이 되었으므로 새 프로그램에서 쓰지 말아야 한다.

## EXAMPLE

<tt>[[pthread_mutexattr_setrobust(3)]]</tt> 참고.

## SEE ALSO

<tt>[[pthread_mutexattr_init(3)]]</tt>, <tt>[[pthread_mutex_lock(3)]]</tt>, <tt>[[pthread_mutexattr_setrobust(3)]]</tt>, <tt>[[pthread_mutexattr_getrobust(3)]]</tt>, <tt>[[pthreads(7)]]</tt>

----

2017-08-20
