## NAME

pthread_create - 새 스레드 만들기

## SYNOPSIS

```c
#include <pthread.h>

int pthread_create(pthread_t *restrict thread,
                   const pthread_attr_t *restrict attr,
                   void *(*start_routine)(void *),
                   void *restrict arg);
```

`-pthread`로 링크.

## DESCRIPTION

`pthread_create()` 함수는 호출 프로세스 내에서 새 스레드를 시작한다. 새 스레드는 `start_routine()` 호출로 실행을 시작하며 `arg`가 `start_routine()`의 유일한 인자로 전달된다.

새 스레드는 다음 중 한 방법으로 종료한다.

* <tt>[[pthread_exit(3)]]</tt>을 호출한다. 지정한 종료 상태 값을 동일 프로세스 내의 다른 스레드가 <tt>[[pthread_join(3)]]</tt>을 호출해서 얻을 수 있다.

* `start_routine()`에서 반환한다. `return` 문에 주는 값으로 <tt>[[pthread_exit(3)]]</tt>을 호출하는 것과 동등하다.

* 취소된다. (<tt>[[pthread_cancel(3)]]</tt> 참고.)

* 프로세스 내의 어느 스레드가 <tt>[[exit(3)]]</tt>을 호출하거나 메인 스레드가 `main()`에서 반환을 수행한다. 그러면 프로세스 내의 모든 스레드들이 종료한다.

`attr` 인자는 `pthread_attr_t` 구조체를 가리키는데, 그 내용물을 스레드 생성 시점에 사용해서 새 스레드의 속성을 결정한다. <tt>[[pthread_attr_init(3)]]</tt> 및 관련 함수들로 이 구조체를 초기화 한다. `attr`이 NULL이면 기본 속성으로 스레드를 생성한다.

`pthread_create()` 호출이 성공하면 반환 전에 새 스레드의 ID를 `thread`가 가리키는 버퍼에 저장한다. 이후의 다른 pthreads 함수 호출에서 이 식별자를 사용해 스레드를 나타낸다.

새 스레드는 생성자 스레드의 시그널 마스크 사본을 물려받는다 (<tt>[[pthread_sigmask(3)]]</tt>). 새 스레드의 미처리 시그널 집합은 비어 있다 (<tt>[[sigpending(2)]]</tt>). 새 스레드는 생성자 스레드의 대체 시그널 스택을 물려받지 않는다 (<tt>[[sigaltstack(2)]]</tt>).

새 스레드는 호출 스레드의 부동소수점 환경을 물려받는다 (<tt>[[fenv(3)]]</tt>).

새 스레드의 CPU 시간 클럭 초기 값은 0이다 (<tt>[[pthread_getcpuclockid(3)]]</tt>).

### 리눅스 한정 세부 사항

새 스레드는 호출 스레드의 역능 집합들(<tt>[[capabilities(7)]]</tt>)과 CPU 친화성 마스크(<tt>[[sched_setaffinity(2)]]</tt>) 사본을 물려받는다.

## RETURN VALUE

성공 시 `pthread_create()`는 0을 반환한다. 오류 시 오류 번호를 반환하며 `*thread` 내용은 규정되어 있지 않다.

## ERRORS

`EAGAIN`
:   스레드를 새로 생성할 자원이 충분하지 않다.

`EAGAIN`
:   시스템이 적용하는 스레드 개수 한계에 부딪쳤다. 이 오류를 일으킬 수 있는 제한들이 여러 가지 있다. 실제 사용자 ID별 프로세스 및 스레드 개수를 제한하는 (<tt>[[setrlimit(2)]]</tt>을 통해 설정한) `RLIMIT_NPROC` 연성 자원 제한에 도달했거나, 커널의 시스템 전체 프로세스 및 스레드 개수 제한인 `/proc/sys/kernel/threads-max`(<tt>[[proc(5)]]</tt> 참고)에 도달했거나, PID 최대 개수인 `/proc/sys/kernel/pid_max`(<tt>[[proc(5)]]</tt>)에 도달한 것이다.

`EINVAL`
:   `attr`에 유효하지 않은 설정.

