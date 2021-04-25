## NAME

getrlimit, setrlimit, prlimit - 자원 제한 얻기/설정하기

## SYNOPSIS

```c
#include <sys/time.h>
#include <sys/resource.h>

int getrlimit(int resource, struct rlimit *rlim);
int setrlimit(int resource, const struct rlimit *rlim);

int prlimit(pid_t pid, int resource, const struct rlimit *new_limit,
            struct rlimit *old_limit);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`prlimit()`:
:   `_GNU_SOURCE`

## DESCRIPTION

`getrlimit()` 및 `setrlimit()` 시스템 호출은 자원 제한을 얻고 설정한다. 각 자원에는 연성(soft) 제한과 경성(hard) 제한이 있으며 `rlimit` 구조체도 그렇게 정의돼 있다.

```c
struct rlimit {
    rlim_t rlim_cur;  /* 연성 제한 */
    rlim_t rlim_max;  /* 경성 제한 (rlim_cur의 상한) */
};
```

연성 제한은 해당 자원에 대해 커널이 강제하는 값이다. 경성 제한은 연성 제한의 상한 역할을 한다. 그래서 비특권 프로세스는 자기 연성 제한을 0에서 경성 제한까지 범위의 값으로만 설정할 수 있으며 자기 경성 제한을 (비가역적으로) 낮출 수만 있다. 특권 프로세스는 (리눅스에서: 최초 사용자 네임스페이스에서 `CAP_SYS_RESOURCE` 역능을 가진 프로세스는) 어느 쪽 제한값이든 마음대로 바꿀 수 있다.

`RLIM_INFINITY` 값은 (`getrlimit()`가 반환하는 구조체와 `setrlimit()`로 주는 구조체 모두에서) 자원에 제한이 없음을 나타낸다.

`resource` 인자는 다음 중 하나여야 한다.

`RLIMIT_AS`
:   프로세스 가상 메모리 (주소 공간) 최대 크기이다. 바이트 단위로 제한값을 지정하고 시스템 페이지 크기 배수로 내림 한다. 이 제한이 <tt>[[brk(2)]]</tt>, <tt>[[mmap(2)]]</tt>, <tt>[[mremap(2)]]</tt>에 영향을 주며 제한 초과 시 `ENOMEM` 오류로 실패하게 된다. 더불어 자동 스택 확장이 실패한다. (이 경우 <tt>[[sigaltstack(2)]]</tt>을 통해 대체 스택을 마련해 두지 않았으면 생성되는 `SIGSEGV`가 프로세스를 죽인다.) 값이 `long`이므로 32비트 `long`을 쓰는 머신에서는 이 제한이 최대 2 GiB이든지 아니면 이 자원에 제한이 없다.

`RLIMIT_CORE`
:   프로세스가 최대로 덤프 할 수 있는 *코어* 파일(<tt>[[core(5)]]</tt> 참고)의 바이트 단위 크기이다. 0이면 코어 파일이 생성되지 않는다. 0이 아니면 그보다 큰 덤프가 그 크기로 잘린다.

`RLIMIT_CPU`
:   프로세스가 소모할 수 있는 CPU 시간 양에 대한 초 단위 제한이다. 프로세스가 연성 제한에 도달하면 `SIGXCPU` 시그널을 받는다. 이 시그널에 대한 기본 동작은 프로세스를 종료시키는 것이다. 하지만 시그널을 잡을 수 있고 핸들러에서 메인 프로그램으로 제어를 반환할 수 있다. 프로세스가 계속 CPU 시간을 소모하면 일 초마다 `SIGXCPU`를 받다가 경성 제한에 도달하면 이번엔 `SIGKILL`을 받는다. (뒤쪽 설명은 리눅스의 동작 방식이다. 연성 제한에 도달하고도 계속 CPU 시간을 소모하는 프로세스를 어떻게 다루는지는 구현에 따라 다르다. 이 시그널을 잡아야 하는 이식성 있는 응용에서는 `SIGXCPU`를 처음 받았을 때 질서정연하게 종료를 수행해야 할 것이다.)

`RLIMIT_DATA`
:   프로세스 데이터 세그먼트(초기화 된 데이터, 초기화 안 된 데이터, 힙)의 최대 크기이다. 바이트 단위로 제한을 지정하며 시스템 페이지 크기 배수로 내림 한다. 이 제한이 <tt>[[brk(2)]]</tt>, <tt>[[sbrk(2)]]</tt>, (리눅스 4.7부터) <tt>[[mmap(2)]]</tt>에 영향을 주며 이 자원의 연성 제한을 만나면 `ENOMEM` 오류로 실패하게 된다.

`RLIMIT_FSIZE`
:   프로세스가 생성할 수 있는 파일의 바이트 단위 최대 크기이다. 이 제한을 넘겨서 파일을 확장하려고 하면 `SIGXFSZ` 시그널을 받는다. 기본적으로 이 시그널은 프로세스를 종료시킨다. 하지만 프로세스에서 시그널을 잡을 수 있으며 그 경우 해당 시스템 호출(가령 `write(2)`, <tt>[[truncate(2)]]</tt>)이 `EFBIG` 오류로 실패하게 된다.

`RLIMIT_LOCKS` (리눅스 2.4.0에서 2.4.24까지)
:   프로세스에서 설정할 수 있는 <tt>[[flock(2)]]</tt> 락 개수와 <tt>[[fcntl(2)]]</tt> 리스 수를 합친 것에 대한 제한이다.

`RLIMIT_MEMLOCK`
:   RAM에 최대로 고정해 둘 수 있는 메모리 바이트 수이다. 이 제한값을 가장 가까운 시스템 페이지 크기 배수로 내려서 적용한다. 이 제한이 <tt>[[mlock(2)]]</tt>, <tt>[[mlockall(2)]]</tt>, <tt>[[mmap(2)]]</tt> `MAP_LOCKED` 동작에 영향을 준다. 리눅스 2.6.9부터는 <tt>[[shmctl(2)]]</tt> `SHM_LOCK` 동작에도 영향을 주는데, 호출 프로세스의 실제 사용자 ID가 고정해 둘 수 있는 공유 메모리 세그먼트(<tt>[[shmget(2)]]</tt> 참고) 바이트 총합의 상한을 설정한다. <tt>[[shmctl(2)]]</tt> `SHM_LOCK` 방식 고정은 <tt>[[mlock(2)]]</tt>, <tt>[[mlockall(2)]]</tt>, <tt>[[mmap(2)]]</tt> `MAP_LOCKED`로 설정하는 프로세스별 메모리 고정과는 따로 계산한다. 즉 한 프로세스가 두 기준 각각에서 이 제한까지 고정할 수 있다.

    리눅스 커널 2.6.9 전에서는 이 제한이 특권 프로세스가 고정할 수 있는 메모리 양을 통제했다. 리눅스 2.6.9부터는 특권 프로세스가 고정할 수 있는 메모리 양에 어떤 한계도 두지 않으며 이 제한은 비특권 프로세스가 고정할 수 있는 메모리 양을 통제한다.

`RLIMIT_MSGQUEUE` (리눅스 2.6.8부터)
:   호출 프로세스의 실제 사용자 ID별로 POSIX 메시지 큐에 할당할 수 있는 바이트 수에 대한 제한이다. <tt>[[mq_open(3)]]</tt>에 이 제한이 적용된다. 사용자가 만드는 메시지 큐 각각에 대해 (제거하기 전까지) 다음 식에 따라 계산해서 이 제한을 적용한다.

    리눅스 3.5부터:
    :   

            bytes = attr.mq_maxmsg * sizeof(struct msg_msg) +
                    min(attr.mq_maxmsg, MQ_PRIO_MAX) *
                          sizeof(struct posix_msg_tree_node)+
                                    /* 오버헤드 */
                    attr.mq_maxmsg * attr.mq_msgsize;
                                    /* 메시지 데이터 */

    리눅스 3.4 및 이전:
    :   

            bytes = attr.mq_maxmsg * sizeof(struct msg_msg *) +
                                    /* 오버헤드 */
                    attr.mq_maxmsg * attr.mq_msgsize;
                                    /* 메시지 데이터 */

    여기서 `attr`은 <tt>[[mq_open(3)]]</tt>에 네 번째 인자로 지정하는 `mq_attr` 구조체이며 `msg_msg` 및 `posix_msg_tree_node` 구조체는 커널 내부용 구조체이다.

    위 식에서 "오버헤드" 부분은 구현에서 필요로 하는 오버헤드 바이트 수에 해당하며 사용자가 길이 0인 메시지를 무한정 만들 수 없게 한다. (그런 메시지 역시도 각각 유지에 얼마간의 시스템 메모리가 든다.)

`RLIMIT_NICE` (리눅스 2.6.12부터, 단 아래 BUGS 참고)
:   <tt>[[setpriority(2)]]</tt>나 <tt>[[nice(2)]]</tt>로 올릴 수 있는 프로세스 나이스 값의 상한을 나타낸다. 실제 나이스 값 상한은 `20 - rlim_cur`로 계산한다. 그래서 이 제한에 사용 가능한 범위는 1(나이스 값 19에 대응)에서 40(나이스 값 -20에 대응)까지다. 범위를 이렇게 특이하게 정한 건 보통 특수한 의미가 있는 음수를 자원 제한값으로 지정할 수 없기 때문이다. 예를 들어 `RLIM_INFINITY`는 보통 -1과 같다. 나이스 값에 대한 자세한 내용은 <tt>[[sched(7)]]</tt> 참고.

`RLIMIT_NOFILE`
:   이 프로세스가 최대로 열 수 있는 파일 디스크립터 개수보다 1만큼 큰 값을 지정한다. 이 제한을 초과하려는 시도(<tt>[[open(2)]]</tt>, <tt>[[pipe(2)]]</tt>, <tt>[[dup(2)]]</tt> 등)는 `EMFILE` 오류를 일으키게 된다. (BSD에서는 이 제한의 이름이 `RLIMIT_OFILE`인 적이 있었다.)

    리눅스 4.5부터는 (`CAP_SYS_RESOURCE` 역능 없는) 비특권 프로세스가 유닉스 도메인 소켓으로 전달해서 다른 프로세스로 "이송 중"일 수 있는 파일 디스크립터의 최대 개수까지 이 제한으로 규정된다. <tt>[[sendmsg(2)]]</tt> 시스템 호출에 이 제한이 적용된다. 자세한 내용은 <tt>[[unix(7)]]</tt> 참고.

`RLIMIT_NPROC`
:   호출 프로세스의 실제 사용자 ID별 현존 프로세스(정확히 말해 리눅스에서는 스레드) 수에 대한 제한이다. 프로세스의 실제 사용자 ID에게 속한 현재 프로세스 수가 이 제한값 이상인 동안에는 <tt>[[fork(2)]]</tt>가 `EAGAIN`으로 실패한다.

    `CAP_SYS_ADMIN`이나 `CAP_SYS_RESOURCE` 역능을 가진 프로세스에게는 `RLIMIT_NPROC` 제한이 적용되지 않는다.

`RLIMIT_RSS`
:   프로세스 상주 집합(램에 상주하는 가상 페이지 수)에 대한 (바이트 단위) 제한이다. 리눅스 2.4.x (x < 30)에서만 효력이 있으며 `MADV_WILLNEED`를 지정한 <tt>[[madvise(2)]]</tt> 호출에만 영향을 준다.

`RLIMIT_RTPRIO` (리눅스 2.6.12부터, 단 BUGS 참고)
:   <tt>[[sched_setscheduler(2)]]</tt>와 <tt>[[sched_setparam(2)]]</tt>으로 프로세스에 설정할 수 있는 실시간 우선순위의 상한을 지정한다.

    실시간 스케줄링 정책에 대한 자세한 내용은 <tt>[[sched(7)]]</tt> 참고.

`RLIMIT_RTTIME` (리눅스 2.6.25부터)
:   실시간 스케줄링 정책 하의 프로세스가 블로킹 시스템 호출을 부르지 않은 채로 소모할 수 있는 CPU 시간 양에 대한 (마이크로초 단위) 제한이다. 이 제한의 목적상 블록 하는 시스템 호출을 프로세스에서 부를 때마다 소모 CPU 시간이 0으로 재설정된다. 프로세스가 계속 CPU를 이용하려고 하는데 선점되거나, 타임 슬라이스가 만료되거나, <tt>[[sched_yield(2)]]</tt>를 호출하는 경우에는 CPU 시간이 재설정되지 않는다.

    연성 제한 도달 시 프로세스가 `SIGXCPU` 시그널을 받는다. 프로세스가 이 시그널을 잡거나 무시하고서 계속 CPU 시간을 소모하면 일 초마다 `SIGXCPU`를 받다가 경성 제한에 도달하면 이번엔 `SIGKILL` 시그널을 받는다.

    이 제한의 기본 용도는 제어 불능인 실시간 프로세스를 멈춰서 시스템이 먹통이 되지 않게 하는 것이다.

    실시간 스케줄링 정책에 대한 자세한 내용은 <tt>[[sched(7)]]</tt> 참고.

`RLIMIT_SIGPENDING` (리눅스 2.6.8부터)
:   호출 프로세스의 실제 사용자 ID별로 큐에 들어가 있을 수 있는 시그널 수에 대한 제한이다. 이 제한을 확인하는 데 있어선 표준 시그널과 실시간 시그널을 함께 센다. 하지만 제한이 적용되는 건 <tt>[[sigqueue(3)]]</tt>에서만이다. 즉 언제든 <tt>[[kill(2)]]</tt>을 사용해서 큐에 들어 있지 않던 시그널 종류의 인스턴스 하나를 큐에 넣는 게 가능하다.

`RLIMIT_STACK`
:   프로세스 스택 최대 크기이고 바이트 단위다. 이 제한에 도달하면 `SIGSEGV` 시그널이 생성된다. 프로세스에서 이 시그널을 다루려면 대체 시그널 스택(<tt>[[sigaltstack(2)]]</tt>)을 이용해야 한다.

    리눅스 2.6.23부터는 프로세스 명령행 인자 및 환경 변수에 쓰이는 공간의 양까지 이 제한이 결정한다. 자세한 내용은 <tt>[[execve(2)]]</tt> 참고.

### `prlimit()`

리눅스 전용인 `prlimit()` 시스템 호출은 `setrlimit()`와 `getrlimit()`의 기능을 합치고 확장한 것이다. 이걸 쓰면 임의 프로세스의 자원 제한을 설정하면서 얻을 수 있다.

`resource` 인자의 의미는 `setrlimit()` 및 `getrlimit()`에서와 같다.

`new_limit` 인자가 NULL이 아니면 가리키는 `rlimit` 구조체를 써서 `resource`에 대한 연성 및 경성 제한에 새 값을 설정한다. `old_limit` 인자가 NULL이 아니면 `prlimit()` 성공 호출 시 `resource`에 대한 이전 연성 및 경성 제한값이 `old_limit`가 가리키는 `rlimit` 구조체에 들어간다.

`pid` 인자는 호출이 동작할 프로세스의 ID를 나타낸다. `pid`가 0이면 호출 프로세스에게 호출이 적용된다. 자기 외 다른 프로세스의 제한값을 설정하거나 얻으려면 자원 제한이 변경될 프로세스의 사용자 네임스페이스에서 호출자가 `CAP_SYS_RESOURCE` 역능을 가지고 있거나, 대상 프로세스의 실제, 실효, saved set 사용자 ID가 호출자의 실제 사용자 ID와 일치하고 *동시에* 대상 프로세스의 실제, 실효, saved set 그룹 ID가 호출자의 실제 그룹 ID와 일치해야 한다.

## RETURN VALUE

성공 시 이 시스템 호출들은 0을 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EFAULT`
:   포인터 인자가 접근 가능한 주소 공간 밖의 위치를 가리킨다.

