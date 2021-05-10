## NAME

close_range - 지정한 범위의 파일 디스크립터 모두 닫기

## SYNOPSIS

```c
#include <linux/close_range.h>

int close_range(unsigned int first, unsigned int last,
                unsigned int flags);
```

*주의*: 이 시스템 호출에 대한 glibc 래퍼가 없다. NOTES 참고.

## DESCRIPTION

`close_range()` 시스템 호출은 `first`에서 `last`까지 (경계 포함) 열린 파일 디스크립터들을 모두 닫는다.

현재는 파일 디스크립터를 닫는 중의 오류를 무시한다.

`flags`는 다음을 0개 이상 담은 비트 마스크다.

`CLOSE_RANGE_CLOEXEC` (리눅스 5.11부터)
:   지정한 파일 디스크립터들을 즉시 닫는 게 아니라 exec에서 닫기 플래그를 설정한다.

`CLOSE_RANGE_UNSHARE`
:   지정한 파일 디스크립터들을 닫기 전에 다른 프로세스와 공유 해제해서 파일 디스크립터 테이블을 공유하는 다른 스레드와의 경쟁을 피한다.

## RETURN VALUE

성공 시 `close_range()`는 0을 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EINVAL`
:   `flags`가 유효하지 않거나, `first`가 `last`보다 크다.

`CLOSE_RANGE_UNSHARE`에서 (새 디스크립터 테이블을 구성하면서) 다음 오류가 발생할 수 있다.

`EMFILE`
:   열린 파일 디스크립터 수가 `/proc/sys/fs/nr_open`에 지정된 제한을 초과한다. (<tt>[[proc(5)]]</tt> 참고.) `CLOSE_RANGE_UNSHARE` 플래그를 지정한 `close_range()` 호출에 앞서 그 제한을 낮춘 경우에 이 오류가 발생할 수 있다.

`ENOMEM`
:   사용 가능한 커널 메모리가 충분하지 않다.

## VERSIONS

리눅스 5.9에서 `close_range()`가 처음 등장했다.

## CONFORMING TO

`close_range()`는 비표준 함수이며 FreeBSD에도 있다.

## NOTES

glibc에서 이 시스템 호출의 래퍼를 제공하지 않는다. <tt>[[syscall(2)]]</tt>을 이용해 호출해야 한다.

### 모든 파일 디스크립터 닫기

가능한 파일 디스크립터 범위의 파일 디스크립터들을 무턱대고 닫는 방식을 피하기 위해 (리눅스에서) `/proc/self/fd/`의 열린 파일 디스크립터들을 나열해서 각각에 <tt>[[close(2)]]</tt>를 호출하는 경우가 있다. `close_range()`는 이를 `/proc` 필요 없이 한 번의 시스템 호출에서 대신 해 주며, 그래서 상당한 성능 이득을 준다.

### exec 전에 파일 디스크립터 닫기

다음처럼 해서 파일 디스크립터들을 안전하게 닫을 수 있다.

```c
/* stderr 다음은 다 필요 없는 경우 */
close(range(3, ~0U, CLOSE_RANGE_UNSHARE);
execve(....);
```

`CLOSE_RANGE_UNSHARE`는 개념적으로 다음과 동등하되 더 효율적이다.

```c
unshare(CLONE_FILES);
close_range(first, last, 0);
```

공유 해제 범위가 호출자의 파일 디스크립터 테이블에 현재 할당된 파일 디스크립터 최대 번호를 넘어가는 경우에 (흔히 하듯 `last`가 ~0U이면 그렇게 된다.) 커널에선 `first`까지만 호출자의 새 파일 디스크립터 테이블을 공유 해제해서 파일 디스크립터를 가급적 적게 복사한다. 그러면 이어지는 <tt>[[close(2)]]</tt> 호출이 아예 필요치 않게 된다. 즉, 테이블을 공유 해제하고 나면 전체 동작이 완료된다.

### exec 시 파일 닫기

**exec** 전의 여러 구성 단계들이 서로 충돌할 위험이 있는 경우에 특히 유용하다. 예를 들어 <tt>[[seccomp(2)]]</tt> 프로파일 설정이 `close_range()` 호출과 충돌할 수 있다. 만약 <tt>[[seccomp(2)]]</tt> 프로파일을 설정하기 전에 파일 디스크립터들을 닫는다면 프로파일 설정 단계에서 그 디스크립터들을 이용할 수도, 그리고 닫는 걸 통제할 수도 없다. 만약 설정 후에 파일 디스크립터들을 닫는다면 seccomp 프로파일에서 `close_range()` 호출 내지 대체 호출을 차단할 수 없다. `CLOSE_RANGE_CLOEXEC`를 쓰면 이 상황을 피할 수 있다. 디스크립터에 표시를 한 다음 <tt>[[seccomp(2)]]</tt> 프로파일을 설정하면 프로파일에서 호출 프로세스에 영향을 끼치지 않고 `close_range()` 접근을 통제할 수 있다.

## EXAMPLES

아래 프로그램은 명령행 인자가 가리키는 파일들을 열고, (`/proc/PID/fd`의 항목들을 순회해서) 연 파일들의 목록을 표시하고, `close_range()`로 3 이상의 파일 디스크립터를 모두 닫고, 다시 프로세스의 열린 파일 목록을 표시한다. 다음 예가 프로그램 사용 방식을 보여 준다.

```text
$ touch /tmp/a /tmp/b /tmp/c
$ ./a.out /tmp/a /tmp/b /tmp/c
/tmp/a opened as FD 3
/tmp/b opened as FD 4
/tmp/c opened as FD 5
/proc/self/fd/0 ==> /dev/pts/1
/proc/self/fd/1 ==> /dev/pts/1
/proc/self/fd/2 ==> /dev/pts/1
/proc/self/fd/3 ==> /tmp/a
/proc/self/fd/4 ==> /tmp/b
/proc/self/fd/5 ==> /tmp/b
/proc/self/fd/6 ==> /proc/9005/fd
========= About to call close_range() =======
/proc/self/fd/0 ==> /dev/pts/1
/proc/self/fd/1 ==> /dev/pts/1
/proc/self/fd/2 ==> /dev/pts/1
/proc/self/fd/3 ==> /proc/9005/fd
```

참고로 경로명 `/proc/9005/fd`가 나오는 행은 <tt>[[opendir(3)]]</tt> 호출로 인한 것이다.

### 프로그램 소스

```c
#define _GNU_SOURCE
#include <fcntl.h>
#include <linux/close_range.h>
#include <stdio.h>
#include <stdlib.h>
#include <sys/syscall.h>
#include <string.h>
#include <unistd.h>
#include <dirent.h>

/* /proc/self/fd의 심볼릭 링크들 내용 보여 주기 */

static void
show_fds(void)
{
    DIR *dirp = opendir("/proc/self/fd");
    if (dirp  == NULL) {
        perror("opendir");
        exit(EXIT_FAILURE);
    }

    for (;;) {
        struct dirent *dp = readdir(dirp);
        if (dp == NULL)
            break;

        if (dp->d_type == DT_LNK) {
            char path[PATH_MAX], target[PATH_MAX];
            snprintf(path, sizeof(path), "/proc/self/fd/%s",
                     dp->d_name);

            ssize_t len = readlink(path, target, sizeof(target));
            printf("%s ==> %.*s\n", path, (int) len, target);
        }
    }

    closedir(dirp);
}

int
main(int argc, char *argv[])
{
    for (int j = 1; j < argc; j++) {
        int fd = open(argv[j], O_RDONLY);
        if (fd == -1) {
            perror(argv[j]);
            exit(EXIT_FAILURE);
        }
        printf("%s opened as FD %d\n", argv[j], fd);
    }

    show_fds();

    printf("========= About to call close_range() =======\n");

    if (syscall(__NR_close_range, 3, ~0U, 0) == -1) {
        perror("close_range");
        exit(EXIT_FAILURE);
    }

    show_fds();
    exit(EXIT_FAILURE);
}
```

## SEE ALSO

<tt>[[close(2)]]</tt>

----

2021-03-22
