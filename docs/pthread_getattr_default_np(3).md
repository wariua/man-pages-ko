## NAME

pthread_getattr_default_np, pthread_setattr_default_np - 기본 스레드 생성 속성들을 얻거나 설정하기

## SYNOPSIS

```c
#define _GNU_SOURCE             /* feature_test_macros(7) 참고 */
#include <pthread.h>

int pthread_getattr_default_np(pthread_attr_t *attr);
int pthread_setattr_default_np(pthread_attr_t *attr);
```

`-pthread`로 컴파일 및 링크.

## DESCRIPTION

`pthread_setattr_default_np()` 함수는 새 스레드 생성에 쓰는 기본 속성들, 즉 두 번째 인자를 NULL로 해서 <tt>[[pthread_create(3)]]</tt>을 호출할 때 쓰는 속성들을 설정한다. `*attr`로 준 속성들을 이용해 기본 속성들을 설정하며, `*attr`은 미리 설정해 둔 스레드 속성 객체이다. 제공 속성 객체에 대해서 다음 사항들에 유의하다.

* 객체의 속성 설정들이 유효해야 한다.

* 객체에 *스택 주소* 속성을 설정해서는 안 된다.

* *스택 크기* 속성을 0으로 설정하는 것은 기본 스택 크기를 바꾸지 않는다는 뜻이다.

`pthread_getattr_default_np()` 함수는 `attr`이 가리키는 스레드 속성 객체를 스레드 생성에 쓰는 기존 속성들을 담도록 설정한다.

## ERRORS

`EINVAL`
:   (`pthread_setattr_default_np()`) `attr`의 한 속성 설정이 유효하지 않거나, `attr`에 스택 주소 속성이 설정돼 있다.

`ENOMEM`
:   (`pthread_setattr_default_np()`) 메모리 부족.

## VERSIONS

glibc 버전 2.18부터 이 함수들이 사용 가능하다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `pthread_getattr_default_np()`,<br>`pthread_setattr_default_np()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

이 함수들은 비표준 GNU 확장이다. 그래서 이름 뒤에 "_np"(nonportable: 이식성 없음)가 붙어 있다.

## EXAMPLE

아래 프로그램에서는 `pthread_getattr_default_np()`를 사용해 기본 스레드 생성 속성들을 가져온 다음 반환받은 스레드 속성 객체의 여러 설정들을 표시한다. 프로그램 실행 시 다음 출력을 보게 된다.

```c
$ ./a.out
Stack size:         8388608
Guard size:         4096
Scheduling policy:  SCHED_OTHER
Schduling priority: 0
Detach state:       JOINABLE
Inherit scheduler:  INHERIT
```

### 프로그램 소스

```c
#define _GNU_SOURCE
#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>

#define errExitEN(en, msg) \
                        do { errno = en; perror(msg); \
                             exit(EXIT_FAILURE); } while (0)

static void
display_pthread_attr(pthread_attr_t *attr)
{
    int s;
    size_t stacksize;
    size_t guardsize;
    int policy;
    struct sched_param schedparam;
    int detachstate;
    int inheritsched;

    s = pthread_attr_getstacksize(attr, &stacksize);
    if (s != 0)
        errExitEN(s, "pthread_attr_getstacksize");
    printf("Stack size:          %zd\n", stacksize);

    s = pthread_attr_getguardsize(attr, &guardsize);
    if (s != 0)
        errExitEN(s, "pthread_attr_getguardsize");
    printf("Guard size:          %zd\n", guardsize);

    s = pthread_attr_getschedpolicy(attr, &policy);
    if (s != 0)
        errExitEN(s, "pthread_attr_getschedpolicy");
    printf("Scheduling policy:   %d\n",
            (policy == SCHED_FIFO) ? "SCHED_FIFO" :
            (policy == SCHED_RR) ? "SCHED_RR" :
            (policy == SCHED_OTHER) ? "SCHED_OTHER" : "[unknown]");

    s = pthread_attr_getschedparam(attr, &schedparam);
    if (s != 0)
        errExitEN(s, "pthread_attr_getschedparam");
    printf("Scheduling priority: %d\n", schedparam.sched_priority);

    s = pthread_attr_getdetachstate(attr, &detachstate);
    if (s != 0)
        errExitEN(s, "pthread_attr_getdetachstate");
    printf("Detach state:        %s\n",
            (detachstate == PTHREAD_CREATE_DETACHED) ? "DETACHED" :
            (detachstate == PTHREAD_CREATE_JOINABLE) ? "JOINABLE" :
            "???");

    s = pthread_attr_getinheritsched(attr, &inheritsched);
    if (s != 0)
        errExitEN(s, "pthread_attr_getinheritsched");
    printf("Inherit scheduler:   %s\n",
            (inheritsched == PTHREAD_INHERIT_SCHED) ? "INHERIT" :
            (inheritsched == PTHREAD_EXPLICIT_SCHED) ? "EXPLICIT" :
            "???");
}

int
main(int argc, char *argv[])
{
    int s;
    pthread_attr_t attr;

    s = pthread_getattr_default_np(&attr);
    if (s != 0)
        errExitEN(s, "pthread_getattr_default_np");

    display_pthread_attr(&attr);

    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[pthread_attr_getaffinity_np(3)]]</tt>, <tt>[[pthread_attr_getdetachstate(3)]]</tt>, <tt>[[pthread_attr_getguardsize(3)]]</tt>, <tt>[[pthread_attr_getinheritsched(3)]]</tt>, <tt>[[pthread_attr_getschedparam(3)]]</tt>, <tt>[[pthread_attr_getschedpolicy(3)]]</tt>, <tt>[[pthread_attr_getscope(3)]]</tt>, <tt>[[pthread_attr_getstack(3)]]</tt>, <tt>[[pthread_attr_getstackaddr(3)]]</tt>, <tt>[[pthread_attr_getstacksize(3)]]</tt>, <tt>[[pthread_attr_init(3)]]</tt>, <tt>[[pthread_create(3)]]</tt>, <tt>[[pthreads(7)]]</tt>

----

2019-03-06
