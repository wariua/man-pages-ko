## NAME

pthread_setschedparam, pthread_getschedparam - 스레드의 스케줄링 정책 및 매개변수 설정하기/얻기

## SYNOPSIS

```c
#include <pthread.h>

int pthread_setschedparam(pthread_t thread, int policy,
                          const struct sched_param *param);
int pthread_getschedparam(pthread_t thread, int *policy,
                          struct sched_param *param);
```

`-pthread`로 컴파일 및 링크.

## DESCRIPTION

`pthread_setschedparam()` 함수는 스레드 `thread`의 스케줄링 정책과 매개변수를 설정한다.

`policy`가 `thread`를 위한 새 스케줄링 정책을 나타낸다. `policy`의 지원 값들과 그 의미는 <tt>[[sched(7)]]</tt>에서 기술한다.

`param`이 가리키는 구조체가 `thread`를 위한 새 스케줄링 매개변수를 나타낸다. 다음 구조체에 스케줄링 매개변수를 담는다.

```c
struct sched_param {
    int sched_priority;     /* 스케줄링 우선순위 */
};
```

보다시피 한 가지 스케줄링 매개변수만 지원한다. 각 스케줄링 정책에서 허용하는 스케줄링 우선순위 범위에 대한 세부 내용은 <tt>[[sched(7)]]</tt>를 보라.

`pthread_getschedparam()` 함수는 스레드 `thread`의 스케줄링 정책과 매개변수를 각각 `policy` 및 `param`이 가리키는 버퍼로 반환한다. 반환되는 우선순위 값은 `thread`에 영향을 준 가장 최근의 `pthread_setschedparam()`이나 <tt>[[pthread_setschedprio(3)]]</tt>, <tt>[[pthread_create(3)]]</tt> 호출에서 설정한 값이다. 반환되는 우선순위는 우선순위 상속 내지 우선순위 상한 지정 함수 호출로 인한 일시적 우선순위 조정을 반영하지 않는다. (예로 <tt>[[pthread_mutexattr_setprioceiling(3)]]</tt> 및 <tt>[[pthread_mutexattr_setprotocol(3)]]</tt> 참고.)

## RETURN VALUE

성공 시 이 함수들은 0을 반환한다. 오류 시 0 아닌 오류 번호를 반환한다. `pthread_setschedparam()`가 실패한 경우 `thread`의 스케줄링 정책 및 매개변수가 바뀌지 않는다.

## ERRORS

두 함수 모두 다음 오류로 실패할 수 있다.

`ESRCH`
:   ID가 `thread`인 스레드를 찾을 수 없다.

`pthread_setschedparam()`이 추가로 다음 오류로 실패할 수도 있다.

`EINVAL`
:   `policy`가 알 수 없는 정책이거나, `policy`에서 `param`이 말이 되지 않는다.

`EPERM`
:   호출자가 지정한 스케줄링 정책 및 매개변수를 설정하기 위한 적절한 특권을 가지고 있지 않다.

POSIX.1에서는 `pthread_setschedparam()`에서 `ENOTSUP` 오류("스케줄링 정책 및 매개변수를 지원하지 않는 값으로 설정하려고 시도했음")도 적고 있다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `pthread_setschedparam()`,<br>`pthread_getschedparam()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## NOTES

스레드의 스케줄링 우선순위를 바꾸는 데 필요한 권한과 그 효과, 각 스케줄링 정책에서 허용하는 우선순위 범위에 대한 설명은 <tt>[[sched(7)]]</tt>를 보라.

## EXAMPLE

아래 프로그램은 `pthread_setschedparam()`과 `pthread_getschedparam()`의 사용 방식뿐 아니라 여러 다른 스케줄링 관련 pthreads 함수들의 사용 방식도 보여 준다.

아래 실행에서 주 스레드는 자기 스케줄링 정책을 우선순위 10인 `SCHED_FIFO`로 설정하고, 스레드 속성 객체를 스케줄링 정책 속성 `SCHED_RR`과 스케줄링 우선순위 속성 20으로 초기화 한다. 그러고서 프로그램은 (<tt>[[pthread_attr_setinheritsched(3)]]</tt>을 이용해) 스레드 속성 객체의 스케줄러 상속 속성을 `PTHREAD_EXPLICIT_SCHED`로 설정하는데, 이는 이 속성 객체를 사용해 생성한 스레드들이 스레드 속성 객체로부터 자기 스케줄링 속성들을 가져와야 한다는 의미이다. 그러고서 프로그램이 그 스레드 속성 객체를 이용해 스레드를 만들고, 그 스레드가 자기 스케줄링 정책과 우선순위를 표시한다.

```
$ su      # 실시간 스케줄링 정책 설정을 위해 특권 필요
Password:
# ./a.out -mf10 -ar20 -i e
Scheduler settings of main thread
    policy=SCHED_FIFO, priority=10

