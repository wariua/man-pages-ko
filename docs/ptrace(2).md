## NAME

ptrace - 프로세스 추적

## SYNOPSIS

```c
#include <sys/ptrace.h>

long ptrace(enum __ptrace_request request, pid_t pid,
            void *addr, void *data);
```

## DESCRIPTION

`ptrace()` 시스템 호출은 한 프로세스("추적자(tracer)")가 다른 프로세스("피추적자(tracee)")의 실행을 관찰 및 제어하고 피추적자의 메모리와 레지스터를 검사 및 변경할 수 있는 방법을 제공한다. 중지점(breakpoint) 디버깅과 시스템 호출 추적을 구현하는 데 주로 쓰인다.

먼저 피추적자를 추적자에 붙여야 한다. 붙이기와 이어지는 명령들은 스레드별이다. 즉, 다중 스레드 프로세스에서 각 스레드를 (다를 수도 있는) 추적자에게 개별적으로 붙이거나, 붙이지 않고 놔둬서 디버깅 하지 않을 수 있다. 따라서 "피추적자"는 항상 "(단일) 스레드"를 뜻하며 절대 "(다중 스레드일 수도 있는) 프로세스"를 뜻하는 게 아니다. ptrace 명령은 항상 다음 형태의 호출을 이용해 특정 피추적자에게 보낸다.

```c
ptrace(PTRACE_foo, pid, ...)
```

여기서 `pid`는 해당 리눅스 스레드의 스레드 ID이다.

(참고로 이 페이지에서 "다중 스레드 프로세스"란 <tt>[[clone(2)]]</tt> `CLONE_THREAD` 플래그로 생성한 스레드들로 이뤄진 스레드 그룹을 뜻한다.)

프로세스가 <tt>[[fork(2)]]</tt>를 호출하고 그래서 생긴 자식이 (보통) 이어지는 <tt>[[execve(2)]]</tt> 전에 `PTRACE_TRACEME`를 하게 해서 추적을 개시할 수 있다. 또는 한 프로세스가 `PTRACE_ATTACH`나 `PTRACE_SEIZE`를 이용해 다른 프로세스 추적을 시작할 수도 있다.

추적되고 있는 동안 피추적자는 시그널이 전달될 때마다 멈추게 된다. 무시하고 있는 시그널이라도 그렇다. (`SIGKILL`은 예외이며 평상시와 효과가 같다.) 추적자는 다음 번 <tt>[[waitpid(2)]]</tt> 호출에서 (또는 유사한 "wait" 시스템 호출들 중 하나에서) 알림을 받게 된다. 그 호출은 피추적자가 멈춘 이유를 나타내는 정보를 담은 `status` 값을 반환하게 된다. 피추적자가 멈춰 있는 동안 추적자가 다양한 ptrace 요청을 사용해 피추적자를 조사하고 변경할 수 있다. 그러고서 추적자는 피추적자가 실행을 계속하게 하는데, 선택적으로 전달됐던 시그널을 무시하게 할 수 있다. (또는 다른 시그널을 대신 전달할 수도 있다.)

`PTRACE_O_TRACEEXEC` 옵션이 적용 중이 아니면 피추적 프로세스가 <tt>[[execve(2)]]</tt> 성공 호출 시 `SIGTRAP` 시그널을 받게 되며, 그래서 새 프로그램이 실행을 시작하기 전에 부모에게 제어권을 얻을 기회를 준다.

추적자가 추적을 마쳤을 때는 `PTRACE_DETACH`를 통해 피추적자가 정상적인 비추적 모드로 실행을 계속하게 할 수 있다.

`request` 값이 수행할 행동을 결정한다.

`PTRACE_TRACEME`
:   이 프로세스가 부모에 의해 추적될 것임을 나타낸다. 부모가 추적할 예정이 아니라면 이 요청을 하지 말아야 할 것이다. (`pid`, `addr`, `data`는 무시한다.)

    `PTRACE_TRACEME` 요청은 피추적자에서만 사용한다. 나머지 요청들은 추적자에서만 사용한다. 이어지는 요청들에서 `pid`는 동작 대상 피추적자의 스레드 ID를 나타낸다. `PTRACE_ATTACH`, `PTRACE_SEIZE`, `PTRACE_INTERRUPT`, `PTRACE_KILL` 외의 요청들에서는 피추적자가 멈춰 있어야 한다.

`PTRACE_PEEKTEXT`, `PTRACE_PEEKDATA`
:   피추적자 메모리의 주소 `addr`에서 워드를 읽고 그 워드를 `ptrace()` 호출의 결과로 반환한다. 리눅스에서 텍스트와 데이터의 주소 공간이 따로 있지 않으므로 이 두 요청은 현재 동등하다. (`data`는 무시한다. 하지만 NOTES 참고.)

`PTRACE_PEEKUSER`
:   피추적자 USER 영역의 오프셋 `addr`에서 워드를 읽는다. USER 영역은 레지스터들과 기타 프로세스에 대한 정보를 담고 있다 (`<sys/user.h>` 참고). 그 워드를 `ptrace()` 호출의 결과로 반환한다. 보통은 오프셋이 워드에 정렬되어 있어야 하지만 아키텍처에 따라 다를 수도 있다. NOTES 참고. (`data`는 무시한다. 하지만 NOTES 참고.)

`PTRACE_POKETEXT`, `PTRACE_POKEDATA`
:   워드 `data`를 피추적자 메모리의 주소 `addr`로 복사한다. `PTRACE_PEEKTEXT` 및 `PTRACE_PEEKDATA`처럼 이 두 요청은 현재 동등하다.

`PTRACE_POKEUSER`
:   워드 `data`를 피추적자 USER 영역의 오프셋 `addr`로 복사한다. `PTRACE_PEEKUSER`처럼 오프셋이 보통은 워드에 정렬되어 있어야 한다. 커널의 무결성을 유지하기 위해 USER 영역에 대한 일부 변경은 허용하지 않는다.

`PTRACE_GETREGS`, `PTRACE_GETFPREGS`
:   각각 피추적자의 범용 레지스터들이나 부동 소수점 레지스터들을 추적자 내의 주소 `data`로 복사한다. 이 데이터의 형식에 대한 정보는 `<sys/user.h>`를 보라. (`addr`은 무시한다.) 참고로 SPARC 시스템에서는 `data`와 `addr`의 의미가 뒤집혀 있다. 즉, `data`를 무시하고 주소 `addr`로 레지스터들을 복사한다. 모든 아키텍처에 `PTRACE_GETREGS`와 `PTRACE_GETFPREGS`가 있는 건 아니다.

`PTRACE_GETREGSET` (리눅스 2.6.34부터)
:   피추적자의 레지스터들을 읽는다. `addr`이 아키텍처별 방식으로 읽을 레지스터들의 종류를 나타낸다. `NT_PRSTATUS`(숫자 값 1)는 일반적으로 범용 레지스터들을 읽게 만든다. 예를 들어 CPU에 부동 소수점 및/또는 벡터 레지스터가 있으면 `addr`을 대응 `NT_foo` 상수로 설정해서 그 레지스터들을 가져올 수 있다. `data`는 목적지 버퍼의 위치와 길이를 기술하는 `struct iovec`을 가리킨다. 반환 시 실제 반환되는 바이트 수를 나타내도록 커널이 `iov.len`을 변경한다.

`PTRACE_SETREGS`, `PTRACE_SETFPREGS`
:   각각 피추적자의 범용 레지스터들이나 부동 소수점 레지스터들을 추적자 내의 주소 `data`에서 온 값으로 변경한다. `PTRACE_POKEUSER`처럼 일부 범용 레지스터 변경이 허용되지 않을 수도 있다. (`addr`은 무시한다.) 참고로 SPARCS 시스템에서는 `data`와 `addr`의 의미가 뒤집혀 있다. 즉, `data`를 무시하고 주소 `addr`로부터 레지스터들을 복사한다. 모든 아키텍처에 `PTRACE_SETREGS`와 `PTRACE_SETFPREGS`가 있는 건 아니다.

`PTRACE_SETREGSET` (리눅스 2.6.34부터)
:   피추적자의 레지스터들을 변경한다. `addr`과 `data`의 의미는 `PTRACE_GETREGSET`과 유사하다.

`PTRACE_GETSIGINFO` (리눅스 2.3.99-pre6부터)
:   정지를 유발한 시그널에 대한 정보를 가져온다. 피추적자로부터 `siginfo_t` 구조체(<tt>[[sigaction(2)]]</tt> 참고)를 추적자 내의 주소 `data`로 복사한다. (`addr`은 무시한다.)

`PTRACE_SETSIGINFO` (리눅스 2.3.99-pre6부터)
:   시그널 정보를 설정한다. 추적자 내의 주소 `data`로부터 `siginfo_t` 구조체를 피추적자로 복사한다. 피추적자에게 정상적으로 전달되었을 것이면서 추적자에게 잡힌 시그널에만 영향을 주게 된다. 정상적인 시그널과 `ptrace()` 자체에서 생성한 인조 시그널을 구별하는 것이 어려울 수도 있다. (`addr`은 무시한다.)

`PTRACE_PEEKSIGINFO` (리눅스 3.10부터)
:   큐에서 시그널을 제거하지 않으면서 `siginfo_t` 구조체를 가져온다. `addr`은 몇 번째 시그널부터 몇 개나 복사해야 할지 지정하는 `ptrace_peeksiginfo_args` 구조체 포인터이다. `data`가 가리키는 버퍼로 `siginfo_t` 구조체들을 복사한다. 반환 값은 복사한 시그널 수를 담고 있다. (0은 지정한 위치에 해당하는 시그널이 없음을 나타낸다.) 반환되는 `siginfo` 구조체의 `si_code` 필드가 다른 방식으로는 사용자 공간에 노출되지 않는 정보(`__SI_CHLD`, `__SI_FAULT` 등)를 포함한다.

        struct ptrace_peeksiginfo_args {
            u64 off;    /* 시그널 복사를 시작할 큐에서의 위치 */
            u32 flags;  /* PTRACE_PEEKSIGINFO_SHARED 또는 0 */
            s32 nr;     /* 복사할 시그널 개수 */
        };

    현재 유일하게 있는 플래그는 프로세스별 시그널 큐에서 시그널을 가져오기 위한 `PTRACE_PEEKSIGINFO_SHARED`이다. 이 플래그가 설정돼 있지 않으면 지정한 스레드의 스레드별 큐에서 시그널을 읽는다.

`PTRACE_GETSIGMASK` (리눅스 3.11부터)
:   블록 된 시그널 마스크(<tt>[[sigprocmask(2)]]</tt> 참고) 사본을 `data`가 가리키는 버퍼에 집어넣는다. `data`는 `sigset_t` 타입 버퍼에 대한 포인터여야 한다. `addr` 인자는 `data`가 가리키는 버퍼의 크기를 (즉 `sizeof(sigset_t)`를) 담는다.

`PTRACE_SETSIGMASK` (리눅스 3.11부터)
:   블록 된 시그널 마스크(<tt>[[sigprocmask(2)]]</tt> 참고)를 `data`가 가리키는 버퍼에 지정한 값으로 바꾼다. `data`는 `sigset_t` 타입 버퍼에 대한 포인터여야 한다. `addr` 인자는 `data`가 가리키는 버퍼의 크기를 (즉 `sizeof(sigset_t)`를) 담는다.

