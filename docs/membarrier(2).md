## NAME

membarrier - 스레드 집합에 메모리 배리어 주기

## SYNOPSIS

```c
#include <linux/membarrier.h>

int membarrier(int cmd, int flags);
```

## DESCRIPTION

`membarrier()` 시스템 호출은 다중 코어 시스템에서 메모리 접근 순서 제어에 필요한 메모리 배리어 인스트럭션의 오버헤드를 줄일 수 있게 해 준다. 하지만 이 시스템 호출이 메모리 배리어보다 무거우며, 그래서 효과적으로 이용하려면 메모리 배리어를 이 시스템 호출로 교체하기만 하면 되는 것이 *아니라* 아래 세부 내용에 대한 이해가 필요하다.

메모리 배리어를 사용할 때 고려해야 하는 것은 메모리 배리어에 항상 상대가 되는 메모리 배리어 짝이 있어야 할 수도 있고, 아키텍처 메모리 모델에서 그런 대응 배리어를 요구하지 않을 수도 있다는 점이다.

대응 배리어 중 한쪽(이하 "빠른 쪽")이 다른 쪽(이하 "느린 쪽")보다 훨씬 자주 실행되는 경우가 있다. 그런 경우가 `membarrier()`를 사용할 주요 대상이다. 핵심 아이디어는 그런 대응 배리어에서 빠른 쪽 메모리 배리어를 다음과 같은 단순 컴파일러 배리어로 교체하고 느린 쪽 메모리 배리어를 `membarrier()` 호출로 교체하는 것이다.

```c
asm volatile ("" : : : "memory")
```

그러면 느린 쪽에 오버헤드를 더하고 빠른 쪽에서 오버헤드를 없애게 되어 `membarrier()` 호출 오버헤드가 빠른 쪽 성능 이득을 상회하지 않을 만큼 느린 쪽이 드물기만 하다면 결과적으로 전체 성능이 올라가게 된다.

`cmd` 인자는 다음 중 하나이다.

`MEMBARRIER_CMD_QUERY` (리눅스 4.3부터)
:   지원 명령 집합을 질의한다. 호출 반환 값은 지원 명령들의 비트 마스크이다. `MEMBARRIER_CMD_QUERY`의 값은 0이므로 그 비트 마스크에 포함되지 않는다. (`membarrier()`를 제공하는 커널에서) 이 명령은 항상 지원한다.

`MEMBARRIER_CMD_GLOBAL` (리눅스 4.16부터)
:   `membarrier()` 시스템 호출 진입과 반환 사이에서 시스템 상의 모든 프로세스의 스레드 모두가 사용자 공간 주소에 대한 모든 메모리 접근이 프로그램 순서와 일치하는 상태를 거치도록 한다. 이 명령은 시스템의 스레드 모두를 대상으로 한다.

`MEMBARRIER_CMD_GLOBAL_EXPEDITED` (리눅스 4.16부터)
:   `MEMBARRIER_CMD_REGISTER_GLOBAL_EXPEDITED`로 미리 등록한 프로세스 모두의 실행 중인 스레드 모두에서 메모리 배리어를 실행한다.

    시스템 호출 반환 시에, 실행 중인 스레드 모두가 사용자 공간 주소에 대한 모든 메모리 접근이 프로그램 순서와 일치하는 상태를 시스템 호출 진입과 반환 사이에서 거쳤다는 보장을 호출 스레드가 얻게 된다. (실행 중이 아닌 스레드는 실질적으로 그런 상태에 있는 것이다.) `MEMBARRIER_CMD_REGISTER_GLOBAL_EXPEDITED`로 미리 등록한 프로세스의 스레드에 대해서만 보장이 이뤄진다.

    등록은 배리어를 받아들이겠다는 의도에 대한 것이므로 `MEMBARRIER_CMD_REGISTER_GLOBAL_EXPEDITED`를 쓴 적 없는 프로세스에서 `MEMBARRIER_CMD_GLOBAL_EXPEDITED`를 호출하는 것이 유효하다.

    "신속 처리(expedited)" 명령은 그렇지 않은 명령보다 빨리 완료된다. 절대 블록 하지 않는다. 하지만 추가 오버헤드를 유발하는 단점이 있다.

`MEMBARRIER_CMD_REGISTER_GLOBAL_EXPEDITED` (리눅스 4.16부터)
:   `MEMBARRIER_CMD_GLOBAL_EXPEDITED` 메모리 배리어를 받아들이려는 프로세스의 의도를 알린다.

