## NAME

pthread_mutexattr_getpshared, pthread_mutexattr_setpshared - 프로세스 공유 뮤텍스 속성 얻기/설정하기

## SYNOPSIS

```c
#include <pthread.h>

int pthread_mutexattr_getpshared(const pthread_mutexattr_t *attr,
                                 int *pshared);
int pthread_mutexattr_setpshared(pthread_mutexattr_t *attr,
                                 int pshared);
```

`-pthread`로 컴파일 및 링크.

## DESCRIPTION

이 함수들은 뮤텍스 속성 객체의 프로세스 공유 속성을 얻고 설정한다. 이 속성 객체를 이용해 생성하는 뮤텍스가 정확하고 효율적으로 동작할 수 있으려면 이 속성을 적절하게 설정해야 한다.

프로세스 공유 속성은 다음 값들 중 하나일 수 있다.

`PTHREAD_PROCESS_PRIVATE`
:   이 속성 객체로 생성하는 뮤텍스를 뮤텍스 초기화를 한 프로세스 내의 스레드들 사이에서만 공유한다. 프로세스 공유 뮤텍스 속성의 기본값이다.

`PTHREAD_PROCESS_SHARED`
:   이 속성 객체로 생성하는 뮤텍스를 다른 프로세스의 스레드를 포함해 그 객체를 담은 메모리에 접근권이 있는 모든 스레드들 사이에 공유할 수 있다.

`pthread_mutexattr_getpshared()`는 `attr`이 가리키는 뮤텍스 속성 객체의 프로세스 공유 속성 값을 `pshared`가 가리키는 위치에 넣는다.

`pthread_mutexattr_setpshared()`는 `attr`이 가리키는 뮤텍스 속성 객체의 프로세스 공유 속성 값을 `pshared`에 지정한 값으로 설정한다.

`attr`이 초기화 된 뮤텍스 속성 객체를 가리키고 있지 않은 경우 동작 방식이 규정되어 있지 않다.

## RETURN VALUE

성공 시 이 함수들은 0을 반환한다. 오류 시 양수 오류 번호를 반환한다.

## ERRORS

`EINVAL`
:   `pshared`에 지정한 값이 유효하지 않다.

`ENOTSUP`
:   `pshared`가 `PTHREAD_PROCESS_SHARED`인데 구현에서 프로세스 공유 뮤텍스를 지원하지 않는다.

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## SEE ALSO

<tt>[[pthread_mutexattr_init(3)]]</tt>, <tt>[[pthreads(7)]]</tt>

----

2017-09-13
