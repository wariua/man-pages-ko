## NAME

core - 코어 덤프 파일

## DESCRIPTION

어떤 시그널들의 기본 동작은 프로세스를 종료시키고 *코어 덤프 파일*, 즉 종료 시점의 프로세스 메모리 이미지를 담은 디스크 파일을 만드는 것이다. 디버거(가령 <tt>[[gdb(1)]]</tt>)에서 그 이미지를 이용해서 종료 시점의 프로그램 상태를 조사할 수 있다. 프로세스 코어 덤프를 만드는 시그널 목록은 <tt>[[signal(7)]]</tt>에서 볼 수 있다.

프로세스에서 연성 `RLIMIT_CORE` 자원 제한을 설정하면 "코어 덤프" 시그널을 받았을 때 만드는 코어 덤프 파일의 크기에 상한을 줄 수 있다. 자세한 내용은 <tt>[[getrlimit(2)]]</tt>을 보라.

코어 덤프 파일이 생기지 않는 다양한 경우들이 있다.

* 프로세스에게 코어 파일 쓰기를 할 권한이 없다. (기본적으로 코어 파일의 이름은 `core`나 `core.PID`이고 현재 작업 디렉터리에 생긴다. 이름에 대한 자세한 내용은 아래 참고.) 파일을 생성할 디렉터리에 쓰기가 가능하지 않거나, 같은 이름의 파일이 존재하는데 그 파일이 쓰기 가능하지 않거나 정규 파일이 아니면 (가령 디렉터리나 심볼릭 링크이면) 코어 파일 쓰기가 실패한다.

* 코어 덤프로 사용할 이름을 가진 (쓰기 가능한 정규) 파일이 이미 존재하는데 그 파일에 대한 하드 링크가 여러 개이다.

* 코어 덤프 파일을 생성할 파일 시스템이 가득 차 있거나, 아이노드가 남아 있지 않거나, 읽기 전용으로 마운트 돼 있거나, 사용자별 할당 용량을 다 썼다.

* 코어 덤프 파일을 생성할 디렉터리가 존재하지 않는다.

* 프로세스의 `RLIMIT_CORE`(코어 파일 크기) 또는 `RLIMIT_FSIZE`(파일 크기) 자원 제한값이 0으로 설정돼 있다. <tt>[[getrlimit(2)]]</tt> 및 셸의 `ulimit` 명령 (`csh(1)`에서는 `limit`) 설명 참고.

* 프로세스가 실행 중인 바이너리에 읽기 권한이 켜져 있지 않다.

* 프로세스가 set-user-ID(set-group-ID) 프로그램을 실행 중인데 프로그램 소유 사용자(그룹)가 프로세스의 실제 사용자(그룹) ID와 다르거나, 프로세스가 파일 역능(<tt>[[capabilities(7)]]</tt> 참고)을 가진 프로그램을 실행 중이다. (하지만 <tt>[[prctl(2)]]</tt> `PR_SET_DUMPABLE` 동작 설명과 <tt>[[proc(5)]]</tt>의 `/proc/sys/fs/suid_dumpable` 설명도 볼 것.)

* `/proc/sys/kernel/core_pattern`이 비어 있고 `/proc/sys/kernel/core_uses_pid` 값이 0이다. (이 파일들은 아래에서 설명한다.) 참고로 `/proc/sys/kernel/core_pattern`이 비어 있고 `/proc/sys/kernel/core_uses_pid` 값이 1이면 코어 덤프 파일 이름이 `.PID` 형태가 되고, 그래서 `ls(1)` `-a` 옵션을 쓰지 않으면 보이지 않는다.

* (리눅스 3.7부터) `CONFIG_COREDUMP`를 빼고 커널을 구성했다.

그리고 <tt>[[madvise(2)]]</tt>의 `MADV_DONTDUMP` 플래그를 사용했다면 프로세스 주소 공간 중 일부가 코어 덤프에서 제외될 수 있다.

`init` 프레임워크로 `systemd(1)`를 쓰는 시스템에서는 코어 덤프 위치를 `systemd(1)`가 결정할 수 있다. 자세한 내용은 아래 참고.

### 코어 덤프 파일 이름

기본적으로 코어 덤프 파일의 이름은 `core`지만 (리눅스 2.6 및 2.4.21부터는) `/proc/sys/kernel/core_pattern` 파일을 통해 코어 덤프 파일의 이름 형식을 설정할 수 있다. 형식에 % 지시자가 들어갈 수 있으며 코어 파일 생성 시 다음 값들로 바뀐다.

