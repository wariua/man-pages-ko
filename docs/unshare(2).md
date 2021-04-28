## NAME

unshare - 프로세스 실행 문맥 일부를 분리하기

## SYNOPSIS

```c
#define _GNU_SOURCE
#include <sched.h>

int unshare(int flags);
```

## DESCRIPTION

`unshare()`를 통해 프로세스(또는 스레드)가 현재 다른 프로세스(또는 스레드)와 공유 중인 실행 문맥 일부를 분리시킬 수 있다. 실행 문맥 중에서 마운트 네임스페이스 같은 요소는 <tt>[[fork(2)]]</tt>나 <tt>[[vfork(2)]]</tt>로 새 프로세스를 만들 때 암묵적으로 공유되지만 가상 메모리 같은 다른 요소는 <tt>[[clone(2)]]</tt>으로 프로세스 내지 스레드를 만들 때 명시적으로 요청해서 공유할 수 있다.

`unshare()`의 주된 용도는 프로세스가 새 프로세스를 만들지 않고도 자기 공유 실행 문맥을 제어할 수 있게 하는 것이다.

`flags` 인자는 실행 문맥의 어느 요소들을 공유 해제할지 지정하는 비트 마스크이다. 다음 상수들을 0개 이상 OR 해서 인자를 지정한다.

`CLONE_FILES`
:   <tt>[[clone(2)]]</tt> `CLONE_FILES` 플래그의 효과를 뒤집는다. 파일 디스크립터 테이블을 공유 해제해서 호출 프로세스가 더이상 다른 프로세스와 파일 디스크립터를 공유하지 않게 한다.

`CLONE_FS`
:   <tt>[[clone(2)]]</tt> `CLONE_FS` 플래그의 효과를 뒤집는다. 파일 시스템 속성들을 공유 해제해서 호출 프로세스가 더이상 다른 프로세스와 루트 디렉터리(<tt>[[chroot(2)]]</tt>), 현재 디렉터리(`chdir(2)`), umask(<tt>[[umask(2)]]</tt>) 속성을 공유하지 않게 한다.

`CLONE_NEWCGROUP` (리눅스 4.6부터)
:   이 플래그는 <tt>[[clone(2)]]</tt> `CLONE_NEWCGROUP` 플래그와 효과가 같다. cgroup 네임스페이스를 공유 해제한다. `CLONE_NEWCGROUP`을 사용하려면 `CAP_SYS_ADMIN` 역능이 필요하다.

`CLONE_NEWIPC` (리눅스 2.6.19부터)
:   이 플래그는 <tt>[[clone(2)]]</tt> `CLONE_NEWIPC` 플래그와 효과가 같다. IPC 네임스페이스를 공유 해제해서 호출 프로세스가 다른 프로세스와 공유하지 않는 IPC 네임스페이스의 개별 사본을 가지도록 한다. 이 플래그 지정은 자동으로 `CLONE_SYSVSEM`까지 함의한다. `CLONE_NEWIPC`를 사용하려면 `CAP_SYS_ADMIN` 역능이 필요하다.

`CLONE_NEWNET` (리눅스 2.6.24부터)
:   이 플래그는 <tt>[[clone(2)]]</tt> `CLONE_NEWNET` 플래그와 효과가 같다. 네트워크 네임스페이스를 공유 해제해서 호출 프로세스가 기존 어느 프로세스와도 공유하지 않는 새 네트워크 네임스페이스로 이동하게 한다. `CLONE_NEWNET`을 사용하려면 `CAP_SYS_ADMIN` 역능이 필요하다.

`CLONE_NEWNS`
:   이 플래그는 <tt>[[clone(2)]]</tt> `CLONE_NEWNS` 플래그와 효과가 같다. 마운트 네임스페이스를 공유 해제해서 호출 프로세스가 다른 프로세스와 공유하지 않는 그 네임스페이스의 개별 사본을 가지도록 한다. 이 플래그 지정은 자동으로 `CLONE_FS`까지 함의한다. `CLONE_NEWNS`를 사용하려면 `CAP_SYS_ADMIN` 역능이 필요하다. 추가 내용은 <tt>[[mount_namespaces(7)]]</tt> 참고.

