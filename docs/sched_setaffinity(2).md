## NAME

sched_setaffinity, sched_getaffinity - 스레드의 CPU 친화성 마스크 설정하고 얻기

## SYNOPSIS

```c
#define _GNU_SOURCE             /* feature_test_macros(7) 참고 */
#include <sched.h>

int sched_setaffinity(pid_t pid, size_t cpusetsize,
                      const cpu_set_t *mask);
int sched_getaffinity(pid_t pid, size_t cpusetsize,
                      cpu_set_t *mask);
```

## DESCRIPTION

스레드의 CPU 친화성 마스크는 그 스레드가 어떤 CPU들 위에서 돌 수 있는지 결정한다. 다중 프로세서 시스템에서 CPU 친화성 마스크를 설정하는 것으로 성능 이득을 얻을 수 있다. 예를 들어 한 CPU를 특정 스레드에게 독점시키면 (즉, 그 스레드의 친화성 마스크를 한 CPU만 나타내게 설정하고 다른 모든 스레드의 친화성 마스크를 그 CPU를 배제하게 설정하면) 그 스레드에 최대한의 실행 속도를 보장하는 것이 가능하다. 스레드가 한 CPU에서만 돌도록 제약하면 스레드가 한 CPU에서 실행을 중단하고 다른 CPU에서 실행을 재개할 때 발생하는 캐시 무효화로 인한 성능 비용을 피하게 되기도 한다.

`mask`가 가리키는 "CPU 세트"인 `cpu_set_t` 구조체로 CPU 친화성 마스크를 표현한다. CPU 세트를 조작하기 위한 매크로들을 <tt>[[CPU_SET(3)]]</tt>에서 기술한다.

`sched_setaffinity()`는 ID가 `pid`인 스레드의 CPU 친화성 마스크를 `mask`가 나타내는 값으로 설정한다. `pid`가 0이면 호출 스레드를 사용한다. `cpusetsize` 인자는 `mask`가 가리키는 데이터의 (바이트 단위) 길이이다. 보통은 이 인자를 `sizeof(cpu_set_t)`라고 지정하게 될 것이다.

`pid`로 지정한 스레드가 현재 `mask`에 지정한 CPU들 중 하나에서 돌고 있지 않으면 `mask`에 지정한 CPU들 중 하나로 스레드를 옮긴다.

`sched_getaffinity()`는 ID가 `pid`인 스레드의 친화성 마스크를 `mask`가 가리키는 `cpu_set_t` 구조체에 써넣는다. `cpusetsize` 인자는 `mask`의 (바이트 단위) 크기를 나타낸다. `pid`가 0이면 호출 스레드의 마스크를 반환한다.

## RETURN VALUE

