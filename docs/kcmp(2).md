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

<dl>
<dt><code>KCMP_FILE</code></dt>
<dd>
프로세스 <code>pid1</code> 내의 파일 디스크립터 <code>idx1</code>이 프로세스 <code>pid2</code> 내의 파일 디스크립터 <code>idx2</code>와 같은 열린 파일 기술 항목(<tt>[[open(2)]]</tt> 참고)을 가리키는지 확인한다. <tt>[[dup(2)]]</tt> (및 유사 함수), <tt>[[fork(2)]]</tt>, 도메인 소켓을 통한 파일 디스크립터 전달(<tt>[[unix(7)]]</tt> 참고)의 결과로 같은 열린 파일 기술 항목을 가리키는 두 개의 파일 디스크립터가 존재할 수 있다.
</dd>

<dt><code>KCMP_FILES</code></dt>
<dd>
프로세스들이 동일한 열린 파일 디스크립터 집합을 공유하는지 확인한다. <code>idx1</code> 및 <code>idx2</code> 인자는 무시한다. <tt>[[clone(2)]]</tt>의 <code>CLONE_FILES</code> 플래그 논의 참고.
</dd>

<dt><code>KCMP_FS</code></dt>
<dd>
프로세스들이 같은 파일 시스템 정보(즉 파일 모드 생성 마스크, 작업 디렉터리, 파일 시스템 루트)를 공유하는지 확인한다. <code>idx1</code> 및 <code>idx2</code> 인자는 무시한다. <tt>[[clone(2)]]</tt>의 <code>CLONE_FS</code> 플래그 논의 참고.
</dd>

<dt><code>KCMP_IO</code></dt>
<dd>
프로세스들이 I/O 문맥을 공유하는지 확인한다. <code>idx1</code> 및 <code>idx2</code> 인자는 무시한다. <tt>[[clone(2)]]</tt>의 <code>CLONE_IO</code> 플래그 논의 참고.
</dd>

<dt><code>KCMP_SIGHAND</code></dt>
<dd>
프로세스들이 동일한 시그널 처리 방식 테이블을 공유하는지 확인한다. <code>idx1</code> 및 <code>idx2</code> 인자는 무시한다. <tt>[[clone(2)]]</tt>의 <code>CLONE_SIGHAND</code> 플래그 논의 참고.
</dd>

<dt><code>KCMP_SYSVSEM</code></dt>
<dd>
프로세스들이 동일한 시스템 V 세마포어 작업 취소 목록을 공유하는지 확인한다. <code>idx1</code> 및 <code>idx2</code> 인자는 무시한다. <tt>[[clone(2)]]</tt>의 <code>CLONE_SYSVSEM</code> 플래그 논의 참고.
</dd>

<dt><code>KCMP_VM</code></dt>
<dd>
프로세스들이 동일한 주소 공간을 공유하는지 확인한다. <code>idx1</code> 및 <code>idx2</code> 인자는 무시한다. <tt>[[clone(2)]]</tt>의 <code>CLONE_VM</code> 플래그 논의 참고.
</dd>

<dt><code>KCMP_EPOLL_TFD</code></dt>
<dd>

프로세스 <code>pid1</code>의 파일 디스크립터 <code>idx1</code>이 프로세스 <code>pid2</code>의 <code>idx2</code>가 기술하는 <tt>[[epoll(7)]]</tt> 인스턴스 내에 있는지 확인한다. <code>idx2</code> 인자는 대상 파일을 기술하는 구조체에 대한 포인터이다. 그 구조체는 다음 형태이다.

```c
struct kcmp_epoll_slot {
    __u32 efd;
    __u32 tfd;
    __u64 toff;
};
```

이 구조체에서 <code>efd</code>는 <tt>[[epoll_create(2)]]</tt>가 반환한 epoll 파일 디스크립터이고, <code>tfd</code>는 대상 파일 디스크립터 번호이고, <code>toff</code>는 0부터 세는 대상 파일 오프셋이다. 같은 파일 디스크립터 번호로 서로 다른 여러 대상을 등록할 수 있기 때문에 오프셋 지정이 각각을 조사하는 데 도움이 된다.
</dd>
</dl>

참고로 `kcmp()`는 프로세스들이 현재 동작 중인 경우 발생할 수 있는 거짓 양성에 대한 대비가 되어 있지 않다. 유의미한 결과를 얻으려면 이 시스템 호출로 조사하기에 앞서 `SIGSTOP`을 보내서 (<tt>[[signal(7)]]</tt> 참고) 프로세스를 정지시켜야 한다.

## RETURN VALUE

