## NAME

pthread_attr_init, pthread_attr_destroy - 스레드 속성 객체 초기화 및 파기

## SYNOPSIS

```c
#include <pthread.h>

int pthread_attr_init(pthread_attr_t *attr);
int pthread_attr_destroy(pthread_attr_t *attr);
```

`-pthread`로 컴파일 및 링크.

## DESCRIPTION

`pthread_attr_init()` 함수는 `attr`이 가리키는 스레드 속성 객체를 기본 속성 값들로 초기화 한다. 이 호출 다음에 (SEE ALSO에 나열된) 여러 관련 함수들을 써서 객체의 개별 속성을 설정할 수 있으며, 그러고 나서 그 객체를 스레드를 생성하는 <tt>[[pthread_create(3)]]</tt> 호출에 한 번 또는 그 이상 사용할 수 있다.

이미 초기화 된 스레드 속성 객체에 `pthread_attr_init()`을 호출하는 결과는 규정되어 있지 않다.

스레드 속성 객체가 더는 필요치 않으면 `pthread_attr_destroy()` 함수를 이용해 파기해야 한다. 스레드 속성 객체 파기는 그 객체로 생성된 스레드에 어떤 영향도 주지 않는다.

스레드 속성 객체를 파기하고 나면 `pthread_attr_init()`으로 다시 초기화 할 수 있다. 파기된 스레드 속성 객체를 그 외 어떤 방식으로든 쓰는 결과는 규정되어 있지 않다.

## RETURN VALUE

성공 시 이 함수들은 0을 반환한다. 오류 시 0 아닌 오류 번호를 반환한다.

## ERRORS

POSIX.1에서는 `pthread_attr_init()`에 `ENOMEM` 오류도 적고 있다. 리눅스에서는 이 함수들이 항상 성공한다. (그렇기는 하지만 이식 가능하고 미래를 대비하는 응용에서는 가능한 오류 반환을 처리해야 할 것이다.)

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `pthread_attr_init()`,<br>`pthread_attr_destroy()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## NOTES

`pthread_attr_t` 타입을 불투명한 것으로 취급해야 한다. pthreads 함수를 통하지 않은 방식으로 객체에 접근하는 것은 이식성이 없으며 규정되지 않은 결과를 유발한다.

## EXAMPLE

아래 프로그램에서는 선택적으로 `pthread_attr_init()`과 여러 관련 함수들을 이용해 스레드 속성 객체를 초기화하고 스레드 한 개를 생성한다. 생성된 스레드는 <tt>[[pthread_getattr_np(3)]]</tt> 함수(비표준 GNU 확장)로 스레드의 속성을 얻어 와서 그 속성들을 표시한다.

명령행 인자 없이 프로그램을 실행하면 <tt>[[pthread_create(3)]]</tt>의 `attr` 인자로 NULL을 전달하며, 그래서 기본 속성으로 스레드를 생성한다. NPTL 스레딩 구현이 있는 리눅스/x86-32에서 프로그램을 돌리면 다음과 같이 나온다.

```text
$ ulimit -s       # 스택 제한 없음 ==> 기본 스택 크기 2MB
unlimited
$ ./a.out
Thread attributes:
        Detach state        = PTHREAD_CREATE_JOINABLE
        Scope               = PTHREAD_SCOPE_SYSTEM
        Inherit scheduler   = PTHREAD_INHERIT_SCHED
        Scheduling policy   = SCHED_OTHER
        Scheduling priority = 0
        Guard size          = 4096 bytes
        Stack address       = 0x40196000
        Stack size          = 0x201000 bytes
```

명령행 인자로 스택 크기를 주면 프로그램에서 스레드 속성 객체를 초기화 하고, 그 객체의 여러 속성들을 설정하고, 그 객체에 대한 포인터를 <tt>[[pthread_create(3)]]</tt> 호출에 전달한다. NTPL 스레딩 구현이 있는 리눅스/x86-32에서 프로그램을 돌리면 다음과 같이 나온다.

```text
$ ./a.out 0x3000000
posix_memalign() allocated at 0x40197000
Thread attributes:
        Detach state        = PTHREAD_CREATE_DETACHED
        Scope               = PTHREAD_SCOPE_SYSTEM
        Inherit scheduler   = PTHREAD_EXPLICIT_SCHED
        Scheduling policy   = SCHED_OTHER
        Scheduling priority = 0
        Guard size          = 0 bytes
        Stack address       = 0x40197000
        Stack size          = 0x3000000 bytes
```

### 프로그램 소스

```c
#define _GNU_SOURCE     /* pthread_getattr_np() 선언을 위해 */
#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <errno.h>

#define handle_error_en(en, msg) \
        do { errno = en; perror(msg); exit(EXIT_FAILURE); } while (0)

