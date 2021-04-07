## NAME

setns - 스레드를 네임스페이스로 재연계하기

## SYNOPSIS

```c
#define _GNU_SOURCE         /* feature_test_macros(7) 참고 */
#include <sched.h>

int setns(int fd, int nstype);
```

## DESCRIPTION

네임스페이스를 가리키는 파일 디스크립터를 받아서 호출 스레드를 그 네임스페이스로 다시 연계한다.

`fd` 인자는 `/proc/[pid]/ns/` 디렉터리 안의 네임스페이스 항목들 중 하나를 가리키는 파일 디스크립터이다. `/proc/[pid]/ns/`에 대한 추가 정보는 <tt>[[namespaces(7)]]</tt>를 보라. 대응하는 네임스페이스로 호출 스레드가 재연계될 때 `nstype` 인자별 제약이 있으면 적용을 받는다.

`nstype` 인자는 호출 스레드를 어떤 종류의 네임스페이스로 재연계할 수 있는지 나타낸다. 이 인자는 다음 값들 중 하나일 수 있다.

<dl>
<dt><code>0</code></dt>
<dd>어떤 종류의 네임스페이스라도 참여를 허용한다.</dd>
<dt><code>CLONE_NEWCGROUP</code> (리눅스 4.6부터)</dt>
<dd><code>fd</code>가 cgroup 네임스페이스를 가리켜야 한다.</dd>
<dt><code>CLONE_NEWIPC</code> (리눅스 3.0부터)</dt>
<dd><code>fd</code>가 IPC 네임스페이스를 가리켜야 한다.</dd>
<dt><code>CLONE_NEWNET</code> (리눅스 3.0부터)</dt>
<dd><code>fd</code>가 네트워크 네임스페이스를 가리켜야 한다.</dd>
<dt><code>CLONE_NEWNS</code> (리눅스 3.8부터)</dt>
<dd><code>fd</code>가 마운트 네임스페이스를 가리켜야 한다.</dd>
<dt><code>CLONE_NEWPID</code> (리눅스 3.8부터)</dt>
<dd><code>fd</code>가 자손 PID 네임스페이스를 가리켜야 한다.</dd>
<dt><code>CLONE_NEWUSER</code> (리눅스 3.8부터)</dt>
<dd><code>fd</code>가 사용자 네임스페이스를 가리켜야 한다.</dd>
<dt><code>CLONE_NEWUTS</code> (리눅스 3.0부터)</dt>
<dd><code>fd</code>가 UTS 네임스페이스를 가리켜야 한다.</dd>
</dl>

`fd`가 어떤 종류의 네임스페이스를 가리키는지 호출자가 알고 있다면 (또는 신경쓰지 않는다면) `nstype`을 0으로 지정하면 충분하다. `nstype`을 0 아닌 값으로 지정하는 게 유용한 경우는 `fd`가 가리키는 네임스페이스의 종류를 호출자가 알지 못하고 그 네임스페이스가 특정 종류임을 보장하고 싶을 때이다. (다른 프로세스가 파일 디스크립터를 열고서 가령 유닉스 도메인 소켓을 통해 호출자에게 전달했다면 `fd`가 가리키는 네임스페이스 종류를 호출자가 알지 못할 수도 있다.)

### 각 네임스페이스 종류의 세부 사항

각 네임스페이스 종류로 재연계할 때 다음 세부 사항 및 제약에 유의해야 한다.

<dl>
<dt>사용자 네임스페이스</dt>
<dd>

스스로를 사용자 네임스페이스에 재연계하려는 프로세스는 대상 사용자 네임스페이스 내에서 `CAP_SYS_ADMIN` 역능이 있어야 한다. (이 때문에 자손 사용자 네임스페이스에만 참여가 가능하다.) 프로세스가 사용자 네임스페이스에 성공적으로 참여하면 사용자 ID 및 그룹 ID와 상관없이 그 네임스페이스 내의 모든 역능이 인가된다.

다중 스레드 프로세스는 `setns()`로 사용자 네임스페이스를 바꿀 수 없다.

`setns()`를 이용해 호출자의 현재 사용자 네임스페이스로 재진입하는 것이 허용되지 않는다. 이는 역능을 버린 호출자가 `setns()` 호출을 통해 그 역능을 다시 얻는 것을 막는다.

