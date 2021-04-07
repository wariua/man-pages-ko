## NAME

seccomp - 프로세스의 안전 컴퓨팅 상태 조작하기

## SYNOPSIS

```c
#include <linux/seccomp.h>
#include <linux/filter.h>
#include <linux/audit.h>
#include <linux/signal.h>
#include <sys/ptrace.h>

int seccomp(unsigned int operation, unsigned int flags, void *args);
```

## DESCRIPTION

`seccomp()` 시스템 호출은 호출 프로세스의 안전 컴퓨팅(Secure Computing; seccomp) 상태를 조작한다.

현재 리눅스는 다음 `operation` 값들을 지원한다.

<dl>
<dt><code>SECCOMP_SET_MODE_STRICT</code></dt>
<dd>

호출 스레드에게 허용되는 시스템 호출이 <code>read(2)</code>, <code>write(2)</code>, <tt>[[_exit(2)]]</tt> (<tt>[[exit_group(2)]]</tt>은 안 됨), <tt>[[sigreturn(2)]]</tt>뿐이다. 다른 시스템 호출은 <code>SIGKILL</code> 시그널 전달을 일으킨다. 엄격한 안전 컴퓨팅 모드는 파이프나 소켓 등을 읽어서 얻은 비신뢰 바이트 코드를 실행해야 하는 계산 위주 응용에 유용하다.

참고로 호출 스레드에서 더 이상 <tt>[[sigprocmask(2)]]</tt>를 호출할 수 없기는 하지만 <tt>[[sigreturn(2)]]</tt>을 이용해 <code>SIGKILL</code>과 <code>SIGSTOP</code>을 제외한 모든 시그널들을 차단할 수 있다. 따라서 (예를 들어) 프로세스의 실행 시간을 제약하는 데 <tt>[[alarm(2)]]</tt>으로는 충분치 않다. 확실하게 프로세스를 끝내려면 대신 <code>SIGKILL</code>을 사용해야 한다. <tt>[[timer_create(2)]]</tt>을 <code>SIGEV_SIGNAL</code>로 하고 <code>sigev_signo</code>를 <code>SIGKILL</code>로 설정해서 사용하거나 <tt>[[setrlimit(2)]]</tt>를 이용해 <code>RLIMIT_CPU</code>에 경성 제한을 설정하면 된다.

이 동작은 커널 구성에 <code>CONFIG_SECCOMP</code>가 켜져 있는 경우에만 사용 가능하다.

<code>flags</code>의 값이 0이어야 하고 <code>args</code>가 NULL이어야 한다.

이 동작은 다음 호출과 기능적으로 동일하다.

```c
prctl(PR_SET_SECCOMP, SECCOMP_MODE_STRICT);
```
</dd>

<dt><code>SECCOMP_SET_MODE_FILTER</code></dt>
<dd>

<code>args</code>를 통해 전달하는 버클리 패킷 필터(BPF) 포인터가 허용 시스템 호출을 규정한다. 이 인자는 <code>struct sock_fprog</code>에 대한 포인터이다. 임의의 시스템 호출 및 시스템 호출 인자를 걸러내도록 설계할 수 있다. 필터가 유효하지 않으면 <code>seccomp()</code>가 실패하며 <code>errno</code>로 <code>EINVAL</code>을 반환한다.

필터에서 <tt>[[fork(2)]]</tt>나 <tt>[[clone(2)]]</tt>을 허용하는 경우 자식 프로세스들은 부모와 같은 시스템 호출 필터의 제약을 받게 된다. <tt>[[execve(2)]]</tt>가 허용되는 경우 <tt>[[execve(2)]]</tt> 호출을 거치면서 기존 필터가 보존된다.

<code>SECCOMP_SET_MODE_FILTER</code> 동작을 사용하기 위해선 호출 스레드가 자기 네임스페이스에서 <code>CAP_SYS_ADMIN</code> 역능을 가지고 있어야 한다. 아니면 스레드에 이미 <code>no_new_privs</code> 비트가 설정되어 있어야 하는데, 스레드 선조가 그 비트를 이미 설정하지 않았다면 스레드에서 다음 호출을 해야 한다.

```c
prctl(PR_SET_NO_NEW_PRIVS, 1);
```

그렇지 않으면 <code>SECCOMP_SET_MODE_FILTER</code> 동작이 실패하며 <code>errno</code>로 <code>EACCES</code>를 반환한다. 이 요구 사항은 비특권 프로세스가 악의적 필터를 적용한 후 <tt>[[execve(2)]]</tt>를 이용해 set-user-ID 내지 기타 특권 프로그램을 호출해서 그 프로그램을 탈취할 가능성을 막는다. (예를 들어 <tt>[[setuid(2)]]</tt>로 호출자의 사용자 ID를 0 아닌 값으로 설정하려는 시도를 악의적 필터가 실제 시스템 호출 실행 없이 0을 반환하게 만들 수 있을 것이다. 그래서 어떤 환경에서 프로그램을 속여서 수퍼유저 특권을 유지하게 하고 위험한 동작을 유도하는 것이 가능할 수도 있다.)

