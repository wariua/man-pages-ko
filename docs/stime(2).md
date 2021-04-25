## NAME

stime - 시간 설정하기

## SYNOPSIS

```c
#include <time.h>

int stime(const time_t *t);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`stime()`:
:   glibc 2.19부터:
    :   `_DEFAULT_SOURCE`

    glibc 2.19 및 이전:
    :   `_SVID_SOURCE`

## DESCRIPTION

**주의**: 이 함수는 제거 예정이다. 대신 <tt>[[clock_settime(2)]]</tt>을 사용하라.

`stime()`은 시스템의 시간 및 날짜 개념을 설정한다. `t`가 가리키는 시간은 에포크 1970-01-01 00:00:00 +0000 (UTC) 이후의 초로 측정한다. 수퍼유저만 `stime()`을 실행할 수 있다.

## RETURN VALUE

성공 시 0을 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EFAULT`
:   사용자 공간의 정보를 얻어 오는 과정에 오류.

`EPERM`
:   호출 프로세스에게 충분한 특권이 없다. 리눅스에서는 `CAP_SYS_TIME` 특권이 필요하다.

## CONFORMING TO

SVr4.

## NOTES

glibc 2.31부터 새로 링크하는 응용에서 이 함수를 더이상 사용할 수 없으며 `<time.h>`에 더이상 함수가 선언돼 있지 않다.

## SEE ALSO

`date(1)`, <tt>[[settimeofday(2)]]</tt>, <tt>[[capabilities(7)]]</tt>

----

2021-03-22