| | |
| - | - |
| `%%` | % 문자 |
| `%c` | 죽는 프로세스의 코어 파일 크기 연성 자원 제한 (리눅스 2.6.24부터) |
| `%d` | 덤프 모드. <tt>[[prctl(2)]]</tt> `PR_GET_DUMPABLE`이 반환하는 값과 같음. (리눅스 3.7부터) |
| `%e` | 실행 파일 이름. 경로 선두부 제외. |
| `%E` | 실행 파일 이름. 슬래시('/')를 느낌표('!')로 바꿔서. (리눅스 3.0부터) |
| `%g` | 덤프 대상 프로세스의 (숫자로 된) 실제 GID |
| `%h` | 호스트명. <tt>[[uname(2)]]</tt>이 반환하는 `nodename`과 같음. |
| `%i` | 코어 덤프를 유발한 스레드의 TID. 스레드가 위치한 PID 네임스페이스 기준. (리눅스 3.18부터) |
| `%I` | 코어 덤프를 유발한 스레드의 TID. 최초 PID 네임스페이스 기준. (리눅스 3.18부터) |
| `%p` | 덤프 대상 프로세스의 PID. 프로세스가 위치한 PID 네임스페이스 기준. |
| `%P` | 덤프 대상 프로세스의 PID. 최초 PID 네임스페이스 기준. (리눅스 3.12부터) |
| `%s` | 덤프를 일으킨 시그널의 번호 |
| `%t` | 덤프 시간. 에포크, 즉 1970-01-01 00:00:00 +0000 (UTC) 이후 경과한 초의 수로 표현. |
| `%u` | 덤프 대상 프로세스의 (숫자로 된) 실제 UID |

형식 끝에 %가 한 개만 있으면 코어 파일 이름에서 빠진다. % 뒤에 위에 나열된 것 외의 문자가 조합된 경우도 마찬가지다. 그 외의 다른 문자들은 모두 코어 파일 이름에 그대로 들어간다. 형식에 '/' 문자가 포함될 수 있으며 디렉터리 이름 구분자로 해석한다. 결과로 나오는 코어 파일 이름의 최대 길이는 128바이트이다. (2.6.19 전의 커널에서는 64바이트였다.) 이 파일의 기본 값은 "core"이다. 하위 호환성을 위해서 `/proc/sys/kernel/core_pattern`에 `%p`가 없고 `/proc/sys/kernel/core_uses_pid`(아래 참고)가 0이 아니면 코어 파일 이름에 .PID를 덧붙인다.

경로 해석은 죽는 프로세스에서 활성인 설정들에 따라 이뤄진다. 설정이란 죽는 프로세스의 마운트 네임스페이스(<tt>[[mount_namespaces(7)]]</tt> 참고), (`getcwd(2)`를 통해 얻은) 현재 작업 디렉터리, 루트 디렉터리(<tt>[[chroot(2)]]</tt> 참고)이다.

버전 2.4부터 리눅스에서는 코어 덤프 파일 이름을 제어하는 더 단순한 방법을 함께 제공했다. `/proc/sys/kernel/core_uses_pid` 파일에 0 값이 들어 있으면 코어 덤프 파일의 이름이 그냥 `core`이다. 그 파일에 0 아닌 값이 들어 있으면 코어 덤프 파일 이름에 프로세스 ID가 포함되어 `core.PID` 형태가 된다.

리눅스 3.6부터는 `/proc/sys/fs/suid_dumpable`을 2("suidsafe")로 설정하는 경우 패턴이 ('/' 문자로 시작하는) 절대 경로명이거나 아래 설명하는 파이프여야 한다.

### 파이프로 코어 덤프를 프로그램으로 보내기

리눅스 커널 2.6.19부터 `/proc/sys/kernel/core_pattern` 파일에서 또 다른 문법을 지원한다. 파일의 첫 번째 문자가 파이프 기호(|)이면 행 나머지를 실행해야 하는 사용자 공간 프로그램 (내지 스크립트) 명령행으로 해석한다. 코어 덤프는 디스크 파일에 기록되지 않고 그 프로그램에 표준 입력으로 주어진다. 다음 사항에 유의해야 한다.

