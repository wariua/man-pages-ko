## NAME

pthread_getattr_np - 생성된 스레드의 속성 얻기

## SYNOPSIS

```c
#define _GNU_SOURCE             /* feature_test_macros(7) 참고 */
#include <pthread.h>

int pthread_getattr_np(pthread_t thread, pthread_attr_t *attr);
```

`-pthread`로 컴파일 및 링크.

## DESCRIPTION

`pthread_getattr_np()` 함수는 `attr`이 가리키는 스레드 속성 객체를 동작 중인 스레드 `thread`를 기술하는 실제 속성 값들을 담도록 설정한다.

반환되는 속성 값이 <tt>[[pthread_create(3)]]</tt>으로 스레드를 생성할 때 썼던 `attr` 객체로 전달한 대응 속성 값과 다를 수도 있다. 특히 다음 속성들이 다를 수 있다.

* 분리 상태. 생성 후 합류 가능 스레드가 스스로를 분리했을 수 있다.

* 스택 크기. 구현에서 적당한 경계로 정렬했을 수 있다.

* 방호 구역 크기. 구현에서 페이지 크기의 배수로 올림 할 수 있으며, 또는 응용에서 자체적으로 스택을 할당하는 경우 무시할 수 (즉 0으로 다룰 수) 있다.

여기에 더해서, 스레드 생성에 쓰인 스레드 속성 객체에 스택 주소 속성이 설정돼 있지 않았다면 구현에서 그 스레드에 골라 준 실제 스택 주소를 반환되는 스레드 속성 객체가 알려 주게 된다.

`pthread_getattr_np()`가 반환한 스레드 속성 객체가 더이상 필요하지 않으면 <tt>[[pthread_attr_destroy(3)]]</tt>로 파기하는 게 좋다.

## RETURN VALUE

성공 시 이 함수들은 0을 반환한다. 오류 시 0 아닌 오류 번호를 반환한다.

## ERRORS

`ENOMEM`
:   메모리 부족.

더불어 `thread`가 메인 스레드를 가리키는 경우에는 여러 기반 호출들의 오류 때문에 `pthread_getattr_np()`가 실패할 수 있다. 가령 <tt>[[fopen(3)]]</tt>이 `/proc/self/maps`를 열 수 없거나 <tt>[[getrlimit(2)]]</tt>가 `RLIMIT_STACK` 자원 제한을 지원하지 않는 경우이다.

## VERSIONS

glibc 버전 2.2.3부터 이 함수가 사용 가능하다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `pthread_getattr_np()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

이 함수들은 비표준 GNU 확장이다. 그래서 이름 뒤에 "\_np"(nonportable: 이식성 없음)가 붙어 있다.

## EXAMPLES

아래 프로그램은 `pthread_getattr_np()` 사용 방식을 보여 준다. 프로그램에서 스레드를 생성한 다음 `pthread_getattr_np()`를 이용해 방호 구역 크기, 스택 주소, 스택 크기 속성을 가져와 표시한다. 명령행 인자를 이용해 스레드 생성 시 기본과 다른 값으로 이 속성들을 설정할 수 있다. 아래 셸 세션은 프로그램 사용 방식을 보여 준다.

x86-32 시스템 상의 첫 번째 실행에서는 기본 속성들로 스레드를 만든다.

```text
$ ulimit -s       # 스택 제한 없음 ==> 기본 스택 크기 2MB
unlimited
$ ./a.out
Attributes of created thread:
        Guard size          = 4096 bytes
        Stack address       = 0x40196000 (EOS = 0x40397000)
        Stack size          = 0x201000 (2101248) bytes
```

다음 실행에서는 방호 구역 크기를 지정하면 시스템 페이지 크기(x86-32에서 4096바이트)의 다음 번 배수로 올림 되는 것을 볼 수 있다.

```text
$ ./a.out -g 4097
Thread attributes object after initializations:
        Guard size          = 4097 bytes
        Stack address       = (nil)
        Stack size          = 0x0 (0) bytes

Attributes of created thread:
        Guard size          = 8192 bytes
        Stack address       = 0x40196000 (EOS = 0x40397000)
        Stack size          = 0x201000 (2101248) bytes
```

마지막 실행에서는 스레드를 위한 스택을 프로그램에서 직접 할당한다. 이 경우 방호 구역 크기 속성이 무시된다.

```text
$ ./a.out -g 4096 -s 0x8000 -a
Allocated thread stack at 0x804d000

Thread attributes object after initializations:
        Guard size          = 4096 bytes
        Stack address       = 0x804d000 (EOS = 0x8055000)
        Stack size          = 0x8000 (32768) bytes

Attributes of created thread:
        Guard size          = 0 bytes
        Stack address       = 0x804d000 (EOS = 0x8055000)
        Stack size          = 0x8000 (32768) bytes
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
display_stack_related_attributes(pthread_attr_t *attr, char *prefix)
{
    int s;
    size_t stack_size, guard_size;
    void *stack_addr;

    s = pthread_attr_getguardsize(attr, &guard_size);
    if (s != 0)
        handle_error_en(s, "pthread_attr_getguardsize");
    printf("%sGuard size          = %zu bytes\n", prefix, guard_size);

    s = pthread_attr_getstack(attr, &stack_addr, &stack_size);
    if (s != 0)
        handle_error_en(s, "pthread_attr_getstack");
    printf("%sStack address       = %p", prefix, stack_addr);
    if (stack_size > 0)
        printf(" (EOS = %p)", (char *) stack_addr + stack_size);
    printf("\n");
    printf("%sStack size          = %#zx (%zu) bytes\n",
            prefix, stack_size, stack_size);
}

