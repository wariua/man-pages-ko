## NAME

acct - 프로세스 통계 파일

## SYNOPSIS

```c
#include <sys/acct.h>
```

## DESCRIPTION

프로세스 통계 옵션(`CONFIG_BSD_PROCESS_ACCT`)을 켜서 커널을 빌드 한 경우에 다음과 같이 <tt>[[acct(2)]]</tt>를 호출하면 프로세스 통계 수집을 시작한다.

```c
acct("/var/log/pacct");
```

프로세스 통계를 켜면 시스템의 각 프로세스가 종료할 때마다 커널이 통계 파일에 레코드를 기록한다. 그 레코드는 종료 프로세스에 대한 정보를 담고 있으며 `<sys/acct.h>`에 다음과 같이 정의돼 있다.

```c
#define ACCT_COMM 16

typedef u_int16_t comp_t;

struct acct {
    char ac_flag;           /* 통계 플래그 */
    u_int16_t ac_uid;       /* 사용자 ID */
    u_int16_t ac_gid;       /* 그룹 ID */
    u_int16_t ac_tty;       /* 제어 터미널 */
    u_int32_t ac_btime;     /* 프로세스 생성 시각
                               (에포크 이후 초) */
    comp_t    ac_utime;     /* 사용자 CPU 시간 */
    comp_t    ac_stime;     /* 시스템 CPU 시간 */
    comp_t    ac_etime;     /* 경과 시간 */
    comp_t    ac_mem;       /* 평균 메모리 사용량 (kB) */
    comp_t    ac_io;        /* 전송한 문자 수 (사용 안 함) */
    comp_t    ac_rw;        /* 읽거나 쓴 블록 수 (사용 안 함) */
    comp_t    ac_minflt;    /* 마이너 페이지 폴트 */
    comp_t    ac_majflt;    /* 메이저 페이지 폴트 */
    comp_t    ac_swaps;     /* 스왑 개수 (사용 안 함) */
    u_int32_t ac_exitcode;  /* 프로세스 종료 상태
                               (wait(2) 참고) */
    char      ac_comm[ACCT_COMM+1];
                            /* 명령 이름 (마지막 실행 명령의
                               basename. 널 종료) */
    char      ac_pad[X];    /* 패딩 바이트 */
};

enum {          /* ac_flag 필드에 설정될 수 있는 비트들 */
    AFORK = 0x01,           /* fork는 실행하고 exec는 실행하지 않았음 */
    ASU   = 0x02,           /* 수퍼유저 특권 사용했음 */
    ACORE = 0x08,           /* 코어 덤프 했음 */
    AXSIG = 0x10            /* 시그널로 죽었음 */
};
```

데이터 타입 `comp_t`는 3비트 8진수 지수와 13비트 가수로 이뤄진 부동소수점 값이다. 다음과 같이 해서 이 타입의 값 `c`를 (long 형) 정수로 변환할 수 있다.

```c
v = (c & 0x1fff) << (((c >> 13) & 0x7) * 3);
```

`ac_utime`, `ac_stime`, `ac_etime` 필드는 "클럭 틱" 단위로 측정한 시간이다. 그 값을 `sysconf(_SC_CLK_TCK)`으로 나누면 초 단위로 바꿀 수 있다.

### 버전 3 통계 파일 형식

커널 2.6.8부터는 커널을 빌드 할 때 `CONFIG_BSD_PROCESS_ACCT_V3` 옵션을 설정하면 다른 버전으로 통계 파일을 만들어 낼 수 있다. 이 옵션이 설정돼 있으면 통계 파일에 기록되는 레코드에 몇 가지 필드가 추가되며 `ac_uid` 및 `ac_gid` 필드가 (리눅스 2.4 및 이후에서의 UID 및 GID 크기 증가에 맞춰) 16비트에서 32비트로 커진다. 레코드가 다음과 같이 정의돼 있다.

```c
struct acct_v3 {
    char      ac_flag;      /* 플래그 */
    char      ac_version;   /* 항상 ACCT_VERSION(3)으로 설정 */
    u_int16_t ac_tty;       /* 제어 터미널 */
    u_int32_t ac_exitcode;  /* 프로세스 종료 상태 */
    u_int32_t ac_uid;       /* 실제 사용자 ID */
    u_int32_t ac_gid;       /* 실제 그룹 ID */
    u_int32_t ac_pid;       /* 프로세스 ID */
    u_int32_t ac_ppid;      /* 부모 프로세스 ID */
    u_int32_t ac_btime;     /* 프로세스 생성 시각 */
    float     ac_etime;     /* 경과 시간 */
    comp_t    ac_utime;     /* 사용자 CPU 시간 */
    comp_t    ac_stime;     /* 시스템 시간 */
    comp_t    ac_mem;       /* 평균 메모리 사용량 (kB) */
    comp_t    ac_io;        /* 전송한 문자 수 (사용 안 함) */
    comp_t    ac_rw;        /* 읽거나 쓴 블록 수 (사용 안 함) */
    comp_t    ac_minflt;    /* 마이너 페이지 폴트 */
    comp_t    ac_majflt;    /* 메이저 페이지 폴트 */
    comp_t    ac_swaps;     /* 스왑 개수 (사용 안 함) */
    char      ac_comm[ACCT_COMM]; /* 명령 이름 */
};
```

## VERSIONS

`acct_v3` 구조체는 glibc 버전 2.6부터 정의돼 있다.

## CONFORMING TO

프로세스 통계는 BSD에서 유래한 것이다. 대다수 시스템에 있기는 하지만 표준화는 되어 있지 않아서 시스템마다 세부 내용이 다소 다르다.

## NOTES

통계 파일의 레코드들은 프로세스 종료 시간 순서로 들어가 있다.

커널 2.6.9까지에서는 NPTL 스레드 라이브러리로 생성된 스레드마다 따로 통계 레코드가 기록된다. 리눅스 2.6.10부터는 프로세스의 마지막 스레드 종료 때 프로세스 전체에 대한 통계 레코드 하나만 기록한다.

<tt>[[proc(5)]]</tt>에서 설명하는 `/proc/sys/kernel/acct` 파일에는 디스크 공간이 모자랄 때 프로세스 통계 기능의 동작 방식을 제어하는 설정들이 있다.

## SEE ALSO

`lastcomm(1)`, <tt>[[acct(2)]]</tt>, `accton(8)`, `sa(8)`

----

2021-03-22