Scheduler settings in 'attr'
    policy=SCHED_RR, priority=20
    inheritsched is EXPLICIT

Scheduler attributes of new thread
    policy=SCHED_RR, priority=20
```

위 출력을 보면 스레드 속성 객체에 지정된 값들에서 스케줄링 정책과 우선순위를 가져왔음을 알 수 있다.

다음 실행은 앞서와 같되 스케줄러 상속 속성을 `PTHREAD_INHERIT_SCHED`로 설정하는데, 이는 스레드 속성 객체를 사용해 생성한 스레드들이 그 스레드 객체에 지정된 스케줄링 속성들을 무시하고 대신 생성을 하는 스레드에게서 스케줄링 속성들을 가져와야 한다는 의미이다.

```
# ./a.out -mf10 -ar20 -i i
Scheduler settings of main thread
    policy=SCHED_FIFO, priority=10

Scheduler settings in 'attr'
    policy=SCHED_RR, priority=20
    inheritsched is INHERIT

Scheduler attributes of new thread
    policy=SCHED_FIFO, priority=10
```

위 출력을 보면 스레드 속성 객체가 아니라 생성을 하는 스레드에게서 스케줄링 정책과 우선순위를 가져왔음을 알 수 있다.

참고로 `-i i` 옵션을 생략했어도 출력이 동일했을 것이다. 스케줄러 상속 속성에 `PTHREAD_INHERIT_SCHED`가 기본값이기 때문이다.

### 프로그램 소스

```c
/* pthreads_sched_test.c */

#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <errno.h>

#define handle_error_en(en, msg) \
        do { errno = en; perror(msg); exit(EXIT_FAILURE); } while (0)

static void
usage(char *prog_name, char *msg)
{
    if (msg != NULL)
        fputs(msg, stderr);

    fprintf(stderr, "Usage: %s [options]\n", prog_name);
    fprintf(stderr, "Options are:\n");
#define fpe(msg) fprintf(stderr, "\t%s", msg);          /* 짧게 */
    fpe("-a<policy><prio> Set scheduling policy and priority in\n");
    fpe("                 thread attributes object\n");
    fpe("                 <policy> can be\n");
    fpe("                     f  SCHED_FIFO\n");
    fpe("                     r  SCHED_RR\n");
    fpe("                     o  SCHED_OTHER\n");
    fpe("-A               Use default thread attributes object\n");
    fpe("-i {e|i}         Set inherit scheduler attribute to\n");
    fpe("                 'explicit' or 'inherit'\n");
    fpe("-m<policy><prio> Set scheduling policy and priority on\n");
    fpe("                 main thread before pthread_create() call\n");
    exit(EXIT_FAILURE);
}

static int
get_policy(char p, int *policy)
{
    switch (p) {
    case 'f': *policy = SCHED_FIFO;     return 1;
    case 'r': *policy = SCHED_RR;       return 1;
    case 'o': *policy = SCHED_OTHER;    return 1;
    default:  return 0;
    }
}

static void
display_sched_attr(int policy, struct sched_param *param)
{
    printf("    policy=%s, priority=%d\n",
            (policy == SCHED_FIFO)  ? "SCHED_FIFO" :
            (policy == SCHED_RR)    ? "SCHED_RR" :
            (policy == SCHED_OTHER) ? "SCHED_OTHER" :
            "???",
            param->sched_priority);
}

static void
display_thread_sched_attr(char *msg)
{
    int policy, s;
    struct sched_param param;

    s = pthread_getschedparam(pthread_self(), &policy, &param);
    if (s != 0)
        handle_error_en(s, "pthread_getschedparam");

    printf("%s\n", msg);
    display_sched_attr(policy, &param);
}

static void *
thread_start(void *arg)
{
    display_thread_sched_attr("Scheduler attributes of new thread");

    return NULL;
}