`CLONE_NEWPID` (리눅스 3.8부터)
:   이 플래그는 <tt>[[clone(2)]]</tt> `CLONE_NEWPID` 플래그와 효과가 같다. PID 네임스페이스를 공유 해제해서 호출 프로세스가 기존 어느 프로세스와도 공유하지 않는 자식들을 위한 새 PID 네임스페이스를 가지도록 한다. 호출 프로세스가 새 네임스페이스로 이동하지 *않는다*. 호출 프로세스가 생성하는 첫 번째 자식이 프로세스 ID 1을 가지게 되어 그 새 네임스페이스에서 `init(1)`의 역할을 맡는다. `CLONE_NEWPID`는 자동으로 `CLONE_THREAD`까지 함의한다. `CLONE_NEWPID`를 사용하려면 `CAP_SYS_ADMIN` 역능이 필요하다. 추가 내용은 <tt>[[pid_namespaces(7)]]</tt> 참고.

`CLONE_NEWTIME` (리눅스 5.6부터)
:   시간 네임스페이스 공유를 해제해서 호출 프로세스가 기존 어느 프로세스와도 공유하지 않는 자식 프로세스들을 위한 새 시간 네임스페이스를 가지도록 한다. 호출 프로세스는 새 네임스페이스로 이동하지 *않는다*. `CLONE_NEWTIME`을 사용하려면 `CAP_SYS_ADMIN` 역능이 필요하다. 추가 내용은 <tt>[[time_namespaces(7)]]</tt> 참고.

`CLONE_NEWUSER` (리눅스 3.8부터)
:   이 플래그는 <tt>[[clone(2)]]</tt> `CLONE_NEWUSER` 플래그와 효과가 같다. 사용자 네임스페이스를 공유 해제해서 호출 프로세스가 기존 어느 프로세스와도 공유하지 않는 새 사용자 네임스페이스로 이동하게 한다. `CLONE_NEWUSER` 플래그를 쓴 <tt>[[clone(2)]]</tt>으로 생성하는 자식 프로세스에서처럼 호출자가 새 네임스페이스에서 완전한 역능 집합을 얻는다.

    `CLONE_NEWUSER`를 위해선 호출 프로세스가 다중 스레드가 아니어야 한다. `CLONE_NEWUSER` 지정은 자동으로 `CLONE_THREAD`를 함의한다. 리눅스 3.9부터 `CLONE_NEWUSER`가 자동으로 `CLONE_FS`를 함의하기도 한다. `CLONE_NEWUSER`를 위해선 호출 프로세스의 사용자 ID 및 그룹 ID가 호출 시점의 호출 프로세스 사용자 네임스페이스 내의 사용자 ID 및 그룹 ID로 매핑 되어 있어야 한다.

    사용자 네임스페이스에 대한 추가 내용은 <tt>[[user_namespaces(7)]]</tt> 참고.

`CLONE_NEWUTS` (리눅스 2.6.19부터)
:   이 플래그는 <tt>[[clone(2)]]</tt> `CLONE_NEWUTS` 플래그와 효과가 같다. UTS 네임스페이스를 공유 해제해서 호출 프로세스가 다른 프로세스와 공유하지 않는 UTS 네임스페이스의 개별 사본을 가지도록 한다. `CLONE_NEWUTS`를 사용하려면 `CAP_SYS_ADMIN` 역능이 필요하다.

`CLONE_SYSVSEM` (리눅스 2.6.26부터)
:   이 플래그는 <tt>[[clone(2)]]</tt> `CLONE_SYSVSEM` 플래그의 효과를 뒤집는다. 시스템 V 세마포어 조정(`semadj`) 값들을 공유 해제해서 호출 프로세스가 다른 프로세스와 공유하지 않는 새로운 빈 `semadj` 목록을 가지도록 한다. 이 프로세스가 현재 `semadj` 목록에 대한 참조를 가진 마지막 프로세스이면 그 목록의 조정 사항들이 <tt>[[semop(2)]]</tt>에서 기술하는 대로 해당 세마포어들에 적용된다.

