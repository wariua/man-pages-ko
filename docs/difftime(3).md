## NAME

difftime - 시간 차이 계산하기

## SYNOPSIS

```c
#include <time.h>

double difftime(time_t time1, time_t time0);
```

## DESCRIPTION

`difftime()` 함수는 시간 `time0`과 시간 `time1` 사이에 경과한 초 수를 `double`로 반환한다. 각 시간은 달력 시간으로 지정한다. 즉, 그 값은 1970-01-01 00:00:00 +0000 (UTC) 에포크를 기준으로 한 (초 단위) 측정치다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `difftime()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, C89, C99, SVr4, 4.3BSD.

## NOTES

POSIX 시스템에서 `time_t`는 산술 타입이고, 그래서 뺄셈 오버플로를 신경쓰지 않아도 되는 경우에는 그냥 다음처럼 정의할 수도 있다.

```c
#define difftime(t1,t0) (double)(t1 - t0)
```

## SEE ALSO

`date(1)`, <tt>[[gettimeofday(2)]]</tt>, <tt>[[time(2)]]</tt>, <tt>[[ctime(3)]]</tt>, <tt>[[gmtime(3)]]</tt>, <tt>[[localtime(3)]]</tt>

----

2021-03-22