`PTRACE_SETOPTIONS` (리눅스 2.4.6부터. BUGS 절의 경고 참고)
:   `data`에서 가져온 ptrace 옵션들을 설정한다. (`addr`은 무시한다.) `data`는 옵션들의 비트 마스크로 해석하며, 다음 플래그들로 옵션을 지정한다.

    `PTRACE_O_EXITKILL` (리눅스 3.8부터)
    :   추적자가 끝날 때 피추적자에게 `SIGKILL` 시그널을 보낸다. 피추적자가 절대 추적자의 통제를 벗어나지 못하게 하고 싶은 ptrace 간수들에게 이 옵션이 유용하다.

    `PTRACE_O_TRACECLONE` (리눅스 2.5.46부터)
    :   다음 번 <tt>[[clone(2)]]</tt>에서 피추적자를 멈추고 새로 clone 된 프로세스 추적을 자동으로 시작한다. 새 프로세스는 `SIGSTOP`으로, 또는 `PTRACE_SEIZE` 사용 시 `PTRACE_EVENT_STOP`으로 시작하게 된다. 추적자의 <tt>[[waitpid(2)]]</tt>가 다음과 같은 `status` 값을 반환하게 된다.

            status>>8 == (SIGTRAP | (PTRACE_EVENT_CLONE<<8))

        `PTRACE_GETEVENTMSG`로 새 프로세스의 PID를 가져올 수 있다.

        이 옵션이 모든 경우의 <tt>[[clone(2)]]</tt> 호출을 잡지는 못할 수도 있다. 피추적자가 `CLONE_VFORK` 플래그로 <tt>[[clone(2)]]</tt>을 호출한 경우 `PTRACE_O_TRACEVFORK`가 설정돼 있으면 `PTRACE_EVENT_VFORK`가 대신 전달된다. 또는 피추적자가 종료 시그널을 `SIGCHLD`로 설정해서 <tt>[[clone(2)]]</tt>을 호출하는 경우 `PTRACE_O_TRACEFORK`가 설정돼 있으면 `PTRACE_EVENT_FORK`가 전달된다.

    `PTRACE_O_TRACEEXEC` (리눅스 2.5.46부터)
    :   다음 번 <tt>[[execve(2)]]</tt>에서 피추적자를 멈춘다. 추적자의 <tt>[[waitpid(2)]]</tt>가 다음과 같은 `status` 값을 반환하게 된다.

            status>>8 == (SIGTRAP | (PTRACE_EVENT_EXEC<<8))

        exec 하는 스레드가 스레드 그룹 리더가 아닌 경우 이 정지 전에 그 스레드의 ID를 스레드 그룹 리더의 ID로 재설정한다. 리눅스 3.0부터 `PTRACE_GETEVENTMSG`로 이전 스레드 ID를 가져올 수 있다.

    `PTRACE_O_TRACEEXIT` (리눅스 2.5.60부터)
    :   exit에서 피추적자를 멈춘다. 추적자의 <tt>[[waitpid(2)]]</tt>가 다음과 같은 `status` 값을 반환하게 된다.

            status>>8 == (SIGTRAP | (PTRACE_EVENT_EXIT<<8))

        `PTRACE_GETEVENTMSG`로 피추적자의 종료 상태를 가져올 수 있다.

        피추적자가 멈추는 것은 프로세스 종료 초반에 레지스터들이 아직 사용 가능할 때이고, 그래서 어디서 종료가 일어났는지 추적자가 볼 수 있다. 반면 정상적인 종료 알림은 프로세스가 종료를 끝낸 후에 이뤄진다. 문맥에 접근 가능하기는 하지만 이 시점에서 추적자가 종료가 일어나지 않게 막을 수는 없다.

    `PTRACE_O_TRACEFORK` (리눅스 2.5.46부터)
    :   다음 번 <tt>[[fork(2)]]</tt>에서 피추적자를 멈추고 새로 fork 된 프로세스 추적을 자동으로 시작한다. 새 프로세스는 `SIGSTOP`으로, 또는 `PTRACE_SEIZE` 사용 시 `PTRACE_EVENT_STOP`으로 시작하게 된다. 추적자의 <tt>[[waitpid(2)]]</tt>가 다음과 같은 `status` 값을 반환하게 된다.

            status>>8 == (SIGTRAP | (PTRACE_EVENT_FORK<<8))

        `PTRACE_GETEVENTMSG`로 새 프로세스의 PID를 가져올 수 있다.

    `PTRACE_O_TRACESYSGOOD` (리눅스 2.4.6부터)
    :   시스템 호출 트랩을 전달할 때 시그널 번호에 7번 비트를 설정한다. (즉, `SIGTRAP|0x80`을 전달한다.) 이렇게 하면 정상적인 트랩과 시스템 호출에 의한 트랩을 추적자가 쉽게 구별할 수 있다.

    `PTRACE_O_TRACEVFORK` (리눅스 2.5.46부터)
    :   다음 번 <tt>[[vfork(2)]]</tt>에서 피추적자를 멈추고 새로 vfork 된 프로세스 추적을 자동으로 시작한다. 새 프로세스는 `SIGSTOP`으로, 또는 `PTRACE_SEIZE` 사용 시 `PTRACE_EVENT_STOP`으로 시작하게 된다. 추적자의 <tt>[[waitpid(2)]]</tt>가 다음과 같은 `status` 값을 반환하게 된다.

            status>>8 == (SIGTRAP | (PTRACE_EVENT_VFORK<<8))

        `PTRACE_GETEVENTMSG`로 새 프로세스의 PID를 가져올 수 있다.

    `PTRACE_O_TRACEVFORKDONE` (리눅스 2.5.60부터)
    :   다음 번 <tt>[[vfork(2)]]</tt> 완료 시 피추적자를 멈춘다. 추적자의 <tt>[[waitpid(2)]]</tt>가 다음과 같은 `status` 값을 반환하게 된다.

            status>>8 == (SIGTRAP | (PTRACE_EVENT_VFORK_DONE<<8))

        (리눅스 2.6.18부터) `PTRACE_GETEVENTMSG`로 새 프로세스의 PID를 가져올 수 있다.

    `PTRACE_O_TRACESECCOMP` (리눅스 3.5부터)
    :   <tt>[[seccomp(2)]]</tt> `SECCOMP_RET_TRACE` 규칙이 걸렸을 때 피추적자를 멈춘다. 추적자의 <tt>[[waitpid(2)]]</tt>가 다음과 같은 `status` 값을 반환하게 된다.

            status>>8 == (SIGTRAP | (PTRACE_EVENT_SECCOMP<<8))

        이 때문에 `PTRACE_EVENT` 정지가 발생하면 그건 시스템-호출-진입-정지(syscall-enter-stop)와 비슷하다. 자세한 내용은 아래의 `PTRACE_EVENT_SECCOMP`에 대한 내용을 보라. `PTRACE_GETEVENTMSG`로 seccomp 이벤트 메시지 데이터(seccomp 필터 규칙에서 `SECCOMP_RET_DATA` 부분)를 가져올 수 있다.

    `PTRACE_O_SUSPEND_SECCOMP` (리눅스 4.3부터)
    :   피추적자의 seccomp 보호를 일시 중단한다. 모드와 상관없이 적용되며 피추적자가 아직 seccomp 필터를 설치하지 않았을 때 사용할 수 있다. 즉, 유효한 사용 방식은 피추적자가 seccomp 필터를 설치하기 전에 seccomp 보호를 일시 중단하고, 피추적자가 필터를 설치하게 하고, 이후 필터를 재개해야 할 때 이 플래그를 비우는 것이다. 이 옵션을 위해선 추적자에게 `CAP_SYS_ADMIN` 역능이 있어야 하고, 어떤 seccomp 보호도 설치되어 있지 않아야 하고, 자체에 `PTRACE_O_SUSPEND_SECCOMP`가 설정되어 있지 않아야 한다.

`PTRACE_GETEVENTMSG` (리눅스 2.5.46부터)
:   방금 발생한 ptrace 이벤트에 대한 메시지를 (`unsigned long`으로) 가져와서 추적자 내의 주소 `data`에 집어넣는다. `PTRACE_EVENT_EXIT`에서는 피추적자의 종료 상태이다. `PTRACE_EVENT_FORK`, `PTRACE_EVENT_VFORK`, `PTRACE_EVENT_VFORK_DONE`, `PTRACE_EVENT_CLONE`에서는 새 프로세스의 PID이다. `PTRACE_EVENT_SECCOMP`에서는 걸린 규칙과 연계된 <tt>[[seccomp(2)]]</tt> 필터의 `SECCOMP_RET_DATA`이다. (`addr`은 무시한다.)

`PTRACE_CONT`
:   정지된 피추적 프로세스를 재시작한다. `data`가 0이 아니면 피추적자에게 보낼 시그널 번호로 해석한다. 0이면 시그널을 보내지 않는다. 그래서 예를 들어 피추적자로 보낸 시그널이 전달될지 여부를 추적자가 제어할 수 있다. (`addr`은 무시한다.)

`PTRACE_SYSCALL`, `PTRACE_SINGLESTEP`
:   `PTRACE_CONT`처럼 정지된 피추적 프로세스를 재시작하되 다음 번 시스템 호출 진입이나 퇴장에서 또는 한 인스트럭션 실행 후에 피추적자가 멈추도록 해 놓는다. (평상시와 마찬가지로 시그널 수신 시에도 피추적자가 멈추게 된다.) 추적자 관점에서는 피추적자가 `SIGTRAP`을 수신해서 멈춘 것으로 보이게 된다. 그래서 예를 들어 `PTRACE_SYSCALL`의 경우 첫 번째 정지 때 시스템 호출의 인자를 검사하고서 다시 `PTRACE_SYSCALL`을 해서 두 번째 정지 때 그 시스템 호출의 반환 값을 검사할 수 있다. `data` 인자는 `PTRACE_CONT`에서처럼 처리한다.

`PTRACE_SYSEMU`, `PTRACE_SYSEMU_SINGLESTEP` (리눅스 2.6.14부터)
:   `PTRACE_SYSEMU`의 경우 속행 후 다음 시스템 호출 진입에서 정지하며, 그 시스템 호출은 실행되지 않게 된다. 아래의 시스템-호출-정지(syscall-stop)에 대한 내용을 보라. `PTRACE_SYSEMU_SINGLESTEP`의 경우 똑같이 하되 시스템 호출이 아니면 단계 실행(singlestep)을 한다. 피추적자의 모든 시스템 호출을 에뮬레이트 하려 하는 사용자 모드 리눅스(User Mode Linux) 같은 프로그램에서 이 호출을 사용한다. `data` 인자는 `PTRACE_CONT`에서처럼 처리한다. `addr`은 무시한다. 현재 x86에서만 이 요청들을 지원한다.

`PTRACE_LISTEN` (리눅스 3.4부터)
:   정지된 피추적자를 재시작하되 실행은 막는다. 그렇게 하면 피추적자의 상태는 `SIGSTOP`(또는 다른 정지형 시그널)으로 정지된 프로세스와 비슷해진다. 추가적인 내용은 "그룹-정지(group-stop)" 부절을 보라. `PTRACE_LISTEN`은 `PTRACE_SEIZE`로 붙인 피추적자에만 동작한다.

`PTRACE_KILL`
:   피추적자에게 `SIGKILL`을 보내서 종료시킨다. (`addr`과 `data`는 무시한다.)

    *이 동작은 제거 예정이므로 사용하지 말 것!* 대신 <tt>[[kill(2)]]</tt>이나 <tt>[[tgkill(2)]]</tt>을 이용해 직접 `SIGKILL`을 보내면 된다. `PTRACE_KILL`의 문제는 피추적자가 시그널-전달-정지(signal-delivery-stop) 상태여야 하고 안 그러면 동작하지 않을 수 있다는 점이다. (즉 성공적으로 완료하고서 피추적자를 죽이지 않을 수가 있다.) 반면 직접 `SIGKILL`을 보내는 방식에는 그런 제한이 없다.

`PTRACE_INTERRUPT` (리눅스 3.4부터)
:   피추적자를 멈춘다. 피추적자가 커널 공간에서 실행 내지 슬립 중이면 시스템 호출을 중단시키고 시스템-호출-퇴장-정지(syscall-exit-stop)를 보고한다. (중단된 시스템 호출이 피추적자 재시작 때 재시작된다.) 피추적자가 이미 시그널로 정지되었고 그리로 `PTRACE_LISTEN`을 보냈으면 피추적자가 `PTRACE_EVENT_STOP`으로 멈추고 `WSTOPSIG(status)`가 정지 시그널을 반환한다. 다른 ptrace-정지가 동시에 발생하면 (가령 피추적자로 시그널이 전송되면) 그 ptrace-정지가 일어난다. 위의 어느 경우도 적용되지 않으면 (예를 들어 피추적자가 사용자 공간에서 돌고 있으면) `PTRACE_EVENT_STOP`으로 멈추고 `WSTOPSIG(status)`는 `SIGTRAP`이다. `PTRACE_INTERRUPT`는 `PTRACE_SEIZE`로 붙인 피추적자에만 동작한다.