붙인 필터에서 <tt>[[prctl(2)]]</tt>이나 <code>seccomp()</code>를 허용하는 경우 필터를 더 추가할 수도 있다. 평가 시간이 늘어나겠지만 이를 통해 스레드 실행 중에 공격 면적을 더 줄일 수 있다.

<code>SECCOMP_SET_MODE_FILTER</code> 동작은 커널 구성에 <code>CONFIG_SECCOMP_FILTER</code>가 켜져 있는 경우에만 사용 가능하다.

<code>flags</code>가 0이면 이 동작은 다음 호출과 기능적으로 동일하다.

```c
prctl(PR_SET_SECCOMP, SECCOMP_MODE_FILTER, args);
```

인식하는 <code>flags</code>는 다음과 같다.

 <dl>
 <dt><code>SECCOMP_FILTER_FLAG_TSYNC</code></dt>
 <dd>

새 필터를 추가할 때 호출 프로세스의 다른 스레드 모두를 같은 seccomp 필터 트리로 동기화시킨다. "필터 트리"란 스레드에 붙인 필터들의 순서 있는 목록이다. (동일 필터를 별개의 <code>seccomp()</code> 호출로 붙인 결과는 이 관점에서 서로 다른 필터이다.)

한 스레드라도 같은 필터 트리로 동기화할 수 없으면 호출이 새 seccomp 필터를 붙이지 않고 실패하며 동기화할 수 없는 첫 번째 스레드의 ID를 반환한다. 동일 프로세스의 다른 스레드가 <code>SECCOMP_MODE_STRICT</code> 모드이거나 자체적으로 새 seccomp 필터를 붙여서 호출 스레드의 필터 트리에서 벗어나 있는 경우에 동기화가 실패하게 된다.
 </dd>

 <dt><code>SECCOMP_FILTER_FLAG_LOG</code> (리눅스 4.14부터)</dt>
 <dd>
<code>SECCOMP_RET_ALLOW</code>를 제외한 모든 필터 반환 행위들을 기록해야 한다. 관리자가 <code>/proc/sys/kernel/seccomp/actions_logged</code> 파일을 통해 개별 행위의 로그 기록을 막아서 이 플래그를 무시하게 할 수도 있다.
 </dd>

 <dt><code>SECCOMP_GET_ACTION_AVAIL</code> (리눅스 4.17부터)</dt>
 <dd>
추측성 저장 우회(Speculative Store Bypass) 완화를 끈다.
 </dd>
 </dl>
</dd>

<dt><code>SECCOMP_GET_ACTION_AVAIL</code> (리눅스 4.14부터)</dt>
<dd>

어떤 행위를 커널이 지원하는지 검사한다. 최근에 추가된 필터 반환 행위를 커널이 알고 있는지 확인하는 데 도움이 된다. 커널에서는 알지 못하는 행위를 모두 <code>SECCOMP_RET_KILL_PROCESS</code>로 처리한다.

<code>flags</code> 값이 0이어야 하고 <code>args</code>가 부호 없는 32비트 필터 반환 행위에 대한 포인터여야 한다.
</dd>
</dl>

### 필터

`SECCOMP_SET_MODE_FILTER`를 통해 필터를 추가할 때 `args`가 필터 프로그램을 가리킨다.

```c
struct sock_fprog {
    unsigned short     len;     /* BPF 인스트럭션 개수 */
    struct sock_filter *filter; /* BPF 인스트럭션들의
                                   배열에 대한 포인터 */
};
```

각 프로그램은 한 개 이상의 BPF 인스트럭션을 담고 있어야 한다.

```c
struct sock_filter {            /* 필터 블록 */
    __u16 code;                 /* 실제 필터 코드 */
    __u8  jt;                   /* 참 점프 */
    __u8  jf;                   /* 거짓 점프 */
    __u32 k;                    /* 범용 다용도 필드 */
};
```

그 인스트럭션들을 실행할 때 BPF 프로그램은 다음 형태의 (읽기 전용) 버퍼로 준비된 시스템 호출 정보에 대해 (`BPF_ABS` 주소 지정 모드를 사용해) 동작한다.

```c
struct seccomp_data {
    int   nr;                   /* 시스템 호출 번호 */
    __u32 arch;                 /* AUDIT_ARCH_* 값
                                   (<linux/audit.h> 참고) */
    __u64 instruction_pointer;  /* CPU 인스트럭션 포인터 */
    __u64 args[6];              /* 6개까지의 시스템 호출 인자 */
};
```

아키텍처에 따라 시스템 호출 번호가 다르고 일부 아키텍처(가령 x86-64)에서는 사용자 코드가 여러 아키텍처의 호출 규약을 사용할 수 있기 때문에 (그리고 프로세스에서 <tt>[[execve(2)]]</tt>를 이용해 다른 규약을 쓰는 바이너리를 실행하는 경우 도중에 사용 규약이 달라질 수도 있기 때문에) 일반적으로 `arch` 필드의 값을 검사할 필요가 있다.

