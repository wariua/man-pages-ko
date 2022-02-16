## NAME

CPU_SET, CPU_CLR, CPU_ISSET, CPU_ZERO, CPU_COUNT, CPU_AND, CPU_OR, CPU_XOR, CPU_EQUAL, CPU_ALLOC, CPU_ALLOC_SIZE, CPU_FREE, CPU_SET_S, CPU_CLR_S, CPU_ISSET_S, CPU_ZERO_S, CPU_COUNT_S, CPU_AND_S, CPU_OR_S, CPU_XOR_S, CPU_EQUAL_S - CPU 세트 조작을 위한 매크로

## SYNOPSIS

```c
#define _GNU_SOURCE             /* feature_test_macros(7) 참고 */
#include <sched.h>

void CPU_ZERO(cpu_set_t *set);

void CPU_SET(int cpu, cpu_set_t *set);
void CPU_CLR(int cpu, cpu_set_t *set);
int  CPU_ISSET(int cpu, cpu_set_t *set);

int  CPU_COUNT(cpu_set_t *set);

void CPU_AND(cpu_set_t *destset,
             cpu_set_t *srcset1, cpu_set_t *srcset2);
void CPU_OR(cpu_set_t *destset,
             cpu_set_t *srcset1, cpu_set_t *srcset2);
void CPU_XOR(cpu_set_t *destset,
             cpu_set_t *srcset1, cpu_set_t *srcset2);

int  CPU_EQUAL(cpu_set_t *set1, cpu_set_t *set2);

cpu_set_t *CPU_ALLOC(int num_cpus);
void CPU_FREE(cpu_set_t *set);
size_t CPU_ALLOC_SIZE(int num_cpus);

void CPU_ZERO_S(size_t setsize, cpu_set_t *set);

void CPU_SET_S(int cpu, size_t setsize, cpu_set_t *set);
void CPU_CLR_S(int cpu, size_t setsize, cpu_set_t *set);
int  CPU_ISSET_S(int cpu, size_t setsize, cpu_set_t *set);

int  CPU_COUNT_S(size_t setsize, cpu_set_t *set);

void CPU_AND_S(size_t setsize, cpu_set_t *destset,
             cpu_set_t *srcset1, cpu_set_t *srcset2);
void CPU_OR_S(size_t setsize, cpu_set_t *destset,
             cpu_set_t *srcset1, cpu_set_t *srcset2);
void CPU_XOR_S(size_t setsize, cpu_set_t *destset,
             cpu_set_t *srcset1, cpu_set_t *srcset2);

int  CPU_EQUAL_S(size_t setsize, cpu_set_t *set1, cpu_set_t *set2);
```

## DESCRIPTION

`cpu_set_t` 데이터 구조는 CPU들의 집합을 표현한다. <tt>[[sched_setaffinity(2)]]</tt> 및 비슷한 인터페이스들에서 CPU 세트를 사용한다.

`cpu_set_t` 데이터 타입은 비트 마스크로 구현되어 있다. 하지만 이 자료 구조를 불투명한 것으로 다루어야 한다. 즉, 모든 CPU 세트 조작이 이 페이지에서 기술하는 매크로들을 통해 이뤄져야 한다.

CPU 세트 `cpu`에 대해 동작하는 다음 매크로들이 있다.

`CPU_ZERO()`
:   `set`을 비워서 어떤 CPU도 담지 않게 만들기.

`CPU_SET()`
:   CPU `cpu`를 `set`에 추가.

`CPU_CLR()`
:   CPU `cpu`를 `set`에서 제거.

`CPU_ISSET()`
:   CPU `cpu`가 `set`에 속하는지 검사.

`CPU_COUNT()`
:   `set` 내의 CPU 개수 반환.

`cpu` 인자를 지정하는 경우 그 인자에 부대 효과가 없어야 한다. 위 매크로들에서 그 인자를 여러 번 평가할 수도 있기 때문이다.

시스템의 첫 번째 CPU가 `cpu` 값 0에 해당하고 다음 CPU가 `cpu` 값 1에 해당하는 식이다. 특정 CPU가 사용 가능한지에 대해, 또는 CPU 세트가 연속인지에 대해 어떤 가정도 해서는 안 된다. CPU가 동적으로 오프라인이 되거나 다른 이유로 빠져 있을 수 있기 때문이다. 상수 `CPU_SETSIZE`(현재 1024)는 `cpu_set_t`에 저장할 수 있는 가장 큰 CPU 번호보다 1만큼 큰 값을 나타낸다.

다음 매크로들은 CPU 세트에 논리 연산을 수행한다.

`CPU_AND()`
:   세트 `srcset1`과 `srcset2`의 교집합을 (두 세트 중 하나일 수도 있는) `destset`에 저장.

`CPU_OR()`
:   세트 `srcset1`과 `srcset2`의 합집합을 (두 세트 중 하나일 수도 있는) `destset`에 저장.

`CPU_XOR()`
:   세트 `srcset1`과 `srcset2`의 XOR을 (두 세트 중 하나일 수도 있는) `destset`에 저장. XOR은 `srcset1`과 `srcset2` 중 한 쪽에만 있는 CPU들의 집합을 뜻한다.

`CPU_EQUAL()`
:   두 CPU 세트가 정확히 같은 CPU들을 담고 있는지 검사.

### 동적 크기 CPU 세트

CPU 세트 크기를 동적으로 정할 수 있어야 하는 (가령 표준 `cpu_set_t` 데이터 타입이 규정하는 것보다 큰 세트를 할당해야 하는) 응용이 있을 수 있기 때문에 요즘 glibc에서는 이를 지원하기 위한 매크로들을 제공한다.

다음 매크로들을 사용해 CPU 세트를 할당하고 해제한다.