`PTRACE_ATTACH`
:   `pid`로 지정한 프로세스에 붙어서 그 프로세스를 호출 프로세스의 피추적자로 만든다. 피추적자에게 `SIGSTOP`이 전송되지만 이 호출 완료 시점에 피추적자가 반드시 멈춰 있지는 않을 것이다. <tt>[[waitpid(2)]]</tt>를 사용해서 피추적자가 멈추기를 기다리면 된다. 자세한 내용은 "붙기와 떨어지기" 부절을 보라. (`addr`과 `data`는 무시한다.)

    `PTRACE_ATTACH` 수행 권한은 ptrace 접근 모드 `PTRACE_MODE_ATTACH_REALCREDS` 검사로 결정된다. 아래 참고.

`PTRACE_SEIZE` (리눅스 3.4부터)
:   `pid`로 지정한 프로세스에 붙어서 그 프로세스를 호출 프로세스의 피추적자로 만든다. `PTRACE_ATTACH`와 달리 `PTRACE_SEIZE`는 프로세스를 정지시키지 않는다. 그룹-정지(group-stop)는 `PTRACE_EVENT_STOP`으로 보고되고 `WSTOPSIG(status)`가 정지 시그널을 반환한다. 자동으로 붙는 자식들은 `PTRACE_EVENT_STOP`으로 멈추고 `WSTOPSIG(status)`가 `SIGTRAP`을 반환하며 `SIGSTOP` 시그널은 전달되지 않는다. <tt>[[execve(2)]]</tt>에서 추가 `SIGTRAP`이 전달되지 않는다. `PTRACE_SEIZE`로 잡은 프로세스만 `PTRACE_INTERRUPT` 및 `PTRACE_LISTEN` 명령을 받아들일 수 있다. 이런 "장악(seize)" 동작 방식을 `PTRACE_O_TRACEFORK`, `PTRACE_O_TRACEVFORK`, `PTRACE_O_TRACECLONE`으로 자동으로 붙는 자식들이 물려받는다. `addr`이 0이어야 한다. `data`는 즉시 활성화시킬 ptrace 옵션들의 비트 마스크를 담는다.

    `PTRACE_SEIZE` 수행 권한은 ptrace 접근 모드 `PTRACE_MODE_ATTACH_REALCREDS` 검사로 결정된다. 아래 참고.

`PTRACE_SECCOMP_GET_FILTER` (리눅스 4.4부터)
:   이 동작을 통해 추적자가 피추적자의 전통적 BPF 필터를 얻어올 수 있다.

    `addr`은 얻어올 필터의 인덱스를 나타내는 정수이다. 가장 최근 설치된 필터의 인덱스가 0이다. `addr`이 설치된 필터 수보다 크면 동작이 `ENOENT` 오류로 실패한다.

    `data`는 BPF 프로그램을 저장하기에 충분히 큰 `struct sock_filter` 배열에 대한 포인터이거나, 프로그램을 저장하려는 것이 아니면 NULL이다.

    성공 시 반환 값은 BPF 프로그램 내 인스트럭션 수이다. `data`가 NULL이었으면 이 반환 값을 이용해 정확한 크기의 `struct sock_filter` 배열을 후속 호출에 줄 수 있다.

    호출자가 `CAP_SYS_ADMIN` 역능을 가지고 있지 않거나 호출자가 seccomp 엄격 내지 필터 모드에 있으면 이 동작이 `EACCES` 오류로 실패한다. `addr`이 가리키는 필터가 전통적 BPF 필터가 아니면 동작이 `EMEDIUMTYPE` 오류로 실패한다.

    커널을 `CONFIG_SECCOMP_FILTER`와 `CONFIG_CHECKPOINT_RESTORE` 옵션 모두로 구성한 경우에만 이 동작이 사용 가능하다.

`PTRACE_DETACH`
:   `PTRACE_CONT`처럼 정지된 피추적자를 재시작하되 먼저 그 프로세스에서 떨어진다. 리눅스에서는 어떤 방법으로 추적을 개시했던지 간에 이 방식으로 피추적자에서 떨어질 수 있다. (`addr`은 무시한다.)

`PTRACE_GET_THREAD_AREA` (리눅스 2.6.0부터)
:   이 동작은 <tt>[[get_thread_area(2)]]</tt>와 비슷한 일을 수행한다. GDT에서 인덱스가 `addr`인 TLS 항목을 읽어서 그 항목의 사본을 `data`가 가리키는 `struct user_desc`로 복사한다. (<tt>[[get_thread_area(2)]]</tt>와 달리 `struct user_desc`의 `entry_number`를 무시한다.)

`PTRACE_SET_THREAD_AREA` (리눅스 2.6.0부터)
:   이 동작은 <tt>[[set_thread_area(2)]]</tt>와 비슷한 일을 수행한다. GDT에서 인덱스가 `addr`인 TLS 항목을 `data`가 가리키는 `struct user_desc`에 준 데이터로 설정한다. (<tt>[[set_thread_area(2)]]</tt>와 달리 `struct user_desc`의 `entry_number`를 무시한다. 다시 말해 이 ptrace 동작을 사용해 빈 TLS 항목을 할당할 수는 없다.)

### ptrace 하의 죽음

(다중 스레드일 수 있는) 프로세스가 죽이기형(killing) 시그널(처리 방식이 `SIG_DFL`로 설정돼 있고 기본 행위가 프로세스 죽이는 것인 시그널)을 수신하면 모든 스레드들이 끝난다. 피추적자는 자기 추적자(들)에게 자기 죽음을 알린다. 이 사건 알림은 <tt>[[waitpid(2)]]</tt>를 통해 전달된다.

참고로 죽이기형 시그널은 먼저 (피추적자 하나에서만) 시그널-전달-정지를 일으키고, 추적자가 그 시그널을 주입하고 나서야 (또는 추적 대상 아닌 스레드가 가져가고 나서야) 다중 스레드 프로세스 내 *모든* 피추적자들에서 시그널에 의한 죽음이 일어나게 된다. ("시그널-전달-정지"라는 용어는 아래에서 설명한다.)

`SIGKILL`은 시그널-전달-정지를 발생시키지 않고, 그래서 추적자가 이를 억제할 수 없다. 시스템 호출 내에 있어도 `SIGKILL`로 죽는다. (`SIGKILL`에 의한 죽음에 앞서 시스템-호출-퇴장-정지(syscall-exit-stop)가 발생하지 않는다.) 결론은 프로세스 내 일부 스레드를 ptrace 하는 경우에도 `SIGKILL`이 항상 프로세스를 (모든 스레드를) 죽인다는 것이다.

피추적자가 <tt>[[_exit(2)]]</tt>를 호출할 때 자기 추적자에게 자기 죽음을 알린다. 다른 스레드들은 영향 받지 않는다.

어느 스레드라도 <tt>[[exit_group(2)]]</tt>을 실행할 때 그 스레드 그룹 내의 모든 피추적자가 자기 추적자에게 자기 죽음을 알린다.

`PTRACE_O_TRACEEXIT` 옵션이 켜져 있으면 실제 죽음 전에 `PTRACE_EVENT_EXIT`가 발생하게 된다. <tt>[[exit(2)]]</tt>, <tt>[[exit_group(2)]]</tt>, 시그널 죽음(`SIGKILL`은 제외이되 커널 버전에 따라 다름. 아래 BUGS 참고)을 통해 끝날 때, 그리고 다중 스레드 프로세스에서 <tt>[[execve(2)]]</tt> 때문에 스레드들이 파기될 때 그렇다.

ptrace에 의해 정지된 피추적자가 존재한다고 추적자가 가정할 수 없다. 피추적자가 정지된 상태에서 죽을 수 있는 (`SIGKILL` 같은) 여러 경우들이 있다. 따라서 추적자는 어느 ptrace 동작에 대해서든 `ESRCH`를 다룰 준비가 되어 있어야 한다. 불행히도 (정지된 피추적자를 필요로 하는 명령에서) 피추적자가 존재하지만 ptrace에 의해 중지되지 않은 경우나 그 ptrace 호출을 한 프로세스에게 추적되고 있지 않은 경우에도 같은 오류가 반환된다. 추적자는 피추적자의 정지/동작 상태를 따라갈 필요가 있으며, 피추적자가 분명히 ptrace-정지에 들어갔다고 알고 있는 경우에만 `ESRCH`를 "피추적자가 예상 못하게 죽었음"으로 해석해야 한다. 참고로 ptrace 동작이 `ESRCH`를 반환한 경우에 `waitpid(WNOHANG)`이 피추적자의 죽음 상태를 신뢰성 있게 알려준다는 보장이 없다. 대신 `waitpid(WNOHANG)`이 0을 반환할 수도 있다. 다시 말해 피추적자가 아직 "완전히 죽지는" 않았으면서 ptrace 요청은 이미 거부하고 있을 수도 있다.

피추적자가 *항상* `WIFEXITED(status)`나 `WIFSIGNALED(status)`를 알려주며 인생을 끝낸다고 추적자가 가정할 수 없다. 그러지 않는 경우들이 있다. 예를 들어 스레드 그룹 리더가 아닌 스레드가 <tt>[[execve(2)]]</tt>를 하면 그 스레드가 사라진다. 그 PID가 다시는 보이지 않게 되며 이후 발생하는 ptrace 정지는 스레드 그룹 리더의 PID로 보고된다.

### 정지 상태

피추적자는 동작 또는 정지 중 한 상태에 있을 수 있다. ptrace에 있어서 (`read(2)`, <tt>[[pause(2)]]</tt> 등의) 시스템 호출 내에 블록 돼 있는 피추적자는 설령 긴 시간 동안 블록 되어 있는 경우에도 실행 중인 것으로 본다. `PTRACE_LISTEN` 이후 피추적자의 상태는 다소 애매하다. 어떤 ptrace 정지에도 있지 않으며 (ptrace 명령이 먹히지 않으며 <tt>[[waitpid(2)]]</tt> 알림을 전달하게 된다.), 그러면서도 인스트럭션을 실행하고 있지 않으므로 (스케줄 되지 않으므로) "정지" 상태로 볼 수도 있다. 그리고 `PTRACE_LISTEN` 전에 그룹 정지에 있었으면 `SIGCONT`를 수신할 때까지는 시그널에 응답하지 않게 된다.

피추적자가 멈춰 있을 때의 상태가 여러 가지 있는데 ptrace 논의에서 이를 뭉뚱그려 말하는 경우가 많다. 따라서 정확한 용어를 사용하는 것이 중요하다.

이 매뉴얼 페이지에서는 피추적자가 추적자로부터 ptrace 명령을 받아들일 준비가 되어 있는 모든 정지 상태를 *ptrace-정지(ptrace-stop)*라고 한다. ptrace-정지를 다시 *시그널-전달-정지(signal-delivery-stop)*, *그룹-정지(group-stop)*, *시스템-호출-정지(syscall-stop)*, *`PTRACE_EVENT` 정지(`PTRACE_EVENT` stop)* 등으로 나눌 수 있다. 이 정지 상태들을 아래에서 자세히 설명한다.

동작 중인 피추적자가 ptrace-정지에 들어가면 <tt>[[waitpid(2)]]</tt>를 (또는 다른 "wait" 시스템 호출들 중 하나를) 하고 있는 추적자에게 알림을 보낸다. 이 매뉴얼 페이지 대부분에서는 추적자가 다음과 같이 기다린다고 가정한다.

```c
pid = waitpid(pid_or_minus_1, &status, __WALL);
```

0보다 큰 `pid` 반환과 `WIFSTOPPED(status)` 참이 ptrace로 정지된 피추적자임을 알려 준다.

`__WALL` 플래그는 `WSTOPPED`와 `WEXITED` 플래그를 포함하지 않지만 그 기능성을 함의한다.

<tt>[[waitpid(2)]]</tt> 호출 시 `WCONTINUED` 플래그 설정을 권장하지 않는다. "속행됨" 상태는 프로세스별이며 이를 소모하면 피추적자의 실제 부모를 혼란스럽게 만들 수 있다.

