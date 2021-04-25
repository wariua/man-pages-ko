## NAME

sigemptyset, sigfillset, sigaddset, sigdelset, sigismember - POSIX 시그널 집합 연산

## SYNOPSIS

```c
#include <signal.h>

int sigemptyset(sigset_t *set);
int sigfillset(sigset_t *set);

int sigaddset(sigset_t *set, int signum);
int sigdelset(sigset_t *set, int signum);

int sigismember(const sigset_t *set, int signum);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`sigemptyset()`, `sigfillset()`, `sigaddset()`, `sigdelset()`, `sigismember()`:
:   `_POSIX_C_SOURCE`

## DESCRIPTION

이 함수들을 이용해 POSIX 시그널 집합을 조작할 수 있다.

`sigemptyset()`은 `set`으로 준 시그널 집합을 시그널이 모두 빠진 빈 집합으로 초기화 한다.

`sigfillset()`은 `set`을 모든 시그널을 포함한 가득 찬 집합으로 초기화 한다.

`sigaddset()`과 `sigdelset()`은 `set`에 시그널 `signum`을 더하고 뺀다.

`sigismember()`는 `signum`이 `set`에 속하는지 검사한다.

`sigset_t` 타입 객체를 함수 `sigaddset()`, `sigdelset()`, `sigismember()`나 아래 기술하는 추가 glibc 함수들(`sigisemptyset()`, `sigandset()`, `sigorset()`)로 전달하기 전에 `sigemptyset()`이나 `sigfillset()` 호출로 초기화 해야 한다. 그러지 않은 경우 결과는 규정되어 있지 않다.

## RETURN VALUE

`sigemptyset()`, `sigfillset()`, `sigaddset()`, `sigdelset()`은 성공 시 0을 반환하고 오류 시 -1을 반환한다.

`sigismember()`는 `signum`이 `set`에 속하면 1을 반환하고 속하지 않으면 0을 반환하며 오류 시 -1을 반환한다.

오류 시 이 함수들은 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EINVAL`
:   `signum`이 유효한 시그널이 아니다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값
| --- | --- | --- |
| `sigemptyset()`, `sigfillset()`, `sigaddset()`,<br>`sigdelset()`, `sigismember()`, `sigisemptyset()`,<br>`sigorset()`, `sigandset()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX-1.2008.

## NOTES

가득 찬 시그널 집합을 만들 때 glibc의 `sigfillset()` 함수는 NPTL 스레딩 구현 내부에서 쓰는 두 가지 실시간 시그널을 포함시키지 않는다. 자세한 내용은 <tt>[[nptl(7)]]</tt>을 보라.

### glibc 확장

기능 확인 매크로 `_GNU_SOURCE`가 정의되어 있으면 `<signal.h>`에서 시그널 집합 조작을 위한 또 다른 함수 세 가지를 드러낸다.

```c
int sigisemptyset(const sigset_t *set);
int sigorset(sigset_t *dest, const sigset_t *left,
              const sigset_t *right);
int sigandset(sigset_t *dest, const sigset_t *left,
              const sigset_t *right);
```

`sigisemptyset()`은 `set`이 아무 시그널도 담고 있지 않으면 1을 반환하고 그 외의 경우 0을 반환한다.

`sigorset()`은 집합 `left`와 `right`의 합집합을 `dest`에 집어넣는다. `sigandset()`은 집합 `left`와 `right`의 교집합을 `dest`에 집어넣는다. 두 함수 모두 성공 시 0을 반환하고 실패 시 -1을 반환한다.

이 함수들은 비표준이며 (몇몇 다른 시스템에서 비슷한 함수들을 제공함) 이식 가능한 응용에서는 사용을 피해야 한다.

## SEE ALSO

<tt>[[sigaction(2)]]</tt>, <tt>[[sigpending(2)]]</tt>, <tt>[[sigprocmask(2)]]</tt>, <tt>[[sigsuspend(2)]]</tt>

----

2021-03-22