`EPERM`
:   `attr`에 지정한 스케줄링 정책과 매개변수를 설정할 권한이 없다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `pthread_create()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## NOTES

`pthread_create()`이 `*thread`로 반환하는 스레드 ID에 대한 추가 내용은 <tt>[[pthread_self(3)]]</tt>를 보라. 실시간 스케줄링 정책을 사용 중이지 않다면 `pthread_create()` 호출 후에 어느 스레드(호출자 또는 새 스레드)가 다음으로 실행될지가 불확실하다.

스레드는 *합류 가능*이거나 *분리 상태*이다. 스레드가 합류 가능이면 다른 스레드에서 <tt>[[pthread_join(3)]]</tt>을 호출해 그 스레드의 종료를 기다리고 종료 상태를 가져올 수 있다. 종료한 합류 가능 스레드가 합류되고 나서야 스레드의 마지막 자원들이 시스템으로 해제된다. 분리 상태인 스레드가 종료할 때는 그 자원이 자동으로 시스템으로 해제된다. 그리고 종료 상태를 얻기 위해 그 스레드와 합류하는 것이 불가능하다. 응용에서 그 종료 상태에 신경쓰지 않는 일부 데몬 스레드에서 스레드를 분리시키는 것이 유용하다. 스레드를 분리 상태로 생성하도록 (<tt>[[pthread_attr_setdetachstate(3)]]</tt>로) `attr`을 설정하지 않았다면 기본적으로 새 스레드는 합류 가능 상태로 생성된다.

NPTL 스레딩 구현에서 *프로그램 시작 시점에* 연성 자원 제한 `RLIMIT_STACK`이 "무제한" 외의 값을 가지고 있으면 그 값이 새 스레드의 기본 스택 크기를 결정한다. 기본과 다른 스택 크기를 원한다면 스레드 생성에 쓰는 `attr` 인자에 <tt>[[pthread_attr_setstacksize(3)]]</tt>으로 스택 크기 속성을 명시적으로 설정할 수 있다. `RLIMIT_STACK` 자원 제한이 "무제한"으로 설정돼 있으면 스택 크기에 아키텍처별 값을 사용한다. 다음은 몇몇 아키텍처의 값이다.

| 아키텍처 | 기본 스택 크기 |
| -------- | -------------: |
| i386     |           2 MB |
| IA-64    |          32 MB |
| PowerPC  |           4 MB |
| S/390    |           2 MB |
| Sparc-32 |           2 MB |
| Sparc-64 |           4 MB |
| x86_64   |           2 MB |

## BUGS

구식이 된 LinuxThreads 구현에서는 프로세스 내의 각 스레드가 서로 다른 프로세스 ID를 가진다. 이는 POSIX 스레드 명세 위반이며 다른 여러 표준 비준수 사항들의 원천이다. <tt>[[pthreads(7)]]</tt> 참고.

## EXAMPLES

아래 프로그램은 `pthread_create()` 및 pthreads API의 다른 여러 함수들의 사용 방식을 보여 준다.

NPTL 스레딩 구현을 제공하는 시스템에서의 다음 실행에서 스택 크기 기본값은 "스택 크기" 자원 제한의 값이다.

```text
$ ulimit -s
8192            # 스택 크기 제한 8MB (0x800000바이트)
$ ./a.out hola salut servus
Thread 1: top of stack near 0xb7dd03b8; argv_string=hola
Thread 2: top of stack near 0xb75cf3b8; argv_string=salut
Thread 3: top of stack near 0xb6dce3b8; argv_string=servus
Joined with thread 1; returned value was HOLA
Joined with thread 2; returned value was SALUT
Joined with thread 3; returned value was SERVUS
```

다음 실행에서는 프로그램에서 명시적으로 생성 스레드의 스택 크기를 (<tt>[[pthread_attr_setstacksize(3)]]</tt>로) 1MB로 설정한다.

```text
$ ./a.out -s 0x100000 hola salut servus
Thread 1: top of stack near 0xb7d723b8; argv_string=hola
Thread 2: top of stack near 0xb7c713b8; argv_string=salut
Thread 3: top of stack near 0xb7b703b8; argv_string=servus
Joined with thread 1; returned value was HOLA
Joined with thread 2; returned value was SALUT
Joined with thread 3; returned value was SERVUS
```

### 프로그램 소스

```c
#include <pthread.h>
#include <string.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <errno.h>
#include <ctype.h>

