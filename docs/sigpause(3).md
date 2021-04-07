## NAME

sigpause - 차단된 시그널들을 원자적으로 해제하고 인터럽트 기다리기

## SYNOPSIS

```c
#include <signal.h>

int sigpause(int sigmask);  /* BSD (하지만 NOTES 참고) */

int sigpause(int sig);      /* 시스템 V / 유닉스 95 */
```

## DESCRIPTION

이 함수를 사용해선 안 된다. 대신 <tt>[[sigsuspend(2)]]</tt>를 사용하라.

`sigpause()` 함수는 어떤 시그널을 기다리도록 설계되어 있다. 프로세스의 시그널 마스크(차단된 시그널들의 집합)를 바꾸고서 시그널이 도착하기를 기다린다. 시그널 도착 시 원래 시그널 마스크를 복원한다.

## RETURN VALUE

`sigpause()`가 반환한 경우 시그널에 의해 중단된 것이므로 반환 값이 -1이고 `errno`가 `EINTR`로 설정된다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값
| --- | --- | --- |
| `sigpause()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

시스템 V 버전 `sigpause()`가 POSIX.1-2001에서 표준화되었다. POSIX.1-2008에도 명세되어 있는데, 구식으로 표시되어 있다.

## NOTES

### 역사

이 함수의 전통적인 BSD 버전은 4.2BSD에서 등장했다. 프로세스의 시그널 마스크를 `sigmask`로 설정한다. 유닉스 95에서 이 함수의 비호환 시스템 V 버전을 표준화하였는데, 프로세스의 시그널 마스크에서 지정한 시그널 `sig`만 제거한다. 이름이 같은 호환 안 되는 함수 두 가지가 있는 안타까운 상황이 해결된 것은 (`int` 대신) `sigset_t *` 인자를 받는 <tt>[[sigsuspend(2)]]</tt> 함수에 의해서이다.

### 리눅스 참고 사항

리눅스에서는 스팍(sparc64) 아키텍처에서만 이 루틴이 시스템 호출이다.

glibc에서는 기능 확인 매크로 <code>_BSD_SOURCE</code>가 정의되어 있으며 <code>_POSIX_SOURCE</code> <code>_POSIX_C_SOURCE</code>, <code>_XOPEN_SOURCE</code>, <code>_GNU_SOURCE</code>, <code>_SVID_SOURCE</code> 중 어느 것도 정의되어 있지 않은 경우에 BSD 버전을 사용한다. 그 외의 경우에는 시스템 V 버전을 쓰며, 선언을 얻으려면 기능 확인 매크로가 다음과 같이 정의되어 있어야 한다.

* glibc 2.26부터: <code>_XOPEN_SOURCE >= 500</code>

* glibc 2.25 및 이전: <code>_XOPEN_SOURCE</code>

glibc 2.19부터는 `<signal.h>`에서 시스템 V 버전만 드러낸다. 이전에 BSD `sigpause()`를 사용하던 응용은 <tt>[[sigsuspend(2)]]</tt>를 사용하도록 수정해야 한다.

## SEE ALSO

<tt>[[kill(2)]]</tt>, <tt>[[sigaction(2)]]</tt>, <tt>[[sigprocmask(2)]]</tt>, <tt>[[sigsuspend(2)]]</tt>, <tt>[[sigblock(3)]]</tt>, <tt>[[sigvec(3)]]</tt>, <tt>[[feature_test_macros(7)]]</tt>

----

2017-09-15