성공 시 `sched_setaffinity()`와 `sched_getaffinity()`는 0을 반환한다. (하지만 기반 `sched_getaffinity()`의 반환 값 차이를 다루는 아래 "C 라이브러리/커널 차이"를 보라.) 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EFAULT`
:   제공한 메모리 주소가 유효하지 않다.

`EINVAL`
:   친화성 비트 마스크 `mask`가 현재 물리적으로 시스템 상에 있으면서 `cpuset` cgroup이나 <tt>[[cpuset(7)]]</tt>에서 기술하는 "cpuset" 메커니즘으로 부과할 수 있는 제약에 따라 스레드에게 허용된 프로세서를 하나도 포함하고 있지 않다.

`EINVAL`
:   (`sched_getaffinity()`, 그리고 2.6.9 전의 커널에서는 `sched_setaffinity()`도) `cpusetsize`가 커널에서 쓰는 친화성 마스크의 크기보다 작다.

`EPERM`
:   (`sched_setaffinity()`) 호출 스레드가 적절한 특권을 가지고 있지 않다. 호출자의 실효 사용자 ID가 `pid`가 가리키는 스레드의 실제 사용자 ID나 실효 사용자 ID와 같거나 호출자가 스레드 `pid`의 사용자 네임스페이스에서 `CAP_SYS_NICE` 역능을 소유하고 있어야 한다.

`ESRCH`
:   ID가 `pid`인 스레드를 찾을 수 없다.

## VERSIONS

리눅스 커널 2.5.8에서 CPU 친화성 시스템 호출들이 도입되었다. glibc 2.3에서 시스템 호출 래퍼들이 도입되었다. 처음에 glibc 인터페이스에는 `unsigned int` 타입인 `cpusetsize` 인자가 포함돼 있었다. glibc 2.3.3에서 `cpusetsize` 인자가 제거되었다가 glibc 2.3.4에서 `size_t` 타입으로 되살아났다.

## CONFORMING TO

이 시스템 호출들은 리눅스 전용이다.

## NOTES

`sched_setaffinity()` 호출 후에 스레드가 실제로 돌게 될 CPU들의 집합은 `mask` 인자에 지정한 집합과 시스템에 실제 존재하는 CPU들의 집합의 교집합이다. <tt>[[cpuset(7)]]</tt>에서 기술하는 "cpuset" 메커니즘을 사용하고 있으면 스레드가 도는 CPU 집합을 시스템에서 추가로 제약할 수도 있다. 스레드가 돌게 될 실제 CPU들의 집합에 대한 이 제약들은 커널이 조용히 적용한다.

시스템에서 사용 가능한 CPU들의 수를 알아내는 다양한 방법이 있다. `/proc/cpuinfo`의 내용을 확인하거나, <tt>[[sysconf(3)]]</tt>를 이용해 `_SC_NPROCESSORS_CONF` 및 `_SC_NPROCESSORS_ONLN` 매개변수의 값을 얻거나, `/sys/devices/system/cpu/` 하의 CPU 디렉터리 목록을 들여다 볼 수 있다.

<tt>[[sched(7)]]</tt>에서 리눅스 스케줄링 체계를 기술한다.

친화성 마스크는 스레드별 속성이므로 스레드 그룹 내의 스레드마다 독립적으로 조정할 수 있다. <tt>[[gettid(2)]]</tt> 호출이 반환한 값을 `pid` 인자로 전달할 수 있다. `pid`를 0으로 지정하면 호출 스레드의 속성을 설정하게 되고 <tt>[[getpid(2)]]</tt> 호출이 반환한 값을 전달하면 스레드 그룹의 주 스레드의 속성을 설정하게 된다. (POSIX 스레드 API를 쓰고 있다면 `sched_setaffinity()` 대신 <tt>[[pthread_setaffinity_np(3)]]</tt>를 사용하라.)

`isolcpus`라는 부팅 옵션을 이용해 부팅 시점에 한 개 이상의 CPU를 격리할 수 있다. 그러면 그 CPU들로 어떤 프로세스도 스케줄 되지 않는다. 이 부팅 옵션을 사용한 다음에는 `sched_setaffinity()`나 <tt>[[cpuset(7)]]</tt> 메커니즘을 통해서만 그 격리된 CPU로 프로세스를 스케줄 할 수 있다. 자세한 내용은 커널 소스 파일 `Documentation/admin-guide/kernel-parameters.txt`를 보라. 그 파일에서 언급하듯 `isolcpus`가 CPU를 격리하는 선호 메커니즘이다. (이에 대비되는 것은 시스템 상의 모든 프로세스들의 CPU 친화성을 수동으로 설정하는 것이다.)

<tt>[[fork(2)]]</tt>를 통해 생성된 자식은 부모의 CPU 친화성 마스크를 물려받는다. <tt>[[execve(2)]]</tt>를 거치면서 친화성 마스크가 유지된다.

### C 라이브러리/커널 차이

이 매뉴얼 페이지에서 기술하는 것은 CPU 친화성 호출의 glibc 인터페이스이다. 실제 시스템 호출 인터페이스는 살짝 다른데, CPU 세트의 기반 구현이 단순한 비트 마스크라는 사실을 반영하여 `mask`의 타입이 `unsigned long *`이다.

성공 시 진짜 `sched_getaffinity()` 시스템 호출은 `mask` 버퍼로 복사된 바이트 수를 반환한다. 이는 `cpusetsize`와 커널 내부에서 CPU 집합 표현에 쓰는 `cpumask_t` 데이터 타입의 (바이트 단위) 크기 중 작은 쪽이다.

### CPU 친화성 마스크가 큰 시스템 다루기

(`unsigned long *` 타입 비트 마스크로 CPU 마스크를 표현하는) 기반 시스템 호출에서는 CPU 마스크의 크기에 어떤 제약도 두지 않는다. 하지만 glibc에서 쓰는 `cpu_set_t` 데이터 타입은 크기가 128바이트로 고정되어 있다. 즉, 표현할 수 있는 가장 큰 CPU 번호가 1023이다. 커널의 CPU 친화성 마스크가 1024보다 크면 다음 형태의 호출이 `EINVAL` 오류로 실패한다.

```c
sched_getaffinity(pid, sizeof(cpu_set_t), &mask);
```

그 오류는 `cpusetsize`에 지정된 `mask` 크기가 커널에서 쓰는 친화성 마스크 크기보다 작은 경우에 기반 시스템 호출이 내놓는 것이다. (시스템 CPU 토폴로지에 따라선 커널 친화성 마스크가 시스템 상의 활성 CPU 개수보다 상당히 클 수 있다.)

커널 CPU 친화성 마스크가 큰 시스템에서 작업할 때는 `mask` 인자를 동적으로 할당해야 한다 (<tt>[[CPU_ALLOC(3)]]</tt> 참고). 현재 그렇게 하기 위한 유일한 방법은 마스크 크기를 늘이며 (호출이 `EINVAL` 오류로 실패하지 않을 때까지) `sched_getaffinity()` 호출을 해서 필요한 마스크 크기를 알아내는 것이다.

<tt>[[CPU_ALLOC(3)]]</tt>이 요청한 것보다 살짝 큰 CPU 세트를 할당할 수도 있음에 유의하라. (CPU 세트가 `sizeof(long)` 단위로 할당된 비트 마스크로 구현되어 있기 때문이다.) 그로 인해 `sched_getaffinity()`가 요청한 할당 크기 너머에서 비트를 설정할 수 있는데, 커널은 그 몇 개의 추가 비트를 보기 때문이다. 따라서 호출자가 (할당 요청을 했던 비트 수만큼 순회하는 것이 아니라) 반환된 세트의 비트들을 순회하며 설정된 비트 수를 세다가 <tt>[[CPU_COUNT(3)]]</tt>가 반환한 값에 도달했을 때 멈춰야 한다.

## EXAMPLES

아래 프로그램에서는 자식 프로세스를 만든다. 그리고 부모와 자식은 각각 지정한 CPU에 자기를 할당하고 동일한 루프를 실행해서 CPU 시간을 좀 소모한다. 종료 전에 부모는 자식 프로세스가 끝나기를 기다린다. 프로그램에서 명령 행 인자를 세 개 받는다. 부모용 CPU 번호, 자식용 CPU 번호, 그리고 두 프로세스가 수행해야 할 루프 반복 횟수이다.

아래 견본이 보여 주듯 프로그램이 실행 시 쓰는 실제 시간과 CPU 시간의 양은 코어 내 캐싱 효과와 프로세스들이 같은 CPU를 사용 중인지 여부에 따라 달라지게 된다.

먼저 `lscpu(1)`를 이용해 이 (x86) 시스템에 두 코어가 있고 각각에 두 CPU가 있다는 것을 알아낸다.

```text
$ lscpu | egrep -i 'core.*:|socket'
Thread(s) per core:    2
Core(s) per socket:    2
Socket(s):             1
```

그리고 세 가지 경우에서 예시 프로그램의 동작 시간을 잰다. 두 프로세스가 같은 CPU에서 돌 때, 두 프로세스가 같은 코어의 다른 CPU에서 돌 때, 그리고 두 프로세스가 다른 코어의 다른 CPU에서 돌 때이다.

```text
$ time -p ./a.out 0 0 100000000
real 14.75
user 3.02
sys 11.73
$ time -p ./a.out 0 1 100000000
real 11.52
user 3.98
sys 19.06
$ time -p ./a.out 0 3 100000000
real 7.89
user 3.29
sys 12.07
```

### 프로그램 소스

```c
#define _GNU_SOURCE
#include <sched.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>