* 절대 경로명으로 (또는 루트 디렉터리 `/` 기준 경로명으로) 프로그램을 지정해야 하며 '|' 문자 바로 다음에 경로명이 와야 한다.

* 명령행 인자에 위에 나열한 % 지시자가 뭐든 들어갈 수 있다. 예를 들어 덤프 중인 프로세스의 PID를 전달하려면 인자에 `%p`를 지정하면 된다.

* 그 프로그램을 실행하기 위해 생성되는 프로세스는 사용자와 그룹을 `root`로 해서 돈다.

* `root`로 돈다고 해도 어떤 예외적인 보안 우회도 이뤄지지 않는다. 즉 LSM(가령 SELinux)이 여전히 효력이 있으며, 그래서 핸들러에서 `/proc/[pid]`를 통해 죽은 프로세스에 대한 자세한 정보에 접근하는 걸 막을 수도 있다.

* 항상 최초 마운트 네임스페이스에서 프로그램을 실행하기 때문에 거기를 기준으로 프로그램 경로명을 해석한다. 죽는 프로세스의 설정들(가령 루트 디렉터리, 마운트 네임스페이스, 현재 작업 디렉터리)에 영향을 받지 않는다.

* 죽는 프로세스의 네임스페이스가 아니라 (PID, 마운트, 사용자 등의) 최초 네임스페이스에서 프로세스가 돈다. `%P` 같은 지시자를 이용하면 올바른 `/proc/[pid]` 디렉터리를 알아낼 수 있으므로 필요 시 죽는 프로세스의 네임스페이스에 진입하거나 조사할 수 있다.

* 루트 디렉터리를 현재 작업 디렉터리로 해서 프로세스가 시작한다. 원한다면 `%P` 지시자가 제공한 값을 이용해서 `/proc/[pid]/cwd`를 통해 덤프 중인 프로세스의 작업 디렉터리로 바꾸는 게 가능하다.

* (리눅스 2.6.24부터) 공백으로 구분된 명령행 인자들을 (행의 총 길이 128바이트까지) 프로그램에 줄 수 있다.

* 이 메커니즘을 통해 프로그램으로 전달되는 코어 덤프에는 `RLIMIT_CORE` 제한이 적용되지 않는다.

### `/proc/sys/kernel/core_pipe_limit`

사용자 공간 프로그램으로 파이프 해서 코어 덤프를 수집할 때 죽는 프로세스의 `/proc/[pid]` 디렉터리로부터 얻는 데이터가 수집 프로그램에게 유용할 수 있다. 안전하게 수집을 할 수 있으려면 코어 덤프 수집 프로그램이 끝날 때까지 커널이 대기하게 해서 죽는 프로세스의 `/proc/[pid]` 파일들을 때 이르게 없애지 않도록 해야 한다. 그런데 이렇게 하면 오동작하는 수집 프로그램이 절대 끝나지 않아서 죽은 프로세스 정리를 막게 될 가능성이 생긴다.

리눅스 2.6.32부터는 `/proc/sys/kernel/core_pipe_limit`을 사용해 그런 가능성에 대비할 수 있다. 이 파일의 값은 동시에 얼마나 많은 죽는 프로세스들이 병렬로 사용자 공간 프로그램으로 파이프 될 수 있는지 규정한다. 이 값을 초과하는 경우 그 값 위에서 죽는 프로세스들은 커널 로그에만 기록하고 코어 덤프를 건너뛴다.

이 파일에서 0 값은 특별하다. 프로세스들을 동시에 제한 없이 잡고 있을 수 있음을 나타내며, 또한 대기가 이뤄지지 않음을 (즉 수집 프로그램에서 죽는 프로세스의 `/proc/[pid]`에 접근하지 못할 수도 있음을) 나타낸다. 이 파일의 기본 값은 0이다.

### 코어 덤프에 기록할 매핑 제어

커널 2.6.23부터는 리눅스 전용인 `/proc/[pid]/coredump_filter` 파일을 사용해서 해당 프로세스 ID를 가진 프로세스에 대해 코어 덤프를 수행하는 경우 코어 덤프 파일에 어떤 메모리 세그먼트들이 기록되도록 할지 제어할 수 있다.

파일의 값은 메모리 매핑 종류들(<tt>[[mmap(2)]]</tt> 참고)의 비트 마스크이다. 마스크의 어떤 비트가 설정돼 있으면 대응하는 종류의 메모리 매핑들을 덤프 하고, 아니면 덤프 하지 않는다. 비트들의 의미는 다음과 같다.