int
main(int argc, char *argv[])
{
    int s, opt, inheritsched, use_null_attrib, policy;
    pthread_t thread;
    pthread_attr_t attr;
    pthread_attr_t *attrp;
    char *attr_sched_str, *main_sched_str, *inheritsched_str;
    struct sched_param param;

    /* 명령행 옵션 처리 */

    use_null_attrib = 0;
    attr_sched_str = NULL;
    main_sched_str = NULL;
    inheritsched_str = NULL;

    while ((opt = getopt(argc, argv, "a:Ai:m:")) != -1) {
        switch (opt) {
        case 'a': attr_sched_str = optarg;      break;
        case 'A': use_null_attrib = 1;          break;
        case 'i': inheritsched_str = optarg;    break;
        case 'm': main_sched_str = optarg;      break;
        default:  usage(argv[0], "Unrecognized option\n");
        }
    }

    if (use_null_attrib &&
            (inheritsched_str != NULL || attr_sched_str != NULL))
        usage(argv[0], "Can't specify -A with -i or -a\n");

    /* 선택적으로 주 스레드의 스케줄링 속성을 설정하고
       속성을 표시 */

    if (main_sched_str != NULL) {
        if (!get_policy(main_sched_str[0], &policy))
            usage(argv[0], "Bad policy for main thread (-m)\n");
        param.sched_priority = strtol(&main_sched_str[1], NULL, 0);

        s = pthread_setschedparam(pthread_self(), policy, &param);
        if (s != 0)
            handle_error_en(s, "pthread_setschedparam");
    }

    display_thread_sched_attr("Scheduler settings of main thread");
    printf("\n");

    /* 옵션에 따라 스레드 속성 객체 초기화 */

    attrp = NULL;

    if (!use_null_attrib) {
        s = pthread_attr_init(&attr);
        if (s != 0)
            handle_error_en(s, "pthread_attr_init");
        attrp = &attr;
    }

    if (inheritsched_str != NULL) {
        if (inheritsched_str[0] == 'e')
            inheritsched = PTHREAD_EXPLICIT_SCHED;
        else if (inheritsched_str[0] == 'i')
            inheritsched = PTHREAD_INHERIT_SCHED;
        else
            usage(argv[0], "Value for -i must be 'e' or 'i'\n");

        s = pthread_attr_setinheritsched(&attr, inheritsched);
        if (s != 0)
            handle_error_en(s, "pthread_attr_setinheritsched");
    }

    if (attr_sched_str != NULL) {
        if (!get_policy(attr_sched_str[0], &policy))
            usage(argv[0],
                    "Bad policy for 'attr' (-a)\n");
        param.sched_priority = strtol(&attr_sched_str[1], NULL, 0);

        s = pthread_attr_setschedpolicy(&attr, policy);
        if (s != 0)
            handle_error_en(s, "pthread_attr_setschedpolicy");
        s = pthread_attr_setschedparam(&attr, &param);
        if (s != 0)
            handle_error_en(s, "pthread_attr_setschedparam");
    }

    /* 스레드 속성 객체를 초기화 했으면 객체에 설정된
       스케줄링 속성들을 표시 */

    if (attrp != NULL) {
        s = pthread_attr_getschedparam(&attr, &param);
        if (s != 0)
            handle_error_en(s, "pthread_attr_getschedparam");
        s = pthread_attr_getschedpolicy(&attr, &policy);
        if (s != 0)
            handle_error_en(s, "pthread_attr_getschedpolicy");

        printf("Scheduler settings in 'attr'\n");
        display_sched_attr(policy, &param);

        s = pthread_attr_getinheritsched(&attr, &inheritsched);
        printf("    inheritsched is %s\n",
                (inheritsched == PTHREAD_INHERIT_SCHED)  ? "INHERIT" :
                (inheritsched == PTHREAD_EXPLICIT_SCHED) ? "EXPLICIT" :
                "???");
        printf("\n");
    }

    /* 자기 스케줄링 속성을 표시하는 스레드 생성 */

    s = pthread_create(&thread, attrp, &thread_start, NULL);
    if (s != 0)
        handle_error_en(s, "pthread_create");

    /* 필요 없는 속성 객체 파기 */

    if (!use_null_attrib) {
      s = pthread_attr_destroy(&attr);
      if (s != 0)
          handle_error_en(s, "pthread_attr_destroy");
    }

    s = pthread_join(thread, NULL);
    if (s != 0)
        handle_error_en(s, "pthread_join");

    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[getrlimit(2)]]</tt>, <tt>[[sched_get_priority_min(2)]]</tt>, <tt>[[pthread_attr_init(3)]]</tt>, <tt>[[pthread_attr_setinheritsched(3)]]</tt>, <tt>[[pthread_attr_setschedparam(3)]]</tt>, <tt>[[pthread_attr_setschedpolicy(3)]]</tt>, <tt>[[pthread_create(3)]]</tt>, <tt>[[pthread_self(3)]]</tt>, <tt>[[pthread_setschedprio(3)]]</tt>, <tt>[[pthreads(7)]]</tt>, <tt>[[sched(7)]]</tt>

----

2019-03-06