static void
display_pthread_attr(pthread_attr_t *attr, char *prefix)
{
    int s, i;
    size_t v;
    void *stkaddr;
    struct sched_param sp;

    s = pthread_attr_getdetachstate(attr, &i);
    if (s != 0)
        handle_error_en(s, "pthread_attr_getdetachstate");
    printf("%sDetach state        = %s\n", prefix,
            (i == PTHREAD_CREATE_DETACHED) ? "PTHREAD_CREATE_DETACHED" :
            (i == PTHREAD_CREATE_JOINABLE) ? "PTHREAD_CREATE_JOINABLE" :
            "???");

    s = pthread_attr_getscope(attr, &i);
    if (s != 0)
        handle_error_en(s, "pthread_attr_getscope");
    printf("%sScope               = %s\n", prefix,
            (i == PTHREAD_SCOPE_SYSTEM)  ? "PTHREAD_SCOPE_SYSTEM" :
            (i == PTHREAD_SCOPE_PROCESS) ? "PTHREAD_SCOPE_PROCESS" :
            "???");

    s = pthread_attr_getinheritsched(attr, &i);
    if (s != 0)
        handle_error_en(s, "pthread_attr_getinheritsched");
    printf("%sInherit scheduler   = %s\n", prefix,
            (i == PTHREAD_INHERIT_SCHED)  ? "PTHREAD_INHERIT_SCHED" :
            (i == PTHREAD_EXPLICIT_SCHED) ? "PTHREAD_EXPLICIT_SCHED" :
            "???");

    s = pthread_attr_getschedpolicy(attr, &i);
    if (s != 0)
        handle_error_en(s, "pthread_attr_getschedpolicy");
    printf("%sScheduling policy   = %s\n", prefix,
            (i == SCHED_OTHER) ? "SCHED_OTHER" :
            (i == SCHED_FIFO)  ? "SCHED_FIFO" :
            (i == SCHED_RR)    ? "SCHED_RR" :
            "???");

    s = pthread_attr_getschedparam(attr, &sp);
    if (s != 0)
        handle_error_en(s, "pthread_attr_getschedparam");
    printf("%sScheduling priority = %d\n", prefix, sp.sched_priority);

    s = pthread_attr.getguardsize(attr, &v);
    if (s != 0)
        handle_error_en(s, "pthread_attr_getguardsize");
    printf("%sGuard size          = %zu bytes\n", prefix, v);

    s = pthread_attr_getstack(attr, &stkaddr, &v);
    if (s != 0)
        handle_error_en(s, "pthread_attr_getstack");
    printf("%sStack address       = %p\n", prefix, stkaddr);
    printf("%sStack size          = 0x%zx bytes\n", prefix, v);
}

static void *
thread_start(void *arg)
{
    int s;
    pthread_attr_t gattr;

    /* pthread_getattr_np()는 비표준 GNU 확장이며
       첫 번째 인자에 지정한 스레드의 속성들을 가져옴 */

    s = pthread_getattr_np(pthread_self(), &gattr);
    if (s != 0)
        handle_error_en(s, "pthread_getattr_np");

    printf("Thread attributes:\n");
    display_pthread_attr(&gattr, "\t");

    exit(EXIT_SUCCESS);         /* 모든 스레드 종료 */
}

int
main(int argc, char *argv[])
{
    pthread_t thr;
    pthread_attr_t attr;
    pthread_attr_t *attrp;      /* NULL 또는 &attr */
    int s;

    attrp = NULL;

    /* 명령행 인자가 있으면 그 값으로 스택 크기 속성을 설정하고
       다른 몇 가지 스레드 속성들을 설정한 다음 attrp가 그
       스레드 속성 객체를 가리키도록 설정 */

    if (argc > 1) {
        int stack_size;
        void *sp;

        attrp = &attr;

        s = pthread_attr_init(&attr);
        if (s != 0)
            handle_error_en(s, "pthread_attr_init");

        s = pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_DETACHED);
        if (s != 0)
            handle_error_en(s, "pthread_attr_setdetachstate");

        s = pthread_attr_setinheritsched(&attr, PTHREAD_EXPLICIT_SCHED);
        if (s != 0)
            handle_error_en(s, "pthread_attr_setinheritsched");

        stack_size = strtoul(argv[1], NULL, 0);

        s = posix_memalign(&sp, sysconf(_SC_PAGESIZE), stack_size);
        if (s != 0)
            handle_error_en(s, "posix_memalign");

        printf("posix_memalign() allocated at %p\n", sp);

        s = pthread_attr_setstack(&attr, sp, stack_size);
        if (s != 0)
            handle_error_en(s, "pthread_attr_setstack");
    }

    s = pthread_create(&thr, attrp, &thread_start, NULL);
    if (s != 0)
        handle_error_en(s, "pthread_create");

    if (attrp != NULL) {
        s = pthread_attr_destroy(attrp);
        if (s != 0)
            handle_error_en(s, "pthread_attr_destroy");
    }

    pause();    /* 다른 스레드에서 exit() 호출할 때 종료 */
}
```

## SEE ALSO

<tt>[[pthread_attr_setaffinity_np(3)]]</tt>, <tt>[[pthread_attr_setdetachstate(3)]]</tt>, <tt>[[pthread_attr_setguardsize(3)]]</tt>, <tt>[[pthread_attr_setinheritsched(3)]]</tt>, <tt>[[pthread_attr_setschedparam(3)]]</tt>, <tt>[[pthread_attr_setschedpolicy(3)]]</tt>, <tt>[[pthread_attr_setscope(3)]]</tt>, <tt>[[pthread_attr_setstack(3)]]</tt>, <tt>[[pthread_attr_setstackaddr(3)]]</tt>, <tt>[[pthread_attr_setstacksize(3)]]</tt>, <tt>[[pthread_create(3)]]</tt>, <tt>[[pthread_getattr_np(3)]]</tt>, <tt>[[pthread_setattr_default_np(3)]]</tt>, <tt>[[pthreads(7)]]</tt>

----

2019-03-06
