## NAME

timegm, timelocal - gmtime 및 localtime의 반대 동작

## SYNOPSIS

```c
#include <time.h>

time_t timelocal(struct tm *tm);
time_t timegm(struct tm *tm);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`timelocal()`, `timegm()`:
:   glibc 2.19부터:
    :   `_DEFAULT_SOURCE`

    glibc 2.19 및 이전:
    :   `_BSD_SOURCE || _SVID_SOURCE`

## DESCRIPTION

`timelocal()` 및 `timegm()` 함수는 <tt>[[localtime(3)]]</tt> 및 <tt>[[gmtime(3)]]</tt>의 반대이다. 두 함수 모두 분할 시간을 받아서 달력 시간(에포크 1970-01-01 00:00:00 +0000 UTC 이후 초)으로 변환한다. 두 함수의 차이는 `timelocal()`에서는 변환을 할 때 지역 시간대를 계산에 넣는 반면 `timegm()`에서는 입력 값을 협정 세계시(UTC)로 받아들인다는 점이다.

## RETURN VALUE

성공 시 이 함수들은 `time_t` 타입 값으로 표현한 달력 시간(에포크 이후 초)을 반환한다. 오류 시 `(time_t) -1` 값을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EOVERFLOW`
:   결과를 표현할 수 없다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `timelocal()`, `timegm()` | 스레드 안전성 | MT-Safe env locale |

## CONFORMING TO

이 함수들은 비표준 GNU 확장이며 BSD 계열에도 있다. 사용을 피해야 한다.

## NOTES

`timelocal()` 함수는 POSIX 표준 함수인 <tt>[[mktime(3)]]</tt>과 동등하다. 따라서 쓸 이유가 전혀 없다.

## SEE ALSO

<tt>[[gmtime(3)]]</tt>, <tt>[[localtime(3)]]</tt>, <tt>[[mktime(3)]]</tt>, <tt>[[tzset(3)]]</tt>

----

2021-03-22
