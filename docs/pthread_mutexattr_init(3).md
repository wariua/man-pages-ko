## NAME

pthread_mutexattr_init, pthread_mutexattr_destroy - 뮤텍스 속성 객체 초기화 및 파기

## SYNOPSIS

```c
#include <pthread.h>

int pthread_mutexattr_init(pthread_mutexattr_t *attr);
int pthread_mutexattr_destroy(pthread_mutexattr_t *attr);
```

`-pthread`로 컴파일 및 링크.

## DESCRIPTION

`pthread_mutexattr_init()` 함수는 `attr`이 가리키는 뮤텍스 속성 객체를 구현에서 정의하는 모든 속성에 대해 기본값으로 초기화 한다.

이미 초기화 된 뮤텍스 속성 객체를 초기화 하는 결과는 규정되어 있지 않다.

`pthread_mutexattr_destroy()` 함수는 뮤텍스 속성 객체를 파기한다. (초기화 안 된 상태로 만든다.) 뮤텍스 속성 객체가 파기되고 나면 `pthread_mutexattr_init()`으로 다시 초기화 할 수 있다.

초기화 안 된 뮤텍스 속성 객체를 파기하는 결과는 규정되어 있지 않다.

## RETURN VALUE

성공 시 이 함수들은 0을 반환한다. 오류 시 양수 오류 번호를 반환한다.

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## NOTES

뮤텍스 속성 객체의 이후 변경은 그 객체를 이용해 이미 초기화 한 뮤텍스에 영향을 끼치지 않는다.

## SEE ALSO

<tt>[[pthread_mutex_init(3)]]</tt>, <tt>[[pthread_mutexattr_getpshared(3)]]</tt>, <tt>[[pthread_mutexattr_getrobust(3)]]</tt>, <tt>[[pthreads(7)]]</tt>

----

2019-10-10
