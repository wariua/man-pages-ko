## NAME

namespaces - 리눅스 네임스페이스 개요

## DESCRIPTION

네임스페이스는 전역 시스템 자원을 어떤 추상화로 감싸서 네임스페이스 안의 프로세스들에게는 독립적인 전역 자원 인스턴스가 따로 있는 것처럼 보이도록 만든다. 그 전역 자원에 대한 변경 사항이 네임스페이스 일원인 다른 프로세스들에게는 보이지만 그 외의 프로세스들에게는 보이지 않는다. 네임스페이스의 용도 중 하나가 컨테이너 구현이다.

이 페이지에서는 다양한 네임스페이스 유형에 대한 정보를 ... ,관련 `/proc` 파일들을 설명하고, 네임스페이스를 다루기 위한 API들을 요약한다.

### 네임스페이스 유형

다음 표는 리눅스에서 이용 가능한 네임스페이스 유형을 보여 준다. 표의 두 번째 열은 여러 API에서 그 네임스페이스를 지정하는 데 쓰는 플래그 값을 나타낸다. 세 번째 열은 그 네임스페이스에 대한 자세한 정보를 제공하는 매뉴얼 페이지를 나타낸다. 마지막 열은 그 네임스페이스가 격리하는 자원들을 요약한다.

| 네임스페이스 | 플래그            | 페이지                             | 격리 대상                       |
| ------------ | ----------------- | ---------------------------------- | ------------------------------- |
| cgroup       | `CLONE_NEWCGROUP` | <tt>[[cgroup_namespaces(7)]]</tt>  | cgroup 루트 디렉터리            |
| IPC          | `CLONE_NEWIPC`    | <tt>[[ipc_namespaces(7)]]</tt>     | 시스템 V IPC, POSIX 메시지 큐   |
| 네트워크     | `CLONE_NEWNET`    | <tt>[[network_namespaces(7)]]</tt> | 네트워크 장치, 스택, 포트 등    |
| 마운트       | `CLONE_NEWNS`     | <tt>[[mount_namespaces(7)]]</tt>   | 마운트 포인트                   |
| PID          | `CLONE_NEWPID`    | <tt>[[pid_namespaces(7)]]</tt>     | 프로세스 ID                     |
| 시간         | `CLONE_NEWTIME`   | <tt>[[time_namespaces(7)]]</tt>    | BOOT 및 MONOTONIC 클럭          |
| 사용자       | `CLONE_NEWUSER`   | <tt>[[user_namespaces(7)]]</tt>    | 사용자 ID와 그룹 ID             |
| UTS          | `CLONE_NEWUTS`    | <tt>[[uts_namespaces(7)]]</tt>     | 호스트명과 NIS 도메인 이름      |

### 네임스페이스 API

아래 기술하는 여러 `/proc` 파일들에 더해서 다음 시스템 호출들이 네임스페이스 API에 포함된다.

<tt>[[clone(2)]]</tt>
:   <tt>[[clone(2)]]</tt> 시스템 호출은 새 프로세스를 만든다. 호출의 `flags` 인자에 아래 나열된 `CLONE_NEW*` 플래그를 하나 이상 지정하면 각 플래그에 대한 새 네임스페이스가 생성되고 자식 프로세스가 그 네임스페이스들에 속하게 된다. (이 시스템 호출은 네임스페이스와 무관한 기능들도 여럿 구현하고 있다.)

<tt>[[setns(2)]]</tt>
:   <tt>[[setns(2)]]</tt> 시스템 호출을 통해 호출 프로세스가 기존 네임스페이스에 참여할 수 있다. 아래 설명하는 `/proc/[pid]/ns` 파일들 중 하나를 가리키는 파일 디스크립터를 통해 참여할 네임스페이스를 지정한다.

<tt>[[unshare(2)]]</tt>
:   <tt>[[unshare(2)]]</tt> 시스템 호출은 호출 프로세스를 새 네임스페이스로 옮긴다. 호출의 `flags` 인자가 아래 나열된 `CLONE_NEW*` 플래그를 하나 이상 지정하면 각 플래그에 대한 새 네임스페이스가 생성되고 호출 프로세스가 그 네임스페이스들에 속하게 된다. (이 시스템 호출은 네임스페이스와 무관한 기능들도 여럿 구현하고 있다.)

