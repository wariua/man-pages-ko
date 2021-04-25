## NAME

kcmp - 두 프로세스 비교해서 커널 자원 공유 여부 알아내기

## SYNOPSIS

```c
#include <linux/kcmp.h>

int kcmp(pid_t pid1, pid_t pid2, int type,
         unsigned long idx1, unsigned long idx2);
```

*주의*: 이 시스템 호출에 대한 glibc 래퍼가 없다. NOTES 참고.

## DESCRIPTION

`kcmp()` 시스템 호출을 이용해 `pid1`과 `pid2`로 나타낸 두 프로세스가 가상 메모리나 파일 디스크립터 등의 커널 자원을 공유하는지 확인할 수 있다.

`kcmp()` 이용 권한은 `pid1`과 `pid2` 모두에 대한 ptrace 접근 모드 `PTRACE_MODE_READ_REALCREDS` 검사에 따라 결정된다. <tt>[[ptrace(2)]]</tt> 참고.

`type` 인자는 두 프로세스에서 어떤 자원을 비교할지 나타낸다. 다음 값들 중 하나이다.

`KCMP_FILE`
:   프로세스 `pid1` 내의 파일 디스크립터 `idx1`이 프로세스 `pid2` 내의 파일 디스크립터 `idx2`와 같은 열린 파일 기술 항목(<tt>[[open(2)]]</tt> 참고)을 가리키는지 확인한다. <tt>[[dup(2)]]</tt> (및 유사 함수), <tt>[[fork(2)]]</tt>, 도메인 소켓을 통한 파일 디스크립터 전달(<tt>[[unix(7)]]</tt> 참고)의 결과로 같은 열린 파일 기술 항목을 가리키는 두 개의 파일 디스크립터가 존재할 수 있다.

`KCMP_FILES`
:   프로세스들이 동일한 열린 파일 디스크립터 집합을 공유하는지 확인한다. `idx1` 및 `idx2` 인자는 무시한다. <tt>[[clone(2)]]</tt>의 `CLONE_FILES` 플래그 설명 참고.

`KCMP_FS`
:   프로세스들이 같은 파일 시스템 정보(즉 파일 모드 생성 마스크, 작업 디렉터리, 파일 시스템 루트)를 공유하는지 확인한다. `idx1` 및 `idx2` 인자는 무시한다. <tt>[[clone(2)]]</tt>의 `CLONE_FS` 플래그 설명 참고.

`KCMP_IO`
:   프로세스들이 I/O 문맥을 공유하는지 확인한다. `idx1` 및 `idx2` 인자는 무시한다. <tt>[[clone(2)]]</tt>의 `CLONE_IO` 플래그 설명 참고.

`KCMP_SIGHAND`
:   프로세스들이 동일한 시그널 처리 방식 테이블을 공유하는지 확인한다. `idx1` 및 `idx2` 인자는 무시한다. <tt>[[clone(2)]]</tt>의 `CLONE_SIGHAND` 플래그 설명 참고.

`KCMP_SYSVSEM`
:   프로세스들이 동일한 시스템 V 세마포어 작업 취소 목록을 공유하는지 확인한다. `idx1` 및 `idx2` 인자는 무시한다. <tt>[[clone(2)]]</tt>의 `CLONE_SYSVSEM` 플래그 설명 참고.

`KCMP_VM`
:   프로세스들이 동일한 주소 공간을 공유하는지 확인한다. `idx1` 및 `idx2` 인자는 무시한다. <tt>[[clone(2)]]</tt>의 `CLONE_VM` 플래그 설명 참고.