`MEMBARRIER_CMD_PRIVATE_EXPEDITED` (리눅스 4.14부터)
:   호출 스레드와 같은 프로세스에 속한 실행 중인 스레드 각각에서 메모리 배리어를 실행한다.

    시스템 호출 반환 시에, 실행 중인 형제자매 스레드 모두가 사용자 공간 주소에 대한 모든 메모리 접근이 프로그램 순서와 일치하는 상태를 시스템 호출 진입과 반환 사이에서 거쳤다는 보장을 호출 스레드가 얻게 된다. (실행 중이 아닌 스레드는 실질적으로 그런 상태에 있는 것이다.) 호출 스레드와 같은 프로세스의 스레드에 대해서만 보장이 이뤄진다.

    "신속 처리" 명령은 그렇지 않은 명령보다 빨리 완료된다. 절대 블록 하지 않는다. 하지만 추가 오버헤드를 유발하는 단점이 있다.

    프로세스별 신속 처리 명령을 사용하려는 프로세스는 사용 전에 그 의도를 미리 알려야 한다.

`MEMBARRIER_CMD_REGISTER_PRIVATE_EXPEDITED` (리눅스 4.14부터)
:   `MEMBARRIER_CMD_PRIVATE_EXPEDITED`를 사용하려는 프로세스의 의도를 알린다.

`MEMBARRIER_CMD_PRIVATE_EXPEDITED_SYNC_CORE` (리눅스 4.16부터)
:   `MEMBARRIER_CMD_PRIVATE_EXPEDITED`에서 기술한 메모리 순서 보장에 더해서, 시스템 호출 반환 시에 실행 중인 형제자매 스레드 모두가 코어 직렬화 인스트럭션을 실행했다는 보장을 호출 스레드가 얻게 된다. 호출 스레드와 같은 프로세스의 스레드에 대해서만 이 보장이 이뤄진다.

    "신속 처리" 명령은 그렇지 않은 명령보다 빨리 완료된다. 절대 블록 하지 않는다. 하지만 추가 오버헤드를 유발하는 단점이 있다.

    프로세스별 신속 처리 코어 동기화 명령을 사용하려는 프로세스는 사용 전에 그 의도를 미리 알려야 한다.

`MEMBARRIER_CMD_REGISTER_PRIVATE_EXPEDITED_SYNC_CORE` (리눅스 4.16부터)
:   `MEMBARRIER_CMD_PRIVATE_EXPEDITED_SYNC_CORE`를 사용하려는 프로세스의 의도를 알린다.

`MEMBARRIER_CMD_SHARED` (리눅스 4.3부터)
:   헤더 하위 호환성을 위해 존재하는 `MEMBARRIER_CMD_GLOBAL`의 별칭이다.

`flags` 인자는 현재 사용하지 않으며 0으로 지정해야 한다.

각 대상 스레드에서 프로그램 순서로 수행되는 모든 메모리 접근은 `membarrier()`에 대해서 순서가 지켜진다고(ordered) 보장된다.

배리어 전후로 프로그램 순서에 따라 메모리 접근을 수행하게 강제하는 컴파일러 배리어를 의미론적 `barrier()`로 나타내고 배리어 전후로 완전한 메모리 접근 순서를 강제하는 명시적 메모리 배리어를 `smp_mb()`로 나타낸다면, `barrier()`, `membarrier()`, `smp_mb()`의 각 쌍에 대한 다음 순서 테이블을 얻는다. O는 순서가 지켜지는 것이고 X는 순서가 지켜지지 않는 것이다.

|                | `barrier()` | `smp_mb()` | `membarrier()` |
| -------------- | ----------- | ---------- | -------------- |
| `barrier()`    |      X      |      X     |        O       |
| `smp_mb()`     |      X      |      O     |        O       |
| `membarrier()` |      O      |      O     |        O       |

## RETURN VALUE

성공 시 `MEMBARRIER_CMD_QUERY` 동작은 지원 명령들의 비트 마스크를 반환하며, `MEMBARRIER_CMD_GLOBAL`, `MEMBARRIER_CMD_GLOBAL_EXPEDITED`, `MEMBARRIER_CMD_REGISTER_GLOBAL_EXPEDITED`, `MEMBARRIER_CMD_PRIVATE_EXPEDITED`, `MEMBARRIER_CMD_REGISTER_PRIVATE_EXPEDITED`, `MEMBARRIER_CMD_PRIVATE_EXPEDITED_SYNC_CORE`, `MEMBARRIER_CMD_REGISTER_PRIVATE_EXPEDITED_SYNC_CORE` 동작은 0을 반환한다. 오류 시 -1을 반환하며 `errno`를 적절히 설정한다.

한 명령에 대해서 `flags`를 0으로 설정 시 이 시스템 호출이 재부팅 때까지 항상 같은 값을 반환한다고 보장된다. 즉, 같은 인자로 하는 이후 호출이 같은 결과를 낳는다. 따라서 `flags`를 0으로 설정 시 첫 번째 `membarrier()` 호출에서만 오류 처리가 필요하다.

## ERRORS

