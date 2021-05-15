## NAME

system_data_types - 시스템 데이터 타입들 소개

## DESCRIPTION

### `aiocb`

*INCLUDE*: `<aio.h>`.

```c
struct aiocb {
    int             aio_fildes;    /* 파일 디스크립터 */
    off_t           aio_offset;    /* 파일 오프셋 */
    volatile void  *aio_buf;       /* 버퍼 위치 */
    size_t          aio_nbytes;    /* 전송 길이 */
    int             aio_reqprio;   /* 요청 우선순위 오프셋 */
    struct sigevent aio_sigevent;  /* 시그널 번호와 값 */
    int             aio_lio_opcode;/* 수행할 동작 */
};
```
이 구조체에 대한 추가 정보는 <tt>[[aio(7)]]</tt>를 보라.

*CONFORMING TO*: POSIX.1-2001 및 이후.

*SEE ALSO*: <tt>[[aio_cancel(3)]]</tt>, <tt>[[aio_error(3)]]</tt>, <tt>[[aio_fsync(3)]]</tt>, <tt>[[aio_read(3)]]</tt>, <tt>[[aio_return(3)]]</tt>, <tt>[[aio_suspend(3)]]</tt>, <tt>[[aio_write(3)]]</tt>, <tt>[[lio_listio(3)]]</tt>

### `clock_t`

*INCLUDE*: `<time.h>`나 `<sys/types.h>`. 또는 `<sys/time.h>`.

클럭 틱, 즉 (`<time.h>`에 정의돼 있는) `CLOCKS_PER_SEC`로 된 시스템 시간에 쓴다. POSIX에 따르면 정수 타입 또는 실수 부동소수점 타입이다.

*CONFORMING TO*: C99 및 이후. POSIX.1-2001 및 이후.

*SEE ALSO*: <tt>[[times(2)]]</tt>, <tt>[[clock(3)]]</tt>

### `clockid_t`

*INCLUDE*: `<sys/types.h>`. 또는 `<time.h>`.

클럭 및 타이머 함수들에서 클럭 ID 타입으로 쓴다. POSIX에 따르면 산술형 타입으로 정의돼 있다.

*CONFORMING TO*: POSIX.1.2001 및 이후.

*SEE ALSO*: <tt>[[clock_adjtime(2)]]</tt>, <tt>[[clock_getres(2)]]</tt>, <tt>[[clock_nanosleep(2)]]</tt>, <tt>[[timer_create(2)]]</tt>, <tt>[[clock_getcpuclockid(3)]]</tt>

### `dev_t`

*INCLUDE*: `<sys/types.h>`. 또는 `<sys/stat.h>`.

장치 ID에 쓴다. POSIX에 따르면 정수 타입이다. 이 타입에 대한 자세한 내용은 <tt>[[makedev(3)]]</tt>를 보라.

*CONFORMING TO*: POSIX.1-2001 및 이후.

*SEE ALSO*: <tt>[[mknod(2)]]</tt>, <tt>[[stat(2)]]</tt>

### `div_t`

*INCLUDE*: `<stdlib.h>`.

```c
typedef struct {
    int quot; /* 몫 */
    int rem;  /* 나머지 */
} div_t;
```

<tt>[[div(3)]]</tt> 함수 반환 값의 타입이다.

*CONFORMING TO*: C99 및 이후. POSIX.1-2001 및 이후.

*SEE ALSO*: <tt>[[div(3)]]</tt>

### `double_t`

*INCLUDE*: `<math.h>`.

구현에서 최소 `double` 크기인 가장 효율적인 부동소수점 타입. (`<float.h>`에 정의돼 있는) `FLT_EVAL_METHOD` 매크로의 값에 따라 타입이 달라진다.

| | |
| -- | -- |
| 0 | `double_t`가 `double`. |
| 1 | `double_t`가 `double`. |
| 2 | `double_t`가 `long double`. |

다른 `FLT_EVAL_METHOD` 값에서 `double_t`의 타입은 구현에서 정한다.

*CONFORMING TO*: C99 및 이후. POSIX.1-2001 및 이후.

*SEE ALSO*: 이 페이지의 `float_t` 타입.

### `fd_set`

*INCLUDE*: `<sys/select.h>`. 또는 `<sys/time.h>`.

파일 디스크립터 집합을 나타낼 수 있는 구조체 타입이다. POSIX에 따르면 `fd_set` 구조체에 들어가는 파일 디스크립터 최대 개수가 `FD_SETSIZE` 매크로 값이다.

*CONFORMING TO*: POSIX.1-2001 및 이후.

*SEE ALSO*: <tt>[[select(2)]]</tt>

### `fenv_t`

*INCLUDE*: `<fenv.h>`.

제어 모드와 상태 플래그들을 포함한 부동소수점 환경 전체를 나타내는 타입이다. 자세한 내용은 <tt>[[fenv(3)]]</tt>를 보라.

*CONFORMING TO*: C99 및 이후. POSIX.1-2001 및 이후.

*SEE ALSO*: <tt>[[fenv(3)]]</tt>

### `fexcept_t`

*INCLUDE*: `<fenv.h>`.

부동소수점 상태 플래그들 전체를 나타내는 타입이다. 자세한 내용은 <tt>[[fenv(3)]]</tt>를 보라.

*CONFORMING TO*: C99 및 이후. POSIX.1-2001 및 이후.

*SEE ALSO*: <tt>[[fenv(3)]]</tt>

### `FILE`