`KCMP_EPOLL_TFD`
:   프로세스 `pid1`의 파일 디스크립터 `idx1`이 프로세스 `pid2`의 `idx2`가 기술하는 <tt>[[epoll(7)]]</tt> 인스턴스 내에 있는지 확인한다. `idx2` 인자는 대상 파일을 기술하는 구조체에 대한 포인터이다. 그 구조체는 다음 형태이다.

        struct kcmp_epoll_slot {
            __u32 efd;
            __u32 tfd;
            __u64 toff;
        };

    이 구조체에서 `efd`는 <tt>[[epoll_create(2)]]</tt>가 반환한 epoll 파일 디스크립터이고, `tfd`는 대상 파일 디스크립터 번호이고, `toff`는 0부터 세는 대상 파일 오프셋이다. 같은 파일 디스크립터 번호로 서로 다른 여러 대상을 등록할 수 있기 때문에 오프셋 지정이 각각을 조사하는 데 도움이 된다.

참고로 `kcmp()`는 프로세스들이 현재 동작 중인 경우 발생할 수 있는 거짓 양성에 대한 대비가 되어 있지 않다. 유의미한 결과를 얻으려면 이 시스템 호출로 조사하기에 앞서 `SIGSTOP`을 보내서 (<tt>[[signal(7)]]</tt> 참고) 프로세스를 정지시켜야 한다.

## RETURN VALUE

`kcmp()` 성공 호출의 반환 값은 커널 포인터들의 산술 비교 결과이다. (커널에서 자원을 비교할 때 자원의 메모리 주소를 사용한다.)

가장 쉬운 설명 방법은 예를 드는 것이다. `v1`과 `v2`가 적당한 자원 주소라고 할 때 반환 값은 다음 중 하나이다.

| | |
| --- | --- |
| 0 | `v1`이 `v2`와 같다. 다시 말해 두 프로세스가 자원을 공유한다. |
| 1 | `v1`이 `v2`보다 작다. |
| 2 | `v1`이 `v2`보다 크다. |
| 3 | `v1`이 `v2`와 같지 않되, 순서에 대한 정보가 없다. |

오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

`kcmp()`는 정렬에 적합한 값을 반환하도록 설계되었다. 많은 수의 파일 디스크립터를 비교해야 할 때 이 점이 특히 편리하다.

## ERRORS

`EBADF`
:   `type`이 `KCMP_FILE`이며 `fd1`이나 `fd2`가 열린 파일 디스크립터가 아니다.

`EFAULT`
:   `idx2`가 가리키는 epoll 슬롯이 사용자의 주소 공간 밖에 있다.

`EINVAL`
:   `type`이 유효하지 않다.

`ENOENT`
:   <tt>[[epoll(7)]]</tt> 인스턴스에 대상 파일이 존재하지 않는다.

`EPERM`
:   프로세스 자원을 조사할 권한이 부족하다. 소유하고 있지 않은 프로세스를 조사하려면 `CAP_SYS_PTRACE` 역능이 필요하다. 그리고 다른 ptrace 제약이 적용될 수도 있다. 가령 `CONFIG_SECURITY_YAMA`에서는 `/proc/sys/kernel/yama/ptrace_scope`가 2일 때 `kcmp()`를 자식 프로세스들로 제한한다. <tt>[[ptrace(2)]]</tt> 참고.

`ESRCH`
:   프로세스 `pid1`이나 `pid2`가 존재하지 않는다.

## VERSIONS

리눅스 3.5에서 `kcmp()` 시스템 호출이 처음 등장했다.

## CONFORMING TO

`kcmp()`는 리눅스 전용이므로 이식성이 있어야 하는 프로그램에서는 사용하지 말아야 한다.

## NOTES

glibc에서 이 시스템 호출의 래퍼를 제공하지 않는다. <tt>[[syscall(2)]]</tt>을 이용해 호출해야 한다.

리눅스 5.12 전에선 커널을 `CONFIG_CHECKPOINT_RESTORE`로 구성한 경우에만 이 시스템 호출이 사용 가능하다. 이 시스템 호출의 주 사용처가 사용자 공간 체크포인트/복원(CRIU) 기능이었기 때문이다. (이 시스템 호출의 대안으로는 <tt>[[proc(5)]]</tt> 파일 시스템을 통해 적절한 프로세스 정보를 노출하는 것이 있을 텐데, 보안적 이유로 적절하지 않다고 보았다.) 리눅스 5.12부터 이 시스템 호출이 무조건 사용 가능해졌다.