`EINVAL`
:   `cmd`가 유효하지 않거나, `flags`가 0이 아니거나, CPU 매개변수 `nohz_full`이 설정되어서 `MEMBARRIER_CMD_GLOBAL` 명령이 비활성화되어 있거나, `MEMBARRIER_CMD_PRIVATE_EXPEDITED_SYNC_CORE` 및 `MEMBARRIER_CMD_REGISTER_PRIVATE_EXPEDITED_SYNC_CORE` 명령이 아키텍처에 구현되어 있지 않다.

`ENOSYS`
:   `membarrier()` 시스템 호출이 이 커널에 구현되어 있지 않다.

`EPERM`
:   현재 프로세스가 프로세스별 신속 처리 명령 사용에 앞서 신고를 하지 않았다.

## VERSIONS

리눅스 4.3에서 `membarrier()` 시스템 호출이 추가되었다.

## CONFORMING TO

`membarrier()`는 리눅스 전용이다.

## NOTES

메모리 모델이 약한 순서(weakly-ordered)인 아키텍처에서 인스트럭션 세트에 메모리 배리어 인스트럭션이 포함되어 있다. 다른 코어 상의 대응하는 배리어에 대해서 배리어 전 메모리 접근과 배리어 후 메모리 접근의 순서를 지켜 준다. 예를 들어 저장 fence로 순서를 유지하는 저장들에 대해서 적재 fence가 펜스 전 적재와 펜스 후 적재의 순서를 유지할 수 있다.

프로그램 순서란 프로그램 어셈블리 코드에서 인스트럭션이 배치되는 순서이다.

`membarrier()`가 유용할 수 있는 경우로 Read-Copy-Update 라이브러리나 가비지 컬렉터 구현 등이 있다.

## EXAMPLE

"`fast_path()`"를 매우 자주 실행하고 "`slow_path()`"를 드물게 실행하는 다중 스레드 응용을 가정할 때 다음과 같은 (x86용) 코드를 `membarrier()`를 쓰도록 변형할 수 있다.

```c
#include <stdlib.h>

static volatile int a, b;

static void
fast_path(int *read_b)
{
    a = 1;
    asm volatile ("mfence" : : : "memory");
    *read_b = b;
}

static void
slow_path(int *read_a)
{
    b = 1;
    asm volatile ("mfence" : : : "memory");
    *read_a = a;
}

int
main(int argc, char **argv)
{
    int read_a, read_b;

    /*
     * 실제 응용에서는 fast_path()와 slow_path()를
     * 다른 스레드에서 호출하게 됨. 예시 프로그램을
     * 짧게 만들기 위해 main()에서 호출함.
     */

    slow_path(&read_a);
    fast_path(&read_b);

    /*
     * read_b == 0이면 read_a == 1이고
     * read_a == 0이면 read_b == 1.
     */

    if (read_b == 0 && read_a == 0)
        abort();

    exit(EXIT_SUCCESS);
}
```

위 코드를 `membarrier()`를 쓰도록 변형하면 다음이 된다.

```c
#define _GNU_SOURCE
#include <stdlib.h>
#include <stdio.h>
#include <unistd.h>
#include <sys/syscall.h>
#include <linux/membarrier.h>

static volatile int a, b;

static int
membarrier(int cmd, int flags)
{
    return syscall(__NR_membarrier, cmd, flags);
}

static int
init_membarrier(void)
{
    int ret;

    /* membarrier() 지원하는지 확인 */

    ret = membarrier(MEMBARRIER_CMD_QUERY, 0);
    if (ret < 0) {
        perror("membarrier");
        return -1;
    }

    if (!(ret & MEMBARRIER_CMD_GLOBAL)) {
        fprintf(stderr,
            "membarrier does not support MEMBARRIER_CMD_GLOBAL\n");
        return -1;
    }

    return 0;
}

static void
fast_path(int *read_b)
{
    a = 1;
    asm volatile ("" : : : "memory");
    *read_b = b;
}

static void
slow_path(int *read_a)
{
    b = 1;
    membarrier(MEMBARRIER_CMD_GLOBAL, 0);
    *read_a = a;
}

int
main(int argc, char **argv)
{
    int read_a, read_b;

    if (init_membarrier())
        exit(EXIT_FAILURE);

    /*
     * 실제 응용에서는 fast_path()와 slow_path()를
     * 다른 스레드에서 호출하게 됨. 예시 프로그램을
     * 짧게 만들기 위해 main()에서 호출함.
     */

    slow_path(&read_a);
    fast_path(&read_b);

    /*
     * read_b == 0이면 read_a == 1이고
     * read_a == 0이면 read_b == 1.
     */

    if (read_b == 0 && read_a == 0)
        abort();

    exit(EXIT_SUCCESS);
}
```

----

2018-04-30