static void
display_thread_attributes(pthread_t thread, char *prefix)
{
    int s;
    pthread_attr_t attr;

    s = pthread_getattr_np(thread, &attr);
    if (s != 0)
        handle_error_en(s, "pthread_getattr_np");

    display_stack_related_attributes(&attr, prefix);

    s = pthread_attr_destroy(&attr);
    if (s != 0)
        handle_error_en(s, "pthread_attr_destroy");
}

static void *           /* 생성한 스레드의 시작 함수 */
thread_start(void *arg)
{
    printf("Attributes of created thread:\n");
    display_thread_attributes(pthread_self(), "\t");

    exit(EXIT_SUCCESS);         /* 모든 스레드 종료 */
}

static void
usage(char *pname, char *msg)
{
    if (msg != NULL)
        fputs(msg, stderr);
    fprintf(stderr, "Usage: %s [-s stack-size [-a]]"
            " [-g guard-size]\n", pname);
    fprintf(stderr, "\t\t-a means program should allocate stack\n");
    exit(EXIT_FAILURE);
}

static pthread_attr_t *   /* 명령행에서 스레드 속성 얻기 */
get_thread_attributes_from_cl(int argc, char *argv[],
                              pthread_attr_t *attrp)
{
    int s, opt, allocate_stack;
    size_t stack_size, guard_size;
    void *stack_addr;
    pthread_attr_t *ret_attr_p = NULL;   /* 스레드 속성 객체를 초기화
                                            하는 경우 attrp로 설정 */
    allocate_stack = 0;
    stack_size = -1;
    guard_size = -1;

    while ((opt = getopt(argc, argv, "ag:s:")) != -1) {
        switch (opt) {
        case 'a':   allocate_stack = 1;                     break;
        case 'g':   guard_size = strtoul(optarg, NULL, 0);  break;
        case 's':   stack_size = strtoul(optarg, NULL, 0);  break;
        default:    usage(argv[0], NULL);
        }
    }

    if (allocate_stack && stack_size == -1)
        usage(argv[0], "Specifying -a without -s makes no sense\n");

    if (argc > optind)
        usage(argv[0], "Extraneous command-line arguments\n");

    if (stack_size >= 0 || guard_size > 0) {
        ret_attrp = attrp;

        s = pthread_attr_init(attrp);
        if (s != 0)
            handle_error_en(s, "pthread_attr_init");
    }

    if (stack_size >= 0) {
        if (!allocate_stack) {
            s = pthread_attr_setstacksize(attrp, stack_size);
            if (s != 0)
                handle_error_en(s, "pthread_attr_setstacksize");
        } else {
            s = posix_memalign(&stack_addr, sysconf(_SC_PAGESIZE),
                               stack_size);
            if (s != 0)
                handle_error_en(s, "posix_memalign");
            printf("Allocated thread stack at %p\n\n", stack_addr);

            s = pthread_attr_setstack(attrp, stack_addr, stack_size);
            if (s != 0)
                handle_error_en(s, "pthread_attr_setstacksize");
        }
    }

    if (guard_size >= 0) {
        s = pthread_attr_setguardsize(attrp, guard_size);
        if (s != 0)
            handle_error_en(s, "pthread_attr_setstacksize");
    }

    return ret_attrp;
}

int
main(int argc, char *argv[])
{
    int s;
    pthread_t thr;
    pthread_attr_t attr;
    pthread_attr_t *attrp = NULL;    /* 스레드 속성 객체를 초기화
                                        하는 경우 &attr로 설정 */

    attrp = get_thread_attributes_from_cl(argc, argv, &attr);

    if (attrp != NULL) {
        printf("Thread attributes object after initializations:\n");
        display_stack_related_attributes(attrp, "\t");
        printf("\n");
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

<tt>[[pthread_attr_getaffinity_np(3)]]</tt>, <tt>[[pthread_attr_getdetachstate(3)]]</tt>, <tt>[[pthread_attr_getguardsize(3)]]</tt>, <tt>[[pthread_attr_getinheritsched(3)]]</tt>, <tt>[[pthread_attr_getschedparam(3)]]</tt>, <tt>[[pthread_attr_getschedpolicy(3)]]</tt>, <tt>[[pthread_attr_getscope(3)]]</tt>, <tt>[[pthread_attr_getstack(3)]]</tt>, <tt>[[pthread_attr_getstackaddr(3)]]</tt>, <tt>[[pthread_attr_getstacksize(3)]]</tt>, <tt>[[pthread_attr_init(3)]]</tt>, <tt>[[pthread_create(3)]]</tt>, <tt>[[pthreads(7)]]</tt>

----

2021-03-22