보안적 이유로 한 프로세스가 다른 프로세스와 파일 시스템 관련 속성(<tt>[[clone(2)]]</tt> `CLONE_FS` 플래그로 공유를 제어하는 속성들)을 공유하고 있으면 새 사용자 네임스페이스에 참여할 수 없다.

사용자 네임스페이스에 대한 더 자세한 내용은 <tt>[[user_namespaces(7)]]</tt>를 보라.
</dd>

<dt>마운트 네임스페이스</dt>
<dd>

마운트 네임스페이스를 바꾸기 위해선 호출자 프로세스가 자기 사용자 네임스페이스에서 `CAP_SYS_CHROOT` 및 `CAP_SYS_ADMIN` 역능을 가지고 있고 대상 마운트 네임스페이스를 소유한 사용자 네임스페이스에서 `CAP_SYS_ADMIN` 역능을 가지고 있어야 한다.

프로세스가 다중 스레드이면 새 마운트 네임스페이스로 재연계할 수 없다.

사용자 네임스페이스와 마운트 네임스페이스의 상호작용에 대한 자세한 내용은 <tt>[[user_namespaces(7)]]</tt>를 보라.
</dd>

<dt>PID 네임스페이스</dt>
<dd>

새 PID 네임스페이스에 재연계하기 위해선 호출자가 자기 사용자 네임스페이스와 대상 PID 네임스페이스를 소유한 사용자 네임스페이스 모두에서 <code>CAP_SYS_ADMIN</code> 역능을 가지고 있어야 한다.

`fd`가 PID 네임스페이스를 가리키는 경우 다른 네임스페이스들과 동작 방식이 좀 다르다. 호출 스레드에 PID 네임스페이스를 재연계하면 이후 생성되는 호출자의 자식 프로세스들이 들어갈 PID 네임스페이스를 바꿀 뿐이다. 즉, 호출자 자체의 PID 네임스페이스는 바뀌지 않는다.

`fd`로 지정한 PID 네임스페이스가 호출자의 PID 네임스페이스의 자손(자식, 손자, 등)인 경우에만 PID 네임스페이스 재연계가 허용된다.

PID 네임스페이스에 대한 더 자세한 내용은 <tt>[[pid_namespaces(7)]]</tt>를 보라.
</dd>

<dt>cgroup 네임스페이스</dt>
<dd>

새 cgroup 네임스페이스에 재연계하기 위해선 호출자가 자기 사용자 네임스페이스와 대상 cgroup 네임스페이스를 소유한 사용자 네임스페이스 모두에서 <code>CAP_SYS_ADMIN</code> 역능을 가지고 있어야 한다.

`setns()`로 호출자의 cgroup 네임스페이스를 바꾸어도 호출자의 cgroup 멤버십은 바뀌지 않는다.
</dd>

<dt>네트워크, IPC, UTS 네임스페이스</dt>
<dd>
새 네트워크 내지 IPC, UTS 네임스페이스에 재연계하기 위해선 호출자가 자기 사용자 네임스페이스와 대상 네임스페이스를 소유한 사용자 네임스페이스 모두에서 <code>CAP_SYS_ADMIN</code> 역능을 가지고 있어야 한다.
</dd>

## RETURN VALUE

성공 시 `setns()`는 0을 반환한다. 실패 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

<dl>
<dt><code>EBADF</code></dt>
<dd><code>fd</code>가 유효한 파일 디스크립터가 아니다.</dd>
<dt><code>EINVAL</code></dt>
<dd><code>fd</code>가 가리키는 네임스페이스의 종류가 <code>nstype</code>에 지정한 것과 일치하지 않는다.</dd>
<dt><code>EINVAL</code></dt>
<dd>지정한 네임스페이스로 스레드를 재연계하는 데 문제가 있다.</dd>
<dt><code>EINVAL</code></dt>
<dd>호출자가 선조(부모, 조부모, 등) PID 네임스페이스에 참여하려고 했다.</dd>
<dt><code>EINVAL</code></dt>
<dd>호출자가 이미 참여해 있는 사용자 네임스페이스에 참여하려고 시도했다.</dd>
<dt><code>EINVAL</code></dt>
<dd>호출자가 다른 프로세스와 파일 시스템(<code>CLONE_FS</code>) 상태를 (특히 루트 디렉터리를) 공유하면서 새 사용자 네임스페이스에 참여하려고 했다.</dd>
<dt><code>EINVAL</code></dt>
<dd>호출자가 다중 스레드인데 새 사용자 네임스페이스에 참여하려고 했다.</dd>
<dt><code>ENOMEM</code></dt>
<dd>지정한 네임스페이스를 바꾸는 데 필요한 메모리를 할당할 수 없다.</dd>
<dt><code>EPERM</code></dt>
<dd>호출 스레드가 이 동작에 필요한 역능을 가지고 있지 않다.</dd>
</dl>

