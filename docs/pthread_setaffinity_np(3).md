## NAME

pthread_setaffinity_np, pthread_getaffinity_np - 스레드의 CPU 친화성 설정하기/얻기

## SYNOPSIS

```c
#define _GNU_SOURCE             /* feature_test_macros(7) 참고 */
#include <pthread.h>

int pthread_setaffinity_np(pthread_t thread, size_t cpusetsize,
                           const cpu_set_t *cpuset);
int pthread_getaffinity_np(pthread_t thread, size_t cpusetsize,
                           cpu_set_t *cpuset);
```

`-pthread`로 컴파일 및 링크.

## DESCRIPTION

`pthread_setaffinity_np()` 함수는 스레드 `thread`의 CPU 친화성 마스크를 `cpuset`이 가리키는 CPU 세트로 설정한다. 호출이 성공하고 스레드가 현재 `cpuset`의 한 CPU에서 돌고 있지 않으면 그 CPU들 중 하나로 스레드를 옮긴다.

`pthread_getaffinity_np()` 함수는 스레드 `thread`의 CPU 친화성 마스크를 `cpuset`이 가리키는 버퍼로 반환한다.

CPU 친화성 마스크에 대한 더 자세한 내용은 <tt>[[sched_setaffinity(2)]]</tt>를 보라. CPU 세트 조작과 검사에 사용할 수 있는 매크로들에 대한 설명은 <tt>[[CPU_SET(3)]]</tt>을 보라.

`cpusetsize` 인자는 `cpuset`이 가리키는 버퍼의 (바이트 단위) 길이이다. 보통은 이 인자를 `sizeof(cpu_set_t)`로 지정할 것이다. (<tt>[[CPU_SET(3)]]</tt>에서 기술하는 CPU 세트 동적 할당을 위한 매크로들을 사용하고 있다면 어떤 다른 값일 수도 있다.)

## RETURN VALUE

성공 시 이 함수들은 0을 반환한다. 오류 시 0 아닌 오류 번호를 반환한다.

## ERRORS

`EFAULT`
:   제공한 메모리 주소가 유효하지 않다.

`EINVAL`
:   (`pthread_setaffinity_np()`) 친화성 비트 마스크 `mask`가 현재 물리적으로 시스템 상에 있으면서 <tt>[[cpuset(7)]]</tt>에서 기술하는 "cpuset" 메커니즘으로 부과할 수 있는 제약에 따라 스레드에게 허용된 프로세서를 하나도 포함하고 있지 않다.

`EINVAL`
:   (`pthread_setaffinity_np()`) `cpuset`으로 커널에서 지원하는 집합을 벗어나는 CPU를 지정했다. (커널 구성 옵션 `CONFIG_NR_CPUS`가 CPU 집합 표현에 쓰는 커널 데이터 타입이 지원하는 집합의 범위를 규정한다.)

`EINVAL`
:   (`pthread_getaffinity_np()`) `cpusetsize`가 커널에서 쓰는 친화성 마스크의 크기보다 작다.

`ESRCH`
:   ID가 `thread`인 스레드를 찾을 수 없다.

## VERSIONS

glibc 버전 2.3.4부터 이 함수들을 제공한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `pthread_setaffinity_np()`,<br>`pthread_getaffinity_np()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

이 함수들은 비표준 GNU 확장이다. 그래서 이름 뒤에 "_np"(nonportable: 이식성 없음)가 붙어 있다.

## NOTES

`pthread_setaffinity_np()` 호출 후에 스레드가 실제로 돌게 될 CPU들의 집합은 `cpuset` 인자에 지정한 집합과 시스템에 실제 존재하는 CPU들의 집합의 교집합이다. <tt>[[cpuset(7)]]</tt>에서 기술하는 "cpuset" 메커니즘을 사용하고 있으면 스레드가 도는 CPU 집합을 시스템에서 추가로 제약할 수도 있다. 스레드가 돌게 될 실제 CPU들의 집합에 대한 이 제약들은 커널이 조용히 적용한다.

이 함수들은 <tt>[[sched_setaffinity(2)]]</tt> 및 <tt>[[sched_getaffinity(2)]]</tt> 시스템 호출 위에 구현되어 있다.

glibc 2.3.3에 한해 제공됐던 버전에 `cpusetsize` 인자가 없었다. 기반 시스템 호출에게 주는 CPU 세트 크기가 항상 `sizeof(cpu_set_t)`였다.

<tt>[[pthread_create(3)]]</tt>로 생성한 신규 스레드는 자기를 생성한 스레드의 CPU 친화성 마스크 사본을 물려받는다.

## EXAMPLE

다음 프로그램에서 주 스레드는 `pthread_setaffinity_np()`를 이용해 CPU 0에서 7까지를 (시스템에서 모두 사용 가능하지는 않을 수도 있음) 포함하도록 자기 CPU 친화성 마스크를 설정하고서 `pthread_getaffinity_np()`를 호출해 그렇게 바꾼 스레드의 CPU 친화성 마스크를 확인한다.

```c
#define _GNU_SOURCE
#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>

#define handle_error_en(en, msg) \
        do { errno = en; perror(msg); exit(EXIT_FAILURE); } while (0)

int
main(int argc, char *argv[])
{
    int s, j;
    cpu_set_t cpuset;
    pthread_t thread;

    thread = pthread_self();

    /* CPU 0에서 7까지 포함하도록 친화성 마스크 설정 */

    CPU_ZERO(&cpuset);
    for (j = 0; j < 8; j++)
        CPU_SET(j, &cpuset);

    s = pthread_setaffinity_np(thread, sizeof(cpu_set_t), &cpuset);
    if (s != 0)
        handle_error_en(s, "pthread_setaffinity_np");

    /* 스레드에 친화성 마스크가 실제 부여되었는지 확인 */

    s = pthread_getaffinity_np(thread, sizeof(cpu_set_t), &cpuset);
    if (s != 0)
        handle_error_en(s, "pthread_getaffinity_np");

    printf("Set returned by pthread_getaffinity_np() contained:\n");
    for (j = 0; j < CPU_SETSIZE; j++)
        if (CPU_ISSET(j, &cpuset))
            printf("    CPU %d\n", j);

    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[sched_setaffinity(2)]]</tt>, <tt>[[CPU_SET(3)]]</tt>, <tt>[[pthread_attr_setaffinity_np(3)]]</tt>, <tt>[[pthread_self(3)]]</tt>, <tt>[[sched_getcpu(3)]]</tt>, <tt>[[cpuset(7)]]</tt>, <tt>[[pthreads(7)]]</tt>, <tt>[[sched(7)]]</tt>

----

2019-03-06