`kcmp()` 성공 호출의 반환 값은 커널 포인터들의 산술 비교 결과이다. (커널에서 자원을 비교할 때 자원의 메모리 주소를 사용한다.)

가장 쉬운 설명 방법은 예를 드는 것이다. `v1`과 `v2`가 적당한 자원 주소라고 할 때 반환 값은 다음 중 하나이다.

<dl>
<dt>0</dt>
<dd><code>v1</code>이 <code>v2</code>와 같다. 다시 말해 두 프로세스가 자원을 공유한다.</dd>
<dt>1</dt>
<dd><code>v1</code>이 <code>v2</code>보다 작다.</dd>
<dt>2</dt>
<dd><code>v1</code>이 <code>v2</code>보다 크다.</dd>
<dt>3</dt>
<dd><code>v1</code>이 <code>v2</code>와 같지 않되, 순서에 대한 정보가 없다.</dd>
</dl>

오류 시 -1을 반환하며 `errno`를 적절히 설정한다.

`kcmp()`는 정렬에 적합한 값을 반환하도록 설계되었다. 많은 수의 파일 디스크립터를 비교해야 할 때 이 점이 특히 편리하다.

## ERRORS

<dl>
<dt><code>EBADF</code></dt>
<dd><code>type</code>이 <code>KCMP_FILE</code>이며 <code>fd1</code>이나 <code>fd2</code>가 열린 파일 디스크립터가 아니다.</dd>
<dt><code>EINVAL</code></dt>
<dd><code>type</code>이 유효하지 않다.</dd>
<dt><code>EPERM</code></dt>
<dd>프로세스 자원을 조사할 권한이 부족하다. 소유하고 있지 않은 프로세스를 조사하려면 <code>CAP_SYS_PTRACE</code> 역능이 필요하다. 그리고 다른 ptrace 제약이 적용될 수도 있다. 가령 <code>CONFIG_SECURITY_YAMA</code>에서는 <code>/proc/sys/kernel/yama/ptrace_scope</code>가 2일 때 <code>kcmp()</code>를 자식 프로세스들로 제한한다. <tt>[[ptrace(2)]]</tt> 참고.</dd>
<dt><code>ESRCH</code></dt>
<dd>프로세스 <code>pid1</code>이나 <code>pid2</code>가 존재하지 않는다.</dd>
<dt><code>EFAULT</code></dt>
<dd><code>idx2</code>가 가리키는 epoll 슬롯이 사용자의 주소 공간 밖에 있다.</dd>
<dt><code>ENOENT</code></dt>
<dd><tt>[[epoll(7)]]</tt> 인스턴스에 대상 파일이 존재하지 않는다.</dd>
</dl>

## VERSIONS

리눅스 3.5에서 `kcmp()` 시스템 호출이 처음 등장했다.

## CONFORMING TO

`kcmp()`는 리눅스 전용이므로 이식성이 있어야 하는 프로그램에서는 사용하지 말아야 한다.

## NOTES

glibc에서 이 시스템 호출의 래퍼를 제공하지 않는다. <tt>[[syscall(2)]]</tt>을 이용해 호출해야 한다.

커널을 `CONFIG_CHECKPOINT_RESTORE`로 구성한 경우에만 이 시스템 호출이 사용 가능하다. 이 시스템 호출의 주 사용처는 사용자 공간 체크포인트/복원(CRIU) 기능이다. 이 시스템 호출의 대안으로는 <tt>[[proc(5)]]</tt> 파일 시스템을 통해 적절한 프로세스 정보를 노출하는 것이 있을 텐데, 보안적 이유로 적절하지 않다고 보았다.

이 페이지에서 언급하는 공유 자원들에 대한 배경 정보는 <tt>[[clone(2)]]</tt>을 보라.

## EXAMPLE

아래 프로그램은 `kcmp()`를 이용해 파일 디스크립터 쌍들이 같은 열린 파일 기술 항목을 가리키는지 검사한다. 프로그램 출력에 나와 있는 것처럼 다양한 파일 디스크립터 쌍 경우들을 검사한다. 프로그램 실행 예는 다음과 같다.

```
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
test_kcmp(char *msg, id_t pid1, pid_t pid2, int fd_a, int fd_b)
{
    printf("\t%s\n", msg);
    printf("\t\tkcmp(%ld, %ld, KCMP_FILE, %d, %d) ==> %s\n",
            (long) pid1, (long) pid2, fd_a, fd_b,
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

    printf("Parent PID is %ld\n", (long) getpid());
    printf("Parent opened file on FD %d\n\n", fd1);

    switch (fork()) {
    case -1:
        errExit("fork");

    case 0:
        printf("PID of child of fork() is %ld\n", (long) getpid());

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

2019-03-06