가능하면 화이트리스트 방식을 사용하기를 강력히 권한다. 그 방식이 더 견고하고 단순하기 때문이다. 블랙리스트는 위험할 수 있는 시스템 호출이 (또는 위험한 플래그나 옵션이) 추가될 때마다 갱신해 주어야 하며, 어떤 값의 표현 방식을 의미 변경 없이 바꿔서 블랙리스트 우회로 이어질 수 있는 경우가 많다. 아래의 *경고* 절도 참고하라.

`arch` 필드가 호출 규약 모두에서 유일하지 않다. x86-64 ABI와 x32 ABI는 모두 `arch` 필드에 `AUDIT_ARCH_X86_64`를 사용하며 같은 프로세서 상에서 동작한다. 대신 시스템 호출 번호에 `__X32_SYSCALL_BIT`를 사용해 두 ABI를 구별한다.

즉, x86-64 ABI에 걸쳐 동작하는 seccomp 기반 시스템 호출 블랙리스트를 만들기 위해선 `arch`가 `AUDIT_ARCH_X86_64`와 같은지 확인해야 할 뿐 아니라 `nr`에 `__X32_SYSCALL_BIT`를 담은 시스템 호출을 모두 명시적으로 거부하기도 해야 한다.

`instruction_pointer` 필드는 시스템 호출을 수행한 기계어 인스트럭션의 주소를 알려 준다. `/proc/[pid]/maps`와 함께 사용해서 프로그램의 어떤 영역(매핑)에서 시스템 호출을 수행했는지 검사하는 데 쓸모가 있을 수도 있다. (프로그램들이 그런 검사를 무력화하는 걸 막으려면 아마 <tt>[[mmap(2)]]</tt> 및 <tt>[[mprotect(2)]]</tt> 시스템 호출을 봉쇄하는 게 좋을 것이다.)

`args`의 값들을 블랙리스트로 검사할 때는 인자들이 처리되기 전에, 하지만 seccomp 검사 후에 조용히 잘려나가는 경우가 많다는 것을 염두에 두어야 한다. 예를 들면 x86-64 커널에서 i386 ABI를 사용할 때 그런 경우가 발생한다. 커널은 보통 인자의 하위 32비트 위를 보지 않을 테지만 seccomp 데이터에는 전체 64비트 레지스터 값이 주어진다. 별로 놀랍지 않을 또 다른 예는 x86-64 ABI를 이용해 `int` 타입 인자를 받는 시스템 호출을 수행할 때이다. 인자 레지스터의 상위 절반이 시스템 호출에서는 무시되지만 seccomp 데이터에게는 보인다.

seccomp 필터는 두 부분으로 이뤄진 32비트 값을 반환한다. (상수 `SECCOMP_RET_ACTION_FULL`이 규정하는 마스크에 대응하는) 상위 16비트는 아래 나열된 "행위" 값들 중 하나를 담는다. (상수 `SECCOMP_RET_DATA`가 규정하는) 하위 16비트는 그 반환 값과 관련된 "데이터"이다.

여러 필터가 존재하는 경우 *모두* 실행하며 필터 트리에 추가한 순서 반대로 실행한다. 즉, 가장 최근 설치된 필터가 가장 먼저 실행된다. (참고로 이전 필터들 중 하나가 `SECCOMP_RET_KILL`을 반환했다 해도 모든 필터를 호출한다. 이렇게 하는 건 커널 코드를 단순하게 하고 흔치 않은 경우에 대한 검사를 피해서 필터 세트 실행 속도를 살짝 높이기 위해서이다.) 어떤 시스템 호출을 평가한 반환 값은 필터 전체 실행에서 반환된 것들 중 가장 높은 우선도의 가장 먼저 나온 행위 값(과 수반 데이터)이다.

seccomp 필터가 반환할 수 있는 행위 값들을 우선도 역순으로 나열하면 다음과 같다.

<dl>
<dt><code>SECCOMP_RET_KILL_PROCESS</code> (리눅스 4.14부터)</dt>
<dd>

이 값은 프로세스가 코어 덤프와 함께 즉시 종료되게 한다. 시스템 호출은 실행되지 않는다. 아래의 <code>SECCOMP_RET_KILL_THREAD</code>와 달리 스레드 그룹의 모든 스레드가 종료된다. (스레드 그룹에 대한 논의는 <tt>[[clone(2)]]</tt>의 <code>CLONE_THREAD</code> 플래그 설명을 보라.)

<code>SIGSYS</code> 시그널에 의해 죽은 <em>것처럼</em> 프로세스를 종료시킨다. <code>SIGSYS</code>에 시그널 핸들러를 등록해 두어도 이 경우에는 그 핸들러를 무시하고 항상 프로세스를 종료시킨다. 이 프로세스를 (<tt>[[waitpid(2)]]</tt> 등으로) 기다리고 있는 부모 프로세스에게 반환되는 <code>wstatus</code>는 자식이 <code>SIGSYS</code> 시그널로 종료된 것처럼 표시된다.
</dd>

<dt><code>SECCOMP_RET_KILL_THREAD</code> (또는 <code>SECCOMP_RET_KILL</code>)</dt>
<dd>

이 값은 시스템 호출을 한 스레드가 즉시 종료되게 한다. 시스템 호출은 실행되지 않는다. 같은 스레드 그룹의 다른 스레드들은 실행을 계속한다.

