## NAME

ftime - 날짜와 시간 반환

## SYSNOPSIS

```c
#include <sys/timeb.h>

int ftime(struct timeb *tp);
```

## DESCRIPTION

**주의**: GNU C 라이브러리에서 이 함수를 더이상 제공하지 않는다. 대신 <tt>[[clock_gettime(2)]]</tt>을 사용하라.

이 함수는 에포크 1970-01-01 00:00:00 +0000 (UTC) 이후의 초와 밀리초로 현재 시간을 반환한다. 다음처럼 선언돼 있는 `tp`로 시간을 반환한다.

```c
struct timeb {
    time_t         time;
    unsigned short millitm;
    short          timezone;
    short          dstflag;
};
```

여기서 `time`은 에포크 이후의 초 수이고 `millitm`은 에포크 이후 `time` 초 이후의 밀리초 수이다. `timezone` 필드는 그리니치 서쪽 분 단위로 잰 지역 시간대이다. (음수 값은 그리니치 동쪽을 나타낸다.) `dstflag` 필드는 플래그이며, 0이 아니면 그 해의 적절한 시기 동안 지역적으로 일광 절약 시간이 적용됨을 나타낸다.

POSIX.1-2001에서는 `timezone` 및 `dstflag` 필드의 내용물이 명세돼 있지 않다고 말한다. 그 둘에 의지하는 것을 피해야 한다.

## RETURN VALUE

이 함수는 항상 0을 반환한다. (POSIX.1-2001에서 -1 오류 반환을 명세하고 있으며 일부 시스템에도 기록돼 있다.)

## VERSIONS

glibc 2.33부터 `ftime()` 함수와 `<sys/timeb.h>` 헤더가 제거되었다. 구식 바이너리들을 지원하기 위해 glibc 2.32 및 이전 버전에 링크 된 응용을 위한 호환 심볼을 glibc에서 계속 제공한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `ftime()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

4.2BSD, POSIX.1-2001. POSIX.1-2008에서 `ftime()` 명세를 제거했다.

이 함수는 구식화되었다. 쓰지 말아야 한다. 초 단위 시간으로 충분하다면 <tt>[[time(2)]]</tt>을 쓸 수 있고 <tt>[[gettimeofday(2)]]</tt>에서는 마이크로초까지 나온다. <tt>[[clock_gettime(2)]]</tt>에서는 나노초까지 나오지만 널리 사용 가능하지는 않다.

## BUGS

초기 glibc2에 버그가 있어서 `millitm` 필드에 0을 반환한다. glibc 2.1.1에서 다시 올바르게 됐다.

## SEE ALSO

<tt>[[gettimeofday(2)]]</tt>, <tt>[[time(2)]]</tt>

----

2021-03-22