`WNOHANG` 플래그를 사용하면 알림이 있을 것임을 추적자가 알고 있는 경우에도 <tt>[[waitpid(2)]]</tt>가 0("아직 사용 가능한 대기 결과 없음")을 반환하게 될 수 있다. 예:

```c
errno = 0;
ptrace(PTRACE_CONT, pid, 0L, 0L);
if (errno == ESRCH) {
    /* 피추적자가 죽었음 */
    r = waitpid(tracee, &status, __WALL | WNOHANG);
    /* 여기서 r이 아직도 0일 수 있다! */
}
```

존재하는 ptrace 정지의 종류로 시그널-전달-정지, 그룹-정지, `PTRACE_EVENT` 정지, 시스템-호출-정지가 있다. 모두 <tt>[[waitpid(2)]]</tt>로 알 수 있고 `WIFSTOPPED(status)`가 참이다. 종류를 구별하고 싶으면 `status>>8` 값을 검사하거나, 그 값에 모호한 점이 있는 경우 `PTRACE_GETSIGINFO`를 질의해 보면 된다. (참고: 이 검사를 수행하는 데 `WSTOPSIG(status)` 매크로를 사용할 수는 없다. `(status>>8) & 0xff` 값을 반환하기 때문이다.)

### 시그널-전달-정지

(다중 스레드일 수 있는) 프로세스가 `SIGKILL` 외의 시그널을 수신했을 때 커널에서는 그 시그널을 처리할 스레드를 임의로 선정한다. (시그널을 <tt>[[tgkill(2)]]</tt>로 생성하는 경우에는 추적자가 대상 스레드를 명시적으로 선택할 수 있다.) 선택된 스레드가 추적되고 있으면 시그널-전달-정지로 들어간다. 이 시점에서 시그널은 아직 프로세스에게 전달되지 않았고 추적자에 의해 억제될 수 있다. 추적자가 시그널을 억제하지 않는 경우 다음 ptrace 재시작 요청에서 피추적자에게 시그널을 보내게 된다. 시그널 전달의 이 두 번째 단계를 이 매뉴얼에서 *시그널 주입*이라고 한다. 참고로 시그널이 블록 되어 있으면 블록이 해제될 때까지 시그널-전달-정지가 일어나지 않는다. 단 블록 할 수 없는 `SIGSTOP`은 언제나처럼 예외이다.

<tt>[[waitpid(2)]]</tt>가 `WIFSTOPPED(status)`를 참으로 반환하는 것으로 추적자가 시그널-전달-정지를 목격하며, `WSTOPSIG(status)`가 시그널을 반환한다. 시그널이 `SIGTRAP`이면 다른 종류의 ptrace 정지일 수도 있다. 자세한 내용은 아래의 "시스템-호출-정지" 및 "execve" 절을 보라. `WSTOPSIG(status)`가 정지형(stopping) 시그널을 반환하는 경우 그룹-정지일 수도 있다. 아래를 보라.

### 시그널 주입과 억제

추적자가 시그널-전달-정지를 목격한 후에 다음 호출로 피추적자를 재시작해야 한다.

```c
ptrace(PTRACE_restart, pid, 0, sig)
```

여기서 `PTRACE_restart`는 ptrace 재시작 요청들 중 하나이다. `sig`가 0이면 시그널을 전달하지 않는다. 그렇지 않으면 시그널 `sig`를 전달한다. 이 매뉴얼 페이지에서는 이 동작을 *시그널 주입*이라고 해서 시그널-전달-정지와 구분한다.

`sig` 값이 `WSTOPSIG(status)` 값과 다를 수도 있다. 즉, 추적자가 다른 시그널을 주입시킬 수 있다.

참고로 억제된 시그널 역시도 시스템 호출이 일찍 반환되게 한다. 이 경우 시스템 호출이 재시작된다. 추적자가 `PTRACE_SYSCALL`을 사용하는 경우 피추적자가 중단됐던 시스템 호출을 재실행 하는 것을 (또는 재시작에 다른 메커니즘을 사용하는 일부 시스템 호출에서 <tt>[[restart_syscall(2)]]</tt> 시스템 호출을) 보게 될 것이다. 시그널 후에 재시작 가능하지 않은 (<tt>[[poll(2)]]</tt> 같은) 시스템 호출들도 시그널 억제 후에는 재시작된다. 하지만 피추적자에게 어떤 관찰 가능한 시그널도 주입하지 않는데도 일부 시스템 호출이 `EINTR`로 실패하게 하는 커널 버그가 존재한다.

시그널-전달-정지 아닌 ptrace 정지에서 내린 재시작 ptrace 명령에서는 `sig`가 0이 아니어도 시그널 주입이 보장되지 않는다. 어떤 오류 보고도 없이 0 아닌 `sig`가 그냥 무시될 수 있다. ptrace 사용자는 이 방식으로 "새로운 시그널 생성"을 하려고 하지 않아야 한다. <tt>[[tgkill(2)]]</tt>을 사용하면 된다.

시그널-전달-정지 아닌 ptrace 정지 후 피추적자를 재시작할 때 시그널 주입 요청이 무시될 수도 있다는 점이 ptrace 사용자들에게 혼란을 일으킬 수 있다. 흔한 사례 하나는 추적자가 그룹-정지를 목격하고서 시그널-전달-정지로 착각하고, 다음으로 피추적자를 재시작하는 것이다.

```c
ptrace(PTRACE_restart, pid, 0, stopsig)
```

`stopsig`를 주입하려는 의도이지만 `stopsig`가 무시되고 피추적자가 실행을 계속한다.

`SIGCONT` 시그널에는 그룹-정지인 프로세스(의 스레드 모두)를 깨우는 부대 효과가 있다. 이 부대 효과는 시그널-전달-정지 전에 발생한다. 추적자가 이 부대 효과를 억제할 수 없다. (시그널 주입을 억제할 수 있을 뿐이며, 그래서 `SIGCONT` 핸들러가 설치되어 있을 때 그 핸들러가 실행되지 않게 할 수 있을 뿐이다.) 그리고 실제로는 `SIGCONT` 전달 시점에 대기 중인 시그널이 있었다면 그룹-정지에서 깨어난 다음에 `SIGCONT` *아닌* 시그널에 대한 시그널-전달-정지가 올 수도 있다. 다시 말해 `SIGCONT` 전송 후 피추적자에게 보이는 첫 번째 시그널이 `SIGCONT`가 아닐 수도 있다.

정지형 시그널은 프로세스(의 스레드 모두)가 그룹-정지에 들어가게 한다. 이 부대 효과는 시그널 주입 후에 일어나며, 따라서 추적자가 억제할 수 있다.

리눅스 2.4와 그 전에서는 `SIGSTOP` 시그널을 주입할 수 없다.

`PTRACE_GETSIGINFO`를 이용해 전달 시그널에 대응하는 `siginfo_t` 구조체를 가져올 수 있다. `PTRACE_SETSIGINFO`를 이용해 변경할 수도 있다. `PTRACE_SETSIGINFO`를 사용해 `siginfo_t`를 바꾸는 경우 `si_signo` 필드와 재시작 명령의 `sig` 매개변수가 일치해야 하며, 그렇지 않을 때의 결과가 규정되어 있지 않다.

### 그룹-정지

(다중 스레드일 수 있는) 프로세스가 정지형 시그널을 수신하면 모든 스레드가 멈춘다. 그 중 추적 대상인 스레드가 있으면 그룹-정지로 들어간다. 참고로 정지형 시그널은 먼저 (피추적자 하나에서만) 시그널-전달-정지를 일으키고, 추적자가 그 시그널을 주입하고 나서야 (또는 추적 대상 아닌 스레드가 가져가고 나서야) 다중 스레드 프로세스 내 *모든* 피추적자에서 그룹-정지가 개시된다. 언제나처럼 모든 피추적자가 대응하는 추적자에게 각기 자신의 그룹-정지를 알린다.

<tt>[[waitpid(2)]]</tt>가 `WIFSTOPPED(status)`를 참으로 반환하는 것으로 추적자가 그룹-정지를 목격하며, `WSTOPSIG(status)`를 통해 그 정지형 시그널을 얻을 수 있다. 몇몇 다른 ptrace 정지 유형에서도 같은 결과를 반환하므로 다음 호출을 수행해 보기를 권장한다.

```c
ptrace(PTRACE_GETSIGINFO, pid, 0, &siginfo)
```

시그널이 `SIGSTOP`, `SIGTSTP`, `SIGTTIN`, `SIGTTOU`가 아니면 호출을 피할 수 있다. 이 네 가지 시그널만 정지형 시그널이기 때문이다. 추적자에게 다른 뭔가가 보인다면 그룹-정지일 수가 없다. 그 외 경우에 추적자가 `PTRACE_GETSIGINFO`를 호출할 필요가 있다. `PTRACE_GETSIGINFO`가 `EINVAL`로 실패한다면 확실히 그룹-정지이다. (다른 실패 코드도 가능하다. 가령 `SIGKILL` 때문에 피추적자가 죽었으면 `ESRCH`("no such process")로 실패한다.)

`PTRACE_SEIZE`로 피추적자에게 붙었다면 `PTRACE_EVENT_STOP`이, 즉 `status>>16 == PTRACE_EVENT_STOP`이 그룹-정지를 나타낸다. 그래서 추가적인 `PTRACE_GETSIGINFO` 호출을 할 필요 없이 그룹-정지를 탐지할 수 있다.

리눅스 2.6.38 기준으로, 추적자가 피추적자의 ptrace 정지를 본 다음 재시작하거나 죽이기 전까지는 피추적자가 돌지 않으며, 추적자가 다른 <tt>[[waitpid(2)]]</tt> 호출로 들어가는 경우에도 추적자에게 (`SIGKILL` 죽음을 제외하고) 알림을 보내지 않게 된다.

앞 문단에서 기술한 커널 동작 방식이 정지형 시그널을 투명하게 처리하는 데 문제를 일으킨다. 추적자가 그룹-정지 후에 피추적자를 재시작하면 그 정지형 시그널이 실질적으로 무시된다. 즉, 피추적자가 정지돼 있지 않고 돈다. 추적자가 다음 <tt>[[waitpid(2)]]</tt> 진입 전에 피추적자를 재시작하지 않으면 이후의 `SIGCONT` 시그널이 추적자에게 보고되지 않게 된다. 그러면 `SIGCONT` 시그널이 피추적자에게 아무 효과도 주지 못하게 될 것이다.

리눅스 3.4부터는 이 문제를 극복할 방법이 있다. `PTRACE_CONT` 대신 `PTRACE_LISTEN` 명령을 사용하면 피추적자가 실행은 하지 않지만 (`SIGCONT`로 재시작될 때처럼) <tt>[[waitpid(2)]]</tt>를 통해 알릴 수 있는 새 이벤트를 기다리게 되는 방식으로 피추적자를 재시작할 수 있다.

### `PTRACE_EVENT` 정지

추적자가 `PTRACE_O_TRACE_*` 옵션을 설정하면 피추적자가 `PTRACE_EVENT` 정지라고 하는 ptrace 정지에 들어가게 된다.

<tt>[[waitpid(2)]]</tt>가 `WIFSTOPPED(status)`를 반환하는 것으로 추적자가 그룹-정지를 목격하며, `WSTOPSIG(status)`는 `SIGTRAP`을 반환한다. 상태 워드의 상위 바이트에 비트가 추가로 설정되어 `status>>8` 값이 다음과 같이 된다.

```c
(SIGTRAP | PTRACE_EVENT_foo << 8)
```

다음 이벤트들이 있다.

`PTRACE_EVENT_VFORK`
:   <tt>[[vfork(2)]]</tt>나 `CLONE_VFORK` 플래그 사용 <tt>[[clone(2)]]</tt>에서 반환하기 전에 멈춘다. 이 정지 후에 피추적자를 속행시키면 자식이 exit/exec 하기를 기다린 후에 실행을 계속할 것이다. (즉 일반적인 <tt>[[vfork(2)]]</tt> 동작이다.)

`PTRACE_EVENT_FORK`
:   <tt>[[fork(2)]]</tt>나 종료 시그널을 `SIGCHLD`로 설정한 <tt>[[clone(2)]]</tt>에서 반환하기 전에 멈춘다.