#define handle_error_en(en, msg) \
        do { errno = en; perror(msg); exit(EXIT_FAILURE); } while (0)

#define handle_error(msg) \
        do { perror(msg); exit(EXIT_FAILURE); } while (0)

struct thread_info {    /* thread_start() 인자로 사용 */
    pthread_t thread_id;        /* pthread_create()이 반환하는 ID */
    int       thread_num;       /* 응용에서 정의한 스레드 번호 */
    char     *argv_string;      /* 명령행 인자에서 온 값 */
};

/* 스레드 시작 함수: 스택 상단 근처의 주소를 표시하고
   argv_string을 대문자로 바꾼 사본을 반환한다. */

static void *
thread_start(void *arg)
{
    struct thread_info *tinfo = arg;
    char *uargv;

    printf("Thread %d: top of stack near %p; argv_string=%s\n",
            tinfo->thread_num, (void *) &tinfo, tinfo->argv_string);

    uargv = strdup(tinfo->argv_string);
    if (uargv == NULL)
        handle_error("strdup");

    for (char *p = uargv; *p != '\0'; p++)
        *p = toupper(*p);

    return uargv;
}

int
main(int argc, char *argv[])
{
    int s, opt, num_threads;
    pthread_attr_t attr;
    ssize_t stack_size;
    void *res;

    /* "-s" 옵션은 스레드의 스택 크기를 나타낸다. */

    stack_size = -1;
    while ((opt = getopt(argc, argv, "s:")) != -1) {
        switch (opt) {
        case 's':
            stack_size = strtoul(optarg, NULL, 0);
            break;

        default:
            fprintf(stderr, "Usage: %s [-s stack-size] arg...\n",
                    argv[0]);
            exit(EXIT_FAILURE);
        }
    }

    num_threads = argc - optind;

    /* 스레드 생성 속성 초기화. */

    s = pthread_attr_init(&attr);
    if (s != 0)
        handle_error_en(s, "pthread_attr_init");

    if (stack_size > 0) {
        s = pthread_attr_setstacksize(&attr, stack_size);
        if (s != 0)
            handle_error_en(s, "pthread_attr_setstacksize");
    }

    /* pthread_create() 인자를 위한 메모리 할당하기. */

    struct thread_info *tinfo = calloc(num_threads, sizeof(*tinfo));
    if (tinfo == NULL)
        handle_error("calloc");

    /* 명령행 인자마다 하나씩 스레드 생성하기. */

    for (int tnum = 0; tnum < num_threads; tnum++) {
        tinfo[tnum].thread_num = tnum + 1;
        tinfo[tnum].argv_string = argv[optind + tnum];

        /* pthread_create() 호출이 tinfo[]의 해당 항목에
           스레드 ID를 저장한다. */

        s = pthread_create(&tinfo[tnum].thread_id, &attr,
                           &thread_start, &tinfo[tnum]);
        if (s != 0)
            handle_error_en(s, "pthread_create");
    }

    /* 더는 필요치 않으므로 스레드 속성 객체 파기하기. */

    s = pthread_attr_destroy(&attr);
    if (s != 0)
        handle_error_en(s, "pthread_attr_destroy");

    /* 이제 각 스레드와 합류하고 반환 값을 표시하기. */

    for (int tnum = 0; tnum < num_threads; tnum++) {
        s = pthread_join(tinfo[tnum].thread_id, &res);
        if (s != 0)
            handle_error_en(s, "pthread_join");

        printf("Joined with thread %d; returned value was %s\n",
                tinfo[tnum].thread_num, (char *) res);
        free(res);      /* 스레드가 할당한 메모리 해제 */
    }

    free(tinfo);
    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[getrlimit(2)]]</tt>, <tt>[[pthread_attr_init(3)]]</tt>, <tt>[[pthread_cancel(3)]]</tt>, <tt>[[pthread_detach(3)]]</tt>, <tt>[[pthread_equal(3)]]</tt>, <tt>[[pthread_exit(3)]]</tt>, <tt>[[pthread_getattr_np(3)]]</tt>, <tt>[[pthread_join(3)]]</tt>, <tt>[[pthread_self(3)]]</tt>, <tt>[[pthread_setattr_default_np(3)]]</tt>, <tt>[[pthreads(7)]]</tt>

----

2021-03-22
