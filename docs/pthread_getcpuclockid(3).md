## NAME

pthread_getcpuclockid - 스레드의 CPU 시간 클럭의 ID 얻기

## SYNOPSIS

```c
#include <pthread.h>
#include <time.h>

int pthread_getcpuclockid(pthread_t thread, clockid_t *clock_id);
```

`-pthread`로 컴파일 및 링크.

## DESCRIPTION

`pthread_getcpuclockid()` 함수는 스레드 `thread`의 CPU 시간 클럭에 대한 클럭 ID를 반환한다.

## RETURN VALUE

성공 시 이 함수는 0을 반환한다. 오류 시 0 아닌 오류 번호를 반환한다.

## ERRORS

`ENOENT`
:   시스템에서 스레드별 CPU 시간 클럭을 지원하지 않는다.

`ESRCH`
:   `thread`라는 ID를 가진 스레드를 찾을 수 없다.

## VERSIONS

glibc 버전 2.2부터 이 함수가 사용 가능하다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `pthread_getcpuclockid()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## NOTES

`thread`가 호출 스레드를 가리킬 때 이 함수가 반환하는 식별자가 가리키는 클럭은 클럭 ID로 `CLOCK_THREAD_CPUTIME_ID`를 주어 <tt>[[clock_gettime(2)]]</tt> 및 <tt>[[clock_settime(2)]]</tt>으로 조작하는 것과 같은 클럭이다.

## EXAMPLE

아래 프로그램은 스레드를 생성한 다음 <tt>[[clock_gettime(2)]]</tt>을 써서 총 프로세스 CPU 시간을 얻고서 두 스레드가 소모한 스레드별 CPU 시간을 얻는다. 다음 셸 세션이 실행 예를 보여 준다.

```
$ ./a.out
Main thread sleeping
Subthread starting infinite loop
Main thread consuming some CPU time...
Process total CPU time:    1.368
Main thread CPU time:      0.376
Subthread CPU time:        0.992
```

### 프로그램 소스

```c
/* "-lrt"로 링크 */

#include <time.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <pthread.h>
#include <string.h>
#include <errno.h>

#define handle_error(msg) \
        do { perror(msg); exit(EXIT_FAILURE); } while (0)

#define handle_error_en(en, msg) \
        do { errno = en; perror(msg); exit(EXIT_FAILURE); } while (0)

static void *
thread_start(void *arg)
{
    printf("Subthread starting infinite loop\n");
    for (;;)
        continue;
}

static void
pclock(char *msg, clockid_t cid)
{
    struct timespec ts;

    printf("%s", msg);
    if (clock_gettime(cid, &ts) == -1)
        handle_error("clock_gettime");
    printf("%4ld.%03ld\n", ts.tv_sec, ts.tv_nsec / 1000000);
}

int
main(int argc, char *argv[])
{
    pthread_t thread;
    clockid_t cid;
    int j, s;

    s = pthread_create(&thread, NULL, thread_start, NULL);
    if (s != 0)
        handle_error_en(s, "pthread_create");

    printf("Main thread sleeping\n");
    sleep(1);

    printf("Main thread consuming some CPU time...\n");
    for (j = 0; j < 2000000; j++)
        getppid();

    pclock("Process total CPU time: ", CLOCK_PROCESS_CPUTIME_ID);

    s = pthread_getcpuclockid(pthread_self(), &cid);
    if (s != 0)
        handle_error_en(s, "pthread_getcpuclockid");
    pclock("Main thread CPU time:   ", cid);

    /* 앞의 코드 4줄을 다음으로 대체할 수도 있음:
       pclock("Main thread CPU time:   ", CLOCK_THREAD_CPUTIME_ID); */

    s = pthread_getcpuclockid(thread, &cid);
    if (s != 0)
        handle_error_en(s, "pthread_getcpuclockid");
    pclock("Subthread CPU time: 1    ", cid);

    exit(EXIT_SUCCESS);         /* 두 스레드 모두 종료 */
}
```

## SEE ALSO

<tt>[[clock_gettime(2)]]</tt>, <tt>[[clock_settime(2)]]</tt>, <tt>[[timer_create(2)]]</tt>, <tt>[[clock_getcpuclockid(3)]]</tt>, <tt>[[pthread_self(3)]]</tt>, <tt>[[pthreads(7)]]</tt>, <tt>[[time(7)]]</tt>

----

2019-03-06