<code>SIGSYS</code> 시그널에 의해 죽은 <em>것처럼</em> 스레드를 종료시킨다. 위의 <code>SECCOMP_RET_KILL_PROCESS</code> 참고.

리눅스 4.11 전에는 이 방식으로 종료되는 프로세스가 코어 덤프를 유발하지 않았다. (<tt>[[signal(7)]]</tt>에는 <code>SIGSYS</code>의 기본 행위가 코어 덤프 하는 종료라고 적혀 있다.) 리눅스 4.11부터는 단일 스레드 프로세스가 이 방식으로 종료되면 코어를 덤프 한다.

리눅스 4.14에서 <code>SECCOMP_RET_KILL_PROCESS</code>가 추가되면서 두 행위를 명확히 구별할 수 있도록 <code>SECCOMP_RET_KILL</code>의 동의어로 <code>SECCOMP_RET_KILL_THREAD</code>가 추가되었다.
</dd>

<dt><code>SECCOMP_RET_TRAP</code></dt>
<dd>

이 값은 커널이 유발 프로세스에게 스레드 지향 <code>SIGSYS</code> 시그널을 보내게 한다. (시스템 호출은 실행되지 않는다.) <code>siginfo_t</code> 구조체(<tt>[[sigaction(2)]]</tt> 참고)의 시그널과 관련된 여러 필드들이 설정된다.

* <code>si_signo</code>가 <code>SIGSYS</code>를 담는다.

* <code>si_call_addr</code>이 시스템 호출 인스트럭션의 주소를 보여 준다.

* <code>si_syscall</code>과 <code>si_arch</code>가 어떤 시스템 호출이 시도됐는지 나타낸다.

* <code>si_code</code>가 <code>SYS_SECCOMP</code>를 담는다.

* <code>si_errno</code>가 필터 반환 값의 <code>SECCOMP_RET_DATA</code> 부분을 담는다.

프로그램 카운터는 시스템 호출이 이뤄진 것처럼 되어 있을 것이다. (즉, 프로그램 카운터가 시스템 호출 인스트럭션을 가리키지 않는다.) 반환 값 레지스터는 아키텍처별로 다른 값을 담는데, 실행 재개 시 그 시스템 호출에 적절한 어떤 값으로 설정한다. (이 아키텍처 의존성은 <code>ENOSYS</code>로 바꿔 버리면 어떤 유용한 정보를 덮어 쓸 수도 있을 것이기 때문이다.)
</dd>

<dt><code>SECCOMP_RET_ERRNO</code></dt>
<dd>
이 값은 시스템 호출을 실행하지 않고 필터 반환 값의 <code>SECCOMP_RET_DATA</code> 부분이 <code>errno</code> 값으로 사용자 공간으로 전달되게 한다.
</dd>

<dt><code>SECCOMP_RET_TRACE</code></dt>
<dd>

반환 시 이 값은 시스템 호출 실행 전에 커널이 <tt>[[ptrace(2)]]</tt> 기반 추적자에게 알림을 시도하게 한다. 추적자가 없으면 시스템 호출을 실행하지 않고 <code>errno</code>를 <code>ENOSYS</code>로 설정해서 실패 상태를 반환한다.

추적자가 <code>ptrace(PTRACE_SETOPTIONS)</code>를 이용해 <code>PTRACE_O_TRACESECCOMP</code>를 요청하면 알림을 받게 된다. <code>PTRACE_EVENT_SECCOMP</code>로 알림을 받게 되며 추적자에서 <code>PTRACE_GETEVENTMSG</code>로 필터 반환 값을 사용할 수 있다.

추적자에서 시스템 호출 번호를 -1로 바꿔서 그 시스템 호출을 건너뛸 수 있다. 또는 추적자에서 시스템 호출을 유효한 시스템 호출 번호로 바꿔서 요청된 시스템 호출을 바꿀 수 있다. 시스템 호출을 건너뛰는 경우 추적자가 반환 값 레지스터에 넣은 값을 그 시스템 호출이 반환하는 것처럼 보이게 된다.

커널 4.8 전에서, 추적자에게 알림을 준 후에는 seccomp 검사를 다시 실행하지 않는다. (따라서 이전 커널에서 seccomp 기반 샌드박스에서는 극히 주의하지 않는 한 <tt>[[ptrace(2)]]</tt> 사용을, 설령 다른 샌드박스 된 프로세스에서라 해도, <strong>절대</strong> 허용해서는 안 된다. ptrace 사용 프로세스가 이 메커니즘을 이용해 seccomp 샌드박스에서 탈출할 수 있다.)
</dd>

<dt><code>SECCOMP_RET_LOG</code> (리눅스 4.14부터)</dt>
<dd>
이 값은 필터 반환 행위를 로그로 기록한 후 시스템 호출이 실행되게 한다. 관리자가 <code>/proc/sys/kernel/seccomp/actions_logged</code> 파일을 통해 이 행위의 기록 동작을 무시하게 할 수도 있다.
</dd>