`PTRACE_EVENT_CLONE`
:   <tt>[[clone(2)]]</tt>에서 반환하기 전에 멈춘다.

`PTRACE_EVENT_VFORK_DONE`
:   <tt>[[vfork(2)]]</tt>나 `CLONE_VFORK` 플래그 사용 <tt>[[clone(2)]]</tt>에서 반환하기 전에, 그러면서 자식이 exit나 exec로 피추적자를 풀어 준 후에 멈춘다.

위의 네 가지 정지 모두 새로 생성된 스레드가 아니라 부모(즉 피추적자)에서 정지가 일어난다. `PTRACE_GETEVENTMSG`를 이용해 새 스레드의 ID를 가져올 수 있다.

`PTRACE_EVENT_EXEC`
:   <tt>[[execve(2)]]</tt> 반환 전에 정지한다. 리눅스 3.0부터 `PTRACE_GETEVENTMSG`가 이전 스레드 ID를 반환한다.

`PTRACE_EVENT_EXIT`
:   종료(<tt>[[exit_group(2)]]</tt>으로 죽는 것 포함)나 시그널 죽음, 다중 스레드 프로세스에서 <tt>[[execve(2)]]</tt>에 의한 종료 전에 정지한다. `PTRACE_GETEVENTMSG`가 종료 상태를 반환한다. ("진짜" 종료가 일어났을 때와 달리) 레지스터를 조사할 수 있다. 피추적자가 여전히 살아 있다. `PTRACE_CONT`나 `PTRACE_DETACH` 해 주어야 종료가 마무리된다.

`PTRACE_EVENT_STOP`
:   `PTRACE_INTERRUPT` 명령이나 그룹-정지, 새 자식에 붙었을 때의 (`PTRACE_SEIZE`로 붙은 경우에만) 최초 ptrace 정지에 의해 유발되는 정지.

`PTRACE_EVENT_SECCOMP`
:   추적자가 `PTRACE_O_TRACESECCOMP`를 설정했을 때 피추적자 시스템 호출 진입 시 <tt>[[seccomp(2)]]</tt> 규칙에 의한 정지. seccomp 이벤트 메시지 데이터(seccomp 필터 규칙의 `SECCOMP_RET_DATA` 부분)를 `PTRACE_GETEVENTMSG`로 가져올 수 있다. 이 정지의 동작 방식을 아래의 별도 절에서 자세히 설명한다.

`PTRACE_EVENT` 정지에서 `PTRACE_GETSIGINFO`는 `si_signo`에 `SIGTRAP`을 반환하며 `si_code`가 `(event<<8) | SIGTRAP`으로 설정되어 있다.

### 시스템-호출-정지

피추적자가 `PTRACE_SYSCALL`이나 `PTRACE_SYSEMU`로 재시작됐으면 피추적자가 시스템 호출에 진입하기 직전에 시스템-호출-정지에 들어간다. (`PTRACE_SYSEMU`로 재시작이 이뤄졌으면 이 지점에서 레지스터를 어떻게 바꾸고 이 정지 후에 피추적자를 어떻게 재시작하는지와 상관없이 그 시스템 호출은 실행되지 않는다.) 시스템-호출-정지를 유발한 방법이 무엇이든 간에, 추적자가 `PTRACE_SYSCALL`로 피추적자를 재시작하면 시스템 호출이 끝날 때, 또는 시그널로 중단될 때 피추적자가 시스템-호출-퇴장-정지에 들어간다. (말하자면 시그널-전달-정지가 절대로 시스템-호출-진입-정지와 시스템-호출-퇴장-정지 사이에서 일어나지 않는다. 시스템-호출-퇴장-정지 *후에* 일어난다.) 다른 방법(`PTRACE_SYSEMU` 포함)으로 피추적자를 속행시키면 시스템-호출-퇴장-정지가 일어나지 않는다. 참고로 `PTRACE_SYSEMU`에 대한 언급 내용이 모두 `PTRACE_SYSEMU_SINGLESTEP`에 동일하게 적용된다.

하지만 `PTRACE_SYSCALL`로 피추적자를 속행시킨 경우에도 다음 정지가 시스템-호출-퇴장-정지가 된다고 보장되지는 않는다. 그렇지 않고 피추적자가 (seccomp 정지를 포함한) `PTRACE_EVENT` 정지에서 멈추거나, (<tt>[[_exit(2)]]</tt>나 <tt>[[exit_group(2)]]</tt>에 들어갔던 것이면) 종료하거나, `SIGKILL`에 의해 죽거나, 조용히 죽을 수도 (피추적자가 스레드 그룹 리더이고, 다른 스레드에서 <tt>[[execve(2)]]</tt>가 이뤄지고, 그 스레드를 동일 추적자가 추적하고 있지 않을 때. 잠시 후 이 경우를 논의함.) 있다.

<tt>[[waitpid(2)]]</tt>가 `WIFSTOPPED(status)`를 참으로 반환하는 것으로 추적자가 시스템-호출-진입-정지와 시스템-호출-퇴장-정지를 목격하며, `WSTOPSIG(status)`는 `SIGTRAP`을 내놓는다. 추적자가 `PTRACE_O_TRACESYSGOOD` 옵션을 설정했으면 `WSTOPSIG(status)`가 `(SIGTRAP | 0x80)` 값을 내놓게 된다.

시스템-호출-정지를 `SIGTRAP` 시그널-전달-정지와 구별하려면 `PTRACE_GETSIGINFO`를 질의해서 다음 경우를 확인하면 된다.

`si_code <= 0`
:   시스템 호출(<tt>[[tgkill(2)]]</tt>, <tt>[[kill(2)]]</tt>, <tt>[[sigqueue(3)]]</tt> 등) 같은 사용자 공간 행동이나 POSIX 타이머의 만료, POSIX 메시지 큐에서의 상태 변화, 비동기 I/O 요청의 완료에 의해서 `SIGTRAP`이 전달되었다.

`si_code == SI_KERNEL` (0x80)
:   커널이 `SIGTRAP`을 보냈다.

`si_code == SIGTRAP` 또는 `si_code == (SIGTRAP|0x80)`
:   시스템-호출-정지이다.

하지만 시스템-호출-정지가 매우 자주 (시스템 호출당 두 번씩) 일어나므로 시스템-호출-정지마다 `PTRACE_GETSIGINFO`를 수행하는 것은 비용이 좀 높을 수도 있다.

일부 아키텍처에서는 레지스터를 검사해서 그 경우들을 구별할 수 있다. 예를 들어 x86에서는 시스템-호출-진입-정지에서 `rax == -ENOSYS`이다. `SIGTRAP`이 (다른 시그널들과 마찬가지로) 언제나 시스템-호출-퇴장-정지 후에 발생하며 그 시점에 `rax`가 `-ENOSYS`를 담을 가능성은 거의 없으므로 `SIGTRAP`은 "시스템-호출-진입-정지 아닌 시스템-호출-정지"처럼 보인다. 다시 말해 "짝 잃은 시스템-호출-퇴장-정지"처럼 보이므로 이 방법으로 탐지할 수 있다. 하지만 그런 탐지 방법은 잘못되기 쉬우므로 가급적 피하는 게 좋다.

시스템-호출-정지를 다른 종류의 ptrace 정지들과 구별하기 위한 권장하는 방식은 `PTRACE_O_TRACESYSGOOD` 옵션을 쓰는 것이다. 믿을 수 있으며 성능 비용을 유발하지 않는다.

추적자에게 시스템-호출-진입-정지와 시스템-호출-퇴장-정지는 구별이 불가능하다. 시스템-호출-진입-정지를 시스템-호출-퇴장-정지로, 또는 그 반대로 잘못 해석하지 않으려면 추적자에서 ptrace 정지들을 추적할 필요가 있다. 일반적으로 시스템-호출-진입-정지 다음에는 항상 시스템-호출-퇴장-정지나 `PTRACE_EVENT` 정지, 또는 피추적자의 죽음이 따라온다. 그 사이에서 다른 어떤 종류의 ptrace 정지도 일어날 수 없다. 하지만 seccomp 정지(아래 참고)는 선행하는 시스템-호출-진입-정지 없이 시스템-호출-퇴장-정지를 일으킬 수 있다. seccomp를 사용하는 경우 그런 정지를 시스템-호출-진입-정지로 잘못 해석하지 않도록 주의를 기울일 필요가 있다.

시스템-호출-진입-정지 후에 추적자가 `PTRACE_SYSCALL` 외의 재시작 명령을 사용하면 시스템-호출-퇴장-정지가 발생하지 않는다.

시스템-호출 정지에서 `PTRACE_GETSIGINFO`는 `si_signo`에 `SIGTRAP`을 반환하며 `si_code`가 `SIGTRAP`이나 `(SIGTRAP|0x80)`으로 설정되어 있다.

### `PTRACE_EVENT_SECCOMP` 정지 (리눅스 3.5에서 4.7까지)

`PTRACE_EVENT_SECCOMP` 정지의 동작과 다른 ptrace 정지들과의 상호 작용 방식이 커널 버전에 따라 바뀌었다. 여기서는 도입 때부터 리눅스 4.7까지의 동작 방식을 적는다. 이후 커널 버전에서의 동작은 다음 절에 적는다.

`SECCOMP_RET_TRACE` 규칙이 걸릴 때마다 `PTRACE_EVENT_SECCOMP` 정지가 일어난다. 어떤 방법으로 시스템 호출을 재시작했는지와는 무관하다. 특히 `PTRACE_SYSEMU`로 피추적자를 재시작해서 이 시스템 호출을 무조건 건너뛰는 경우에도 seccomp가 동작한다.

이 정지에서 재시작하면 해당 시스템 호출 바로 전에서 정지가 일어났던 것처럼 동작하게 된다. 특히 `PTRACE_SYSCALL`과 `PTRACE_SYSEMU` 모두 정상적으로 이어지는 시스템-호출-진입-정지를 일으키게 된다. 하지만 `PTRACE_EVENT_SECCOMP` 후에 시스템 호출 번호가 음수이면 시스템-호출-진입-정지와 시스템 호출 자체를 건너뛰게 된다. 즉, `PTRACE_EVENT_SECCOMP` 후에 시스템 호출 번호가 음수이고 `PTRACE_SYSCALL`로 피추적자를 재시작하는 경우에 다음으로 목격하는 정지는 어쩌면 예상했을 시스템-호출-진입-정지가 아니라 시스템-호출-퇴장-정지가 된다.

### `PTRACE_EVENT_SECCOMP` 정지 (리눅스 4.8부터)

리눅스 4.8부터는 `PTRACE_EVENT_SECCOMP` 정지가 시스템-호출-진입-정지와 시스템-호출-퇴장-정지 사이에서 일어나도록 순서가 바뀌었다. 그래서 `PTRACE_SYSEMU` 때문에 시스템 호출을 건너뛰는 경우 seccomp이 더 이상 돌지 않는다. (그래서 `PTRACE_EVENT_SECCOMP`가 보고되지 않는다.)

기능적으로 `PTRACE_EVENT_SECCOMP` 정지는 시스템-호출-진입-정지와 비슷하게 기능한다. (즉, `PTRACE_SYSCALL`로 속행시키면 시스템-호출-퇴장-정지가 발생하고, 시스템 호출 번호를 바꿀 수 있으며, 다른 레지스터를 변경하면 그 내용 역시 실행될 시스템 호출에게 보인다.) 참고로 선행 시스템-호출-진입-정지가 있을 수도 있지만 꼭 있어야 하는 것은 아니다.

`PTRACE_EVENT_SECCOMP` 정지 후에 `SECCOMP_RET_ALLOW`와 같은 기능을 하는 `SECCOMP_RET_TRACE` 규칙으로 seccomp를 다시 돌리게 된다. 이게 분명하게 뜻하는 바는 `PTRACE_EVENT_SECCOMP` 정지 중 레지스터들을 변경하지 않으면 시스템 호출이 허용된다는 것이다.

### `PTRACE_SINGLESTEP` 정지

[이 정지 유형에 대한 세부 내용은 아직 문서화가 이뤄지지 않았다.]

### 정보형 및 재시작형 ptrace 명령

