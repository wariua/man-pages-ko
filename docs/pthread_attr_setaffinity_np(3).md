## NAME

pthread_attr_setaffinity_np, pthread_attr_getaffinity_np - 스레드 속성 객체의 CPU 친화성 속성 설정하기/얻기

## SYNOPSIS

```c
#define _GNU_SOURCE             /* feature_test_macros(7) 참고 */
#include <pthread.h>

int pthread_attr_setaffinity_np(pthread_attr_t *attr,
                   size_t cpusetsize, const cpu_set_t *cpuset);
int pthread_attr_getaffinity_np(const pthread_attr_t *attr,
                   size_t cpusetsize, cpu_set_t *cpuset);
```

`-pthread`로 컴파일 및 링크.

## DESCRIPTION

`pthread_attr_setaffinity_np()` 함수는 `attr`이 가리키는 스레드 속성 객체의 CPU 친화성 마스크 속성을 `cpuset`에 지정한 값으로 설정한다. 이 속성은 스레드 속성 객체 `attr`을 이용해 생성하는 스레드의 CPU 친화성 마스크를 결정한다.

`pthread_attr_getaffinity_np()` 함수는 `attr`이 가리키는 스레드 속성 객체의 CPU 친화성 마스크를 `cpuset`이 가리키는 버퍼로 반환한다.

`cpusetsize` 인자는 `cpuset`이 가리키는 버퍼의 (바이트 단위) 길이이다. 보통은 이 인자를 `sizeof(cpu_set_t)`로 지정할 것이다.

CPU 친화성 마스크에 대한 더 자세한 내용은 <tt>[[sched_setaffinity(2)]]</tt>를 보라. CPU 세트 조작과 검사에 사용할 수 있는 매크로들에 대한 설명은 <tt>[[CPU_SET(3)]]</tt>을 보라.

## RETURN VALUE

성공 시 이 함수들은 0을 반환한다. 오류 시 0 아닌 오류 번호를 반환한다.

## ERRORS

`EINVAL`
:   (`pthread_attr_setaffinity_np()`) `cpuset`으로 커널에서 지원하는 집합을 벗어나는 CPU를 지정했다. (커널 구성 옵션 `CONFIG_NR_CPUS`가 CPU 집합 표현에 쓰는 커널 데이터 타입이 지원하는 집합의 범위를 규정한다.)

`EINVAL`
:   (`pthread_attr_getaffinity_np()`) `attr`이 가리키는 스레드 속성 객체의 친화성 마스크에서 어느 CPU가 `cpusetsize`로 지정한 범위 밖에 있다. (즉, `cpuset`이/`cpusetsize`가 너무 작다.)

`ENOMEM`
:   (`pthread_attr_setaffinity_np()`) 메모리를 할당하지 못했다.

## VERSIONS

glibc 버전 2.3.4부터 이 함수들을 제공한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `pthread_attr_setaffinity_np()`,<br>`pthread_attr_getaffinity_np()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

이 함수들은 비표준 GNU 확장이다. 그래서 이름 뒤에 "\_np"(nonportable: 이식성 없음)가 붙어 있다.

## NOTES

glibc 2.3.3에 한해 제공됐던 버전에 `cpusetsize` 인자가 없었다. 기반 시스템 호출에게 주는 CPU 세트 크기가 항상 `sizeof(cpu_set_t)`였다.

## SEE ALSO

<tt>[[sched_setaffinity(2)]]</tt>, <tt>[[pthread_attr_init(3)]]</tt>, <tt>[[pthread_setaffinity_np(3)]]</tt>, <tt>[[cpuset(3)]]</tt>, <tt>[[pthreads(7)]]</tt>

----

2021-03-22