<dt><code>SECCOMP_RET_ALLOW</code></dt>
<dd>
이 값은 시스템 호출이 실행되게 한다.
</dd>
</dl>

위와 다른 행위 값을 지정하는 경우에는 필터 행위를 `SECCOMP_RET_KILL_PROCESS`(리눅스 4.14부터)나 `SECCOMP_RET_KILL_THREAD`(리눅스 4.13까지)로 처리한다.

### `/proc` 인터페이스

`/proc/sys/kernel/seccomp` 디렉터리 내의 파일들이 seccomp 정보 및 설정 인터페이스를 추가로 제공한다.

<dl>
<dt><code>actions_avail</code> (리눅스 4.14부터)</dt>
<dd>
seccomp 필터 반환 행위들의 문자열 형태로 된 읽기 전용 순서 있는 목록이다. 왼쪽에서 오른쪽으로의 순서가 우선도가 내려가는 순서이다. 커널이 지원하는 seccomp 필터 반환 행위들의 집합을 나타낸다.
</dd>

<dt><code>actions_logged</code> (리눅스 4.14부터)</dt>
<dd>

로그 기록이 허용되는 seccomp 필터 반환 행위들의 읽기-쓰기 순서 있는 목록이다. 파일에 쓸 때 순서를 지킬 필요가 없으며, 그래도 파일을 읽을 때는 <code>actions_avail</code> 파일과 같은 순서가 된다.

어떤 태스크를 감사(audit)하도록 감사 서브시스템이 구성되어 있을 때 <code>actions_logged</code> 값이 특정 필터 반환 행위들이 기록되는 것을 막지 않는다는 점에 유의해야 한다. <code>actions_logged</code> 파일에 행위가 없는 경우 그 태스크에 대한 행위를 감사할지 여부에 대한 최종 결정은 감사 서브시스템이 <code>SECCOMP_RET_ALLOW</code> 외 모든 필터 반환 행위들에 대해 어떻게 할지 내리는 판단에 달려 있다.

<code>actions_logged</code> 파일에서 문자열 "allow"는 받아들이지 않는다. <code>SECCOMP_RET_ALLOW</code> 행위를 기록하는 것은 불가능하기 때문이다. 파일에 "allow"를 쓰려고 하면 <code>EINVAL</code> 오류로 실패한다.
</dd>
</dl>

### seccomp 행위 감사 기록

리눅스 4.14부터 커널은 seccomp 필터가 반환하는 행위를 감사 로그에 기록하는 장치를 제공한다. 커널은 행위의 종류, `actions_logged` 파일에 그 행위가 있는지 여부, 커널 감사가 (가령 커널 부트 옵션 `audit=1`을 통해) 켜져 있는지 여부에 따라서 행위를 기록할지를 판단한다. 규칙은 다음과 같다.

* 행위가 `SECCOMP_RET_ALLOW`이면 그 행위를 기록하지 않는다.

* 그 외 경우에, 행위가 `SECCOMP_RET_KILL_PROCESS`나 `SECCOMP_RET_KILL_THREAD`이며 행위가 `actions_logged` 파일에 등장하면 그 행위를 기록한다.

* 그 외 경우에, 필터에서 기록을 요청했으며 (`SECCOMP_FILTER_FLAG_LOG` 플래그) 행위가 `actions_logged` 파일에 등장하면 그 행위를 기록한다.

* 그 외 경우에, 커널 감사 기능이 켜져 있으며 프로세스를 감사하는 중이면 (`autrace(8)`) 그 행위를 기록한다.

* 그 외 경우에는 그 행위를 기록하지 않는다.

## RETURN VALUE

성공 시 `seccomp()`는 0을 반환한다. 오류 시 `SECCOMP_FILTER_FLAG_TSYNC`를 사용했으면 반환 값은 동기화 실패를 유발한 스레드의 ID이다. (이 ID는 <tt>[[clone(2)]]</tt>이나 <tt>[[gettid(2)]]</tt>가 반환하는 종류의 커널 스레드 ID이다.) 다른 오류 시 -1을 반환하며 오류 원인을 나타내도록 `errno`를 설정한다.

## ERRORS

`seccomp()`가 다음 이유로 실패할 수 있다.