이 페이지에서 언급하는 공유 자원들에 대한 배경 정보는 <tt>[[clone(2)]]</tt>을 보라.

## EXAMPLES

아래 프로그램은 `kcmp()`를 이용해 파일 디스크립터 쌍들이 같은 열린 파일 기술 항목을 가리키는지 검사한다. 프로그램 출력에 나와 있는 것처럼 다양한 파일 디스크립터 쌍 경우들을 검사한다. 프로그램 실행 예는 다음과 같다.

```text
$ ./a.out
Parent PID is 1144
Parent opened file on FD 3

PID of child of fork() is 1145
     Compare duplicate FDs from different processes:
          kcmp(1145, 1144, KCMP_FILE, 3, 3) ==> same
Child opened file on FD 4
     Compare FDs from distinct open()s in same process:
          kcmp(1145, 1145, KCMP_FILE, 3, 4) ==> different
Child duplicated FD 3 to create FD 5
     Compare duplicated FDs in same process:
          kcmp(1145, 1145, KCMP_FILE, 3, 5) ==> same
```

### 프로그램 소스

```c
#include _GNU_SOURCE
#include <sys/syscall.h>
#include <sys/wait.h>
#include <sys/stat.h>
#include <stdint.h>
#include <stdlib.h>
#include <stdio.h>
#include <unistd.h>
#include <fcntl.h>
#include <linux/kcmp.h>

#define errExit(msg)    do { perror(msg); exit(EXIT_FAILURE); \
                        } while (0)

static int
kcmp(pid_t pid1, pid_t pid2, int type,
     unsigned long idx1, unsigned long idx2)
{
    return syscall(SYS_kcmp, pid1, pid2, type, idx1, idx2);
}

static void
test_kcmp(char *msg, pid_t pid1, pid_t pid2, int fd_a, int fd_b)
{
    printf("\t%s\n", msg);
    printf("\t\tkcmp(%jd, %jd, KCMP_FILE, %d, %d) ==> %s\n",
            (intmax_t) pid1, (intmax_t) pid2, fd_a, fd_b,
            (kcmp(pid1, pid2, KCMP_FILE, fd_a, fd_b) == 0) ?
                        "same" : "different");
}

int
main(int argc, char *argv[])
{
    int fd1, fd2, fd3;
    char pathname[] = "/tmp/kcmp.text";

    fd1 = open(pathname, O_CREAT | O_RDWR, S_IRUSR | S_IWUSR);
    if (fd1 == -1)
        errExit("open");

    printf("Parent PID is %jd\n", (intmax_t) getpid());
    printf("Parent opened file on FD %d\n\n", fd1);

    switch (fork()) {
    case -1:
        errExit("fork");

    case 0:
        printf("PID of child of fork() is %jd\n", (intmax_t) getpid());

        test_kcmp("Compare duplicate FDs from different processes:",
                getpid(), getppid(), fd1, fd1);

        fd2 = open(pathname, O_CREAT | O_RDWR, S_IRUSR | S_IWUSR);
        if (fd2 == -1)
            errExit("open");
        printf("Child opened file on FD %d\n", fd2);

        test_kcmp("Compare FDs from distinct open()s in same process:",
                getpid(), getpid(), fd1, fd2);

        fd3 = dup(fd1);
        if (fd3 == -1)
            errExit("dup");
        printf("Child duplicated FD %d to create FD %d\n", fd1, fd3);

        test_kcmp("Compare duplicated FDs in same process:",
                getpid(), getpid(), fd1, fd3);
        break;

    default:
        wait(NULL);
    }

    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[clone(2)]]</tt>, <tt>[[unshare(2)]]</tt>

----

2021-03-22
