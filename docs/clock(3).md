## NAME

clock - 프로세서 시간 알아내기

## SYNOPSIS

```c
#include <time.h>

clock_t clock(void);
```

## DESCRIPTION

`clock()` 함수는 프로그램이 이용한 프로세서 시간의 근사치를 반환한다.

## RETURN VALUE

반환되는 값은 지금까지 쓴 CPU 시간을 `clock_t`로 나타낸 것이다. 이용한 초 수를 얻으려면 `CLOCKS_PER_SEC`로 나누면 된다. 쓴 프로세서 시간을 알아낼 수 없거나 그 값을 표현할 수 없으면 함수가 `(clock_t) -1` 값을 반환한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `clock()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, C89, C99. XSI에서는 실제 해상도와 상관없이 `CLOCKS_PER_SEC`가 1000000과 같기를 요구한다.

## NOTES

C 표준에서는 프로그램 시작 때 임의 값을 반환할 수 있다고 허용한다. 따라서 최대한의 호환성을 얻으려면 프로그램 시작 때의 `clock()` 호출 반환 값만큼 빼 줘야 한다.

시간이 넘쳐서 되돌아갈 수 있다는 점에 유의하라. 어느 32비트 시스템에서 `CLOCKS_PER_SEC`가 1000000이라면 대략 72분마다 이 함수가 같은 값을 반환하게 된다.

다른 몇몇 구현들에서는 <tt>[[wait(2)]]</tt>(또는 다른 wait 계열 호출)을 통해 상태를 수집한 자식들의 시간까지 `clock()` 반환 값에 포함된다. 리눅스에서는 대기한 자식들의 시간을 `clock()` 반환 값에 포함시키지 않는다. 호출자 및 자식들에 대해 (구별된) 정보를 명확하게 반환하는 <tt>[[times(2)]]</tt> 함수가 바람직할 수 있다.

glibc 2.17 및 이전에서는 <tt>[[times(2)]]</tt> 상에서 `clock()`을 구현했다. 정확도를 높이기 위해 glibc 2.18부터는 <tt>[[clock_gettime(2)]]</tt> 상에서 (`CLOCK_PROCESS_CPUTIME_ID` 클럭을 써서) 구현한다.

## SEE ALSO

<tt>[[clock_gettime(2)]]</tt>, <tt>[[getrusage(2)]]</tt>, <tt>[[times(2)]]</tt>

----

2021-03-22