*INCLUDE*: `<stdio.h>`. 또는 `<wchar.h>`.

스트림에 쓰는 오브젝트 타입.

*CONFORMING TO*: C99 및 이후. POSIX.1-2001 및 이후.

*SEE ALSO*: <tt>[[fclose(3)]]</tt>, <tt>[[flockfile(3)]]</tt>, <tt>[[fopen(3)]]</tt>, <tt>[[fprintf(3)]]</tt>, `fread(3)`, <tt>[[fscanf(3)]]</tt>, <tt>[[stdin(3)]]</tt>, <tt>[[stdio(3)]]</tt>

### `float_t`

*INCLUDE*: `<math.h>`.

구현에서 최소 `float` 크기인 가장 효율적인 부동소수점 타입. (`<float.h>`에 정의돼 있는) `FLT_EVAL_METHOD` 매크로의 값에 따라 타입이 달라진다.

| | |
| -- | -- |
| 0 | `float_t`가 `float`. |
| 1 | `float_t`가 `double`. |
| 2 | `float_t`가 `long double`. |

다른 `FLT_EVAL_METHOD` 값에서 `float_t`의 타입은 구현에서 정한다.

*CONFORMING TO*: C99 및 이후. POSIX.1-2001 및 이후.

*SEE ALSO*: 이 페이지의 `double_t` 타입.

### `gid_t`

*INCLUDE*: `<sys/types.h>`. 또는 `<grp.h>`, `<pwd.h>`, `<signal.h>`, `<stropts.h>`, `<sys/ipc.h>`, `<sys/stat.h>`, `<unistd.h>`.

그룹 ID를 담는 데 쓰는 타입. POSIX에 따르면 정수 타입이다.

*CONFORMING TO*: POSIX.1-2001 및 이후.

*SEE ALSO*: <tt>[[chown(2)]]</tt>, <tt>[[getgid(2)]]</tt>, <tt>[[getegid(2)]]</tt>, <tt>[[getgroups(2)]]</tt>, <tt>[[getresgid(2)]]</tt>, <tt>[[getgrnam(2)]]</tt>, <tt>[[credentials(7)]]</tt>

### `id_t`

*INCLUDE*: `<sys/types.h>`. 또는 `<sys/resource.h>`.

일반 식별자를 담는 데 쓰는 타입. POSIX에 따르면 `pid_t`, `uid_t`, `gid_t`를 담을 수 있는 정수 타입이다.

*CONFORMING TO*: POSIX.1-2001 및 이후.

*SEE ALSO*: <tt>[[getpriority(2)]]</tt>, <tt>[[waitid(2)]]</tt>

### `imaxdiv_t`

*INCLUDE*: `<inttypes.h>`.

```c
typedef struct {
    intmax_t    quot; /* 몫 */
    intmax_t    rem;  /* 나머지 */
} imaxdiv_t;
```

<tt>[[imaxdiv(3)]]</tt> 함수 반환 값의 타입이다.

*CONFORMING TO*: C99 및 이후. POSIX.1-2001 및 이후.

*SEE ALSO*: <tt>[[imaxdiv(3)]]</tt>

### `intmax_t`

*INCLUDE*: `<stdint.h>`. 또는 `<inttypes.h>`.

구현에서 지원하는 어떤 부호 있는 정수 타입의 값이든 표현 가능한 부호 있는 정수 타입. C 언어 표준에 따르면 [`INTMAX_MIN`, `INTMAX_MAX`] 범위의 값을 저장할 수 있다.

`INTMAX_C()` 매크로는 인자를 `intmax_t` 타입의 정수 상수로 확장해 준다.

<tt>[[printf(3)]]</tt> 및 <tt>[[scanf(3)]]</tt> 함수군에서 `intmax_t`의 길이 변환자는 `j`다. 그래서 `intmax_t` 값을 찍을 때 `%jd`나 `%ji`를 흔히 쓰게 된다.

*CONFORMING TO*: C99 및 이후. POSIX.1-2001 및 이후.

*BUGS*: `__int128`이 정의돼 있고 `long long`이 128비트보다 작은 구현에서 `intmax_t`가 `__int128` 타입 값을 표현할 만큼 크지 않다.

*SEE ALSO*: 이 페이지의 `uintmax_t` 타입.

### `intN_t`

*INCLUDE*: `<stdint.h>`. 또는 `<inttypes.h>`.

`int8_t`, `int16_t`, `int32_t`, `int64_t`

정확히 N비트 고정 크기인 부호 있는 정수 타입. N은 타입 이름에 있는 값이다. C 언어 표준에 따르면 [`INTN_MIN`, `INTN_MAX`] 범위의 값을 저장할 수 있다.

POSIX에 따르면 `int8_t`, `int16_t`, `int32_t`가 반드시 있다. `int64_t`는 64비트 정수 타입을 제공하는 구현에서만 반드시 있다. 이 형태의 그 외 타입들은 모두 선택적이다.

<tt>[[printf(3)]]</tt> 함수군에서 `intN_t` 타입의 길이 변환자는 (`<inttypes.h>`에 정의돼 있는) `PRIdN` 및 `PRIiN` 형태 매크로를 확장한 것이다. 예를 들어 `int64_t` 값을 찍을 때는 `%"PRId64"`나 `%"PRIi64"`가 된다. <tt>[[scanf(3)]]</tt> 함수군에서 `intN_t` 타입의 길이 변환자는 (`<inttypes.h>`에 정의돼 있는) `SCNdN` 및 `SCNiN` 형태 매크로를 확장한 것이다. 예를 들어 `int8_t` 값을 탐색할 때는 `%"SCNd8"`이나 `%"SCNi8"`이 된다.

