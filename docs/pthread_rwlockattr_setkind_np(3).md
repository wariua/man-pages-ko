## NAME

pthread_rwlockattr_setkind_np, pthread_rwlockattr_getkind_np - 스레드 읽기-쓰기 락 속성 객체의 읽기-쓰기 락 종류 설정하기/얻기

## SYNOPSIS

```c
#include <pthread.h>

int pthread_rwlockattr_setkind_np(pthread_rwlockattr_t *attr,
                                  int pref);
int pthread_rwlockattr_getkind_np(const pthread_rwlockattr_t *attr,
                                  int *pref);
```

`-pthread`로 컴파일 및 링크.

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`pthread_rwlockattr_setkind_np()`, `pthread_rwlockattr_getkind_np()`:
:   `_XOPEN_SOURCE >= 500 || _POSIX_C_SOURCE >= 200809L`

## DESCRIPTION

`pthread_rwlockattr_setkind_np()` 함수는 `attr`이 가리키는 읽기-쓰기 락 속성 객체의 "락 종류" 속성을 `pref`에 지정한 값으로 설정한다. 인자 `pref`를 다음 중 하나로 설정할 수 있다.

`PTHREAD_RWLOCK_PREFER_READER_NP`
:   기본값이다. 한 스레드가 읽기 락을 여러 번 잡을 수 있다. 즉 읽기 락이 재귀적이다. 단일 유닉스 규격에 따르면 읽기 쪽이 락을 잡으려 하는데 쓰기 락은 없지만 쓰기 쪽이 대기 중일 때의 동작 방식이 명세되어 있지 않다. `PTHREAD_RWLOCK_PREFER_READER_NP`를 설정해서 읽기 쪽을 우선한다는 것은 쓰기 쪽이 대기 중인 경우에도 읽기 쪽이 요청한 락을 받게 된다는 의미이다. 읽기 쪽이 있는 동안은 쓰기 쪽이 굶주리게 된다.

`PTHREAD_RWLOCK_PREFER_WRITER_NP`
:   `PTHREAD_RWLOCK_PREFER_READER_NP`의 쓰기 락 짝으로 의도된 것이다. glibc에서는 무시하는데, 재귀적 쓰기 락을 지원해야 한다는 POSIX 요구 사항 때문에 이 옵션이 사소한 교착들을 만들게 되기 때문이다. 대신 `PTHREAD_RWLOCK_PREFER_WRITER_NONRECURSIVE_NP`를 쓰면 되는데, 응용 개발자가 재귀적 읽기 락을 잡지 않도록 해서 교착을 피한다.

`PTHREAD_RWLOCK_PREFER_WRITER_NONRECURSIVE_NP`
:   락 종류를 이 값으로 설정하면 읽기 락이 재귀적 방식으로 이뤄지지만 않으면 쓰기 쪽 굶주림을 피하게 된다.

`pthread_rwlockattr_getkind_np()` 함수는 `attr`이 가리키는 읽기-쓰기 락 속성 객체의 락 종류 속성 값을 포인터 `pref`로 반환한다.

## RETURN VALUE

성공 시 이 함수들은 0을 반환한다. 유효한 포인터 인자가 주어지면 `pthread_rwlockattr_getkind_np()`는 항상 성공한다. 오류 시 `pthread_rwlockattr_setkind_np()`는 0 아닌 오류 번호를 반환한다.

## ERRORS

`EINVAL`
:   `pref`가 지원하지 않는 값을 나타낸다.

## VERSIONS

glibc 2.1에서 `pthread_rwlockattr_getkind_np()` 및 `pthread_rwlockattr_setkind_np()` 함수가 처음 등장했다.

## CONFORMING TO

이 함수들은 비표준 GNU 확장이다. 그래서 이름 뒤에 "\_np"(nonportable: 이식성 없음)가 붙어 있다.

## SEE ALSO

<tt>[[pthreads(7)]]</tt>

----

2019-03-06
