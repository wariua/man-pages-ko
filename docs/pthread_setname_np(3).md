## NAME

pthread_setname_np, pthread_getname_np - 스레드 이름 설정하기/얻기

## SYNOPSIS

```c
#define _GNU_SOURCE             /* feature_test_macros(7) 참고 */
#include <pthread.h>

int pthread_setname_np(pthread_t thread, const char *name);
int pthread_getname_np(pthread_t thread, char *name, size_t len);
```

`-pthread`로 컴파일 및 링크.

## DESCRIPTION

기본적으로 <tt>[[pthread_create(3)]]</tt>를 이용해 만든 스레드들은 모두 프로그램 이름을 물려받는다. `pthread_setname_np()` 함수를 사용해 스레드에 고유한 이름을 설정할 수 있는데, 다중 스레드 응용을 디버깅 할 때 유용할 수 있다. 스레드 이름은 유의미한 C 문자열로, 종료용 널 바이트(`'\0'`) 포함 16개 문자로 길이가 제약되어 있다. `thread` 인자는 이름을 바꿀 스레드를 나타내고 `name`은 새 이름을 나타낸다.

`pthread_getname_np()` 함수를 사용해 스레드의 이름을 얻어올 수 있다. `thread` 인자는 이름을 가져올 스레드를 나타낸다. 버퍼 `name`을 이용해 스레드 이름을 반환하며 `len`은 `name`에서 사용 가능한 바이트 개수를 나타낸다. `name`으로 지정한 버퍼는 최소 16개 문자 길이여야 한다. 출력 버퍼로 반환되는 스레드 이름은 널로 끝나게 된다.

## RETURN VALUE

성공 시 이 함수들은 0을 반환한다. 오류 시 0 아닌 오류 번호를 반환한다.

## ERRORS

`pthread_setname_np()` 함수가 다음 오류로 실패할 수 있다.

`ERANGE`
:   `name`으로 지정한 문자열의 길이가 허용 한계를 초과한다.

`pthread_getname_np()` 함수가 다음 오류로 실패할 수 있다.

`ERANGE`
:   `name`과 `len`으로 지정한 버퍼가 스레드 이름을 담기에 너무 작다.

이 함수들이 `/proc/self/task/[tid]/comm`을 여는 데 실패하면 <tt>[[open(2)]]</tt>에서 기술하는 오류들 중 하나로 실패할 수도 있다.

## VERSIONS

glibc 버전 2.12에 이 함수들이 처음 등장했다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `pthread_setname_np()`,<br>`pthread_getname_np()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

이 함수들은 비표준 GNU 확장이다. 그래서 이름 뒤에 "\_np"(nonportable: 이식성 없음)가 붙어 있다.

## NOTES

내부적으로 `pthread_setname_np()`는 `/proc` 파일 시스템 내의 스레드별 `comm` 파일인 `/proc/self/task/[tid]/comm`에 쓰기를 한다. `pthread_getname_np()`는 같은 위치에서 이름을 가져온다.

## EXAMPLES

아래 프로그램이 `pthread_setname_np()`와 `pthread_getname_np()` 사용 방식을 보여 준다.

다음 셸 세션이 프로그램 실행 예를 보여 준다.

```text
$ ./a.out
Created a thread. Default name is: a.out
The thread name after setting it is THREADFOO.
^Z                           # 프로그램 정지
[1]+  Stopped           ./a.out
$ ps H -C a.out -o 'pid tid cmd comm'
  PID   TID CMD                         COMMAND
 5990  5990 ./a.out                     a.out
 5990  5991 ./a.out                     THREADFOO
$ cat /proc/5990/task/5990/comm
a.out
$ cat /proc/5990/task/5991/comm
THREADFOO
```

### 프로그램 소스

```c
#define _GNU_SOURCE
#include <pthread.h>
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <errno.h>
#include <stdlib.h>

#define NAMELEN 16

#define errExitEN(en, msg) \
                        do { errno = en; perror(msg); \
                             exit(EXIT_FAILURE); } while (0)

static void *
threadfunc(void *parm)
{
    sleep(5);          // 주 프로그램에서 스레드 이름 설정할 수 있도록
    return NULL;
}

int
main(int argc, char **argv)
{
    pthread_t thread;
    int rc;
    char thread_name[NAMELEN];

    rc = pthread_create(&thread, NULL, threadfunc, NULL);
    if (rc != 0)
        errExitEN(rc, "pthread_create");

    rc = pthread_getname_np(thread, thread_name, NAMELEN);
    if (rc != 0)
        errExitEN(rc, "pthread_getname_np");

    printf("Created a thread. Default name is: %s\n", thread_name);
    rc = pthread_setname_np(thread, (argc > 1) ? argv[1] : "THREADFOO");
    if (rc != 0)
        errExitEN(rc, "pthread_setname_np");

    sleep(2);

    rc = pthread_getname_np(thread, thread_name,
                            (argc > 2) ? atoi(argv[1]) : NAMELEN);
    if (rc != 0)
        errExitEN(rc, "pthread_getname_np");
    printf("The thread name after setting it is %s.\n", thread_name);

    rc = pthread_join(thread, NULL);
    if (rc != 0)
        errExitEN(rc, "pthread_join");

    printf("Done\n");
    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[prctl(2)]]</tt>, <tt>[[pthread_create(3)]]</tt>, <tt>[[pthreads(7)]]</tt>

----

2021-03-22
