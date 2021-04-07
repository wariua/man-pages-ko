## NAME

get_nprocs, get_nprocs_conf - 프로세서 개수 얻기

## SYNOPSIS

```c
#include <sys/sysinfo.h>

int get_nprocs(void);
int get_nprocs_conf(void);
```

## DESCRIPTION

`get_nprocs_conf()` 함수는 운영 체제가 구성한 프로세서들의 수를 반환한다.

`get_nprocs()` 함수는 현재 시스템에서 사용 가능한 프로세서들의 수를 반환한다. `get_nprocs_conf()`가 반환하는 수보다 작을 수도 있는데, (가령 핫플러그 가능 시스템에서) 프로세서가 오프라인일 수도 있기 때문이다.

## RETURN VALUE

DESCRIPTION의 내용 대로.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `get_nprocs()`,<br>`get_nprocs_conf()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

이 함수들은 GNU 확장이다.

## NOTES

이 함수들의 현재 구현은 동작 비용이 꽤 크다. 호출 때마다 `/sys` 파일 시스템의 파일을 열어서 파싱 하기 때문이다.

다음 <tt>[[sysconf(3)]]</tt> 호출은 이 페이지에 적힌 함수들을 이용해서 동일 정보를 반환한다.

```c
np = sysconf(_SC_NPROCESSORS_CONF);     /* 구성된 프로세서들 */
np = sysconf(_SC_NPROCESSORS_ONLN);     /* 사용 가능 프로세서들 */
```

## EXAMPLE

다음 예는 `get_nprocs()` 및 `get_nprocs_conf()` 사용 방식을 보여 준다.

```c
#include <stdlib.h>
#include <stdio.h>
#include <sys/sysinfo.h>

int
main(int argc, char *argv[])
{
    printf("This system has %d processors configured and "
            "%d processors available.\n",
            get_nprocs_conf(), get_nprocs());
    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

`nproc(1)`

----

2019-03-06