추가로 호출자가 단일 스레드이면 (즉 다른 프로세스 내지 스레드와 주소 공간을 공유하고 있지 않으면) `flags`에 `CLONE_THREAD`, `CLONE_SIGHAND`, `CLONE_VM`을 지정할 수 있다. 그 경우에 그 플래그들은 효과가 없다. (그리고 `CLONE_THREAD` 지정이 자동으로 `CLONE_VM`을 함의하고 `CLONE_VM` 지정이 자동으로 `CLONE_SIGHAND`를 함의하기도 한다.) 프로세스가 다중 스레드인 경우에는 이 플래그 사용 시 오류 결과가 나온다.

`flags`를 0으로 지정하는 경우 `unshare()`는 no-op이다. 현재 프로세스의 실행 문맥에 어떤 변경도 하지 않는다.

## RETURN VALUE

성공 시 0을 반환한다. 실패 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EINVAL`
:   `flags`에 유효하지 않은 비트를 지정했다.

`EINVAL`
:   `flags`에 `CLONE_THREAD`나 `CLONE_SIGHAND`, `CLONE_VM`을 지정했으며 호출자가 다중 스레드이다.

`EINVAL`
:   `flags`에 `CLONE_NEWIPC`를 지정했는데 커널이 `CONFIG_SYSVIPC` 및 `CONFIG_IPC_NS` 옵션으로 구성되지 않았다.

`EINVAL`
:   `flags`에 `CLONE_NEWNET`를 지정했는데 커널이 `CONFIG_NET_NS` 옵션으로 구성되지 않았다.

`EINVAL`
:   `flags`에 `CLONE_NEWPID`를 지정했는데 커널이 `CONFIG_PID_NS` 옵션으로 구성되지 않았다.

`EINVAL`
:   `flags`에 `CLONE_NEWUSER`를 지정했는데 커널이 `CONFIG_USER_NS` 옵션으로 구성되지 않았다.

`EINVAL`
:   `flags`에 `CLONE_NEWUTS`를 지정했는데 커널이 `CONFIG_UTS_NS` 옵션으로 구성되지 않았다.

`EINVAL`
:   `flags`에 `CLONE_NEWPID`를 지정했는데 프로세스가 앞서 `CLONE_NEWPID` 플래그로 `unshare()`를 호출했다.

`ENOMEM`
:   공유 해제해야 하는 호출자의 문맥 요소들을 복사할 충분한 메모리를 할당할 수 없다.

`ENOSPC` (리눅스 3.7부터)
:   `flags`에 `CLONE_NEWPID`를 지정했는데 PID 네임스페이스 중첩 깊이 제한을 초과하게 되었다. <tt>[[pid_namespaces(7)]]</tt> 참고.

`ENOSPC` (리눅스 4.9부터. 전에는 `EUSERS`)
:   `flags`에 `CLONE_NEWUSER`를 지정했는데 중첩 사용자 네임스페이스 수 제한을 초과하게 되었다. <tt>[[user_namespaces(7)]]</tt> 참고.

    리눅스 3.11부터 리눅스 4.8까지는 이 경우 진단 오류가 `EUSERS`였다.

`ENOSPC` (리눅스 4.9부터)
:   `flags`의 한 값이 새 사용자 네임스페이스 생성을 나타내지만 그렇게 하면 `/proc/sys/user` 안의 대응 파일에 규정된 제한을 초과하게 된다. 자세한 내용은 <tt>[[namespaces(7)]]</tt> 참고.

`EPERM`
:   호출 프로세스가 이 동작에 필요한 특권을 가지고 있지 않다.

`EPERM`
:   `flags`에 `CLONE_NEWUSER`를 지정했는데 호출자의 실효 사용자 ID나 실효 그룹 ID 중 하나가 부모 네임스페이스에 매핑 되어 있지 않다. (<tt>[[user_namespaces(7)]]</tt> 참고.)

`EPERM` (리눅스 3.9부터)
:   `flags`에 `CLONE_NEWUSER`를 지정했는데 호출자가 chroot 환경 안에 있다. (즉, 호출자의 루트 디렉터리가 호출자가 위치한 마운트 네임스페이스의 루트 디렉터리와 일치하지 않는다.)

`EUSERS` (리눅스 3.11부터 리눅스 4.8까지)
:   `flags`에 `CLONE_NEWUSER`를 지정했는데 중첩 사용자 네임스페이스 수 제한을 초과하게 되었다. 위의 `ENOSPC` 오류 설명 참고.

## VERSIONS

리눅스 커널 2.6.16에서 `unshare()` 시스템 호출이 추가되었다.

## CONFORMING TO

`unshare()` 시스템 호출은 리눅스 전용이다.

## NOTES

<tt>[[clone(2)]]</tt>으로 새 프로세스를 생성할 때 공유할 수 있는 프로세스 속성들을 모두 `unshare()`로 공유 해제할 수 있는 건 아니다. 구체적으로 커널 3.8 현재 `unshare()`는 `CLONE_SIGHAND`, `CLONE_THREAD`, `CLONE_VM`의 효과를 뒤집는 플래그를 구현하고 있지 않다. 필요하면 향후 그런 기능이 추가될 수도 있다.

## EXAMPLES

아래 프로그램은 `unshare(1)` 명령을 간단하게 구현한 것이다. 네임스페이스 한 가지 또는 그 이상을 공유 해제하고 명령행 인자로 받은 명령을 실행한다. 다음은 이 프로그램 사용 예시인데, 새 마운트 네임스페이스에서 셸을 실행하고서 원래 셸과 새 셸이 분리된 마운트 네임스페이스에 있는지 확인한다.

```text
$ readlink /proc/$$/ns/mnt
mnt:[4026531840]
$ sudo ./unshare -m /bin/bash
# readlink /proc/$$/ns/mnt
mnt:[4026532325]
```

두 `readlink(1)` 명령 출력이 다른데, 이는 두 셸이 다른 마운트 네임스페이스에 있음을 보여 준다.

### 프로그램 소스

```c
/* unshare.c

   간단한 unshare(1) 명령 구현: 네임스페이스를 unshare 하고
   명령을 실행
*/
#define _GNU_SOURCE
#include <sched.h>
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>