대부분의 (`PTRACE_ATTACH`, `PTRACE_SEIZE`, `PTRACE_TRACEME`, `PTRACE_INTERRUPT`, `PTRACE_KILL`을 제외한 모든) ptrace 명령에는 ptrace 정지 상태의 피추적자가 필요하며, 없으면 `ESRCH`로 실패한다.

피추적자가 ptrace 정지 상태에 있을 때 추적자가 정보형 명령들을 이용해 피추적자의 데이터를 읽거나 쓸 수 있다. 이 명령들은 피추적자를 ptrace 정지 상태 그대로 둔다.

```cc
ptrace(PTRACE_PEEKTEXT/PEEKDATA/PEEKUSER, pid, addr, 0);
ptrace(PTRACE_POKETEXT/POKEDATA/POKEUSER, pid, addr, long_val);
ptrace(PTRACE_GETREGS/GETFPREGS, pid, 0, &struct);
ptrace(PTRACE_SETREGS/SETFPREGS, pid, 0, &struct);
ptrace(PTRACE_GETREGSET, pid, NT_foo, &iov);
ptrace(PTRACE_SETREGSET, pid, NT_foo, &iov);
ptrace(PTRACE_GETSIGINFO, pid, 0, &siginfo);
ptrace(PTRACE_SETSIGINFO, pid, 0, &siginfo);
ptrace(PTRACE_GETEVENTMSG, pid, 0, &long_var);
ptrace(PTRACE_SETOPTIONS, pid, 0, PTRACE_O_flags);
```

참고로 일부 오류들은 보고가 되지 않는다. 예를 들어 일부 ptrace 정지들에서는 시그널 정보(`siginfo`) 설정이 효과가 없을 수도 있는데, 그래도 호출이 성공을 반환할 (0을 반환하고 `errno`를 설정하지 않을) 수 있다. 또 현재 ptrace 정지에서 어떤 의미 있는 이벤트 메시지를 반환한다고 되어 있지 않은데 `PTRACE_GETEVENTMSG` 질의를 하면 성공하면서 어떤 임의 값을 반환할 수 있다.

다음 호출은 한 피추적자에 영향을 준다.

```c
ptrace(PTRACE_SETOPTIONS, pid, 0, PTRACE_O_flags);
```

그 피추적자의 현재 플래그가 바뀐다. `PTRACE_O_TRACEFORK`, `PTRACE_O_TRACEVFORK`, `PTRACE_O_TRACECLONE` 옵션을 통해 생성 및 "자동 붙기" 된 새 피추적자들이 그 플래그를 물려받는다.

또 다른 명령들은 ptrace 정지 상태인 피추적자가 돌게 한다. 다음 형태이다.

```c
ptrace(cmd, pid, 0, sig);
```

여기서 `cmd`는 `PTRACE_CONT`, `PTRACE_LISTEN`, `PTRACE_DETACH`, `PTRACE_SYSCALL`, `PTRACE_SINGLESTEP`, `PTRACE_SYSEMU`, `PTRACE_SYSEMU_SINGLESTEP` 중 하나이다. 피추적자가 시그널-전달-정지에 있는 경우 `sig`는 (0 아닌 경우) 주입할 시그널이다. 그 외의 경우 `sig`는 무시될 수 있다. (시그널-전달-정지 아닌 ptrace 정지에서 피추적자를 재시작할 때 권장하는 방식은 `sig`에 항상 0을 주는 것이다.)

### 붙기와 떨어지기

다음 중 한 호출을 이용해 스레드를 추적자에게 붙일 수 있다.

```c
ptrace(PTRACE_ATTACH, pid, 0, 0);
```

```c
ptrace(PTRACE_SEIZE, pid, 0, PTRACE_O_flags);
```

`PTRACE_ATTACH`는 이 스레드에게 `SIGSTOP`을 보낸다. 이 `SIGSTOP`이 효력이 없기를 원하면 추적자는 이를 억제해야 한다. 참고로 붙이기 도중 이 스레드에게 다른 시그널을 동시에 보내면 피추적자가 그 다른 시그널로 먼저 시그널-전달-정지에 들어가는 것을 추적자가 볼 수도 있다! 일반적인 관행은 `SIGSTOP`을 볼 때까지 그 시그널들을 재주입하고서 `SIGSTOP` 주입을 억제하는 것이다. 여기서의 설계 버그는 ptrace 붙이기와 그와 동시에 전달되는 `SIGSTOP`이 경쟁할 수 있고, 그래서 그 동시 `SIGSTOP`이 유실될 수도 있다는 점이다.

붙이기를 하면 `SIGSTOP`이 가는데 일반적으로 추적자가 이를 억제하므로 "시그널 주입과 억제" 절에서 서술한 것처럼 현재 실행 중인 시스템 호출에서 느닷없이 `EINTR` 반환이 일어나게 할 수도 있다.

리눅스 3.4부터 `PTRACE_ATTACH` 대신 `PTRACE_SEIZE`를 쓸 수 있다. `PTRACE_SEIZE`는 붙은 프로세스를 멈추지 않는다. 붙은 후에 (또는 다른 어느 때에도) 아무 시그널도 보내지 않고 그 프로세스를 멈춰야 하면 `PTRACE_INTERRUPT` 명령을 사용하면 된다.

다음 요청은 호출 스레드를 피추적자로 바꾼다.

```c
ptrace(PTRACE_TRACEME, 0, 0, 0);
```

스레드가 실행을 계속한다. (ptrace 정지에 들어가지 않는다.) 그리고 흔히 `PTRACE_TRACEME`에 이어 다음을 해서 (이제 추적자가 된) 부모가 시그널-전달-정지를 목격하게 한다.

```c
raise(SIGSTOP);
```

`PTRACE_O_TRACEFORK`나 `PTRACE_O_TRACEVFORK`, `PTRACE_O_TRACECLONE` 옵션이 적용 중이면 각기 <tt>[[vfork(2)]]</tt>나 `CLONE_VFORK` 플래그 사용 <tt>[[clone(2)]]</tt>, <tt>[[fork(2)]]</tt>나 종료 시그널을 `SIGCHLD`로 설정한 <tt>[[clone(2)]]</tt>, 다른 종류의 <tt>[[clone(2)]]</tt>으로 생성된 자식이 부모를 추적하던 동일 추적자에 자동으로 붙는다. 자식에게 `SIGSTOP`이 전달되어 자식을 생성한 시스템 호출에서 빠져나온 후 자식이 시그널-전달-정지에 들어가게 된다.

다음 호출로 피추적자에서 떨어진다.

```c
ptrace(PTRACE_DETACH, pid, 0, sig);
```

`PTRACE_DETACH`는 재시작형 동작이다. 따라서 피추적자가 ptrace 정지 상태여야 한다. 피추적자가 시그널-전달-정지에 있으면 시그널을 주입할 수 있다. 그 외의 경우 `sig` 매개변수가 조용히 무시될 수 있다.

추적자가 떨어지고 싶을 때 피추적자가 실행 중인 경우 일반적인 해결책은 `SIGSTOP`을 보내고 (올바른 스레드로 가도록 하기 위해 <tt>[[tgkill(2)]]</tt> 사용), 피추적자가 `SIGSTOP`에 대한 시그널-전달-정지에서 멈추기를 기다리고, (`SIGSTOP`을 주입을 억제하면서) 떨어지는 것이다. 여기의 설계 버그는 동시에 발생한 `SIGSTOP`과 경쟁할 수 있다는 점이다. 다른 문제는 피추적자가 다른 ptrace 정지에 들어갈 수도 있어서 `SIGSTOP`을 볼 때까지 다시 재시작하고 기다려야 한다는 것이다. 그리고 또 다른 문제는 피추적자가 이미 ptrace 정지에 있지 않음을 확실히 하는 것이다. 그때는 어떤 시그널 전달도 (`SIGSTOP`도) 이뤄지지 않기 때문이다.

추적자가 죽으면 그룹-정지에 있었던 게 아니면 모든 피추적자들이 자동으로 떨어지고 재시작된다. 그룹-정지에서의 재시작 처리에는 현재 버그가 있지만 "계획 상" 동작은 피추적자가 그대로 멈춰서 `SIGCONT`를 기다리게 두는 것이다. 피추적자가 시그널-전달-정지에서 재시작되는 경우 대기 중인 시그널이 주입된다.

### ptrace 하의 <tt>[[execve(2)]]</tt>

다중 스레드 프로세스의 한 스레드가 <tt>[[execve(2)]]</tt>를 호출하면 커널에서 그 프로세스의 다른 스레드들을 모두 없애고 exec 한 스레드의 스레드 ID를 스레드 그룹 ID(프로세스 ID)로 재설정한다. (다른 식으로 말하면, 다중 스레드 프로세스에서 <tt>[[execve(2)]]</tt>를 하면 어떤 스레드가 <tt>[[execve(2)]]</tt>를 했는지와 무관하게 호출 완료 시점에는 스레드 그룹 리더에서 <tt>[[execve(2)]]</tt>가 일어난 것처럼 보인다.) 이런 스레드 ID 재설정이 추적자에게는 혼란스러워 보인다.

* `PTRACE_O_TRACEEXIT` 옵션이 켜졌으면 다른 스레드 모두가 `PTRACE_EVENT_EXIT` 정지에서 멈춘다. 그러고서 스레드 그룹 리더를 제외한 다른 스레드 모두가 종료 코드 0으로 <tt>[[_exit(2)]]</tt>을 통해 끝난 것처럼 죽음을 보고한다.

* exec 하는 피추적자가 <tt>[[execve(2)]]</tt> 내에 있는 동안 자기 스레드 ID를 바꾼다. (기억하겠지만 ptrace에서 <tt>[[waitpid(2)]]</tt>가 반환하거나 ptrace 호출에 넣어 주는 "pid"는 피추적자의 스레드 ID이다.) 즉 피추적자의 스레드 ID가 프로세스 ID, 즉 스레드 그룹 리더의 스레드 ID와 같아지게 재설정된다.

* 그리고 `PTRACE_O_TRACEEXEC` 옵션이 켜졌으면 `PTRACE_EVENT_EXEC` 정지가 일어난다.

* 이 시점 전에 스레드 그룹 리더가 `PTRACE_EVENT_EXIT`를 보고했다면 추적자에게는 죽은 스레드 리더가 "난데없이 다시 나타난" 것처럼 보인다. (참고: 적어도 한 개의 다른 살아 있는 스레드가 있기 전에는 스레드 그룹 리더가 `WIFEXITED(status)`를 통해 죽음을 보고하지 않는다. 이 때문에 추적자에게 스레드 그룹 리더가 죽었다가 다시 나타나는 것으로 보일 가능성이 없어진다.) 스레드 그룹 리더가 아직 살아 있었다면 추적자에게 스레드 그룹 리더가 들어간 것과 다른 시스템 호출에서 반환하는 것처럼 보이거나, 심지어 "아무 시스템 호출 안에도 있지 않다가 시스템 호출에서 반환"하는 것으로 보일 수도 있다. 스레드 그룹 리더를 추적하고 있지 않았다면 (또는 다른 추적자가 추적하고 있었다면) <tt>[[execve(2)]]</tt> 과정에서 exec 한 피추적자의 추적자의 피추적자가 된 것처럼 보일 것이다.

위의 효과 모두가 피추적자 스레드 ID 변경의 산물이다.

`PTRACE_O_TRACEEXEC` 옵션이 이런 상황에 대처하기 위한 권장 도구이다. 첫째로, <tt>[[execve(2)]]</tt> 반환 전에 일어나는 `PTRACE_EVENT_EXEC` 정지를 켠다. 그 정지에서 추적자가 `PTRACE_GETEVENTMSG`를 사용해서 피추적자의 이전 스레드 ID를 가져올 수 있다. (이 기능은 리눅스 3.0에서 추가되었다.) 둘째로, `PTRACE_O_TRACEEXEC` 옵션이 <tt>[[execve(2)]]</tt>에 대한 구식 `SIGTRAP` 생성을 끈다.

추적자가 `PTRACE_EVENT_EXEC` 정지 알림을 받을 때 그 피추적자와 스레드 그룹 리더를 제외하고 프로세스의 다른 어떤 스레드도 살아 있지 않다는 것이 보장된다.