<tt>[[ioctl(2)]]</tt>
:   다양한 <tt>[[ioctl(2)]]</tt> 동작을 이용해 네임스페이스에 대한 정보를 알아낼 수 있다. 그 동작들을 <tt>[[ioctl_ns(2)]]</tt>에서 설명한다.

<tt>[[clone(2)]]</tt>과 <tt>[[unshare(2)]]</tt>로 새 네임스페이스를 만들려면 대부분의 경우 `CAP_SYS_ADMIN` 역능이 필요하다. 새 네임스페이스 안에서 생성 프로세스가 이후 그 네임스페이스에서 생성되거나 그 네임스페이스에 참여할 다른 프로세스들에게 보이는 전역 자원들을 변경할 능력을 가지게 되기 때문이다. 사용자 네임스페이스가 예외인데, 리눅스 3.8부터는 사용자 네임스페이스를 만드는 데 특권이 필요 없다.

### `/proc/[pid]/ns/` 디렉터리

각 프로세스마다 서브디렉터리 `/proc/[pid]/ns/`가 있고 그 안에는 <tt>[[setns(2)]]</tt>를 통한 조작을 지원하는 네임스페이스마다 항목이 하나씩 있다.

```text
$ ls -l /proc/$$/ns | awk '{print $1, $9, $10, $11}'
total 0
lrwxrwxrwx. cgroup -> cgroup:[4026531835]
lrwxrwxrwx. ipc -> ipc:[4026531839]
lrwxrwxrwx. mnt -> mnt:[4026531840]
lrwxrwxrwx. net -> net:[4026531969]
lrwxrwxrwx. pid -> pid:[4026531836]
lrwxrwxrwx. pid_for_children -> pid:[4026531834]
lrwxrwxrwx. time -> time:[4026531834]
lrwxrwxrwx. time_for_children -> time:[4026531834]
lrwxrwxrwx. user -> user:[4026531837]
lrwxrwxrwx. uts -> uts:[4026531838]
```

이 디렉터리의 파일들 중 하나를 파일 시스템의 다른 어딘가로 바인드 마운트(<tt>[[mount(2)]]</tt> 참고) 하면 `pid`로 지정한 프로세스의 해당 네임스페이스가 그 네임스페이스의 현재 프로세스가 모두 종료하더라도 유지된다.

이 디렉터리의 파일들 중 하나를 (또는 그 파일들 중 하나를 바인드 마운트 한 파일을) 열면 `pid`로 지정한 프로세스의 해당 네임스페이스의 파일 핸들이 반환된다. 그 파일 디스크립터가 열려 있는 동안에는 그 네임스페이스의 프로세스가 모두 종료하더라도 네임스페이스가 계속 살아 있게 된다. 그 파일 디스크립터를 <tt>[[setns(2)]]</tt>에 줄 수 있다.

리눅스 3.7 및 이전에서는 이 파일들이 하드 링크처럼 보였다. 리눅스 3.8부터는 심볼릭 링크로 보인다. 두 프로세스가 같은 네임스페이스에 있으면 그 `/proc/[pid]/ns/xxx` 심볼릭 링크의 장치 ID와 아이노드 번호가 동일하게 된다. 응용에서는 <tt>[[stat(2)]]</tt>이 반환하는 `stat.st_dev` 및 `stat.st_ino` 필드를 이용해 확인할 수 있다. 이 심볼릭 링크의 내용물은 다음처럼 네임스페이스 유형과 아이노드 번호를 담은 문자열이다.

```text
$ readlink /proc/$$/ns/uts
uts:[4026531838]
```

이 서브디렉터리 내의 심볼릭 링크들은 다음과 같다.

`/proc/[pid]/ns/cgroup` (리눅스 4.6부터)
:   이 파일은 프로세스의 cgroup 네임스페이스에 대한 핸들이다.

`/proc/[pid]/ns/ipc` (리눅스 3.0부터)
:   이 파일은 프로세스의 IPC 네임스페이스에 대한 핸들이다.

`/proc/[pid]/ns/mnt` (리눅스 3.8부터)
:   이 파일은 프로세스의 마운트 네임스페이스에 대한 핸들이다.

`/proc/[pid]/ns/net` (리눅스 3.0부터)
:   이 파일은 프로세스의 네트워크 네임스페이스에 대한 핸들이다.