*CONFORMING TO*: C99 및 이후. POSIX.1-2001 및 이후.

*SEE ALSO*: 이 페이지의 `intmax_t`, `uintN_t`, `uintmax_t` 타입.

### `intptr_t`

*INCLUDE*: `<stdint.h>`. 또는 `<inttypes.h>`.

유효한 어떤 (`void *`) 값이든 변환해서 넣고 뺄 수 있는 부호 있는 정수 타입. C 언어 표준에 따르면 [`INTPTR_MIN`, `INTPTR_MAX`] 범위의 값을 저장할 수 있다.

<tt>[[printf(3)]]</tt> 함수군에서 `intptr_t`의 길이 변환자는 (`<inttypes.h>`에 정의돼 있는) `PRIdPTR` 및 `PRIiPTR` 매크로를 확장한 것이다. 그래서 `intptr_t` 값을 찍을 때 `%"PRIdPTR"`이나 `%"PRIiPTR"`을 흔히 쓰게 된다. <tt>[[scanf(3)]]</tt> 함수군에서 `intptr_t`의 길이 변환자는 (`<inttypes.h>`에 정의돼 있는) `SCNdPTR` 및 `SCNiPTR` 매크로를 확장한 것이다. 그래서 `intptr_t` 값을 탐색할 때 `%"SCNdPTR"`이나 `%"SCNiPTR"`을 흔히 쓰게 된다.

*CONFORMING TO*: C99 및 이후. POSIX.1-2001 및 이후.

*SEE ALSO*: 이 페이지의 `uintptr_t` 및 `void *` 타입.

### `lconv`

*INCLUDE*: `<locale.h>`

```c
struct lconv {                  /* "C" 로캘에서의 값: */
    char   *decimal_point;      /* "." */
    char   *thousands_sep;      /* "" */
    char   *grouping;           /* "" */
    char   *mon_decimal_point;  /* "" */
    char   *mon_thousands_sep;  /* "" */
    char   *mon_grouping;       /* "" */
    char   *positive_sign;      /* "" */
    char   *negative_sign;      /* "" */
    char   *currency_symbol;    /* "" */
    char    frac_digits;        /* CHAR_MAX */
    char    p_cs_precedes;      /* CHAR_MAX */
    char    n_cs_precedes;      /* CHAR_MAX */
    char    p_sep_by_space;     /* CHAR_MAX */
    char    n_sep_by_space;     /* CHAR_MAX */
    char    p_sign_posn;        /* CHAR_MAX */
    char    n_sign_posn;        /* CHAR_MAX */
    char   *int_curr_symbol;    /* "" */
    char    int_frac_digits;    /* CHAR_MAX */
    char    int_p_cs_precedes;  /* CHAR_MAX */
    char    int_n_cs_precedes;  /* CHAR_MAX */
    char    int_p_sep_by_space; /* CHAR_MAX */
    char    int_n_sep_by_space; /* CHAR_MAX */
    char    int_p_sign_posn;    /* CHAR_MAX */
    char    int_n_sign_posn;    /* CHAR_MAX */
};
```

수 값의 서식과 관련된 멤버들을 담고 있다. "C" 로캘에서 멤버들의 값이 위 주석에 있다.

*CONFORMING TO*: C11 및 이후. POSIX.1-2001 및 이후.

*SEE ALSO*: <tt>[[setlocale(3)]]</tt>, <tt>[[localeconv(3)]]</tt>, <tt>[[charsets(7)]]</tt>, <tt>[[locale(7)]]</tt>

### `ldiv_t`

*INCLUDE*: `<stdlib.h>`.

```c
typedef struct {
    long    quot; /* 몫 */
    long    rem;  /* 나머지 */
} ldiv_t;
```

<tt>[[ldiv(3)]]</tt> 함수 반환 값의 타입이다.

*CONFORMING TO*: C99 및 이후. POSIX.1-2001 및 이후.

*SEE ALSO*: <tt>[[ldiv(3)]]</tt>

### `lldiv_t`

*INCLUDE*: `<stdlib.h>`.

```c
typedef struct {
    long long   quot; /* 몫 */
    long long   rem;  /* 나머지 */
} lldiv_t;
```

<tt>[[lldiv(3)]]</tt> 함수 반환 값의 타입이다.

*CONFORMING TO*: C99 및 이후. POSIX.1-2001 및 이후.

*SEE ALSO*: <tt>[[lldiv(3)]]</tt>

### `off64_t`

*INCLUDE*: `<sys/types.h>`.

파일 크기에 쓴다. 64비트 부호 있는 정수 타입이다.

*CONFORMING TO*: glibc에 있다. C 언어 표준이나 POSIX에 표준화되어 있지 않다.

*NOTES*: 이 타입을 이용하려면 기능 확인 매크로 `_LARGEFILE64_SOURCE`가 정의돼 있어야 한다.

*SEE ALSO*: <tt>[[copy_file_range(2)]]</tt>, <tt>[[readahead(2)]]</tt>, <tt>[[sync_file_range(2)]]</tt>, <tt>[[lseek64(3)]]</tt>, <tt>[[feature_test_macros(7)]]</tt>

이 페이지의 `off_t` 타입도 참고.

### `off_t`

*INCLUDE*: `<sys/types.h>`. 또는 `<aio.h>`, `<fcntl.h>`, `<stdio.h>`, `<sys/mman.h>`, `<sys/stat.h>`, `<unistd.h>`.