## VERSIONS

리눅스 커널 3.0에서 `setns()` 시스템 호출이 처음 등장했다. glibc 버전 2.14에서 라이브러리 지원이 추가되었다.

## CONFORMING TO

`setns()` 시스템 호출은 리눅스 전용이다.

## NOTES

<tt>[[clone(2)]]</tt>으로 새 스레드를 생성할 때 공유할 수 있는 속성들을 모두 `setns()`로 바꿀 수 있는 것은 아니다.

## EXAMPLES

아래 프로그램은 둘 이상의 인자를 받는다. 첫 번째 인자는 기존 `/proc/[pid]/ns/` 디렉터리 내의 네임스페이스 파일 경로를 나타낸다. 나머지 인자들은 명령과 그 인자들을 지정한다. 프로그램은 네임스페이스 파일을 열고, `setns()`를 이용해 그 네임스페이스에 참여하고, 지정한 명령을 그 네임스페이스 안에서 실행한다.

다음 셸 세션은 (`ns_exec`라는 바이너리로 컴파일 한) 이 프로그램을 (`newuts`라는 바이너리로 컴파일 한) <tt>[[clone(2)]]</tt> 맨 페이지의 `CLONE_NEWUTS` 예시 프로그램과 함께 사용하는 것을 보여 준다.

먼저 <tt>[[clone(2)]]</tt>의 예시 프로그램을 배경으로 실행한다. 그 프로그램은 별도의 UTS 네임스페이스에서 자식을 생성한다. 자식이 자기 네임스페이스에서 호스트명을 바꾼 후 두 프로세스 모두 자기 UTS 네임스페이스 내의 호스트명을 표시하며, 그래서 둘이 어떻게 다른지 볼 수 있다.

```
$ su                   # 네임스페이스 작업에 특권 필요함
Password:
# ./newuts bizarro &
[1] 3549
clone() returned 3550
uts.nodename in child:  bizarro
uts.nodename in parent: antero
# uname -n             # 셸에서 호스트명 확인
antero
```

다음으로 아래와 같이 프로그램을 실행해서 셸을 실행하게 한다. 그 셸 내에서 호스트명이 첫 번째 프로그램이 만든 자식이 설정한 것과 같은지 확인한다.

```
# ./ns_exec /proc/3550/ns/uts /bin/bash
# uname -n             # ns_exec가 시작한 셸에서 실행
bizarro
```

### 프로그램 소스

```c
#define _GNU_SOURCE
#include <fcntl.h>
#include <sched.h>
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>

#define errExit(msg)    do { perror(msg); exit(EXIT_FAILURE); \
                        } while (0)

int
main(int argc, char *argv[])
{
    int fd;

    if (argc < 3) {
        fprintf(stderr, "%s /proc/PID/ns/FILE cmd args...\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    fd = open(argv[1], O_RDONLY); /* 네임스페이스 파일 디스크립터 얻기 */
    if (fd == -1)
        errExit("open");

    if (setns(fd, 0) == -1)       /* 그 네임스페이스에 참여 */
        errExit("setns");

    execvp(argv[2], &argv[2]);    /* 네임스페이스에서 명령 실행 */
    errExit("execvp");
}
```

## SEE ALSO

`nsenter(1)`, <tt>[[clone(2)]]</tt>, <tt>[[fork(2)]]</tt>, <tt>[[unshare(2)]]</tt>, <tt>[[vfork(2)]]</tt>, <tt>[[namespaces(7)]]</tt>, <tt>[[unix(7)]]</tt>

----

2019-03-06
