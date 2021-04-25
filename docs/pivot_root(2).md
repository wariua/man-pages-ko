## NAME

pivot_root - 루트 마운트 바꾸기

## SYNOPSIS

```c
int pivot_root(const char *new_root, const char *put_old);
```

*주의*: 이 시스템 호출에 대한 glibc 래퍼가 없다. NOTES 참고.

## DESCRIPTION

`pivot_root()`는 호출 프로세스의 마운트 스페이스에서 루트 마운트를 바꾼다. 더 정확하게는, 루트 마운트를 `put_old`로 옮기고 `new_root`를 새 루트 마운트로 만든다. 호출자의 마운트 네임스페이스를 소유한 사용자 네임스페이스에서 호출 프로세스가 `CAP_SYS_ADMIN` 역능을 가지고 있어야 한다.

`pivot_root()`는 같은 마운트 네임스페이스의 프로세스 내지 스레드 각각의 루트 디렉터리 및 현재 작업 디렉터리가 이전 루트 디렉터리를 가리키고 있으면 `new_root`로 바꾼다. (NOTES 절도 참고.) 한편으로 `pivot_root()`는 호출자의 작업 디렉터리를 (이전 루트 디렉터리에 있지 않았다면) 바꾸지 않으며, 그래서 뒤이어 `chdir("/")` 호출이 와야 한다.

다음 제약들이 적용된다.

* `new_root`와 `put_old`가 디렉터리여야 한다.

* `new_root`와 `put_old`가 현재 루트와 같은 마운트 상에 있어선 안 된다.

* `put_old`가 `new_root`와 같거나 그 아래에 있어야 한다. 즉, `put_old`가 가리키는 경로명 앞에 "`/..`"를 0개 이상 붙여서 `new_root`와 같은 디렉터리가 나와야 한다.

* `new_root`가 마운트 지점의 경로여야 하되, `"/"`일 수 없다. 마운트 지점이 아닌 경우에는 그 경로를 스스로에게 바인드 마운트 해서 마운트 지점으로 바꿀 수 있다.

* `new_root`의 부모 마운트 및 현재 작업 디렉터리의 부모 마운트의 전파 유형이 `MS_SHARED`여선 안 된다. 마찬가지로 `put_old`가 기존 마운트 지점인 경우 그 전파 유형이 `MS_SHARED`여선 안 된다. 이 제약은 `pivot_root()`로 인해 다른 마운트 네임스페이스로 어떤 변화도 전파되지 않게 한다.

* 현재 루트 디렉터리가 마운트 지점이어야 한다.

## RETURN VALUE

성공 시 0을 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

<tt>[[stat(2)]]</tt>과 같은 오류로 `pivot_root()`가 실패할 수 있다. 더불어 다음 오류로 실패할 수 있다.

`EBUSY`
:   `new_root`나 `put_old`가 현재 루트 마운트다. (이 오류는 `new_root`가 `"/"`인 이상한 경우를 포괄한다.)

`EINVAL`
:   `new_root`가 마운트 지점이 아니다.

`EINVAL`
:   `put_old`가 `new_root`에 또는 그 아래에 있지 않다.

`EINVAL`
:   (선행한 <tt>[[chroot(2)]]</tt> 때문에) 현재 루트 디렉터리가 마운트 지점이 아니다.

`EINVAL`
:   현재 루트가 rootfs(초기 ramfs) 마운트 상에 있다. NOTES 참고.

`EINVAL`
:   `new_root`의 마운트 지점이나 그 마운트 지점의 부모 마운트가 전파 유형이 `MS_SHARED`다.

`EINVAL`
:   `put_old`가 마운트 지점이고 전파 유형이 `MS_SHARED`다.

`ENOTDIR`
:   `new_root`나 `put_old`가 디렉터리가 아니다.

`EPERM`
:   호출 프로세스가 `CAP_SYS_ADMIN` 역능을 가지고 있지 않다.

## VERSIONS

리눅스 2.3.41에서 `pivot_root()`가 도입됐다.

## CONFORMING TO

`pivot_root()`는 리눅스 전용이므로 이식성이 없다.

## NOTES

glibc에서 이 시스템 호출의 래퍼를 제공하지 않는다. <tt>[[syscall(2)]]</tt>을 이용해 호출해야 한다.

`pivot_root(8)`가 이 시스템 호출의 명령행 인터페이스를 제공한다.