파일 크기에 쓴다. POSIX에 따르면 부호 있는 정수 타입이다.

*VERSIONS*: POSIX.1-2008부터 `<aio.h>`와 `<stdio.h>`가 `off_t`를 정의한다.

*CONFORMING TO*: POSIX.1-2001 및 이후.

*NOTES*: 어떤 아키텍처에선 기능 확인 매크로 `_FILE_OFFSET_BITS`로 이 타입의 크기를 제어할 수 있다.

*SEE ALSO*: <tt>[[lseek(2)]]</tt>, <tt>[[mmap(2)]]</tt>, <tt>[[posix_fadvise(2)]]</tt>, <tt>[[pread(2)]]</tt>, <tt>[[truncate(2)]]</tt>, `fseeko(3)`, <tt>[[lockf(3)]]</tt>, <tt>[[posix_fallocate(3)]]</tt>, <tt>[[feature_test_macros(7)]]</tt>

이 페이지의 `off64_t` 타입도 참고.

### `pid_t`

*INCLUDE*: `<sys/types.h>`. 또는 `<fcntl.h>`, `<sched.h>`, `<signal.h>`, `<spawn.h>`, `<sys/msg.h>`, `<sys/sem.h>`, `<sys/shm.h>`, `<sys/wait.h>`, `<termios.h>`, `<time.h>`, `<unistd.h>`, `<utmpx.h>`.

프로세스 ID, 프로세스 그룹 ID, 세션 ID 저장에 쓰는 타입. POSIX에 따르면 부호 있는 정수 타입이며, 구현에서 `pid_t` 크기가 `long` 타입보다 크지 않은 프로그래밍 환경을 한 가지 이상 지원한다.

*CONFORMING TO*: POSIX.1-2001 및 이후.

*SEE ALSO*: <tt>[[fork(2)]]</tt>, <tt>[[getpid(2)]]</tt>, <tt>[[getppid(2)]]</tt>, <tt>[[getsid(2)]]</tt>, <tt>[[gettid(2)]]</tt>, <tt>[[getpgid(2)]]</tt>, <tt>[[kill(2)]]</tt>, <tt>[[pidfd_open(2)]]</tt>, <tt>[[sched_setscheduler(2)]]</tt>, <tt>[[waitpid(2)]]</tt>, <tt>[[sigqueue(3)]]</tt>, <tt>[[credentials(7)]]</tt>

### `ptrdiff_t`

*INCLUDE*: `<stddef.h>`.

항목 개수와 배열 인덱스에 쓴다. 두 포인터로 뺄셈을 한 결과다. C 언어 표준에 따르면 [`PTRDIFF_MIN`, `PTRDIFF_MAX`] 범위의 값을 저장할 수 있다.

<tt>[[printf(3)]]</tt> 및 <tt>[[scanf(3)]]</tt> 함수군에서 `ptrdiff_t`의 길이 변환자는 `t`다. 그래서 `ptrdiff_t` 값을 찍을 때 `%td`나 `%ti`를 흔히 쓰게 된다.

*CONFORMING TO*: C99 및 이후. POSIX.1-2001 및 이후.

*SEE ALSO*: 이 페이지의 `size_t` 및 `ssize_t` 타입.

### `regex_t`

*INCLUDE*: `<regex.h>`

```c
typedef struct {
    size_t  re_nsub; /* 괄호로 감싼 하위식 수 */
} regex_t;
```

정규 표현식 검사에 쓰는 구조체 타입이다. <tt>[[regcomp(3)]]</tt>로 컴파일한 정규 표현식을 담는다.

*CONFORMING TO*: POSIX.1-2001 및 이후.

*SEE ALSO*: <tt>[[regex(3)]]</tt>

### `regmatch_t`

*INCLUDE*: `<regex.h>`

```c
typedef struct {
    regoff_t    rm_so; /* 문자열 시작점 기준
                          하위 문자열 시작점의 바이트 오프셋 */
    regoff_t    rm_eo; /* 문자열 시작점 기준
                          하위 문자열 끝 다음 첫 문자의
                          바이트 오프셋 */
} regmatch_t;
```

정규 표현식 검사에 쓰는 구조체 타입이다.

*CONFORMING TO*: POSIX.1-2001 및 이후.

*SEE ALSO*: <tt>[[regexec(3)]]</tt>

### `regoff_t`

*INCLUDE*: `<regex.h>`

POSIX에 따르면 `ptrdiff_t` 타입이나 `ssize_t` 타입 중 한쪽에 저장할 수 있는 가장 큰 값을 저장할 수 있는 부호 있는 정수 타입이다.

*VERSIONS*: POSIX.1-2008 전에선 `off_t` 타입이나 `ssize_t` 타입 중 한쪽에 저장할 수 있는 가장 큰 값을 저장할 수 있었다.

*CONFORMING TO*: POSIX.1-2001 및 이후.

*SEE ALSO*: 이 페이지의 `regmatch_t` 구조체와 `ptrdiff_t` 및 `ssize_t` 타입.

### `sigevent`

*INCLUDE*: `<signal.h>`. 또는 `<aio.h>`, `<mqueue.h>`, `<time.h>`.

```c
struct sigevent {
    int             sigev_notify; /* 알림 종류 */
    int             sigev_signo;  /* 시그널 번호 */
    union sigval    sigev_value;  /* 시그널 값 */
    void          (*sigev_notify_function)(union sigval);
                                  /* 알림 함수 */
    pthread_attr_t *sigev_notify_attributes;
                                  /* 알림 속성 */
};
```

