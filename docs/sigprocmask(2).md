## NAME

sigprocmask, rt_sigprocmask - 블록 된 시그널 조사하고 바꾸기

## SYNOPSIS

```c
#include <signal.h>

/* glibc 래퍼 함수 원형 */
int sigprocmask(int how, const sigset_t *set, sigset_t *oldset);

/* 기반 시스템 호출 원형 */
int rt_sigprocmask(int how, const kernel_sigset_t *set,
                   kernel_sigset_t *oldset, size_t sigsetsize);

/* 구식 시스템 호출 원형 (제거 예정) */
int sigprocmask(int how, const old_kernel_sigset_t *set,
                old_kernel_sigset_t *oldset);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`sigprocmask()`:
:   `_POSIX_C_SOURCE`

## DESCRIPTION

`sigprocmask()`를 사용해 호출 스레드의 시그널 마스크를 가져오고 바꾼다. 시그널 마스크는 현재 호출자에게 전달이 차단된 시그널들의 집합이다. (더 자세한 내용은 <tt>[[signal(7)]]</tt>도 참고.)

다음과 같이 `how` 값에 따라 호출의 동작 방식이 달라진다.

`SIG_BLOCK`
:   현재 집합과 `set` 인자의 교집합이 차단 시그널 집합이 된다.

`SIG_UNBLOCK`
:   현재 차단 시그널 집합에서 `set`의 시그널들을 제거한다. 막혀 있지 않은 시그널을 제거하려는 것도 허용한다.

`SIG_SETMASK`
:   차단 시그널 집합을 `set` 인자로 설정한다.

`oldset`이 NULL이 아니면 이전 시그널 마스크 값을 `oldset`에 저장한다.

`set`이 NULL이면 시그널 마스크가 바뀌지 않는다. (즉, `how`를 무시한다.) 그렇다고 해도 현재 시그널 마스크 값을 `oldset`으로 (NULL이 아니라면) 반환한다.

`sigset_t`("시그널 집합") 타입 변수를 변경하고 검사하는 함수들을 <tt>[[sigsetops(3)]]</tt>에서 설명한다.

다중 스레드 프로세스에서의 `sigprocmask()` 사용은 명세되어 있지 않다. <tt>[[pthread_sigmask(3)]]</tt>를 보라.

## RETURN VALUE

`sigprocmask()`는 성공 시 0을 반환하고 오류 시 -1을 반환한다. 오류 때는 원인을 나타내도록 `errno`를 설정한다.

## ERRORS

`EFAULT`
:   `set`이나 `oldset` 인자가 프로세스에게 할당된 주소 공간 밖을 가리키고 있다.

`EINVAL`
:   `how`로 지정한 값이 유효하지 않거나 `sigsetsize`로 전달한 크기를 커널이 지원하지 않는다.

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## NOTES

`SIGKILL`이나 `SIGSTOP`을 막는 것은 불가능하다. 그렇게 하려는 시도는 조용히 무시된다.

프로세스의 각 스레드가 각자의 시그널 마스크를 가지고 있다.

<tt>[[fork(2)]]</tt>를 통해 생성된 자식은 부모의 시그널 마스크 사본을 물려받는다. <tt>[[execve(2)]]</tt>를 거치면서 시그널 마스크가 보존된다.

`SIGBUG`, `SIGFPE`, `SIGILL`, `SIGSEGV`가 블록 된 상태에서 생성되는 경우 <tt>[[kill(2)]]</tt>이나 <tt>[[sigqueue(3)]]</tt>, <tt>[[raise(3)]]</tt>로 시그널이 생성된 게 아니면 결과가 규정되어 있지 않다.

시그널 집합 조작에 대한 자세한 내용은 <tt>[[sigsetops(3)]]</tt>를 보라.

참고로 `set`과 `oldset`을 모두 NULL로 지정하는 것도 (아주 유용하지는 않지만) 허용된다.

### C 라이브러리/커널 차이

커널의 `sigset_t` 정의는 C 라이브러리에서 쓰는 것과 크기가 다르다. 이 매뉴얼 페이지에서는 전자를 `kernel_sigset_t`라고 부른다. (그렇지만 커널 소스에서의 이름은 `sigset_t`이다.)

glibc의 `sigprocmask()` 래퍼 함수에서는 NPTL 스레딩 구현 내부에서 쓰는 두 가지 실시간 시그널을 막으려는 시도를 조용히 무시한다. 자세한 내용은 <tt>[[nptl(7)]]</tt>을 보라.

원래 리눅스 시스템 호출의 이름은 `sigprocmask()`였다. 하지만 리눅스 2.2에 실시간 시그널이 추가되면서 그 시스템 호출이 지원하던 고정 크기 32비트 `sigset_t` 타입(이 매뉴얼 페이지에서 `old_kernel_sigset_t`)이 더는 용도에 맞지 않게 되었다. 그에 따라 확장된 `sigset_t` 타입(이 매뉴얼 페이지에서 `kernel_sigset_t`)을 지원하기 위해 새로운 시스템 호출 `rt_sigprocmask()`가 추가되었다. 새 시스템 호출에서 네 번째 인자로 `size_t sigsetsize`를 받는데, 이는 `set`과 `oldset`의 시그널 집합의 바이트 단위 크기를 나타낸다. 현재는 이 인자가 고정된 아키텍처별 값(`sizeof(kernel_sigset_t)`)을 가져야 한다.

glibc의 `sigprocmask()` 래퍼 함수에서 이런 세부 사항을 감추고 커널이 제공할 때 투명하게 `rt_sigprocmask()`를 호출한다.

## SEE ALSO

<tt>[[kill(2)]]</tt>, <tt>[[pause(2)]]</tt>, <tt>[[sigaction(2)]]</tt>, <tt>[[signal(2)]]</tt>, <tt>[[sigpending(2)]]</tt>, <tt>[[sigsuspend(2)]]</tt>, <tt>[[pthread_sigmask(3)]]</tt>, <tt>[[sigqueue(3)]]</tt>, <tt>[[sigsetops(3)]]</tt>, <tt>[[signal(7)]]</tt>

----

2017-09-15