`EINVAL`
:   `resource`에 지정한 값이 유효하지 않다. 또는 `setrlimit()`나 `prlimit()`에서 `rlim->lim_cur`이 `rlim->lim_max`보다 크다.

`EPERM`
:   비특권 프로세스가 경성 제한을 올리려고 했다. 그러려면 `CAP_SYS_RESOURCE` 역능이 필요하다.

`EPERM`
:   호출자가 `RLIMIT_NOFILE` 제한을 `/proc/sys/fs/nr_open`(<tt>[[proc(5)]]</tt> 참고)에 규정된 최대치 너머로 올리려고 했다.

`EPERM`
:   (`prlimit()`) 호출 프로세스에게 `pid`로 지정한 프로세스의 제한값을 설정할 권한이 없다.

`ESRCH`
:   `pid`에 지정한 ID의 프로세스를 찾을 수 없다.

## VERSIONS

리눅스 2.6.36부터 `prlimit()` 시스템 호출이 사용 가능하다. glibc 2.13부터 라이브러리 지원이 제공된다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `getrlimit()`, `setrlimit()`, `prlimit()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

`getrlimit()`, `setrlimit()`: POSIX.1-2001, POSIX.1-2008, SVr4, 4.3BSD.

`prlimit()`: 리눅스 전용.

`RLIMIT_MEMLOCK`과 `RLIMIT_NPROC`은 BSD에서 온 것이며 POSIX.1에는 명세돼 있지 않다. BSD 계열과 리눅스에는 있지만 다른 구현에는 거의 없다. `RLIMIT_RSS`는 BSD에서 온 것이며 POSIX.1에는 명세돼 있지 않다. 하지만 대다수 구현들에 존재한다. `RLIMIT_MSGQUEUE`, `RLIMIT_NICE`, `RLIMIT_RTPRIO`, `RLIMIT_RTTIME`, `RLIMIT_SIGPENDING`은 리눅스 전용이다.

## NOTES

<tt>[[fork(2)]]</tt>를 통해 생성된 자식 프로세스는 부모의 자원 제한들을 물려받는다. <tt>[[execve(2)]]</tt>를 거치면서 자원 제한들이 보존된다.

자원 제한은 프로세스별 속성이므로 프로세스 내 모든 스레드가 공유한다.

자원의 연성 제한을 프로세스가 현재 그 자원을 쓰고 있는 것보다 낮게 내려도 성공한다. (하지만 프로세스가 그 자원 사용량을 더 높이지 못하게 된다.)

셸의 내장 명령 `ulimit`(`csh(1)`에서는 `limit`)을 사용해 셸의 자원 제한을 설정할 수 있다. 셸에서 명령 실행을 위해 생성하는 프로세스들이 그 자원 제한을 물려받는다.

리눅스 2.6.24부터 `/proc/[pid]/limits`를 통해 모든 프로세스의 자원 제한값을 들여다볼 수 있다. <tt>[[proc(5)]]</tt> 참고.

아주 옛날 시스템에선 `setrlimit()`과 용도가 비슷한 `vlimit()` 함수를 제공했다. 하위 호환성을 위해 glibc에서도 `vlimit()`를 제공한다. 새로 작성하는 응용은 모두 `setrlimit()`을 쓰도록 작성해야 한다.

### C 라이브러리/커널 ABI 차이

버전 2.12부터 glibc의 `getrlimit()` 및 `setrlimit()` 래퍼 함수에서는 더이상 대응하는 시스템 호출을 부르지 않고 대신 `prlimit()`을 이용한다. 이는 BUGS에서 설명하는 이유들 때문이다.

glibc 래퍼 함수의 이름이 `prlimit()`이다. 기반 시스템 호출은 `prlimit64()`이다.

## BUGS

구식 리눅스 커널들에서는 프로세스가 연성 및 경성 `RLIMIT_CPU` 제한을 만났을 때 (CPU 사용 시간으로) 일 초 늦게 `SIGXCPU` 및 `SIGKILL` 시그널이 전달된다. 커널 2.6.8에서 수정됐다.

2.6.x 커널에서 2.6.17 전까지에서는 `RLIMIT_CPU` 제한값 0을 (`RLIM_INFINITY`처럼) "제한 없음"으로 잘못 처리한다. 2.6.17부터 제한값을 0으로 설정하는 것은 효력이 없다. 단 실제로는 제한값 1초로 다룬다.

커널 2.6.12에서는 커널 버그 때문에 `RLIMIT_RTPRIO`가 동작하지 않는다. 커널 2.6.13에서 문제가 수정됐다.

커널 2.6.12에서는 <tt>[[getpriority(2)]]</tt>가 반환하는 우선순위 범위와 `RLIMIT_NICE` 간에 1만큼의 불일치가 있었다. 이 때문에 나이스 값 실제 상한을 `19 - rlim_cur`로 계산했다. 커널 2.6.13에서 수정됐다.

리눅스 2.6.12부터는 프로세스가 `RLIMIT_CPU` 제한에 도달했는데 `SIGXCPU`에 대한 핸들러가 설치돼 있는 경우 커널에서 시그널 핸들러를 호출하고 추가로 연성 제한을 1초만큼 올린다. 프로세스가 계속 CPU 시간을 소모하면 이 동작이 되풀이되고, 그러다 경성 제한에 도달하면 프로세스가 죽는다. 다른 구현들에서는 이런 식으로 `RLIMIT_CPU` 연성 제한을 바꾸지 않으며 리눅스의 동작 방식은 표준에 부합하지 않을 수 있다. 따라서 이식 가능한 응용에서는 리눅스 한정인 이 동작 방식에 의존하는 걸 피해야 한다. 리눅스 한정인 `RLIMIT_RTTIME` 제한 역시 연성 제한을 만났을 때 같은 동작을 보인다.

커널 2.4.22 전에서는 `setrlimit()`에서 `rlim->rlim_cur`가 `rlim->rlim_max`보다 클 때 `EINVAL` 오류로 진단하지 않았다.

리눅스에서는 호환성 문제 때문에 `RLIMIT_CPU` 설정 시도가 실패했을 때 오류를 반환하지 않는다.

### 32비트 플랫폼에서 "큰" 자원 제한값 표현하기

glibc의 `getrlimit()` 및 `setrlimit()` 래퍼 함수는 32비트 플랫폼에서도 64비트짜리 `rlim_t` 데이터 타입을 쓴다. 하지만 `getrlimit()` 및 `setrlimit()` 시스템 호출에서 쓰는 `rlim_t` 데이터 타입은 (32비트인) `unsigned long`이다. 또한 리눅스의 커널에서는 32비트 플랫폼에서 `unsigned long`으로 자원 제한을 표현한다. 하지만 32비트 데이터 타입은 충분히 크지 않다. 특히 그런 경우가 파일이 최대로 커질 수 있는 크기를 지정하는 `RLIMIT_FSIZE`이다. 이 제한이 유용하려면 파일 오프셋을 나타내는 데 쓰는 타입만큼은 큰 타입을 써서 제한을 표현해야 한다. 즉 (프로그램이 `_FILE_OFFSET_BITS=64`로 컴파일 되었다고 가정하면) 64비트인 `off_t`만큼은 커야 한다.

커널의 이런 한계를 피하기 위해 프로그램에서 자원 제한값을 32비트 `unsigned long`으로 표현 가능한 것보다 크게 설정하는 경우 glibc의 `setrlimit()` 래퍼 함수에서는 그 제한값을 조용히 `RLIM_INFINITY`로 바꿨다. 달리 말해 요청한 자원 제한 설정이 조용히 무시됐다.

버전 2.13부터 glibc에서는 `getrlimit()` 및 `setrlimit()` 시스템 호출의 한계를 피하기 위해 `prlimit()`를 호출하는 래퍼 함수 형태로 `setrlimit()` 및 `getrlimit()`를 구현한다.

## EXAMPLES

아래 프로그램은 `prlimit()` 사용 방식을 보여 준다.

```c
#define _GNU_SOURCE
#define _FILE_OFFSET_BITS 64
#include <stdint.h>
#include <stdio.h>
#include <time.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/resource.h>

