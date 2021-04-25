## NAME

wait, waitpid, waitid - 프로세스 상태 변경 기다리기

## SYNOPSIS

```c
#include <sys/types.h>
#include <sys/wait.h>

pid_t wait(int *wstatus);
pid_t waitpid(pid_t pid, int *wstatus, int options);

int waitid(idtype_t idtype, id_t id, siginfo_t *infop, int options);
                /* glibc 및 POSIX의 인터페이스. 기반 시스템
                   호출에 대한 정보는 NOTES 참고. */
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`waitid()`:
:   glibc 2.26부터:
    :   `_XOPEN_SOURCE >= 500 || _POSIX_C_SOURCE >= 200809L`
 
    glibc 2.25 및 이전:
    :   `_XOPEN_SOURCE`<br>
        `    || /* glibc 2.12부터: */ _POSIX_C_SOURCE >= 200809L`<br>
        `    || /* glibc <= 2.19: */ _BSD_SOURCE`

## DESCRIPTION

이 시스템 호출들은 모두 호출 프로세스의 자식에서의 상태 변경을 기다리고 상태가 바뀐 자식에 대한 정보를 얻는 데 사용하는 것이다. 자식이 종료하는 것, 자식이 시그널에 의해 중지되는 것, 자식이 시그널에 의해 재개되는 것을 상태 변경으로 본다. 자식이 종료하는 경우에 wait을 수행하면 시스템이 자식과 연관된 자원들을 해제할 수 있게 된다. wait을 수행하지 않으면 종료한 자식이 "좀비" 상태로 남는다. (아래 NOTES 참고.)

자식이 이미 상태를 바꾸었으면 이 호출들이 즉시 반환한다. 그렇지 않으면 자식이 상태를 바꾸거나 시그널 핸들러가 호출을 중단시킬 때까지 (<tt>[[sigaction(2)]]</tt>의 `SA_RESTART` 플래그로 시스템 호출을 자동 재시작하지 않는다고 가정) 블록 한다. 이 페이지 나머지에서는 상태가 바뀌었지만 아직 이 시스템 호출들 중 하나로 기다리지 않은 자식을 *기다릴 수 있다(waitable)*고 표현한다.

### `wait()`과 `waitpid()`

`wait()` 시스템 호출은 자식들 중 하나가 종료할 때까지 호출 스레드의 실행을 중지한다. `wait(&wstatus)` 호출은 다음과 동등하다.

```c
waitpid(-1, &wstatus, 0);
```

`waitpid()` 시스템 호출은 `pid` 인자로 지정한 자식이 상태를 바꿀 때까지 호출 스레드의 실행을 중지한다. 기본적으로 `waitpid()`는 자식 종료만 기다리지만 아래에서 설명하듯 `options` 인자를 통해 동작 방식을 변경할 수 있다.

`pid`의 값은 다음일 수 있다.

`< -1`
:   프로세스 그룹 ID가 `pid`의 절댓값과 같은 아무 자식 프로세스나 기다리기

`-1`
:   아무 자식 프로세스나 기다리기

`0`
:   `waitpid()` 호출 시점에 프로세스 그룹 ID가 호출 프로세스의 프로세스 그룹 ID와 같은 아무 자식 프로세스나 기다리기

`> 0`
:   프로세스 ID가 `pid` 값과 같은 자식 기다리기

`options`의 값은 다음 상수들을 0개 또는 그 이상 OR 한 것이다.

`WNOHANG`
:   종료한 자식이 없으면 즉시 반환한다.

`WUNTRACED`
:   자식이 정지한 (하지만 <tt>[[ptrace(2)]]</tt> 추적 대상은 아닌) 경우에도 반환한다. 정지했으면서 *추적 대상인* 자식의 상태는 이 옵션을 지정하지 않아도 제공된다.

`WCONTINUED` (리눅스 2.6.10부터)
:   정지했던 자식이 `SIGCONT` 전달로 재개된 경우에도 반환한다.

(리눅스 전용 옵션들에 대해선 아래 참고.)

`wstatus`가 NULL이 아니면 `wait()`과 `waitpid()`가 그 `int`에 상태 정보를 저장해 준다. 다음 매크로들로 그 정수를 조사할 수 있다. (`wait()`과 `waitpid()`에서처럼 정수 포인터를 받는 게 아니라 정수 자체를 인자로 받는다!)

`WIFEXITED(wstatus)`
:   자식이 정상적으로, 즉 <tt>[[exit(3)]]</tt>나 <tt>[[_exit(2)]]</tt>를 호출하거나 `main()`에서 반환하여 종료했으면 참을 반환한다.

`WEXITSTATUS(wstatus)`
:   자식의 종료 상태를 반환한다. 이 값은 자식이 <tt>[[exit(3)]]</tt>나 <tt>[[_exit(2)]]</tt> 호출에 지정한 `status` 인자나 `main()`의 return 문에 지정한 인자의 하위 8비트이다. 이 매크로는 `WIFEXITED`가 참을 반환했을 때만 써야 한다.

`WIFSIGNALED(wstatus)`
:   자식 프로세스가 시그널로 종료되었으면 참을 반환한다.

`WTERMSIG(wstatus)`
:   자식 프로세스를 종료시킨 시그널의 번호를 반환한다. 이 매크로는 `WIFSIGNALED`가 참을 반환했을 때만 써야 한다.

`WCOREDUMP(wstatus)`
:   자식이 코어 덤프를 만들었으면 참을 반환한다. (<tt>[[core(5)]]</tt> 참고.) 이 매크로는 `WIFSIGNALED`가 참을 반환했을 때만 써야 한다.

    이 매크로는 POSIX.1-2001에 명세되어 있지 않으며 어떤 유닉스 구현(가령 AIX, SunOS)에서는 사용 가능하지 않다. 그러니 `#ifdef WCOREDUMP ... #endif`로 감싸서 사용하라.