`pivot_root()`를 통해 호출자가 새 루트 파일 시스템으로 전환하면서 동시에 이전 루트 마운트를 `new_root` 아래 위치에 두게 되며, 이를 이후에 언마운트 할 수 있다. (루트 디렉터리나 현재 작업 디렉터리가 이전 루트 디렉터리인 프로세스들이 모두 새 루트로 옮겨지므로 이전 루트 디렉터리 사용자가 없게 돼서 이전 루트 마운트를 언마운트하기가 더 쉬워진다.)

`pivot_root()`를 쓰는 곳 중 하나가 시스템 시동 때인데, 시스템에서 임시 루트 파일 시스템(가령 <tt>[[initrd(4)]]</tt>)을 마운트 한 다음 실제 루트 파일 시스템을 마운트 하고서 마지막으로 후자를 유의미한 모든 프로세스 및 스레드의 루트 디렉터리가 되게 한다. 더 최신 용도는 컨테이너 생성 과정에서 루트 파일 시스템을 구성하는 것이다.

DESCRIPTION에서 언급한 방식으로 `pivot_root()`에서 프로세스의 루트 디렉터리 및 현재 작업 디렉터리를 변경하는 게 필요한 이유는 커널 스레드에서 파일 시스템에 전혀 접근하지 않는 경우에도 그 루트 디렉터리 및 현재 작업 디렉터리 때문에 이전 루트 마운트가 계속 사용 중이 되는 걸 막기 위해서다.

rootfs(초기 ramfs)를 `pivot_root()` 할 수 없다. 이 경우 루트 파일 시스템을 바꾸는 권장하는 방법은 rootfs에서 모든 걸 지우고, 새 root로 rootfs를 덮어서 마운트 하고, 새 `/dev/console`에 `stdin`/`stdout`/`stderr`를 붙이고, 새 `init(1)`을 exec 하는 것이다. 이 과정을 위한 헬퍼 프로그램들이 존재한다. `switch_root(8)` 참고.

### `pivot_root(".", ".")`

`new_root`와 `put_old`가 같은 디렉터리일 수도 있다. 특히 다음처럼 하면 임시 디렉터리를 만들고 지울 필요 없이 pivot-root 동작을 할 수 있다.

```c
chdir(new_root);
pivot_root(".", ".");
umount2(".", MNT_DETACH);
```

이 동작이 성공하는 건 `pivot_root()` 호출에서 `/`의 새 루트 마운트 지점 위에 이전 루트 마운트 지점을 겹쳐 두기 때문이다. 그 시점에 호출 프로세스의 루트 디렉터리 및 현재 작업 디렉터리는 새 루트 마운트 지점(`new_root`)을 가리킨다. 이어지는 `umount()` 호출에서 `"."` 해석이 `new_root`로 시작해서  `/`에 겹쳐진 마운트 목록을 따라 올라가서 결국 이전 루트 마운트 지점이 언마운트 된다.

### 역사

여러 해 동안 매뉴얼 페이지에 다음 내용이 있었다.

> 이전 루트 디렉터리를 쓰는 프로세스 내지 스레드의 현재 루트 및 현재 작업 디렉터리를 `pivot_root()`에서 바꿀 수도 있고 바꾸지 않을 수도 있다. `pivot_root()`를 호출하는 쪽에선 이전 루트를 루트나 현재 작업 디렉터리로 하는 프로세스가 어느 경우에도 올바르게 동작하도록 해야 한다. 그걸 보장하는 손쉬운 방법 하나는 `pivot_root()` 호출 전에 그 프로세스들의 루트 및 현재 작업 디렉터리를 `new_root`로 바꾸는 것이다.

커널의 시스템 호출 구현이 마무리되기도 전에 작성됐던 이 글에선 최종 릴리스 전에 구현 내용이 바뀔 수도 있다고 당시의 사용자들에게 경고하려 했던 것 같다. 하지만 이 시스템 호출이 처음 구현된 이후로 DESCRIPTION에 적힌 동작 방식은 한결같았으며 이제 바뀌지 않을 것이다.

## EXAMPLES

아래 프로그램은 <tt>[[clone(2)]]</tt>으로 만든 마운트 네임스페이스 안에서 `pivot_root()`를 쓰는 것을 보여 준다. 프로그램 첫 번째 명령행 인자에 지정된 루트 디렉터리로 전환한 다음 <tt>[[clone(2)]]</tt>으로 만든 자식에서 나머지 명령행 인자들로 지정된 프로그램을 실행한다.