#define errExit(msg)    do { perror(msg); exit(EXIT_FAILURE); \
                        } while (0)

int
main(int argc, char *argv[])
{
    cpu_set_t set;
    int parentCPU, childCPU;
    int nloops;

    if (argc != 4) {
        fprintf(stderr, "Usage: %s parent-cpu child-cpu num-loops\n",
                argv[0]);
        exit(EXIT_FAILURE);
    }

    parentCPU = atoi(argv[1]);
    childCPU = atoi(argv[2]);
    nloops = atoi(argv[3]);

    CPU_ZERO(&set);

    switch (fork()) {
    case -1:            /* 오류 */
        errExit("fork");

    case 0:             /* 자식 */
        CPU_SET(childCPU, &set);

        if (sched_setaffinity(getpid(), sizeof(set), &set) == -1)
            errExit("sched_setaffinity");

        for (int j = 0; j < nloops; j++)
            getppid();

        exit(EXIT_SUCCESS);

    default:            /* 부모 */
        CPU_SET(parentCPU, &set);

        if (sched_setaffinity(getpid(), sizeof(set), &set) == -1)
            errExit("sched_setaffinity");

        for (int j = 0; j < nloops; j++)
            getppid();

        wait(NULL);     /* 자식 종료 기다리기 */
        exit(EXIT_SUCCESS);
    }
}
```

## SEE ALSO

`lscpu(1)`, `nproc(1)`, `taskset(1)`, <tt>[[clone(2)]]</tt>, <tt>[[getcpu(2)]]</tt>, <tt>[[getpriority(2)]]</tt>, <tt>[[gettid(2)]]</tt>, <tt>[[nice(2)]]</tt>, <tt>[[sched_get_priority_max(2)]]</tt>, <tt>[[sched_get_priority_min(2)]]</tt>, <tt>[[sched_getscheduler(2)]]</tt>, <tt>[[sched_setscheduler(2)]]</tt>, <tt>[[setpriority(2)]]</tt>, <tt>[[CPU_SET(3)]]</tt>, <tt>[[get_nprocs(3)]]</tt>, <tt>[[pthread_setaffinity_np(3)]]</tt>, <tt>[[sched_getcpu(3)]]</tt>, <tt>[[capabilities(7)]]</tt>, <tt>[[cpuset(7)]]</tt>, <tt>[[sched(7)]]</tt>, `numactl(8)`

----

2021-03-22