`WIFSTOPPED(wstatus)`
:   자식 프로세스가 시그널 전달로 인해 정지되었으면 참을 반환한다. `WUNTRACED`를 써서 호출했거나 자식을 추적 중인 경우(<tt>[[ptrace(2)]]</tt> 참고)에만 가능하다.

`WSTOPSIG(wstatus)`
:   자식이 정지하게 만든 시그널의 번호를 반환한다. 이 매크로는 `WIFSTOPPED`가 참을 반환했을 때만 써야 한다.

`WIFCONTINUED(wstatus)`
:   (리눅스 2.6.10부터) 자식 프로세스가 `SIGCONT` 전달로 인해 재개되었으면 참을 반환한다.

### `waitid()`

(리눅스 2.6.9부터 사용 가능한) `waitid()` 시스템 호출에서는 어떤 자식 상태 변경을 기다릴지에 대해 더 정밀한 제어가 가능하다.

`idtype` 및 `id` 인자로 다음과 같이 기다릴 자식(들)을 선택한다.

`idtype == P_PID`
:   프로세스 ID가 `id`와 일치하는 자식 기다리기.

`idtype == P_PIDFD` (리눅스 5.4부터)
:   `id`로 지정한 PID 파일 디스크립터가 나타내는 자식을 기다리기. (PID 파일 디스크립터에 대한 추가 설명은 <tt>[[pidfd_open(2)]]</tt>을 보라.)

`idtype == P_PGID`
:   프로세스 그룹 ID가 `id`와 일치하는 아무 자식이나 기다리기. 리눅스 5.4부터, `id`가 0이면 호출 시점에 호출자의 프로세스 그룹과 같은 프로세스 그룹에 있는 아무 자식이나 기다린다.

`idtype == P_ALL`
:   아무 자식이나 기다리기. `id`는 무시.

`options`에서 다음 플래그들을 하나 이상 OR 하여 기다릴 자식 상태 변경 종류를 지정한다.

`WEXITED`
:   종료한 자식을 기다린다.

`WSTOPPED`
:   시그널 전달로 정지된 자식을 기다린다.

`WCONTINUED`
:   (앞서 정지되었다가) `SIGCONT` 전달로 재개된 자식을 기다린다.

`options`에서 추가로 다음 플래그들을 OR 할 수 있다.

`WNOHANG`
:   `waitpid()`에서와 같음.

`WNOWAIT`
:   자식을 기다릴 수 있는 상태로 남겨둔다. 이후에 `wait` 호출을 다시 사용해 자식 상태 정보를 얻어올 수 있다.