`/proc/[pid]/ns/pid` (리눅스 3.8부터)
:   이 파일은 프로세스의 PID 네임스페이스에 대한 핸들이다. 이 핸들은 프로세스의 수명 동안 불변이다. (즉 프로세스의 PID 네임스페이스 소속은 절대 바뀌지 않는다.)

`/proc/[pid]/ns/pid_for_children` (리눅스 4.12부터)
:   이 파일은 이 프로세스가 생성한 자식 프로세스들의 PID 네임스페이스에 대한 핸들이다. 이 네임스페이스는 <tt>[[unshare(2)]]</tt>와 <tt>[[setns(2)]]</tt> 호출의 결과로 바뀔 수 있으며 (<tt>[[pid_namespaces(7)]]</tt> 참고), 따라서 파일이 `/proc/[pid]/ns/pid`와 다를 수 있다. 그 네임스페이스에 첫 번째 자식 프로세스가 생성된 후에야 심볼릭 링크에 값이 들어간다. (그 전에는 심볼릭 링크를 <tt>[[readlink(2)]]</tt> 하면 빈 버퍼를 반환한다.)

`/proc/[pid]/ns/time` (리눅스 5.6부터)
:   이 파일은 프로세스의 시간 네임스페이스에 대한 핸들이다.

`/proc/[pid]/ns/time_for_children` (리눅스 5.6부터)
:   이 파일은 이 프로세스가 생성한 자식 프로세스들의 시간 네임스페이스에 대한 핸들이다. 이 네임스페이스는 <tt>[[unshare(2)]]</tt>와 <tt>[[setns(2)]]</tt> 호출의 결과로 바뀔 수 있으며 (<tt>[[time_namespaces(7)]]</tt> 참고), 따라서 파일이 `/proc/[pid]/ns/time`과 다를 수 있다.

`/proc/[pid]/ns/user` (리눅스 3.8부터)
:   이 파일은 프로세스의 사용자 네임스페이스에 대한 핸들이다.

`/proc/[pid]/ns/uts` (리눅스 3.0부터)
:   이 파일은 프로세스의 UTS 네임스페이스에 대한 핸들이다.

이 심볼릭 링크들을 따라가거나 읽을 수 있는 권한은 ptrace 접근 모드 `PTRACE_MODE_READ_FSCREDS` 검사에 따라 결정된다. <tt>[[ptrace(2)]]</tt> 참고.

### `/proc/sys/user` 디렉터리

`/proc/sys/user` 디렉터리(리눅스 4.9부터 존재)의 파일들은 여러 유형의 네임스페이스들을 몇 개까지 만들 수 있는지 보여 준다.

`max_cgroup_namespaces`
:   이 파일의 값은 그 사용자 네임스페이스 내에 만들 수 있는 cgroup 네임스페이스 수의 사용자별 제한을 규정한다.

`max_ipc_namespaces`
:   이 파일의 값은 그 사용자 네임스페이스 내에 만들 수 있는 IPC 네임스페이스 수의 사용자별 제한을 규정한다.

`max_mnt_namespaces`
:   이 파일의 값은 그 사용자 네임스페이스 내에 만들 수 있는 마운트 네임스페이스 수의 사용자별 제한을 규정한다.

`max_net_namespaces`
:   이 파일의 값은 그 사용자 네임스페이스 내에 만들 수 있는 네트워크 네임스페이스 수의 사용자별 제한을 규정한다.

`max_pid_namespaces`
:   이 파일의 값은 그 사용자 네임스페이스 내에 만들 수 있는 PID 네임스페이스 수의 사용자별 제한을 규정한다.

`max_time_namespaces` (리눅스 5.7부터)
:   이 파일의 값은 그 사용자 네임스페이스 내에 만들 수 있는 시간 네임스페이스 수의 사용자별 제한을 규정한다.

`max_user_namespaces`
:   이 파일의 값은 그 사용자 네임스페이스 내에 만들 수 있는 사용자 네임스페이스 수의 사용자별 제한을 규정한다.

`max_uts_namespaces`
:   이 파일의 값은 그 사용자 네임스페이스 내에 만들 수 있는 UTC 네임스페이스 수의 사용자별 제한을 규정한다.

이 파일들에 대해 다음 사항에 유의하라.

* 특권 프로세스가 이 파일들의 값을 변경할 수 있다.