이 타입에 대한 자세한 내용은 <tt>[[sigevent(7)]]</tt>를 보라.

*VERSIONS*: POSIX.1-2008부터 `<aio.h>` 및 `<time.h>`가 `sigevent`를 정의한다.

*CONFORMING TO*: POSIX.1-2001 및 이후.

*SEE ALSO*: <tt>[[timer_create(2)]]</tt>, <tt>[[getaddrinfo_a(3)]]</tt>, <tt>[[lio_listio(3)]]</tt>, <tt>[[mq_notify(3)]]</tt>

이 페이지의 `aiocb` 구조체도 참고.

### `siginfo_t`

*INCLUDE*: `<signal.h>`. 또는 `<sys/wait.h>`.

```c
typedef struct {
    int      si_signo;  /* 시그널 번호 */
    int      si_code;   /* 시그널 코드 */
    pid_t    si_pid;    /* 송신 프로세스 ID */
    uid_t    si_uid;    /* 송신 프로세스의 실제 사용자 ID */
    void    *si_addr;   /* 폴트 유발 인스트럭션의 주소 */
    int      si_status; /* 종료 값 또는 시그널 */
    union sigval si_value;  /* 시그널 값 */
} siginfo_t;
```

시그널 연관 정보. 이 구조체에 대한 (리눅스 전용 추가 필드들을 포함한) 자세한 내용은 <tt>[[sigaction(2)]]</tt>을 보라.

*CONFORMING TO*: POSIX.1-2001 및 이후.

*SEE ALSO*: <tt>[[pidfd_send_signal(2)]]</tt>, <tt>[[rt_sigqueueinfo(2)]]</tt>, <tt>[[sigaction(2)]]</tt>, <tt>[[sigwaitinfo(2)]]</tt>, <tt>[[psiginfo(3)]]</tt>

### `sigset_t`

*INCLUDE*: `<signal.h>`. 또는 `<spawn.h>`나 `<sys/select.h>`.

시그널 집합을 나타내는 타입이다. POSIX에 따르면 정수 또는 구조체 타입이다.

*CONFORMING TO*: POSIX.1-2001 및 이후.

*SEE ALSO*: <tt>[[epoll_pwait(2)]]</tt>, <tt>[[ppoll(2)]]</tt>, <tt>[[pselect(2)]]</tt>, <tt>[[sigaction(2)]]</tt>, <tt>[[signalfd(2)]]</tt>, <tt>[[sigpending(2)]]</tt>, <tt>[[sigprocmask(2)]]</tt>, <tt>[[sigsuspend(2)]]</tt>, <tt>[[sigwaitinfo(2)]]</tt>, <tt>[[signal(7)]]</tt>

### `sigval`

*INCLUDE*: `<signal.h>`.

```c
union sigval {
    int     sigval_int; /* 정수 값 */
    void   *sigval_ptr; /* 포인터 값 */
};
```

시그널과 함께 전달되는 데이터.

*CONFORMING TO*: POSIX.1-2001 및 이후.

*SEE ALSO*: <tt>[[pthread_sigqueue(3)]]</tt>, <tt>[[sigqueue(3)]]</tt>, <tt>[[sigevent(7)]]</tt>

이 페이지의 `sigevent` 구조체와 `siginfo_t` 타입도 참고.

### `size_t`

*INCLUDE*: `<stddef.h>`나 `<sys/types.h>`. 또는 `<aio.h>`, `<glob.h>`, `<grp.h>`, `<iconv.h>`, `<monetary.h>`, `<mqueue.h>`, `<ndbm.h>`, `<pwd.h>`, `<regex.h>`, `<search.h>`, `<signal.h>`, `<stdio.h>`, `<stdlib.h>`, `<string.h>`, `<strings.h>`, `<sys/mman.h>`, `<sys/msg.h>`, `<sys/sem.h>`, `<sys/shm.h>`, `<sys/socket.h>`, `<sys/uio.h>`, `<time.h>`, `<unistd.h>`, `<wchar.h>`, `<wordexp.h>`.

바이트 개수에 쓴다. `sizeof` 연산자의 결과다. C 언어 표준에 따르면 [0, `SIZE_MAX`] 범위의 값을 저장할 수 있는 부호 없는 정수 타입이다. POSIX에 따르면 구현에서 `size_t` 크기가 `long` 타입보다 크지 않은 프로그래밍 환경을 한 가지 이상 지원한다.

<tt>[[printf(3)]]</tt> 및 <tt>[[scanf(3)]]</tt> 함수군에서 `size_t`의 길이 변환자는 `z`다. 그래서 `size_t` 값을 찍을 때 `%zu`나 `%zx`를 흔히 쓰게 된다.

*VERSIONS*: POSIX.1-2008부터 `<aio.h>`, `<glob.h>`, `<grp.h>`, `<iconv.h>`, `<mqueue.h>`, `<pwd.h>`, `<signal.h>`, `<sys/socket.h>`가 `size_t`를 정의한다.

*CONFORMING TO*: C99 및 이후. POSIX.1-2001 및 이후.

*SEE ALSO*: `read(2)`, `write(2)`, `fread(3)`, `fwrite(3)`, `memcmp(3)`, `memcpy(3)`, `memset(3)`, <tt>[[offsetof(3)]]</tt>

이 페이지의 `ptrdiff_t` 및 `ssize_t` 타입도 참고.