프로그램 시연을 위해 새 루트 디렉터리 역할을 할 디렉터리를 하나 만들고 (정적으로 링크 된) `busybox(1)` 실행 파일 사본을 그 디렉터리에 둔다.

```text
$ mkdir /tmp/rootfs
$ ls -id /tmp/rootfs    # 새 루트 디렉터리의 아이노드 보기
319459 /tmp/rootfs
$ cp $(which busybox) /tmp/rootfs
$ PS1='bbsh$ ' sudo ./pivot_root_demo /tmp/rootfs /busybox sh
bbsh$ PATH=/
bbsh$ busybox ln busybox ln
bbsh$ ln busybox echo
bbsh$ ln busybox ls
bbsh$ ls
busybox  echo     ln       ls
bbsh$ ls -id /          # 위의 아이노드 번호와 비교해 보자
319459 /
bbsh$ echo 'hello world'
hello world
```

### 프로그램 소스

```c
/* pivot_root_demo.c */

#define _GNU_SOURCE
#include <sched.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>
#include <sys/syscall.h>
#include <sys/mount.h>
#include <sys/stat.h>
#include <limits.h>
#include <sys/mman.h>

#define errExit(msg)    do { perror(msg); exit(EXIT_FAILURE); \
                        } while (0)

static int
pivot_root(const char *new_root, const char *put_old)
{
    return syscall(SYS_pivot_root, new_root, put_old);
}

#define STACK_SIZE (1024 * 1024)

static int              /* 복제된 자식의 시작 함수 */
child(void *arg)
{
    char **args = arg;
    char *new_root = args[0];
    const char *put_old = "/oldrootfs";
    char path[PATH_MAX];

    /* 'new_root'와 그 부모 마운트가 공유 전파를 하지 않게
       (그렇게 하면 pivot_root()가 오류를 반환하게 된다.),
       그리고 마운트 이벤트가 최초 마운트 네임스페이스로
       전파되지 않게 하자. */

    if (mount(NULL, "/", NULL, MS_REC | MS_PRIVATE, NULL) == -1)
        errExit("mount-MS_PRIVATE");

    /* 'new_root'가 마운트 지점이어야 한다. */

    if (mount(new_root, new_root, NULL, MS_BIND, NULL) == -1)
        errExit("mount-MS_BIND");

    /* 이전 루트를 붙일 디렉터리 만들기. */

    snprintf(path, sizeof(path), "%s/%s", new_root, put_old);
    if (mkdir(path, 0777) == -1)
        errExit("mkdir");

    /* 루트 파일 시스템으로 전환하기. */

    if (pivot_root(new_root, path) == -1)
        errExit("pivot_root");

    /* 현재 작업 디렉터리를 "/"로 바꾸기. */

    if (chdir("/") == -1)
        errExit("chdir");

    /* 이전 루트를 언마운트 하고 마운트 지점을 제거하기. */

    if (umount2(put_old, MNT_DETACH) == -1)
        perror("umount2");
    if (rmdir(put_old) == -1)
        perror("rmdir");

    /* argv[1]...에 지정된 명령 실행하기. */

    execv(args[1], &args[1]);
    errExit("execv");
}

int
main(int argc, char *argv[])
{
    /* 새 마운트 네임스페이스에 자식 프로세스 만들기. */

    char *stack = mmap(NULL, STACK_SIZE, PROT_READ | PROT_WRITE,
                       MAP_PRIVATE | MAP_ANONYMOUS | MAP_STACK, -1, 0);
    if (stack == MAP_FAILED)
        errExit("mmap");

    if (clone(child, stack + STACK_SIZE,
                CLONE_NEWNS | SIGCHLD, &argv[1]) == -1)
        errExit("clone");

    /* 부모는 여기로 떨어진다. 자식을 기다리자. */

    if (wait(NULL) == -1)
        errExit("wait");

    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[chdir(2)]]</tt>, <tt>[[chroot(2)]]</tt>, <tt>[[mount(2)]]</tt>, <tt>[[stat(2)]]</tt>, <tt>[[initrd(4)]]</tt>, <tt>[[mount_namespaces(7)]]</tt>, `pivot_root(8)`, `switch_root(8)`

----

2021-03-22