성공 반환 시 `waitid()`는 `infop`가 가리키는 `siginfo_t` 구조체의 다음 필드들을 채운다.

`si_pid`
:   자식의 프로세스 ID.

`si_uid`
:   자식의 실제 사용자 ID. (다른 구현들에서는 대부분 이 필드를 설정하지 않는다.)

`si_signo`
:   항상 `SIGCHLD`로 설정.

`si_status`
:   자식이 <tt>[[_exit(2)]]</tt>에 (또는 <tt>[[exit(3)]]</tt>에) 준 종료 상태, 또는 자식을 종료시키거나 정지하거나 재개한 시그널. 이 필드를 어떻게 해석해야 하는지를 `si_code` 필드로 판단할 수 있다.

`si_code`
:   다음 중 하나로 설정: `CLD_EXITED` (자식이 <tt>[[_exit(2)]]</tt>를 호출했음), `CLD_KILLED` (자식이 시그널로 죽었음), `CLD_DUMPED` (자식이 시그널로 죽었고 코어를 덤프 했음), `CLD_STOPPED` (자식이 시그널로 중지됐음), `CLD_TRAPPED` (추적하는 자식이 트랩에 걸렸음), `CLD_CONTINUED` (자식이 `SIGCONT`로 재개됐음).

`options`에 `WNOHANG`을 지정했고 기다릴 수 있는 상태인 자식이 없으면 `waitid()`가 즉시 0을 반환하며 `infop`가 가리키는 `siginfo_t` 구조체의 상태는 구현에 따라 다르다. 이 경우를 자식이 기다릴 수 있는 상태였던 경우와 (이식성 있는 방식으로) 구별하려면 호출 전에 `si_pid` 필드를 0으로 만들고서 호출 반환 후에 이 필드에 0 아닌 값이 있는지 확인하면 된다.

POSIX.1-2008 기술 정오표 1(2013년)에서는 `options`에 `WNOHANG`을 지정했고 기다릴 수 있는 상태인 자식이 없으면 `waitid()`가 그 구조체의 `si_pid` 및 `si_signo` 필드를 0으로 채워야 한다는 요구 사항을 추가한다. 이 요구 사항을 준수하는 리눅스 및 여타 구현들에서는 `waitid()` 호출 전에 `si_pid` 필드를 0으로 만들 필요가 없다. 하지만 모든 구현들이 이 점에서 POSIX.1 명세를 따르지는 않는다.

## RETURN VALUE

`wait()`: 성공 시 종료한 자식의 프로세스 ID를 반환한다. 실패 시 -1을 반환한다.

`waitpid()`: 성공 시 상태가 바뀐 자식의 프로세스 ID를 반환한다. `WNOHANG`을 지정했고 `pid`로 지정한 자식이 하나 이상 존재하지만 그 자식(들)이 아직 상태를 바꾸지 않았으면 0을 반환한다. 실패 시 -1을 반환한다.

`waitid()`: 성공 시, 또는 `WNOHANG`을 지정했고 `id`로 지정한 어느 자식도 아직 상태를 바꾸지 않았으면 0을 반환한다. 실패 시 -1을 반환한다.

실패 시 이 호출들 각각은 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`ECHILD`
:   (`wait()에서`) 호출 프로세스에게 아직 기다리지 않은 자식이 없다.

`ECHILD`
:   (`waitpid()`나 `waitid()`에서) `pid`나(`waitpid()`) `idtype` 및 `id`로(`waitid()`) 지정한 프로세스가 존재하지 않거나 호출 프로세스의 자식이 아니다. (`SIGCHLD`에 대한 동작이 `SIG_IGN`으로 설정돼 있으면 자기 자식에 대해서도 이렇게 될 수 있다. 스레드에 대한 *리눅스 참고 사항* 절도 참고.)

`EINTR`
:   `WNOHANG`을 설정하지 않았으며 차단 안 된 시그널이나 `SIGCHLD`를 잡았다. <tt>[[signal(7)]]</tt> 참고.

`EINVAL`
:   `options` 인자가 유효하지 않다.

## CONFORMING TO

SVr4, 4.3BSD, POSIX.1-2001.

## NOTES