### `ssize_t`

*INCLUDE*: `<sys/types.h>`. 또는 `<aio.h>`, `<monetary.h>`, `<mqueue.h>`, `<stdio.h>`, `<sys/msg.h>`, `<sys/socket.h>`, `<sys/uio.h>`, `<unistd.h>`.

바이트 개수 또는 오류 표시에 쓴다. POSIX에 따르면 적어도 [-1, `SSIZE_MAX`] 범위의 값을 저장할 수 있는 부호 있는 정수 타입이며, 구현에서 `ssize_t` 크기가 `long` 타입보다 크지 않은 프로그래밍 환경을 한 가지 이상 지원한다.

glibc와 대다수의 다른 구현에서 <tt>[[printf(3)]]</tt> 및 <tt>[[scanf(3)]]</tt> 함수군을 위한 `ssize_t`의 길이 변환자를 `z`로 제공한다. 그래서 `ssize_t` 값을 찍을 때 `%zd`나 `%zi`를 흔히 쓰게 된다. 대다수 구현에서 `ssize_t`에 `z`가 동작하긴 하지만 이식 가능한 POSIX 프로그램에선 이용을 피하는 게 좋다. 예를 들어 값을 `intmax_t`로 변환하고 그에 맞는 길이 변환자(`j`)를 쓰면 된다.

*CONFORMING TO*: POSIX.1-2001 및 이후.

*SEE ALSO*: `read(2)`, <tt>[[readlink(2)]]</tt>, <tt>[[readv(2)]]</tt>, <tt>[[recv(2)]]</tt>, <tt>[[send(2)]]</tt> `write(2)`

이 페이지의 `ptrdiff_t` 및 `size_t` 타입도 참고.

### `suseconds_t`

*INCLUDE*: `<sys/types.h>`. 또는 `<sys/select.h>`나 `<sys/time.h>`.

마이크로초 단위 시간에 쓴다. POSIX에 따르면 적어도 [-1, 1000000] 범위의 값을 저장할 수 있는 부호 있는 정수 타입이며, 구현에서 `suseconds_t` 크기가 `long` 타입보다 크지 않은 프로그래밍 환경을 한 가지 이상 지원한다.

*CONFORMING TO*: POSIX.1-2001 및 이후.

*SEE ALSO*: 이 페이지의 `timeval` 구조체.

### `time_t`

*INCLUDE*: `<time.h>`나 `<sys/types.h>`. 또는 `<sched.h>`, `<sys/msg.h>`, `<sys/select.h>`, `<sys/sem.h>`, `<sys/shm.h>`, `<sys/stat.h>`, `<sys/time.h>`, `<utime.h>`.

초 단위 시간에 쓴다. POSIX에 따르면 정수 타입이다.

*VERSIONS*: POSIX.1-2008부터 `<sched.h>`가 `time_t`를 정의한다.

*CONFORMING TO*: C99 및 이후. POSIX.1-2001 및 이후.

*SEE ALSO*: <tt>[[stime(2)]]</tt>, <tt>[[time(2)]]</tt>, <tt>[[ctime(3)]]</tt>, <tt>[[difftime(3)]]</tt>

### `timer_t`

*INCLUDE*: `<sys/types.h>`. 또는 `<time.h>`.

<tt>[[timer_create(2)]]</tt>가 반환하는 타이머 ID에 쓴다. POSIX에 따르면 이 타입에 대해 정의된 비교 연산자나 할당 연산자가 없다.

*CONFORMING TO*: POSIX.1-2001 및 이후.

*SEE ALSO*: <tt>[[timer_create(2)]]</tt>, <tt>[[timer_delete(2)]]</tt>, <tt>[[timer_getoverrun(2)]]</tt>, <tt>[[timer_settime(2)]]</tt>

### `timespec`

*INCLUDE*: `<time.h>`. 또는 `<aio.h>`, `<mqueue.h>`, `<sched.h>`, `<signal.h>`, `<sys/select.h>`, `<sys/stat.h>`.

```c
struct timespec {
    time_t  tv_sec;  /* 초 */
    long    tv_nsec; /* 나노초 */
};
```

초와 나노초로 시간을 기술한다.

*CONFORMING TO*: C11 및 이후. POSIX.1-2001 및 이후.

*SEE ALSO*: <tt>[[clock_gettime(2)]]</tt>, <tt>[[clock_nanosleep(2)]]</tt>, <tt>[[nanosleep(2)]]</tt>, <tt>[[timerfd_gettime(2)]]</tt>, <tt>[[timer_gettime(2)]]</tt>

### `timeval`

*INCLUDE*: `<sys/time.h>`. 또는 `<sys/resource.h>`, `<sys/select.h>`, `<utmpx.h>`.

```c
struct timeval {
    time_t      tv_sec;  /* 초 */
    suseconds_t tv_usec; /* 마이크로초 */
};
```

초와 마이크로초로 시간을 기술한다.

*CONFORMING TO*: POSIX.1-2001 및 이후.

*SEE ALSO*: <tt>[[gettimeofday(2)]]</tt>, <tt>[[select(2)]]</tt>, <tt>[[utimes(2)]]</tt>, <tt>[[adjtime(3)]]</tt>, <tt>[[futimes(3)]]</tt>, <tt>[[timeradd(3)]]</tt>

### `uid_t`

*INCLUDE*: `<sys/types.h>`. 또는 `<pwd.h>`, `<signal.h>`, `<stropts.h>`, `<sys/ipc.h>`, `<sys/stat.h>`, `<unistd.h>`.