<dl>
<dt><code>EACCES</code></dt>
<dd>호출자가 자기 사용자 네임스페이스에서 <code>CAP_SYS_ADMIN</code> 역능을 가지고 있지 않으며 <code>SECCOMP_SET_MODE_FILTER</code> 사용 전에 <code>no_new_privs</code>를 설정하지 않았다.</dd>
<dt><code>EFAULT</code></dt>
<dd><code>args</code>가 유효한 주소가 아니다.</dd>
<dt><code>EINVAL</code></dt>
<dd>알 수 없는 <code>operation</code>이거나 현재 커널 버전 내지 구성에서 지원하지 않는다.</dd>
<dt><code>EINVAL</code></dt>
<dd>지정한 <code>flags</code>가 해당 <code>operation</code>에서 유효하지 않다.</dd>
<dt><code>EINVAL</code></dt>
<dd><code>operation</code>이 <code>BPF_ABS</code>를 포함하는데 지정한 오프셋이 32비트 경계에 정렬되어 있지 않거나 <code>sizeof(struct seccomp_data)</code>를 초과한다.</dd>
<dt><code>EINVAL</code></dt>
<dd>안전 컴퓨팅 모드를 이미 설정했으며 <code>operation</code>이 기존 설정과 다르다.</dd>
<dt><code>EINVAL</code></dt>
<dd><code>operation</code>이 <code>SECCOMP_SET_MODE_FILTER</code>인데 <code>args</code>가 가리키는 필터가 유효하지 않거나 필터 프로그램 길이가 0이거나 <code>BPF_MAXINSNS</code>(4096)개 인스트럭션을 초과한다.</dd>
<dt><code>ENOMEM</code></dt>
<dd>메모리 부족.</dd>
<dt><code>ENOMEM</code></dt>
<dd>호출 스레드에 붙인 필터 프로그램들의 총 길이가 <code>MAX_INSNS_PER_PATH</code>(32768)개 인스트럭션을 넘게 된다. 참고로 이 제한을 계산할 때 기존 필터 프로그램 각각에는 4개 인스트럭션씩 오버헤드를 더한다.</dd>
<dt><code>EOPNOTSUPP</code></dt>
<dd><code>operation</code>이 <code>SECCOMP_GET_ACTION_AVAIL</code>인데 <code>args</code>로 지정한 필터 반환 행위를 커널이 지원하지 않는다.</dd>
<dt><code>ESRCH</code></dt>
<dd>스레드 동기화 중에 다른 스레드 때문에 실패했는데 그 ID를 알아낼 수 없다.</dd>
</dl>

## VERSIONS

리눅스 3.17에서 `seccomp()` 시스템 호출이 처음 등장했다.

## CONFORMING TO

`seccomp()` 시스템 호출은 비표준 리눅스 확장이다.

## NOTES

아래 예처럼 seccomp 필터를 직접 코딩 하는 대신 `libseccomp` 라이브러리를 이용할 수도 있다. seccomp 필터 생성을 위한 프론트엔드를 제공해 준다.

`/proc/[pid]/status` 파일의 `Seccomp` 필드를 통해 프로세스의 seccomp 모드를 볼 수 있다. <tt>[[proc(5)]]</tt> 참고.

`seccomp()`는 <tt>[[prctl(2)]]</tt> `PR_SET_SECCOMP` 동작이 제공하는 기능(`flags`를 지원하지 않음)의 상위집합을 제공한다.

리눅스 4.4부터 <tt>[[ptrace(2)]]</tt> `PTRACE_SECCOMP_GET_FILTER` 동작을 이용해 프로세스의 seccomp 필터를 얻어올 수 있다.

### seccomp BPF 아키텍처 지원

다음 아키텍처들에서 seccomp BPF 필터링 아키텍처 지원이 사용 가능하다.

* x86-64, i386, x32 (리눅스 3.5부터)
* ARM (리눅스 3.8부터)
* s390 (리눅스 3.8부터)
* MIPS (리눅스 3.16부터)
* ARM-64 (리눅스 3.19부터)
* PowerPC (리눅스 4.3부터)
* Tile (리눅스 4.3부터)
* PA-RISC (리눅스 4.6부터)

### 경고

프로그램에 seccomp 필터를 적용할 때 고려해야 하는 다음과 같은 미묘한 사항들이 있다.

* 몇몇 전통적 시스템 호출은 여러 아키텍처 상에서 <tt>[[vdso(7)]]</tt>에 사용자 공간 구현이 있다. 유명한 예로 <tt>[[clock_gettime(2)]]</tt>, <tt>[[gettimeofday(2)]]</tt>, <tt>[[time(2)]]</tt> 등이 있다. 그런 아키텍처에서 이런 시스템 호출들에는 seccomp 필터의 효과가 없다. (하지만 <tt>[[vdso(7)]]</tt> 구현에서 진짜 시스템 호출로 후퇴할 수 있는 경우가 있어서 그때는 seccomp 필터가 시스템 호출을 보게 된다.)

* seccomp 필터링은 시스템 호출 번호를 기반으로 한다. 하지만 일반적으로 응용에서는 시스템 호출을 직접 부르는 대신 C 라이브러리 래퍼 함수를 호출하고, 그러면 거기서 시스템 호출을 부른다. 따라서 다음을 염두에 두어야 한다.

  * 몇몇 전통적 시스템 호출들의 glibc 래퍼에서 실제로는 커널의 다른 이름의 시스템 호출을 이용할 수 있다. 예를 들어 <tt>[[exit(2)]]</tt> 래퍼 함수가 실제로는 <tt>[[exit_group(2)]]</tt> 시스템 호출을 이용하고 <tt>[[fork(2)]]</tt> 래퍼 함수가 실제로는 <tt>[[clone(2)]]</tt>을 호출한다.

  * 아키텍처에서 제공하는 시스템 호출에 따라 래퍼 함수의 동작 방식이 달라질 수 있다. 다시 말해 같은 래퍼 함수가 다른 아키텍처에서 상이한 시스템 호출을 부를 수도 있다.

  * 마지막으로, glibc 버전에 따라 래퍼 함수의 동작 방식이 달라질 수 있다. 예를 들어 이전 버전에서 <tt>[[open(2)]]</tt>의 glibc 래퍼 함수는 같은 이름의 시스템 호출을 불렀지만 glibc 2.26부터는 모든 아키텍처에서 <tt>[[openat(2)]]</tt>을 호출하는 것으로 구현이 바뀌었다.

