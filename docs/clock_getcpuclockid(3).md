## NAME

clock_getcpuclockid - 프로세스 CPU 시간 클럭의 ID 얻기

## SYNOPSIS

```c
#include <time.h>

int clock_getcpuclockid(pid_t pid, clockid_t *clockid);
```

`-lrt`로 링크 (glibc 버전 2.17 전에서만).

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`clock_getcpuclockid()`:
:   `_POSIX_C_SOURCE >= 200112L`

## DESCRIPTION

`clock_getcpuclockid()` 함수는 ID가 `pid`인 프로세스의 CPU 시간 클럭의 ID를 얻어서 `clockid`가 가리키는 위치를 통해 반환한다. `pid`가 0이면 호출 프로세스의 CPU 시간 클럭의 클럭 ID를 반환한다.

## RETURN VALUE

성공 시 `clock_getcpuclockid()`는 0을 반환한다. 오류 시 ERRORS 절에 나열된 양수 오류 번호 중 하나를 반환한다.

## ERRORS

`ENOSYS`
:   커널이 다른 프로세스의 프로세스별 CPU 시간 클럭 얻기를 지원하지 않는데 `pid`가 호출 프로세스를 나타내고 있지 않다.

`EPERM`
:   `pid`가 나타내는 프로세스의 CPU 시간 클럭에 접근할 권한을 호출자가 가지고 있지 않다. (POSIX.1-2001에서 명세함. 리눅스에서는 커널이 다른 프로세스의 프로세스별 CPU 시간 클럭 얻기를 지원하지 않는 경우가 아니라면 발생하지 않음.)

`ESRCH`
:   ID가 `pid`인 프로세스가 없다.

## VERSIONS

glibc 버전 2.2부터 `clock_getcpuclockid()` 함수가 사용 가능하다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `clock_getcpuclockid()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## NOTES

`pid` 0으로 `clock_getcpuclockid()`를 호출해서 얻은 클럭 ID를 가지고 <tt>[[clock_gettime(2)]]</tt>을 호출하는 것은 클럭 ID `CLOCK_PROCESS_CPUTIME_ID`를 쓰는 것과 같다.

## EXAMPLES

아래의 예시 프로그램은 명령행으로 받은 ID를 가진 프로세스의 CPU 시간 클럭 ID를 얻은 다음 <tt>[[clock_gettime(2)]]</tt>을 이용해 그 클럭의 시간을 얻는다. 동작 예는 다음과 같다.

```text
$ ./a.out 1                 # init 프로세스의 CPU 클럭 보기
CPU-time clock for PID 1 is 2.213466748 seconds
```

### 프로그램 소스

```c
#define _XOPEN_SOURCE 600
#include <stdint.h>
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <time.h>

int
main(int argc, char *argv[])
{
    clockid_t clockid;
    struct timespec ts;

    if (argc != 2) {
        fprintf(stderr, "%s <process-ID>\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    if (clock_getcpuclockid(atoi(argv[1]), &clockid) != 0) {
        perror("clock_getcpuclockid");
        exit(EXIT_FAILURE);
    }

    if (clock_gettime(clockid, &ts) == -1) {
        perror("clock_gettime");
        exit(EXIT_FAILURE);
    }

    printf("CPU-time clock for PID %s is %jd.%09ld seconds\n",
            argv[1], (intmax_t) ts.tv_sec, ts.tv_nsec);
    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[clock_getres(2)]]</tt>, <tt>[[timer_create(2)]]</tt>, <tt>[[pthread_getcpuclockid(3)]]</tt>, <tt>[[time(7)]]</tt>

----

2021-03-22