`PTRACE_EVENT_EXEC` 정지 알림 수신 시 추적자는 그 프로세스의 스레드들을 기술하는 내부 자료 구조를 모두 정리하고 단 하나, 다음 조건에 해당하는 아직 돌고 있는 피추적자를 기술하는 자료 구조만을 유지해야 할 것이다.

```text
스레드 ID == 스레드 그룹 ID == 프로세스 ID
```

예: 두 스레드가 동시에 <tt>[[execve(2)]]</tt> 호출:

```text
*** we get syscall-enter-stop in thread 1: **
PID1 execve("/bin/foo", "foo" <unfinished ...>
*** we issue PTRACE_SYSCALL for thread 1 **
*** we get syscall-enter-stop in thread 2: **
PID2 execve("/bin/bar", "bar" <unfinished ...>
*** we issue PTRACE_SYSCALL for thread 2 **
*** we get PTRACE_EVENT_EXEC for PID0, we issue PTRACE_SYSCALL **
*** we get syscall-exec-stop for PID0: **
PID0 <... execve resumed> )             = 0
```

exec 하는 피추적자에 `PTRACE_O_TRACEEXEC`가 적용 중이지 *않고* `PTRACE_SEIZE`가 아닌 `PTRACE_ATTACH`로 피추적자에 붙었던 경우 <tt>[[execve(2)]]</tt> 반환 후 커널이 피추적자에게 추가 `SIGTRAP`를 전달한다. 이는 평범한 (`kill -TRAP`으로 생성할 수 있는 것과 비슷한) 시그널이며 특별한 종류의 ptrace 정지가 아니다. 이 시그널에 `PTRACE_GETSIGINFO` 하면 `si_code`가 0(`SI_USER`)으로 설정돼서 반환된다. 시그널 마스크로 이 시그널을 블록 할 수 있고, 따라서 (훨씬) 나중에 전달될 수도 있다.

일반적으로 추적자(가령 <tt>[[strace(1)]]</tt>)는 execve 후의 이 추가 `SIGTRAP` 시그널을 사용자에게 보이고 싶지 않을 것이고 피추적자에게 전달되는 것을 억제하려 할 것이다. (`SIGTRAP`이 `SIG_DFL`로 설정되어 있으면 죽이기형 시그널이다.) 하지만 *어떤* `SIGTRAP`을 억제할지 판단하는 것이 쉽지 않다. `PTRACE_O_TRACEEXEC` 옵션을 설정하거나 `PTRACE_SEIZE`를 사용해서 이 추가 `SIGTRAP`을 금하는 것이 권장하는 방식이다.

### 진짜 부모

ptrace API는 <tt>[[waitpid(2)]]</tt>를 통한 표준 유닉스 부모/자식 신호 전달을 이용(내지 오용)한다. 이 때문에 어떤 다른 프로세스가  자식 프로세스를 추적할 때 진짜 부모가 여러 종류의 <tt>[[waitpid(2)]]</tt> 알림을 받지 못하게 되곤 했다.

이런 버그가 많이 고쳐졌지만 리눅스 2.6.38 기준으로 아직 여러 개가 남아 있다. 아래 BUGS 참고.

2.6.38 기준으로 다음 사항들이 올바로 동작하는 것 같다.

* 시그널에 의한 종료/죽음이 먼저 추적자에게 보고되고, 추적자가 <tt>[[waitpid(2)]]</tt> 결과를 소모할 때 진짜 부모에게 (다중 스레드 프로세스 전체가 끝날 때만 진짜 부모에게) 보고된다. 추적자와 진짜 부모가 같은 프로세스이면 보고가 한 번만 간다.

## RETURN VALUE

성공 시 `PTRACE_PEEK*` 요청은 요청한 데이터를 반환하고 (하지만 NOTES 참고), `PTRACE_SECCOMP_GET_FILTER` 요청은 BPF 프로그램의 인스트럭션 수를 반환하며, 다른 요청은 0을 반환한다.

오류 시 모든 요청이 -1을 반환하며 `errno`를 적절히 설정한다. `PTRACE_PEEK*` 요청이 성공 시 반환하는 값이 -1일 수도 있기 때문에 오류가 발생했는지 알려면 호출자가 호출 전에 `errno`를 비우고서 호출 후 검사해야 한다.

## ERRORS

`EBUSY`
:   (i386 한정) 디버그 레지스터 할당 내지 해제 중에 오류가 있었다.

`EFAULT`
:   추적자나 피추적자의 메모리 내의 유효하지 않은 영역에 대한 읽기나 쓰기 시도가 있었다. 아마 그 영역이 매핑 되어 있지 않거나 접근 가능하지 않아서일 것이다. 유감스럽게도 리눅스에서는 이 문제의 여러 변종들이 다소 임의적으로 `EIO`나 `EFAULT`를 반환한다.

`EINVAL`
:   유효하지 않은 옵션을 설정하려고 시도했다.

`EIO`
:   `request`가 유효하지 않거나, 추적자가 피추적자의 메모리 내의 유효하지 않은 영역에 대한 읽기나 쓰기 시도가 있었거나, 워드 정렬 위반이 있었거나, 재시작 요청 중에 유효하지 않은 시그널을 지정했다.

`EPERM`
:   지정한 프로세스를 추적할 수 없다. 추적자가 충분한 특권을 가지고 있지 않아서일 수 있다. (필요한 역능은 `CAP_SYS_PTRACE`이다.) 비특권 프로세스가 시그널을 보낼 수 없는 프로세스들을 추적할 수 없는 것이고, 또 당연한 이유로 set-user-ID/set-group-ID 프로그램을 돌리는 프로세스를 추적할 수 없다. 또는, 프로세스가 이미 추적되고 있거나 (2.6.26 전의 커널에서) 프로세스가 `init(1)` (PID 1)이다.

`ESRCH`
:   지정한 프로세스가 존재하지 않거나, 현재 호출자가 추적 중이 아니거나, (피추적자가 멈춰 있어야 하는 요청들에서) 멈춰있지 않다.

## CONFORMING TO

SVr4, 4.3BSD.

## NOTES

앞서 제시한 원형에 따라 `ptrace()` 인자를 해석하지만 현재 glibc에서는 `ptrace()`를 `request` 인자만 고정된 가변 인자 함수로 선언하고 있다. 요청 동작에서 사용하지 않더라도 항상 인자 네 개를 제공하기를 권장한다. 안 쓰거나 무시하는 인자는 `0L`이나 `(void *) 0`으로 설정하면 된다.

리눅스 커널 2.6.26 전에서는 PID 1 프로세스인 `init(1)`를 추적할 수 없다.

피추적자의 부모는 `execve(2)`를 호출해도 계속 추적자이다.

메모리 및 USER 영역의 내용물 배치는 운영 체제와 아키텍처에 상당히 의존적이다. 제공되는 오프셋과 반환된 데이터가 `struct user` 정의와 완전하게 일치하지는 않을 수도 있을 것이다.

"워드"의 크기는 운영 체제 종류에 따라 정해진다. (가령 32비트 리눅스에서는 32비트이다.)

이 페이지는 현재 리눅스에서 `ptrace()`가 어떻게 동작하는지 적은 것이다. 그 동작 방식은 다른 UNIX 변종들과 상당히 다르다. 어떤 경우이든 `ptrace()` 사용은 운영 체제와 아키텍처에 고도로 의존적이다.

### ptrace 접근 모드 검사

(`ptrace()` 동작뿐 아니라) 커널-사용자 공간 API의 여러 부분에서 소위 "ptrace 접근 모드" 검사를 요구하여 그 결과에 따라 동작을 허용할지 여부를 (또는 일부 경우에 "읽기" 동작이 검열된 데이터를 반환할지 여부를) 결정한다. 한 프로세스가 다른 프로세스에 대한 민감한 정보를 검사하거나 때에 따라 프로세스 상태를 변경할 수 있는 경우에 그런 검사를 수행한다. 검사는 두 프로세스의 크리덴셜과 역능, "대상" 프로세스가 덤프 가능한지 여부, 활성화된 (SELinux, Yama, Smack 같은) 리눅스 보안 모듈(LSM) 및 (항상 호출되는) commoncap LSM이 수행한 검사 결과 같은 인자들에 따라 이뤄진다.

리눅스 2.6.27 전에서는 모든 접근 검사가 한 종류였다. 리눅스 2.6.27부터는 두 가지 접근 모드 단계를 구별한다.

`PTRACE_MODE_READ`
:   "읽기" 동작이나 덜 위험한 동작들: <tt>[[get_robust_list(2)]]</tt>, <tt>[[kcmp(2)]]</tt>, `/proc/[pid]/auxv`이나 `/proc/[pid]/environ`, `/proc/[pid]/stat` 읽기, `/proc/[pid]/ns/*` 파일 <tt>[[readlink(2)]]</tt>

`PTRACE_MODE_ATTACH`
:   "쓰기" 동작이나 더 위험한 동작들: 다른 프로세스에 ptrace 붙기 (`PTRACE_ATTACH`)나 <tt>[[process_vm_writev(2)]]</tt> 호출. (리눅스 2.6.27 전에서는 `PTRACE_MODE_ATTACH`가 기본인 것과 같았다.)

리눅스 4.5부터 위 접근 모드 검사를 다음 수식자 중 하나와 결합(OR)한다.

`PTRACE_MODE_FSCREDS`
:   LSM 검사에 호출자의 파일 시스템 UID/GID (<tt>[[credentials(7)]]</tt> 참고) 또는 실효 역능 사용.

`PTRACE_MODE_REALCREDS`
:   LSM 검사에 호출자의 실제 UID/GID나 허용 역능 사용. 리눅스 4.5 전에서는 이 방식이 기본인 것과 같았다.

위 크리덴셜 수식자와 앞서 언급한 접근 모드를 결합해서 쓰는 게 일반적이므로 커널 소스에는 그 조합들에 대한 매크로가 정의되어 있다.

`PTRACE_MODE_READ_FSCREDS`
:   `PTRACE_MODE_READ | PTRACE_MODE_FSCREDS`로 정의.

`PTRACE_MODE_READ_REALCREDS`
:   `PTRACE_MODE_READ | PTRACE_MODE_REALCREDS`로 정의.

`PTRACE_MODE_ATTACH_FSCREDS`
:   `PTRACE_MODE_ATTACH | PTRACE_MODE_FSCREDS`로 정의.

`PTRACE_MODE_ATTACH_READLCREDS`
:   `PTRACE_MODE_ATTACH | PTRACE_MODE_REALCREDS`로 정의.

접근 모드에 수식자 하나를 더 OR 할 수 있다.

`PTRACE_MODE_NOAUDIT` (리눅스 3.3부터)
:   이 접근 모드 검사를 감사하지 않는다. 이 수식자는 호출자에게 오류를 반환시키기보다는 출력이 걸러지거나 검열되게 하기만 하는 (`/proc/[pid]/stat` 읽을 때의 검사 같은) ptrace 접근 모드 검사에 쓰인다. 그런 경우에 파일 접근은 보안 위반이 아니므로 보안 감사 기록을 생성할 이유가 없다. 이 수식자는 특정 접근 검사에 대해 보안 기록 생성을 억제한다.

참고로 이 부절에서 서술한 `PTRACE_MODE_*` 상수들은 모두 커널 내부용이어서 사용자 공간에 보이지 않는다. 여기서 상수 이름을 언급한 것은 여러 시스템 호출과 여러 (가령 `/proc` 아래의) 가상 파일 접근에 대해 수행하는 다양한 ptrace 접근 모드 검사들의 종류에 이름을 붙이기 위해서이다. 이 이름을 다른 매뉴얼 페이지에서 사용해서 다양한 커널 검사를 간단한 방식으로 지칭한다.

ptrace 접근 모드 검사에 쓰이는 알고리듬은 호출 프로세스가 대상 프로세스에 해당 행위를 수행하는 것이 허용되는지 판단한다. (`/proc/[pid]` 파일 열기의 경우 "호출 프로세스"는 파일을 여는 프로세스이고 해당 PID를 가진 프로세스가 "대상 프로세스"이다.) 알고리듬은 다음과 같다.

1. 호출 스레드와 대상 스레드가 같은 스레드 그룹에 속하면 접근을 항상 허용한다.