/* 간단한 오류 처리 함수: 'errno'의 값에 따라 오류 메시지를
   찍고 호출 프로세스를 종료한다. */
#define errExit(msg)    do { perror(msg); exit(EXIT_FAILURE); \
                        } while (0)

static void
usage(char *pname)
{
    fprintf(stderr, "Usage: %s [options] program [arg...]\n", pname);
    fprintf(stderr, "Options can be:\n");
    fprintf(stderr, "    -C   unshare cgroup namespace\n");
    fprintf(stderr, "    -i   unshare IPC namespace\n");
    fprintf(stderr, "    -m   unshare mount namespace\n");
    fprintf(stderr, "    -n   unshare network namespace\n");
    fprintf(stderr, "    -p   unshare PID namespace\n");
    fprintf(stderr, "    -t   unshare time namespace\n");
    fprintf(stderr, "    -u   unshare UTS namespace\n");
    fprintf(stderr, "    -U   unshare user namespace\n");
    exit(EXIT_FAILURE);
}

int
main(int argc, char *argv[])
{
    int flags, opt;

    flags = 0;

    while ((opt = getopt(argc, argv, "CimnptuU")) != -1) {
        switch (opt) {
        case 'C': flags |= CLONE_NEWCGROUP;     break;
        case 'i': flags |= CLONE_NEWIPC;        break;
        case 'm': flags |= CLONE_NEWNS;         break;
        case 'n': flags |= CLONE_NEWNET;        break;
        case 'p': flags |= CLONE_NEWPID;        break;
        case 't': flags |= CLONE_NEWTIME;       break;
        case 'u': flags |= CLONE_NEWUTS;        break;
        case 'U': flags |= CLONE_NEWUSER;       break;
        default:  usage(argv[0]);
        }
    }

    if (optind >= argc)
        usage(argv[0]);

    if (unshare(flags) == -1)
        errExit("unshare");

    execvp(argv[optind], &argv[optind]);
    errExit("execvp");
}
```

## SEE ALSO

`unshare(1)`, <tt>[[clone(2)]]</tt>, <tt>[[fork(2)]]</tt>, <tt>[[kcmp(2)]]</tt>, <tt>[[setns(2)]]</tt>, <tt>[[vfork(2)]]</tt>, <tt>[[namespaces(7)]]</tt>

리눅스 커널 소스 트리의 `Documentation/userspace-api/unshare.rst` (리눅스 4.12 전에는 `Documentation/unshare.txt`)

----

2021-03-22
