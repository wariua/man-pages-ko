## NAME

pthread_mutexattr_getrobust, pthread_mutexattr_setrobust - 뮤텍스 속성 객체의 견고성 속성 얻기 및 설정하기

## SYNOPSIS

```c
#include <pthread.h>

int pthread_mutexattr_getrobust(const pthread_mutexattr_t *attr,
                                int *robustness);
int pthread_mutexattr_setrobust(pthread_mutexattr_t *attr,
                                int robustness);
```

`-pthread`로 컴파일 및 링크.

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`pthread_mutexattr_getrobust()`, `pthread_mutexattr_setrobust()`:
:   `_POSIX_C_SOURCE >= 200809L`

## DESCRIPTION

`pthread_mutexattr_getrobust()` 함수는 `attr`이 가리키는 뮤텍스 속성 객체의 견고성 속성 값을 `*robustness`에 넣는다. `pthread_mutexattr_setrobust()` 함수는 `attr`이 가리키는 뮤텍스 속성 객체의 견고성 속성 값을 `robustness`에 지정한 값으로 설정한다.

견고성 속성은 소유자 스레드가 뮤텍스를 풀지 않고 죽었을 때 뮤텍스의 동작 방식을 지정한다. `robustness`에 다음 값들이 유효하다.

`PTHREAD_MUTEX_STALLED`
:   뮤텍스 속성 객체의 기본값이다. 뮤텍스를 `PTHREAD_MUTEX_STALLED` 속성으로 초기화 하고 소유자가 풀지 않고 죽으면 뮤텍스가 계속 잠긴 상태로 있으며 향후 그 뮤텍스에 대한 <tt>[[pthread_mutex_lock(3)]]</tt> 호출 시도가 영원히 블록 하게 된다.

`PTHREAD_MUTEX_ROBUST`
:   뮤텍스를 `PTHREAD_MUTEX_ROBUST` 속성으로 초기화 하고 소유자가 풀지 않고 죽으면 향후 그 뮤텍스에 대한 <tt>[[pthread_mutex_lock(3)]]</tt> 호출 시도가 성공하고 `EOWNERDEAD`를 반환하는데, 이는 원래 소유자가 더는 존재하지 않고 뮤텍스가 비일관 상태임을 나타낸다. 일반적으로 `EOWNERDEAD` 반환 후에 다음 소유자가 획득한 뮤텍스에 <tt>[[pthread_mutex_consistent(3)]]</tt>를 호출해서 더 이용하기 전에 뮤텍스를 다시 정상으로 만들어야 한다.

    다음 소유자가 뮤텍스를 정상으로 만들기 전에 <tt>[[pthread_mutex_unlock(3)]]</tt>으로 풀면 뮤텍스가 영구히 사용 불가능해지고 이후 <tt>[[pthread_mutex_lock(3)]]</tt>으로 잠그려는 시도가 `ENOTRECOVERABLE` 오류로 실패하게 된다. 그런 뮤텍스에서 유일하게 가능한 동작은 <tt>[[pthread_mutex_destroy(3)]]</tt>이다.

    다음 소유자가 <tt>[[pthread_mutex_consistent(3)]]</tt> 호출 전에 종료하면 그 뮤텍스에 대한 <tt>[[pthread_mutex_lock(3)]]</tt> 동작이 다시 `EOWNERDEAD`를 반환하게 된다.

`pthread_mutexattr_getrobust()` 및 `pthread_mutexattr_setrobust()`의 `attr` 인자는 <tt>[[pthread_mutexattr_init(3)]]</tt>으로 초기화 한 뮤텍스 속성 객체를 가리켜야 하며, 아니면 동작 방식이 규정되어 있지 않다.

## RETURN VALUE

성공 시 이 함수들은 0을 반환한다. 오류 시 양수 오류 번호를 반환한다.

glibc 구현에서 `pthread_mutexattr_getrobust()`는 항상 0을 반환한다.

## ERRORS

`EINVAL`
:   `pthread_mutexattr_setrobust()`에 `PTHREAD_MUTEX_STALLED`와 `PTHREAD_MUTEX_ROBUST` 외의 값을 전달했다.

## VERSIONS

glibc 버전 2.12에서 `pthread_mutexattr_getrobust()`와 `pthread_mutexattr_setrobust()`가 추가되었다.

## CONFORMING TO

POSIX.1-2008.

## NOTES