위 사항들의 결론은 필터에서 예상과 다른 시스템 호출을 걸러야 할 수도 있다는 것이다. 2부의 여러 매뉴얼 페이지에서 <em>C 라이브러리/커널 차이</em>라는 부절을 통해 래퍼 함수와 기반 시스템 호출의 차이에 대한 유용한 설명을 제공한다.

더불어 응용에서 수행해야 할 법한 적법한 동작에 대해 필터가 예상 외의 실패를 유발하여 seccomp 필터 적용이 응용에 버그를 유발할 위험도 있다. 아주 드물게 쓰이는 응용 코드 경로에서 그런 버그가 발생한다면 seccomp 필터 테스트 때 발견하기 어려울 수도 있다.

### seccomp 관련 BPF 세부 사항

seccomp 필터에 한정된 다음과 같은 BPF 세부 사항이 있다.

* 크기 수식자 `BPF_H`와 `BPF_B`를 지원하지 않는다. 모든 연산은 (4바이트) 워드(`BPF_W`)를 적재하고 저장해야 한다.

* `seccomp_data` 버퍼 내용에 접근하려면 주소 지정 모드 수식자 `BPF_ABS`를 사용하면 된다.

* 주소 지정 모드 수식자 `BPF_LEN`이 즉시 모드 피연산자를 내놓으며 그 값은 `seccomp_data` 버퍼의 크기이다.

## EXAMPLE

아래 프로그램은 4개 이상의 인자를 받는다. 처음 세 인자는 시스템 호출 번호, 숫자로 된 아키텍처 식별자, 오류 번호이다. 프로그램이 그 값들을 이용해 BPF 필터를 만들면 런타임에 다음 검사를 수행한다.

 1. 프로그램이 지정한 아키텍처에서 돌고 있지 않으면 BPF 필터가 `ENOSYS` 오류로 시스템 호출이 실패하게 한다.
 
 2. 프로그램이 지정한 번호의 시스템 호출을 실행하려고 하면 BPF 필터가 시스템 호출이 실패하게 하고 `errno`를 지정한 오류 번호로 설정한다.

나머지 명령행 인자들은 예시 프로그램이 <tt>[[execv(3)]]</tt>(시스템 호출 <tt>[[execve(2)]]</tt>를 사용하는 라이브러리 함수)를 이용해 실행을 시도할 프로그램 경로명과 추가 인자이다. 아래에 몇 가지 프로그램 실행 예가 있다.

먼저 현재 아키텍처(x86-64)를 표시하고 이 아키텍처에서 시스템 호출 번호를 찾는 셸 함수를 만든다.

```
$ uname -m
x86_64
$ syscall_nr() {
    cat /usr/src/linux/arch/x86/syscalls/syscall_64.tbl | \
    awk '$2 != "x32" && $3 == "'$1'" { print $1 }'
}
```

BPF 필터가 시스템 호출을 거부할 때 (위의 2번 경우) 명령행에서 지정한 오류 번호로 시스템 호출이 실패하게 한다. 이 실험에서는 오류 번호 99를 사용할 것이다.

```
$ errno 99
EADDRNOTAVAIL 99 Cannot assign requested address
```

다음 예에서는 `whoami(1)` 명령 실행을 시도한다. 하지만 BPF 필터가 <tt>[[execve(2)]]</tt> 시스템 호출을 거부하므로 명령이 실행조차 되지 않는다.

```
$ syscall_nr execve
59
$ ./a.out
Usage: ./a.out <syscall_nr> <arch> <errno> <prog> [<args>]
Hint for <arch>: AUDIT_ARCH_I386: 0x40000003
                 AUDIT_ARCH_X86_64: 0xC000003E
$ ./a.out 59 0xC000003E 99 /bin/whoami
execv: Cannot assign requested address
```

그 다음에는 BPF 필터가 `write(2)` 시스템 호출을 거부하여 `whoami(1)` 명령이 성공적으로 시작은 하지만 출력을 쓸 수 없게 한다.

```
$ syscall_nr write
1
$ ./a.out 1 0xC000003E 99 /bin/whoami
```

마지막 예에서는 BPF 필터가 `whoami(1)` 명령에서 쓰지 않는 시스템 호출을 거부한다. 명령이 성공적으로 실행되고 출력을 내놓는다.

```
$ syscall_nr preadv
295
$ ./a.out 295 0xC000003E 99 /bin/whoami
cecilia
```

### 프로그램 소스