종료되었는데 기다려 주지 않은 자식은 "좀비"가 된다. 커널에서는 좀비 프로세스에 대한 최소한의 정보(PID, 종료 상태, 자원 사용 정보)를 유지하여 이후 부모가 wait을 수행해서 자식에 대한 정보를 얻을 수 있도록 한다. wait을 통해 시스템에서 제거하지 않는 한 좀비는 커널 프로세스 테이블에서 자리를 차지하고 있는다. 그리고 이 테이블이 가득 차면 프로세스를 새로 만드는 게 불가능해진다. 부모 프로세스가 종료되면 "좀비" 자식들이 (있다면) `init(1)`에게 (또는 <tt>[[prctl(2)]]</tt> `PR_SET_CHILD_SUBREAPER` 동작을 통해 지정한 가장 가까운 "서브리퍼" 프로세스에게) 입양된다. 그러면 `init(1)`이 자동으로 wait을 수행하여 좀비를 제거한다.

POSIX.1-2001에서는 `SIGCHLD` 처리 방식을 `SIG_IGN`로 설정하거나 `SIGCHLD`에 `SA_NOCLDWAIT` 플래그를 설정한 경우에 (<tt>[[sigaction(2)]]</tt> 참고) 종료한 자식이 좀비가 되지 않으며 `wait()`이나 `waitpid()` 호출이 모든 자식이 종료할 때까지 블록 되었다가 `errno`를 `ECHILD`로 해서 실패하게 된다고 명세한다. (원래 POSIX 표준에서는 `SIGCHLD`를 `SIG_IGN`으로 설정했을 때의 동작 방식을 명세하지 않고 남겨두었다. 참고로 `SIGCHLD`의 기본 처리 방식이 "무시"이지만 처리 방식을 명시적으로 `SIG_IGN`으로 설정하면 좀비 프로세스 자식 처리 방식이 달라지는 것이다.)

리눅스 2.6은 POSIX 요구를 준수한다. 하지만 리눅스 2.4(및 이전)에서는 그렇지 않아서 `SIGCHLD`가 무시되는 동안 `wait()`이나 `waitpid()` 호출을 하면 `SIGCHLD`가 무시되고 있지 않은 것처럼 동작한다. 즉, 다음 자식 종료 때까지 호출이 블록 되어 있다가 그 자식의 프로세스 ID와 상태를 반환한다.

### 리눅스 참고 사항

리눅스 커널에서 커널 스케줄 스레드는 프로세스와 뚜렷하게 구별되는 요소가 아니다. 스레드는 그저 리눅스 고유의 <tt>[[clone(2)]]</tt> 시스템 호출로 만들어진 프로세스일 뿐이다. 그리고 이식성 있는 <tt>[[pthread_create(3)]]</tt> 같은 다른 루틴들이 <tt>[[clone(2)]]</tt>으로 구현되어 있다. 리눅스 2.4 전에서 스레드는 프로세스의 특수한 경우일 뿐이었고, 그래서 같은 스레드 그룹에 속할 때에도 한 스레드가 다른 스레드의 자식을 기다릴 수 없었다. 하지만 POSIX에서 그 기능을 규정하면서 리눅스 2.4부터는 스레드가 같은 스레드 그룹 내의 다른 스레드의 자식을 기다릴 수 있으며, 또 기본적으로 그렇게 한다.

다음 리눅스 전용 `options` 값들은 <tt>[[clone(2)]]</tt>으로 만든 자식들에 쓰기 위한 것이다. 리눅스 4.7부터 `waitid()`에도 쓸 수 있다.

`__WCLONE`
:   "클론" 자식만 기다린다. 없으면 "비클론" 자식만 기다린다. ("클론" 자식은 종료 시 자기 부모에게 시그널을 전달하지 않거나 `SIGCHLD` 아닌 시그널을 전달하는 자식이다.) `__WALL`도 지정한 경우 이 옵션은 무시한다.

`__WALL` (리눅스 2.4부터)
:   종류("클론", "비클론")와 상관없이 모든 자식을 기다린다.

`__WNOTHREAD` (리눅스 2.4부터)
:   같은 스레드 그룹의 다른 스레드의 자식을 기다리지 않는다. 리눅스 2.4 전에서는 이게 기본이었다.

리눅스 4.7부터는 자식을 ptrace 하고 있으면 자동으로 `__WALL` 플래그를 함의한다.

### C 라이브러리/커널 차이