리눅스 구현에서 프로세스 공유 견고 뮤텍스를 사용할 때 견고 뮤텍스 소유자가 뮤텍스를 풀지 않고 <tt>[[execve(2)]]</tt>를 수행할 때도 대기 중인 스레드가 `EOWNERDEAD` 알림을 받는다. POSIX.1에서는 이 사항을 명세하고 있지 않지만 적어도 몇몇 다른 구현에서도 같은 동작이 일어난다.

POSIX에 `pthread_mutexattr_getrobust()` 및 `pthread_mutexattr_setrobust()`가 추가되기 전에 glibc에서는 `_GNU_SOURCE`가 정의된 경우 동등한 비표준 함수를 정의하였다.

```c
int pthread_mutexattr_getrobust_np(const pthread_mutexattr_t *attr,
                                   int *robustness);
int pthread_mutexattr_setrobust_np(pthread_mutexattr_t *attr,
                                   int robustness);
```

이에 맞게 상수 `PTHREAD_MUTEX_STALLED_NP`와 `PTHREAD_MUTEX_ROBUST_NP`도 정의하였다.

이 GNU 전용 API는 glibc 2.4에서 처음 등장했으며, 현재는 구식이 되었으므로 새 프로그램에서 쓰지 말아야 한다.

## EXAMPLES

아래 프로그램은 뮤텍스 속성 객체의 견고성 속성 사용 방식을 보여 준다. 이 프로그램에서는 뮤텍스를 잡은 스레드가 뮤텍스를 풀지 않고 일찍 죽는다. 이후 메인 스레드가 뮤텍스를 성공적으로 획득하고 `EOWNERDEAD` 오류를 얻으며, 다음으로 뮤텍스를 정상 상태로 만든다.

다음 셸 세션은 프로그램 실행 시 뭐가 찍히는지 보여 준다.

```text
$ ./a.out
[original owner] Setting lock...
[original owner] Locked. Now exiting without unlocking.
[main] Attempting to lock the robust mutex.
[main] pthread_mutex_lock() returned EOWNERDEAD
[main] Now make the mutex consistent
[main] Mutex is now consistent; unlocking
```

### 프로그램 소스

```c
#include <stdlib.h>
#include <stdio.h>
#include <unistd.h>
#include <pthread.h>
#include <errno.h>

#define handle_error_en(en, msg) \
        do { errno = en; perror(msg); exit(EXIT_FAILURE); } while (0)

static pthread_mutex_t mtx;

static void *
original_owner_thread(void *ptr)
{
    printf("[original owner] Setting lock...\n");
    pthread_mutex_lock(&mtx);
    printf("[original owner] Locked. Now exiting without unlocking.\n");
    pthread_exit(NULL);
}

int
main(int argc, char *argv[])
{
    pthread_t thr;
    pthread_mutexattr_t attr;
    int s;

    pthread_mutexattr_init(&attr);

    pthread_mutexattr_setrobust(&attr, PTHREAD_MUTEX_ROBUST);

    pthread_mutex_init(&mtx, &attr);

    pthread_create(&thr, NULL, original_owner_thread, NULL);

    sleep(2);

    /* 지금쯤 "original_owner_thread"는 끝났을 것이다. */

    printf("[main] Attempting to lock the robust mutex.\n");
    s = pthread_mutex_lock(&mtx);
    if (s == EOWNERDEAD) {
        printf("[main] pthread_mutex_lock() returned EOWNERDEAD\n");
        printf("[main] Now make the mutex consistent\n");
        s = pthread_mutex_consistent(&mtx);
        if (s != 0)
            handle_error_en(s, "pthread_mutex_consistent");
        printf("[main] Mutex is now consistent; unlocking\n");
        s = pthread_mutex_unlock(&mtx);
        if (s != 0)
            handle_error_en(s, "pthread_mutex_unlock");

        exit(EXIT_SUCCESS);
    } else if (s == 0) {
        printf("[main] pthread_mutex_lock() unexpectedly succeeded\n");
        exit(EXIT_FAILURE);
    } else {
        printf("[main] pthread_mutex_lock() unexpectedly failed\n");
        handle_error_en(s, "pthread_mutex_lock");
    }
}
```

## SEE ALSO

<tt>[[get_robust_list(2)]]</tt>, <tt>[[set_robust_list(2)]]</tt>, <tt>[[pthread_mutex_consistent(3)]]</tt>, <tt>[[pthread_mutex_init(3)]]</tt>, <tt>[[pthread_mutex_lock(3)]]</tt>, <tt>[[pthreads(7)]]</tt>

----

2021-03-22