```c
#include <errno.h>
#include <stddef.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <linux/audit.h>
#include <linux/filter.h>
#include <linux/seccomp.h>
#include <sys/prctl.h>

#define X32_SYSCALL_BIT 0x40000000

static int
install_filter(int syscall_nr, int t_arch, int f_errno)
{
    unsigned int upper_nr_limit = 0xffffffff;

    /* AUDIT_ARCH_X86_64가 일반 x86-64 ABI를 뜻한다고 상정
       (x32 ABI에서는 모든 시스템 호출의 'nr' 필드 30번 비트가
       설정되어 있고, 그래서 숫자 값이 >= X32_SYSCALL_BIT임) */
    if (t_arch == AUDIT_ARCH_X86_64)
        upper_nr_limit = X32_SYSCALL_BIT - 1;

    struct sock_filter filter[] = {
        /* [0] 'seccomp_data' 버퍼에서 누산기로 아키텍처 적재 */
        BPF_STMT(BPF_LD | BPF_W | BPF_ABS,
                 (offsetof(struct seccomp_data, arch))),

        /* [1] 아키텍처가 't_arch'와 일치하지 않으면 5개 인스트럭션
               후로 점프 */
        BPF_JUMP(BPF_JMP | BPF_JEQ | BPF_K, t_arch, 0, 5),

        /* [2] 'seccomp_data' 버퍼에서 누산기로 시스템 호출 번호
               적재 */
        BPF_STMT(BPF_LD | BPF_W | BPF_ABS,
                 (offsetof(struct seccomp_data, nr))),

        /* [3] ABI 확인 - x86-64에서 블랙리스트 방식으로 쓸 때만 필요.
               syscall 번호 재적재를 피하기 위해서 비트 마스크로
               검사하는 대신 BPF_JGT 사용. */
        BPF_JUMP(BPF_JMP | BPF_JGT | BPF_K, upper_nr_limit, 3, 0),

        /* [4] 시스템 호출 번호가 'syscall_nr'과 일치하지 않으면
               1개 인스트럭션 후로 점프 */
        BPF_JUMP(BPF_JMP | BPF_JEQ | BPF_K, syscall_nr, 0, 1),

        /* [5] 아키텍처와 시스템 호출 일치: 시스템 호출을 실행하지
               말고 'errno'로 'f_errno' 반환 */
        BPF_STMT(BPF_RET | BPF_K,
                 SECCOMP_RET_ERRNO | (f_errno & SECCOMP_RET_DATA)),

        /* [6] 시스템 호출 번호 불일치 점프 목적지: 다른 시스템 호출들
               허용 */
        BPF_STMT(BPF_RET | BPF_K, SECCOMP_RET_ALLOW),

        /* [7] 아키텍처 불일치 점프 목적지: 태스크 죽이기 */
        BPF_STMT(BPF_RET | BPF_K, SECCOMP_RET_KILL),
    };

    struct sock_fprog prog = {
        .len = (unsigned short) (sizeof(filter) / sizeof(filter[0])),
        .filter = filter,
    };

    if (seccomp(SECCOMP_SET_MODE_FILTER, 0, &prog)) {
        perror("seccomp");
        return 1;
    }

    return 0;
}

int
main(int argc, char **argv)
{
    if (argc < 5) {
        fprintf(stderr, "Usage: "
                "%s <syscall_nr> <arch> <errno> <prog> [<args>]\n"
                "Hint for <arch>: AUDIT_ARCH_I386: 0x%X\n"
                "                 AUDIT_ARCH_X86_64: 0x%X\n"
                "\n", argv[0], AUDIT_ARCH_I386, AUDIT_ARCH_X86_64);
        exit(EXIT_FAILURE);
    }

    if (prctl(PR_SET_NO_NEW_PRIVS, 1, 0, 0, 0)) {
        perror("prctl");
        exit(EXIT_FAILURE);
    }

    if (install_filter(strtol(argv[1], NULL, 0),
                       strtol(argv[2], NULL, 0),
                       strtol(argv[3], NULL, 0)))
        exit(EXIT_FAILURE);

    execv(argv[4], &argv[4]);
    perror("execv");
    exit(EXIT_FAILURE);
}
```

## SEE ALSO

`bpfc(1)`, <tt>[[strace(1)]]</tt>, <tt>[[bpf(2)]]</tt>, <tt>[[prctl(2)]]</tt>, <tt>[[ptrace(2)]]</tt>, <tt>[[sigaction(2)]]</tt>, <tt>[[proc(5)]]</tt>, <tt>[[signal(7)]]</tt>, <tt>[[socket(7)]]</tt>

`libseccomp` 라이브러리에서 온 여러 페이지들: `scmp_sys_resolver(1)`, <tt>[[seccomp_init(3)]]</tt>, <tt>[[seccomp_load(3)]]</tt>, <tt>[[seccomp_rule_add(3)]]</tt>, <tt>[[seccomp_export_bpf(3)]]</tt>

커널 소스 파일 `Documentation/networking/filter.txt`와 `Documentation/userspace-api/seccomp_filter.rst` (리눅스 4.13 전에선 `Documentation/prctl/seccomp_filter.txt`).

McCanne, S. and Jacobson, V. (1992) <em>The BSD Packet Filter: A New Architecture for User-level Packet Capture</em>, Proceedings of the USENIX Winter 1993 Conference (http://www.tcpdump.org/papers/bpf-usenix93.pdf)

----

2019-03-06