`wait()`이 실제로는 (glibc에서) <tt>[[wait4(2)]]</tt> 호출로 구현된 라이브러리 함수이다.

어떤 아키텍처에는 `waitpid()` 시스템 호출이 없다. 대신 <tt>[[wait4(2)]]</tt>를 호출하는 C 라이브러리 래퍼 함수를 통해 이 인터페이스를 구현한다.

진짜 `waitid()` 시스템 호출은 `struct rusage *` 타입의 다섯 번째 인자를 받는다. 이 인자가 NULL이 아니면 <tt>[[wait4(2)]]</tt>와 같은 식으로 이를 이용해 자식에 대한 자원 사용 정보를 반환한다. 자세한 내용은 <tt>[[getrusage(2)]]</tt>를 보라.

## BUGS

POSIX.1-2008에 따르면 `waitid()`를 호출하는 응용에서 `infop`가 `siginfo_t` 구조체를 가리키도록 (즉 널 포인터가 아니도록) 보장해야 한다. 리눅스에서는 `infop`가 NULL이어도 `waitid()`가 성공하여 기다린 자식의 프로세스 ID를 반환한다. 응용에서는 일관성 없고 비표준이며 불필요한 이 동작 방식에 의지하는 것을 피해야 한다.

## EXAMPLES

다음 프로그램은 <tt>[[fork(2)]]</tt>와 `waitpid()` 사용 방식을 보여 준다. 프로그램에서 자식 프로세스를 만든다. 프로그램에 명령행 인자를 주지 않으면 자식이 <tt>[[pause(2)]]</tt>를 이용해 실행을 멈추는데, 그때 사용자가 자식에게 시그널을 보낼 수 있다. 명령행 인자가 있으면 명령행에서 준 정수를 종료 상태로 사용해서 자식이 즉시 종료한다. 부모 프로세스는 루프를 돌면서 `waitpid()`로 자식을 감시하며, 위에서 설명한 `W*()` 매크로들을 이용해 종료 상태 값을 분석한다.

다음 셸 세션이 프로그램 사용 방식을 보여 준다.

```text
$ ./a.out &
Child PID is 32360
[1] 32359
$ kill -STOP 32360
stopped by signal 19
$ kill -CONT 32360
continued
$ kill -TERM 32360
killed by signal 15
[1]+  Done                    ./a.out
$
```

### 프로그램 소스

```c
#include <sys/wait.h>
#include <stdint.h>
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>

int
main(int argc, char *argv[])
{
    pid_t cpid, w;
    int wstatus;

    cpid = fork();
    if (cpid == -1) {
        perror("fork");
        exit(EXIT_FAILURE);
    }

    if (cpid == 0) {            /* 자식이 실행하는 코드 */
        printf("Child PID is %jd\n", (intmax_t) getpid());
        if (argc == 1)
            pause();                    /* 시그널 기다리기 */
        _exit(atoi(argv[1]));

    } else {                    /* 부모가 실행하는 코드 */
        do {
            w = waitpid(cpid, &wstatus, WUNTRACED | WCONTINUED);
            if (w == -1) {
                perror("waitpid");
                exit(EXIT_FAILURE);
            }

            if (WIFEXITED(wstatus)) {
                printf("exited, status=%d\n", WEXITSTATUS(wstatus));
            } else if (WIFSIGNALED(wstatus)) {
                printf("killed by signal %d\n", WTERMSIG(wstatus));
            } else if (WIFSTOPPED(wstatus)) {
                printf("stopped by signal %d\n", WSTOPSIG(wstatus));
            } else if (WIFCONTINUED(wstatus)) {
                printf("continued\n");
            }
        } while (!WIFEXITED(wstatus) && !WIFSIGNALED(wstatus));
        exit(EXIT_SUCCESS);
    }
}
```

## SEE ALSO

<tt>[[_exit(2)]]</tt>, <tt>[[clone(2)]]</tt>, <tt>[[fork(2)]]</tt>, <tt>[[ptrace(2)]]</tt>, <tt>[[sigaction(2)]]</tt>, <tt>[[signal(2)]]</tt>, <tt>[[wait4(2)]]</tt>, <tt>[[pthread_create(3)]]</tt>, <tt>[[credentials(7)]]</tt>, <tt>[[signal(7)]]</tt>

----

2021-03-22