| | |
| - | - |
| 0번 비트 | 익명 비공유 매핑 덤프. |
| 1번 비트 | 익명 공유 매핑 덤프. |
| 2번 비트 | 파일 기반 비공유 매핑 덤프. |
| 3번 비트 | 파일 기반 공유 매핑 덤프. |
| 4번 비트 | (리눅스 2.6.24부터) ELF 헤더 덤프. |
| 5번 비트 | (리눅스 2.6.28부터) 비공유 거대 페이지 덤프. |
| 6번 비트 | (리눅스 2.6.28부터) 공유 거대 페이지 덤프. |
| 7번 비트 | (리눅스 4.4부터) 비공유 DAX 페이지 덤프. |
| 8번 비트 | (리눅스 4.4부터) 공유 DAX 페이지 덤프. |

기본적으로 0번, 1번, 4번 (`CONFIG_CORE_DUMP_DEFAULT_ELF_HEADERS` 커널 구성 옵션이 켜진 경우), 5번 비트가 설정돼 있다. `coredump_filter` 부팅 옵션을 사용하면 부팅 때 이 기본 값을 바꿀 수 있다.

이 파일의 값은 16진수로 표시된다. (따라서 기본 값은 33으로 표시된다.)

`coredump_filter` 값이 어떻든지 프레임 버퍼 같은 메모리 맵 I/O 페이지들은 절대 덤프 하지 않으며 가상 DSO(<tt>[[vdso(7)]]</tt>) 페이지들은 항상 덤프 한다.

<tt>[[fork(2)]]</tt>를 통해 생성된 자식 프로세스는 부모의 `coredump_filter` 값을 물려받는다. <tt>[[execve(2)]]</tt>를 거치면서 `coredump_filter` 값이 보존된다.

프로그램 실행 전에 부모 셸에서 `coredump_filter`를 설정하는 게 유용할 수 있다.

```text
$ echo 0x7 > /proc/self/coredump_filter
$ ./some_program
```

커널을 `CONFIG_ELF_CORE` 구성 옵션으로 빌드 한 경우에만 이 파일이 제공된다.

### 코어 덤프와 systemd

`systemd(1)` `init` 프레임워크를 쓰는 시스템에서는 코어 덤프가 저장되는 위치를 `systemd(1)`에서 결정할 수 있다. 이를 위해 `systemd(1)`에서는 파이프로 코어 덤프를 프로그램으로 보낼 수 있는 `core_pattern` 기능을 이용한다. 코어 덤프가 파이프를 통해 `systemd-coredump(8)` 프로그램으로 가고 있는지 다음과 같이 확인해 볼 수 있다.

```text
$ cat /proc/sys/kernel/core_pattern
|/usr/lib/systemd/systemd-coredump %P %u %g %s %t %c %e
```

이 경우 코어 덤프는 `systemd-coredump(8)`에 설정된 위치로 가게 되는데, 보통은 디렉터리 `/var/lib/systemd/coredump/` 안에 `lz4(1)` 압축 파일로 저장된다. `systemd-coredump(8)`가 기록한 코어 덤프들의 목록을 `coredumpctl(1)`을 이용해 볼 수 있다.

```text
$ coredumpctl list | tail -5
Wed 2017-10-11 22:25:30 CEST  2748 1000 1000 3 present  /usr/bin/sleep
Thu 2017-10-12 06:29:10 CEST  2716 1000 1000 3 present  /usr/bin/sleep
Thu 2017-10-12 06:30:50 CEST  2767 1000 1000 3 present  /usr/bin/sleep
Thu 2017-10-12 06:37:40 CEST  2918 1000 1000 3 present  /usr/bin/cat
Thu 2017-10-12 08:13:07 CEST  2955 1000 1000 3 present  /usr/bin/cat
```

코어 덤프마다 덤프 일시, 덤프 프로세스의 PID, UID, GID, 코어 덤프를 유발한 시그널 번호, 덤프 된 프로세스가 실행하고 있던 실행 파일 경로명 등의 정보가 보인다. `coredumpctl(1)`의 여러 옵션들을 통해 지정한 코어 덤프 파일을 `systemd(1)` 위치에서 지정한 파일로 가져올 수 있다. 예를 들어 위에 있는 PID 2955의 코어 덤프를 현재 디렉터리에 `core`라는 파일로 빼내려면 다음과 같이 하면 된다.