`CPU_ALLOC()`
:   0에서 `num_cpus-1`까지 범위의 CPU들을 담을 만큼 큰 CPU 세트를 할당.

`CPU_ALLOC_SIZE()`
:   0에서 `num_cpus-1`까지 범위의 CPU들을 담는 데 필요한 CPU 세트의 바이트 단위 크기를 반환. 아래에서 설명하는 `CPU_*_S()` 매크로들의 `setsize` 인자로 사용할 수 있는 값을 이 매크로가 제공한다.

`CPU_FREE()`
:   앞서 `CPU_ALLOC()`으로 할당한 CPU 세트 해제.

이름이 "\_S"로 끝나는 매크로들은 이름에 접미사 없는 매크로들과 유사하다. 대응하는 매크로와 같은 일을 수행하되 크기가 `setsize` 바이트인 동적 할당 CPU 세트에 대해 동작한다.

## RETURN VALUE

`CPU_ISSET()` 및 `CPU_ISSET_S()`는 `set`에 `cpu`가 설정되어 있으면 0 아닌 값을 반환한다. 아니면 0을 반환한다.

`CPU_COUNT()` 및 `CPU_COUNT_S()`는 `set` 내의 CPU 개수를 반환한다.

`CPU_EQUAL()` 및 `CPU_EQUAL_S()`는 두 CPU 세트가 같으면 0 아닌 값을 반환한다. 아니면 0을 반환한다.

`CPU_ALLOC()`은 성공 시 포인터를, 실패 시 NULL을 반환한다. (오류는 <tt>[[malloc(3)]]</tt>에서와 같다.)

`CPU_ALLOC_SIZE()`는 지정한 크기의 CPU 세트를 저장하는 데 필요한 바이트 수를 반환한다.

다른 함수들은 값을 반환하지 않는다.

## VERSIONS

glibc 2.3.3에서 `CPU_ZERO()`, `CPU_SET()`, `CPU_CLR()`, `CPU_ISSET()` 매크로가 추가되었다.

glibc 2.6에서 `CPU_COUNT()`가 처음 등장했다.

glibc 2.7에서 `CPU_AND()`, `CPU_OR()`, `CPU_XOR()`, `CPU_EQUAL()`, `CPU_ALLOC()`, `CPU_ALLOC_SIZE()`, `CPU_FREE()`, `CPU_ZERO_S()`, `CPU_SET_S()`, `CPU_CLR_S()`, `CPU_ISSET_S()`, `CPU_AND_S()`, `CPU_OR_S()`, `CPU_XOR_S()`, `CPU_EQUAL_S()`가 처음 등장했다.

## CONFORMING TO

이 인터페이스들은 리눅스 전용이다.

## NOTES

CPU 세트를 복제하려면 <tt>[[memcpy(3)]]</tt>를 쓰면 된다.

CPU 세트는 긴 워드 단위로 할당된 비트 마스크이다. 그래서 동적 할당 CPU 세트의 실제 CPU 수는 가장 가까운 `sizeof(unsigned long)`의 배수로 올림한 것이 된다. 응용에서 나머지 비트들의 내용물은 정의되어 있지 않은 것으로 보아야 한다.

이름의 유사성에도 불구하고 상수 `CPU_SETSIZE`는 `cpu_set_t` 데이터 타입 내의 CPU 수를 나타내는 것이고 (그래서 실질적으로 비트 마스크의 비트 개수이고) `CPU_*_S()` 매크로의 `setsize` 인자는 바이트 단위 크기이다.

SYNOPSIS에 나와 있는 인자와 반환 값의 데이터 타입은 각 경우에 무엇을 예상하는지에 대한 힌트이다. 하지만 이 인터페이스들이 매크로로 구현되어 있기 때문에 그 제안을 어긴 경우에 컴파일러가 반드시 모든 타입 오류를 잡아 주지는 않을 것이다.

## BUGS

glibc가 2.8 이전인 32비트 플랫폼에서 `CPU_ALLOC()`이 필요한 크기 두 배로 공간을 할당하며 `CPU_ALLOC_SIZE()`가 그 두 배 값을 반환한다. 이 버그가 프로그램의 동작 결과에 영향을 주지는 않겠지만 메모리가 낭비되며 동적 할당 CPU 세트에 대해 동작하는 매크로들이 덜 효율적으로 동작하게 된다. glibc 2.9에서 이 버그들이 수정되었다.

## EXAMPLES

다음 프로그램은 동적 할당 CPU 세트에 쓰는 몇 가지 매크로의 사용 방식을 보여 준다.

```c
#define _GNU_SOURCE
#include <sched.h>
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <assert.h>

int
main(int argc, char *argv[])
{
    cpu_set_t *cpusetp;
    size_t size;
    int num_cpus;

    if (argc < 2) {
        fprintf(stderr, "Usage: %s <num-cpus>\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    num_cpus = atoi(argv[1]);

    cpusetp = CPU_ALLOC(num_cpus);
    if (cpusetp == NULL) {
        perror("CPU_ALLOC");
        exit(EXIT_FAILURE);
    }

    size = CPU_ALLOC_SIZE(num_cpus);

    CPU_ZERO_S(size, cpusetp);
    for (int cpu = 0; cpu < num_cpus; cpu += 2)
        CPU_SET_S(cpu, size, cpusetp);

    printf("CPU_COUNT() of set:    %d\n", CPU_COUNT_S(size, cpusetp));

    CPU_FREE(cpusetp);
    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[sched_setaffinity(2)]]</tt>, <tt>[[pthread_attr_setaffinity_np(3)]]</tt>, <tt>[[pthread_setaffinity_np(3)]]</tt>, <tt>[[cpuset(7)]]</tt>

----

2021-03-22