2. 접근 모드에 `PTRACE_MODE_FSCREDS`가 지정돼 있으면 다음 단계의 검사에서 호출자의 파일 시스템 UID 및 GID를 사용한다. (<tt>[[credentials(7)]]</tt>에서 언급하듯 파일 시스템 UID와 GID는 거의 언제나 대응하는 실효 ID와 값이 같다.)

   그렇지 않고 접근 모드에 `PTRACE_MODE_REALCREDS`가 지정돼 있으면 다음 단계의 검사에 호출자의 실제 UID 및 GID를 사용한다. (호출자의 UID와 GID를 검사하는 대부분의 API에서는 실효 ID를 사용한다. 역사적 이유 때문에 `PTRACE_MODE_REALCREDS` 검사에서는 실제 ID를 사용한다.)

3. 다음 중 어느 것도 참이 아니면 접근을 거부한다.

   * 대상의 실제, 실효, saved-set 사용자 ID가 호출자의 사용자 ID와 일치하고 대상의 실제, 실효, saved-set 그룹 ID가 호출자의 그룹 ID와 일치한다.

   * 호출자가 대상의 사용자 네임스페이스에서 `CAP_SYS_PTRACE` 역능을 가지고 있다.

4. 대상 프로세스의 "덤프 가능" 속성이 1 아닌 값을 가지고 있으며 (`SUID_DUMP_USER`. <tt>[[prctl(2)]]</tt>의 `PR_SET_DUMPABLE` 논의 참고) 호출자가 대상 프로세스의 사용자 네임스페이스에서 `CAP_SYS_PTRACE` 역능을 가지고 있지 않으면 접근을 거부한다.

5. 커널 LSM `security_ptrace_access_check()` 인터페이스를 호출해서 ptrace 접근이 허용되는지 알아본다. 결과는 LSM(들)에 달려 있다. commoncap LSM의 이 인터페이스 구현에서는 다음 단계들을 수행한다.

   a) 접근 모드에 `PTRACE_MODE_FSCREDS`가 포함돼 있으면 다음 검사에서 호출자의 *실효* 역능 집합을 사용한다. 그렇지 않으면 (접근 모드에 `PTRACE_MODE_REALCREDS`가 지정돼 있으면) 호출자의 *허용* 역능 집합을 사용한다.

   b) 다음 중 어느 것도 참이 아니면 접근을 거부한다.

      * 호출자와 대상 프로세스가 같은 사용자 네임스페이스 안에 있으며, 호출자의 역능이 대상 프로세스의 *허용* 역능의 상위집합이다.

      * 호출자가 대상 프로세스의 사용자 네임스페이스에서 `CAP_SYS_PTRACE` 역능을 가지고 있다.

      참고로 commoncap LSM에서는 `PTRACE_MODE_READ`와 `PTRACE_MODE_ATTACH`를 구분하지 않는다.

6. 이전 단계들에서 접근이 거부되지 않았으면 접근을 허용한다.

### `/proc/sys/kernel/yama/ptrace_scope`

Yama 리눅스 보안 모듈(LSM)이 설치된 (즉 `CONFIG_SECURITY_YAMA`로 커널을 구성한) 시스템에서는 (리눅스 3.4부터 사용 가능한) `/proc/sys/kernel/yama/ptrace_scope` 파일을 이용해 `ptrace()`로 프로세스를 추적하는 것을 (그래서 <tt>[[strace(1)]]</tt>나 <tt>[[gdb(1)]]</tt> 같은 도구 사용을) 제약할 수 있다. 탈취된 프로세스가 그 사용자가 소유한 다른 민감한 프로세스(가령 GPG 에이전트나 SSH 세션)에 ptrace로 붙어서 메모리 내에 있을 수 있는 추가 크리덴셜을 얻어서 공격 범위를 넓히는 확대 공격을 막는 것이 그 제약의 목적이다.

더 엄밀하게 말해 Yama LSM은 다음 두 종류의 동작을 제한한다.

* ptrace 접근 모드 `PTRACE_MODE_ATTACH` 검사를 수행하는 모든 동작. 예를 들어 `ptrace()`, `PTRACE_ATTACH`. (위의 "ptrace 접근 모드 검사" 참고.)

* `ptrace()` `PTRACE_TRACEME`

`CAP_SYS_PTRACE` 역능을 가진 프로세스가 `/proc/sys/kernel/yama/ptrace_scope` 파일을 다음 값들 중 하나로 갱신할 수 있다.

0 ("전통적 ptrace 권한")
:   `PTRACE_MODE_ATTACH` 검사를 수행하는 동작에 (commoncap과 다른 LSM에서 부과하는 것 이상으로) 추가로 제약을 가하지 않는다.

    `PTRACE_TRACEME` 사용에 변화가 없다.

1 ("제약된 ptrace") [기본값]
:   `PTRACE_MODE_ATTACH` 검사가 필요한 동작을 수행할 때 호출 프로세스가 대상 프로세스의 사용자 네임스페이스에서 `CAP_SYS_PTRACE` 역능을 가지고 있거나 대상 프로세스와 기정 관계를 가지고 있어야 한다. 기본적으로 기정 관계란 대상 프로세스가 호출자의 자손이어야 한다는 것이다.

    대상 프로세스에서 <tt>[[prctl(2)]]</tt> `PR_SET_PTRACER` 동작을 사용해서 그 대상에 `PTRACE_MODE_ATTACH` 동작을 수행할 수 있게 허용할 추가 PID를 선언할 수 있다. 자세한 내용은 커널 소스 파일 `Documentation/admin-guide/LSM/Yama.rst`를 (리눅스 4.13 전에선 `Documentation/security/Yama.txt`를) 보라.

    `PTRACE_TRACEME` 사용에 변화가 없다.

2 ("관리자만 붙기")
:    대상 프로세스의 사용자 네임스페이스에서 `CAP_SYS_PTRACE` 역능을 가진 프로세스만 `PTRACE_MODE_ATTACH` 동작 수행이나 `PTRACE_TRACEME` 사용 자식 추적을 할 수 있다.

3 ("붙기 불가능")
:   어떤 프로세스도 `PTRACE_MODE_ATTACH` 동작 수행이나 `PTRACE_TRACEME` 사용 자식 추적을 수행할 수 없다.

    파일에 이 값을 한번 써넣고 나면 바꿀 수 없다.

1과 2 값과 관련해서, 새 사용자 네임스페이스를 생성하면 Yama가 제공하는 보호가 실질적으로 무력화된다는 점에 유의해야 한다. 실효 UID가 자식 사용자 네임스페이스 생성자의 UID와 일치하는 부모 네임스페이스 내의 프로세스가 그 자식 사용자 네임스페이스 (그리고 더 먼 자손들) 내에서 동작을 수행할 때 (`CAP_SYS_PTRACE`를 포함한) 모든 역능을 가지기 때문이다. 그래서 프로세스가 스스로 샌드박스에 들어가려고 네임스페이스를 사용하려 할 때 Yama LSM이 제공하는 보호를 의도치 않게 약화시키게 된다.

### C 라이브러리/커널 차이

시스템 호출 수준에서 `PTRACE_PEEKTEXT`, `PTRACE_PEEKDATA`, `PTRACE_PEEKUSER` 요청은 API가 다르다. `data` 매개변수로 지정한 주소에 결과를 저장하며 반환 값은 오류 플래그이다. glibc 래퍼 함수가 위 DESCRIPTION의 설명처럼 함수 반환 값을 통해 결과를 반환하는 API를 제공한다.

## BUGS

2.6 커널 헤더 사용 호스트에서 `PTRACE_SETOPTIONS`가 2.4에서와 다른 값으로 선언되어 있다. 이 때문에 2.6 커널 헤더로 컴파일 한 응용을 2.4 커널에서 돌릴 때 문제가 생긴다. `PTRACE_OLDSETOPTIONS`가 정의되어 있으면 `PTRACE_SETOPTIONS`를 그 값으로 재정의해서 피해 갈 수 있다.

그룹-정지 알림이 추적자에게는 가지만 진짜 부모에게는 가지 않는다. 2.6.38.6에서 마지막으로 확인.

스레드 그룹 리더가 추적 대상이면서 <tt>[[_exit(2)]]</tt> 호출로 끝나면 (요청 시) `PTRACE_EVENT_EXIT` 정지가 일어나지만 후속 `WIFEXITED` 알림은 다른 스레드가 모두 끝나기 전까지 전달되지 않는다. 위에서 설명한 것처럼 다른 스레드들 중 하나가 <tt>[[execve(2)]]</tt>를 호출하면 스레드 그룹 리더의 죽음이 *절대* 보고되지 않게 된다. exec 한 스레드를 추적자가 추적하고 있지 않다면 추적자는 <tt>[[execve(2)]]</tt>가 일어났다는 것을 절대 모를 것이다. 이를 피하기 위한 방법 하나는 이런 경우에 스레드 그룹 리더를 재시작 하지 말고 `PTRACE_DETACH` 하는 것이다. 2.6.38.6에서 마지막으로 확인.

`SIGKILL` 시그널이 여전히 실제 시그널 죽음 전에 `PTRACE_EVENT_EXIT` 정지를 유발할 수도 있다. 향후에는 바뀔 수도 있다. `SIGKILL`은 ptrace 하에서도 언제나 태스크를 즉시 죽이도록 되어 있다. 3.13에서 마지막으로 확인.

피추적자에게 시그널이 갔지만 추적자가 전달을 억제한 경우에 일부 시스템 호출들이 `EINTR`으로 반환한다. (그건 아주 흔한 동작이다. 디버거가 일반적으로 붙기를 할 때마다 그렇게 해서 가짜 `SIGSTOP`이 새로 등장하지 않게 한다.) 리눅스 3.2.9 현재, 영향을 받는 시스템 호출들: <tt>[[epoll_wait(2)]]</tt>, <tt>[[inotify(7)]]</tt> 파일 디스크립터 `read(2)`. (이 목록은 아마 불완전할 것이다.) 이 버그의 일반적 증상은 조용히 있는 프로세스에 다음 명령으로 붙을 때,

```text
strace -p <process-ID>
```

다음과 같은 일반적이고 예상 가능한 한 줄 출력 대신

```text
restart_syscall(<... resuming interrupted call ...>_
```

```text
select(6, [5], NULL, [5], NULL_
```

('_'는 커서 위치를 나타낸다.) 가령 다음과 같은 여러 줄을 보게 된다.

```text
clock_gettime(CLOCK_MONOTONIC, {15370, 690928118}) = 0
epoll_wait(4,_
```

여기에는 보이지 않지만 <tt>[[strace(1)]]</tt>가 붙기 전에 프로세스는 <tt>[[epoll_wait(2)]]</tt>에서 블록 되어 있었다. 붙기 때문에 <tt>[[epoll_wait(2)]]</tt>이 `EINTR` 오류로 사용자 공간으로 반환했다. 그리고 이 경우에서 프로그램은 `EINTR`에 대해 현재 시간을 확인하고 다시 <tt>[[epoll_wait(2)]]</tt>을 실행하는 것으로 대응했다. (그런 "이유 없는" `EINTR`를 예상하지 못한 프로그램은 <tt>[[strace(1)]]</tt> 붙기에 대해 의도치 않은 방식으로 동작할 수 있다.)

일반적인 규칙과 달리 glibc의 `ptrace()` 래퍼에서 `errno`를 0으로 설정할 수 있다.

## SEE ALSO

<tt>[[gdb(1)]]</tt>, <tt>[[ltrace(1)]]</tt>, <tt>[[strace(1)]]</tt>, <tt>[[clone(2)]]</tt>, <tt>[[execve(2)]]</tt>, <tt>[[fork(2)]]</tt>, <tt>[[gettid(2)]]</tt>, <tt>[[prctl(2)]]</tt>, <tt>[[seccomp(2)]]</tt>, <tt>[[sigaction(2)]]</tt>, <tt>[[tgkill(2)]]</tt>, <tt>[[vfork(2)]]</tt>, <tt>[[waitpid(2)]]</tt>, <tt>[[exec(3)]]</tt>, <tt>[[capabilities(7)]]</tt>, <tt>[[signal(7)]]</tt>

----

2018-04-30