```text
$ coredumpctl dump 2955 -o core
```

더 자세한 내용은 `coredumpctl(1)` 매뉴얼 페이지를 참고하라.

`systemd(1)`의 코어 덤프 기록 메커니즘을 끄고 전통적인 리눅스 동작 방식을 복원하고 싶다면 다음처럼 `systemd(1)` 메커니즘을 무시하게 설정할 수 있다.

```text
# echo "kernel.core_pattern=core.%p" > /etc/sysctl.d/50-coredump.conf
# /lib/systemd/systemd-sysctl
```

## NOTES

<tt>[[gdb(1)]]</tt>의 `gcore` 명령을 이용하면 실행 중인 프로세스의 코어 덤프를 얻을 수 있다.

리눅스 2.6.27까지의 버전에서는 다중 스레드 프로세스가 (더 정확히는 <tt>[[clone(2)]]</tt>의 `CLONE_VM` 플래그로 생성돼서 다른 프로세스와 메모리를 공유하는 프로세스가) 코어를 덤프 하는 경우에 `/proc/sys/kernel/core_pattern`의 `%p` 지시자를 통해 파일명 어딘가에 이미 프로세스 ID가 포함돼 있지 않으면 항상 코어 파일명에 프로세스 ID를 덧붙인다. (프로세스의 스레드마다 PID가 다른 구식 LinuxThreads 구현을 쓸 때 주로 도움이 된다.)

## EXAMPLE

아래 프로그램을 통해 `/proc/sys/kernel/core_pattern` 파일의 파이프 문법 사용 방식을 볼 수 있다. 다음 셸 세션은 이 프로그램 사용례를 보여 준다. (`core_pattern_pipe_test`라는 실행 파일로 컴파일 함.)

```text
$ cc -o core_pattern_pipe_test core_pattern_pipe_test.c
$ su
Password:
# echo "|$PWD/core_pattern_pipe_test %p UID=%u GID=%g sig=%s" > \
    /proc/sys/kernel/core_pattern
# exit
$ sleep 100
^\                     # Ctrl-\ 입력
Quit (core dumped)
$ cat core.info
argc=5
argc[0]=</home/mtk/core_pattern_pipe_test>
argc[1]=<20575>
argc[2]=<UID=1000>
argc[3]=<GID=100>
argc[4]=<sig=3>
Total bytes in core dump: 282624
```

### 프로그램 소스

```c
/* core_pattern_pipe_test.c */

#define _GNU_SOURCE
#include <sys/stat.h>
#include <fcntl.h>
#include <limits.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

#define BUF_SIZE 1024

int
main(int argc, char *argv[])
{
    int tot, j;
    ssize_t nread;
    char buf[BUF_SIZE];
    FILE *fp;
    char cwd[PATH_MAX];

    /* 죽은 프로세스의 현재 작업 디렉터리로 이동 */

    snprintf(cwd, PATH_MAX, "/proc/%s/cwd", argv[1]);
    chdir(cwd);

    /* 그 디렉터리의 파일 "core.info"로 기록 */

    fp = fopen("core.info", "w+");
    if (fp == NULL)
        exit(EXIT_FAILURE);

    /* core_pattern 파이프 프로그램이 받은 명령행 인자 표시 */

    fprintf(fp, "argc=%d\n", argc);
    for (j = 0; j < argc; j++)
        fprintf(fp, "argc[%d]=<%s>\n", j, argv[j]);

    /* 표준 입력(코어 덤프)의 바이트 수 세기 */

    tot = 0;
    while ((nread = read(STDIN_FILENO, buf, BUF_SIZE)) > 0)
        tot += nread;
    fprintf(fp, "Total bytes in core dump: %d\n", tot);

    fclose(fp);
    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

`bash(1)`, `coredumpctl(1)`, <tt>[[gdb(1)]]</tt>, <tt>[[getrlimit(2)]]</tt>, <tt>[[mmap(2)]]</tt>, <tt>[[prctl(2)]]</tt>, <tt>[[sigaction(2)]]</tt>, <tt>[[elf(5)]]</tt>, <tt>[[proc(5)]]</tt>, <tt>[[pthreads(7)]]</tt>, <tt>[[signal(7)]]</tt>, `systemd-coredump(8)`

----

2019-03-06