사용자 ID를 담는 데 쓰는 타입. POSIX에 따르면 정수 타입이다.

*CONFORMING TO*: POSIX.1-2001 및 이후.

*SEE ALSO*: <tt>[[chown(2)]]</tt>, <tt>[[getuid(2)]]</tt>, <tt>[[geteuid(2)]]</tt>, <tt>[[getresuid(2)]]</tt>, <tt>[[getpwnam(2)]]</tt>, <tt>[[credentials(7)]]</tt>

### `uintmax_t`

*INCLUDE*: `<stdint.h>`. 또는 `<inttypes.h>`.

구현에서 지원하는 어떤 부호 없는 정수 타입의 값이든 표현 가능한 부호 없는 정수 타입. C 언어 표준에 따르면 [0, `UINTMAX_MAX`] 범위의 값을 저장할 수 있다.

`UINTMAX_C()` 매크로는 인자를 `uintmax_t` 타입의 정수 상수로 확장해 준다.

<tt>[[printf(3)]]</tt> 및 <tt>[[scanf(3)]]</tt> 함수군에서 `uintmax_t`의 길이 변환자는 `j`다. 그래서 `uintmax_t` 값을 찍을 때 `%ju`나 `%jx`를 흔히 쓰게 된다.

*CONFORMING TO*: C99 및 이후. POSIX.1-2001 및 이후.

*BUGS*: `unsigned __int128`이 정의돼 있고 `unsigned long long`이 128비트보다 작은 구현에서 `uintmax_t`가 `unsigned __int128` 타입 값을 표현할 만큼 크지 않다.

*SEE ALSO*: 이 페이지의 `intmax_t` 타입.

### `uintN_t`

*INCLUDE*: `<stdint.h>`. 또는 `<inttypes.h>`.

`uint8_t`, `uint16_t`, `uint32_t`, `uint64_t`

정확히 N비트 고정 크기인 부호 없는 정수 타입. N은 타입 이름에 있는 값이다. C 언어 표준에 따르면 [0, `UINTN_MAX`] 범위의 값을 저장할 수 있다.

POSIX에 따르면 `uint8_t`, `uint16_t`, `uint32_t`가 반드시 있다. `uint64_t`는 64비트 정수 타입을 제공하는 구현에서만 반드시 있다. 이 형태의 그 외 타입들은 모두 선택적이다.

<tt>[[printf(3)]]</tt> 함수군에서 `uintN_t` 타입의 길이 변환자는 (`<inttypes.h>`에 정의돼 있는) `PRIuN`, `PRIoN`, `PRIxN`, `PRIXN` 형태 매크로를 확장한 것이다. 예를 들어 `uint32_t` 값을 찍을 때는 `%"PRIu32"`나 `%"PRIx32"`가 된다. <tt>[[scanf(3)]]</tt> 함수군에서 `uintN_t` 타입의 길이 변환자는 (`<inttypes.h>`에 정의돼 있는) `SCNuN`,`SCNoN`, `SCNxN`, `SCNXN` 형태 매크로를 확장한 것이다. 예를 들어 `uint16_t` 값을 탐색할 때는 `%"SCNu16"`이나 `%"SCNx16"`이 된다.

*CONFORMING TO*: C99 및 이후. POSIX.1-2001 및 이후.

*SEE ALSO*: 이 페이지의 `intmax_t`, `intN_t`, `uintmax_t` 타입.

### `uintptr_t`

*INCLUDE*: `<stdint.h>`. 또는 `<inttypes.h>`.

유효한 어떤 (`void *`) 값이든 변환해서 넣고 뺄 수 있는 부호 없는 정수 타입. C 언어 표준에 따르면 [0, `UINTPTR_MAX`] 범위의 값을 저장할 수 있다.

<tt>[[printf(3)]]</tt> 함수군에서 `uintptr_t`의 길이 변환자는 (`<inttypes.h>`에 정의돼 있는) `PRIuPTR`, `PRIoPTR`, `PRIxPTR`, `PRIXPTR` 매크로를 확장한 것이다. 그래서 `uintptr_t` 값을 찍을 때 `%"PRIuPTR"`이나 `%"PRIxPTR"`을 흔히 쓰게 된다. <tt>[[scanf(3)]]</tt> 함수군에서 `uintptr_t`의 길이 변환자는 (`<inttypes.h>`에 정의돼 있는) `SCNuPTR`, `SCNoPTR`, `SCNxPTR`, `SCNXPTR` 매크로를 확장한 것이다. 그래서 `uintptr_t` 값을 탐색할 때 `%"SCNuPTR"`이나 `%"SCNxPTR"`을 흔히 쓰게 된다.

*CONFORMING TO*: C99 및 이후. POSIX.1-2001 및 이후.

*SEE ALSO*: 이 페이지의 `intptr_t` 및 `void *` 타입.

### `va_list`

*INCLUDE*: `<stdarg.h>`. 또는 `<stdio.h>`나 `<wchar.h>`.

가변 타입 가변 개수 인자가 있는 함수에서 쓴다.  함수에서 `va_list` 타입 객체를 선언해야 하며 <tt>[[va_start(3)]]</tt>, <tt>[[va_arg(3)]]</tt>, <tt>[[va_copy(3)]]</tt>, <tt>[[va_end(3)]]</tt> 매크로로 인자 목록을 순회한다.

*CONFORMING TO*: C99 및 이후. POSIX.1-2001 및 이후.

