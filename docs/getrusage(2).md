## NAME

getrusage - 자원 사용량 얻기

## SYNOPSIS

```c
#include <sys/time.h>
#include <sys/resource.h>

int getrusage(int who, struct rusage *usage);
```

## DESCRIPTION

`getrusage()`는 다음 중 하나일 수 있는 `who`에 대해 자원 사용량 측정치를 반환한다.

`RUSAGE_SELF`
:   호출 프로세스의 자원 사용 통계를 반환한다. 프로세스 내 모든 스레드의 자원 사용 합계이다.

`RUSAGE_CHILDREN`
:   종료돼서 대기까지 이뤄진 호출 프로세스의 자식 모두에 대한 자원 사용 통계를 반환한다. 사이의 프로세스들이 모두 종료된 자식에 대기를 했으면 자식 아래 후손들이 쓴 자원까지 이 통계에 포함된다.

`RUSAGE_THREAD` (리눅스 2.6.26부터)
:   호출 스레드의 자원 사용 통계를 반환한다. `<sys/resource.h>`에서 이 상수 정의를 얻으려면 (*어떤* 헤더도 포함시키기 전에) `_GNU_SOURCE` 기능 확인 매크로가 정의돼 있어야 한다.

`usage`가 가리키는 구조체로 자원 사용량이 반환된다. 그 구조체는 다음 형태이다.

```c
struct rusage {
    struct timeval ru_utime; /* 사용한 사용자 CPU 시간 */
    struct timeval ru_stime; /* 사용한 시스템 CPU 시간 */
    long   ru_maxrss;        /* 최대 상주 집합 크기 */
    long   ru_ixrss;         /* 필수 공유 메모리 크기 */
    long   ru_idrss;         /* 필수 비공유 데이터 크기 */
    long   ru_isrss;         /* 필수 비공유 스택 크기 */
    long   ru_minflt;        /* 페이지 재활용 (연성 페이지 폴트) 횟수 */
    long   ru_majflt;        /* 페이지 폴트 (경성 페이지 폴트) 횟수 */
    long   ru_nswap;         /* 스왑 */
    long   ru_inblock;       /* 블록 입력 동작 횟수 */
    long   ru_oublock;       /* 블록 출력 동작 횟수 */
    long   ru_msgsnd;        /* IPC 메시지 전송 횟수 */
    long   ru_msgrcv;        /* IPC 메시지 수신 횟수 */
    long   ru_nsignals;      /* 시그널 수신 횟수 */
    long   ru_nvcsw;         /* 자발적 문맥 전환 횟수 */
    long   ru_nivcsw;        /* 비자발적 문맥 전환 횟수 */
};
```

모든 필드들이 채워지진 않는다. 비지원 필드들은 커널에서 0으로 설정한다. (지원하지 않는데 필드들을 제공하는 건 다른 시스템들과의 호환성을 위해서이기도 하고 언젠가 리눅스에서 지원하게 될 수도 있기 때문이다.) 다음과 같이 필드들을 해석한다.

`ru_utime`
:   사용자 모드에서 실행하며 보낸 시간의 총량을 `timeval` 구조체로 (초와 마이크로초로) 표현한 것이다.

`ru_stime`
:   커널 모드에서 실행하며 보낸 시간의 총량을 `timeval` 구조체로 (초와 마이크로초로) 표현한 것이다.

`ru_maxrss` (리눅스 2.6.32부터)
:   최대로 사용한 상주 집합 (킬로바이트 단위) 크기이다. `RUSAGE_CHILDREN`인 경우 프로세스 트리에서 가장 큰 상주 집합 크기가 아니라 자식들 중 최대 상주 집합 크기이다.

`ru_ixrss` (비지원)
:   현재 리눅스에서는 이 필드를 쓰지 않는다.

`ru_idrss` (비지원)
:   현재 리눅스에서는 이 필드를 쓰지 않는다.

`ru_isrss` (비지원)
:   현재 리눅스에서는 이 필드를 쓰지 않는다.

`ru_minflt`
:   I/O 활동 없이 처리한 페이지 폴트 수. 재할당 대기 페이지 목록의 페이지 프레임을 "재활용"해서 I/O 활동을 피한다.

`ru_majflt`
:   I/O 활동이 필요했던 처리 페이지 폴트 수.

`ru_nswap` (비지원)
:   현재 리눅스에서는 이 필드를 쓰지 않는다.

`ru_inblock` (리눅스 2.6.22부터)
:   파일 시스템에서 입력을 수행해야 했던 횟수.

`ru_oublock` (리눅스 2.6.22부터)
:   파일 시스템에서 출력을 수행해야 했던 횟수.

`ru_msgsnd` (비지원)
:   현재 리눅스에서는 이 필드를 쓰지 않는다.

`ru_msgrcv` (비지원)
:   현재 리눅스에서는 이 필드를 쓰지 않는다.

`ru_nsignals` (비지원)
:   현재 리눅스에서는 이 필드를 쓰지 않는다.

`ru_nvcsw` (리눅스 2.6부터)
:   타임 슬라이스가 끝나기 전에 (일반적으로 자원이 사용 가능해지기를 기다리려고) 프로세스가 자발적으로 프로세서를 포기하여 발생한 문맥 전환 횟수.

`ru_nivcsw` (리눅스 2.6부터)
:   더 높은 우선순위의 프로세스가 실행 가능해지거나 현재 프로세스가 타임 슬라이스를 초과해서 발생한 문맥 전환 횟수.

## RETURN VALUE

성공 시 0을 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EFAULT`
:   `usage`가 접근 가능한 주소 공간 밖을 가리키고 있다.

`EINVAL`
:   `who`가 유효하지 않다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `getrusage()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, SVr4, 4.3BSD. POSIX.1에서 `getrusage()`를 명세하고는 있지만 `ru_utime` 및 `ru_stime` 필드만 명세한다.

`RUSAGE_THREAD`는 리눅스 전용이다.

## NOTES

<tt>[[execve(2)]]</tt>를 거치면서 자원 사용량 수치들이 유지된다.

`<sys/time.h>`를 포함시키는 게 요즘에는 필요치 않지만 이식성을 높여 준다. (사실 `struct timeval`이 `<sys/time.h>`에 정의돼 있다.)

리눅스 커널 버전 2.6.9 전에서는 `SIGCHLD` 처리 방식이 `SIG_IGN`으로 설정돼 있는 경우에 자식 프로세스의 자원 사용량이 자동으로 `RUSAGE_CHILDREN` 반환 값에 포함된다. 하지만 POSIX.1-2001에서는 이를 명확히 금지한다. 리눅스 2.6.9 및 이후에서는 이 불일치가 시정돼 있다.

이 페이지 앞에 나온 구조체 정의는 4.3BSD Reno에서 가져온 것이다.

아주 옛날 시스템들에선 `getrusage()`와 용도가 비슷한 `vtimes()` 함수를 제공했다. 하위 호환성을 위해 glibc에서도 (버전 2.32까지) `vtimes()`를 제공한다. 새로 작성하는 응용은 모두 `getrusage()`를 쓰도록 작성해야 한다. (버전 2.33부터 glibc에서 `vtimes()` 구현체를 더이상 제공하지 않는다.)

<tt>[[proc(5)]]</tt>의 `/proc/[pid]/stat` 설명도 참고.

## SEE ALSO

<tt>[[clock_gettime(2)]]</tt>, <tt>[[getrlimit(2)]]</tt>, <tt>[[times(2)]]</tt>, <tt>[[wait(2)]]</tt>, <tt>[[wait4(2)]]</tt>, <tt>[[clock(3)]]</tt>

----

2021-03-22