* 이 파일들이 내보이는 값은 여는 프로세스가 있는 사용자 네임스페이스에서의 제한이다.

* 제한이 사용자별이다. 같은 사용자 네임스페이스 내의 각 사용자가 각각 규정된 제한만큼 네임스페이스를 만들 수 있다.

* UID 0을 포함한 모든 사용자에게 제한이 적용된다.

* 이 제한들에 더해서 (PID 및 사용자 네임스페이스에서처럼) 다른 네임스페이스별 제한이 적용될 수도 있다.

* 이 제한에 걸리면 <tt>[[clone(2)]]</tt>와 <tt>[[unshare(2)]]</tt>가 `ENOSPC` 오류로 실패한다.

* 최초 사용자 네임스페이스에서 이 파일들의 기본값은 생성 가능 스레드 수 제한(`/proc/sys/kernel/threads-max`)의 절반이다. 모든 자손 사용자 네임스페이스에서는 각 파일의 기본값이 `MAXINT`이다.

* 네임스페이스가 생길 때 조상 네임스페이스들에도 그 객체가 상계된다. 정확히 말하자면,

  + 각 사용자 네임스페이스에는 생성자 UID가 있다.

  + 네임스페이스가 생길 때 조상 사용자 네임스페이스들 각각의 생성자 UID로 상계되며, 커널에서는 그 조상 네임스페이스의 생성자 UID에 대해 대응하는 네임스페이스 제한이 초과하지 않도록 한다.

  + 방금 언급한 점 때문에 사용자 네임스페이스를 새로 만드는 것이 현재 사용자 네임스페이스에 적용 중인 제한을 빠져나가는 방법이 될 수 없다.

### 네임스페이스의 수명

다른 요인이 없을 때 네임스페이스는 그 안의 마지막 프로세스가 종료하거나 네임스페이스를 떠날 때 자동으로 해체된다. 하지만 참여 프로세스가 없는 경우에도 네임스페이스가 존재하도록 고정할 수 있는 다른 요인들이 몇 가지 있다. 다음이 거기에 포함된다.

* 대응하는 `/proc/[pid]/ns/*` 파일에 대한 열린 파일 디스크립터나 바인드 마운트가 존재한다.

* 계층적인 네임스페이스(즉 PID 또는 사용자 네임스페이스)인데 자식 네임스페이스가 있다.

* 사용자 네임스페이스인데 사용자 아닌 네임스페이스를 하나 이상 소유하고 있다.

* PID 네임스페이스인데 심볼릭 링크 `/proc/[pid]/ns/pid_for_children`을 통해 그 네임스페이스를 가리키고 있는 프로세스가 있다.

* 시간 네임스페이스인데 심볼릭 링크 `/proc/[pid]/ns/time_for_children`을 통해 그 네임스페이스를 가리키고 있는 프로세스가 있다.

* IPC 네임스페이스인데 대응하는 `mqueue` 파일 시스템 마운트(<tt>[[mq_overview(7)]]</tt> 참고)가 그 네임스페이스를 가리키고 있다.

* PID 네임스페이스인데 대응하는 <tt>[[proc(5)]]</tt> 파일 시스템 마운트가 그 네임스페이스를 가리키고 있다.

## EXAMPLES

<tt>[[clone(2)]]</tt> 및 <tt>[[user_namespaces(7)]]</tt> 참고.

## SEE ALSO

`nsenter(1)`, `readlink(1)`, `unshare(1)`, <tt>[[clone(2)]]</tt>, <tt>[[ioctl_ns(2)]]</tt>, <tt>[[setns(2)]]</tt>, <tt>[[unshare(2)]]</tt>, <tt>[[proc(5)]]</tt>, <tt>[[capabilities(7)]]</tt>, <tt>[[cgroup_namespaces(7)]]</tt>, <tt>[[cgroups(7)]]</tt>, <tt>[[credentials(7)]]</tt>, <tt>[[ipc_namespaces(7)]]</tt>, <tt>[[network_namespaces(7)]]</tt>, <tt>[[pid_namespaces(7)]]</tt>, <tt>[[user_namespaces(7)]]</tt>, <tt>[[uts_namespaces(7)]]</tt>, `lsns(8)`, `pam_namespace(8)`, `switch_root(8)`

----

2021-03-22