*SEE ALSO*: <tt>[[va_start(3)]]</tt>, <tt>[[va_arg(3)]]</tt>, <tt>[[va_copy(3)]]</tt>, <tt>[[va_end(3)]]</tt>

### `void *`

C 언어 표준에 따르면 모든 객체의 포인터와 `void` 포인터를 서로 변환할 수 있다. 더 나아가 POSIX에서는 함수 포인터까지 포함한 모든 포인터와 `void` 포인터를 서로 변환할 수 있기를 요구한다.

다른 포인터 타입과의 변환은 묵시적으로 이뤄지며 캐스팅이 전혀 필요치 않다. 이런 동작 때문에 어떤 종류의 타입 검사도 불가능해진다는 점에 유의하자. 프로그래머는 `void *` 값을 기반 데이터와 호환되지 않는 타입으로 변환하지 않도록 유의해야 한다. 규정되지 않은 동작이 일어나게 되기 때문이다.

임의 타입의 값을 전달할 수 있으므로 함수 매개변수와 반환 값에 유용하다. `void` 포인터를 통해 함수로 전달되는 데이터의 진짜 타입을 알 수 있는 어떤 메커니즘이 있는 게 보통이다.

이 타입의 값을 따라갈 수 없다. 그러면 `void` 타입이 나올 텐데, 그건 불가능하기 때문이다. 마찬가지로 이 타입으로 포인터 산술을 할 수 없다. 하지만 GNU C에서는 표준 확장으로 포인터 산술을 허용한다. `void`나 함수의 크기를 1로 해서 동작한다. `void`와 함수 타입에 `sizeof`도 허용되며, 1을 반환한다.

<tt>[[printf(3)]]</tt> 및 <tt>[[scanf(3)]]</tt> 함수군에서 `void *`의 변환 지정자는 `p`이다.

*VERSIONS*: `void *`와 함수 포인터 간 호환성에 대한 POSIX 요구 사항은 POSIX.1-2008 기술 정오표 1(2013년)에서 추가되었다.

*CONFORMING TO*: C99 및 이후. POSIX.1-2001 및 이후.

*SEE ALSO*: <tt>[[malloc(3)]]</tt>, `memcmp(3)`, `memcpy(3)`, `memset(3)`

이 페이지의 `intptr_t` 및 `uintptr_t` 타입도 참고.

## NOTES

이 매뉴얼 페이지에서 기술하는 구조체들은 적어도 그 정의에 나와 있는 멤버들을 가지고 있다. 단, 그 순서는 정해져 있지 않다.

이 페이지에서 기술하는 정수 타입 대다수에는 <tt>[[printf(3)]]</tt> 및 <tt>[[scanf(3)]]</tt> 함수군을 위한 대응하는 길이 변환자가 없다. 길이 변환자가 없는 정수 타입의 값을 찍으려면 명시적 캐스팅으로 `intmax_t`나 `uintmax_t`로 변환하는 게 좋다. 길이 변환자가 없는 정수 타입의 변수로 탐색해서 넣으려면 `intmax_t`나 `uintmax_t` 타입의 중간 임시 변수를 쓰는 게 좋다. 그 임시 변수에서 대상 변수로 복사할 때 값이 넘칠 수도 있다. 타입에 상한 및 하한이 있다면 값을 실제 복사하기 전에 값이 그 범위 안에 있는지 검사해 보는 게 좋다. 아래 예에서 어떻게 그런 변환을 해야 하는지 보인다.

### 페이지 작성 방식

"CONFORMING TO"에서는 C99 및 이후, 그리고 POSIX.1-2001 및 이후만 고려한다. 일부 타입은 더 이른 표준 버전에 명세돼 있을 수도 있지만 단순함을 위해 더 이른 표준의 내용은 생략한다.

"INCLUDE"에서는 먼저 C나 POSIX.1 표준에 따라 그 타입을 정의하는 "주된" 헤더(들)을 적는다. "또는" 다음에는 그 타입을 정의하게 된다고 표준에서 명시한 추가 헤더들을 적는다.

## EXAMPLES

아래 프로그램은 길이 변환자가 없는 정수 타입을 문자열에서 탐색하고 변수에 저장된 값을 찍는다. 위의 NOTES 절에서 설명한 대로 적절히 `intmax_t`와 변환하고 적절히 범위 검사를 한다.

```c
#include <stdint.h>
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>

int
main (void)
{
    static const char *const str = "500000 us in half a second";
    suseconds_t us;
    intmax_t    tmp;

    /* 문자열에서 수를 탐색해서 임시 변수에 넣기. */

    sscanf(str, "%jd", &tmp);

    /* 값이 suseconds_t의 유효 범위에 있는지 확인하기. */

    if (tmp < -1 || tmp > 1000000) {
        fprintf(stderr, "Scanned value outside valid range!\n");
        exit(EXIT_FAILURE);
    }

    /* 그 값을 suseconds_t 변수 'us'로 복사하기. */

    us = tmp;

    /* suseconds_t가 -1 값을 담을 수 있긴 하지만 합리적인
       마이크로초 수는 아니다. */

    if (us < 0) {
        fprintf(stderr, "Scanned value shouldn't be negative!\n");
        exit(EXIT_FAILURE);
    }

    /* 값 찍기. */

    printf("There are %jd microseconds in half a second.\n",
            (intmax_t) us);

    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[feature_test_macros(7)]]</tt>, <tt>[[standards(7)]]</tt>

----

2021-03-22