#define errExit(msg) do { perror(msg); exit(EXIT_FAILURE); \
                        } while (0)

int
main(int argc, char *argv[])
{
    struct rlimit old, new;
    struct rlimit *newp;
    pid_t pid;

    if (!(argc == 2 || argc == 4)) {
        fprintf(stderr, "Usage: %s <pid> [<new-soft-limit> "
                "<new-hard-limit>]\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    pid = atoi(argv[1]);        /* 대상 프로세스의 PID */

    newp = NULL;
    if (argc == 4) {
        new.rlim_cur = atoi(argv[2]);
        new.rlim_max = atoi(argv[3]);
        newp = &new;
    }

    /* 대상 프로세스의 CPU 시간 제한 설정.
       이전 제한값 얻어 와서 표시. */

    if (prlimit(pid, RLIMIT_CPU, newp, &old) == -1)
        errExit("prlimit-1");
    printf("Previous limits: soft=%jd; hard=%jd\n",
            (intmax_t) old.rlim_cur, (intmax_t) old.rlim_max);

    /* 새 CPU 시간 제한 얻어 와서 표시. */

    if (prlimit(pid, RLIMIT_CPU, NULL, &old) == -1)
        errExit("prlimit-2");
    printf("New limits: soft=%jd; hard=%jd\n",
            (intmax_t) old.rlim_cur, (intmax_t) old.rlim_max);

    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

`prlimit(1)`, <tt>[[dup(2)]]</tt>, <tt>[[fcntl(2)]]</tt>, <tt>[[fork(2)]]</tt>, <tt>[[getrusage(2)]]</tt>, <tt>[[mlock(2)]]</tt>, <tt>[[mmap(2)]]</tt>, <tt>[[open(2)]]</tt>, <tt>[[quotactl(2)]]</tt>, <tt>[[sbrk(2)]]</tt>, <tt>[[shmctl(2)]]</tt>, <tt>[[malloc(3)]]</tt>, <tt>[[sigqueue(3)]]</tt>, <tt>[[ulimit(3)]]</tt>, <tt>[[core(5)]]</tt>, <tt>[[capabilities(7)]]</tt>, <tt>[[cgroups(7)]]</tt>, <tt>[[credentials(7)]]</tt>, <tt>[[signal(7)]]</tt>

----

2021-03-22
