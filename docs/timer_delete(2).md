## NAME

timer_delete - POSIX 프로세스별 타이머 삭제하기

## SYNOPSIS

```c
#include <time.h>

int timer_delete(timer_t timerid);
```

`-lrt`로 링크.

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`timer_delete()`:
:   `_POSIX_C_SOURCE >= 199309L`

## DESCRIPTION

`timer_delete()`는 `timerid`에 주어진 ID의 타이머를 삭제한다. 이 호출 시점에 타이머가 장전 상태이면 삭제 전에 해제한다. 삭제된 타이머가 생성한 대기 중 시그널의 처리 방법은 명세되어 있지 않다.

## RETURN VALUE

성공 시 `timer_delete()`는 0을 반환한다. 실패 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EINVAL`
:   `timerid`가 유효한 타이머 ID가 아니다.

## VERSIONS

리눅스 2.6부터 이 시스템 호출이 사용 가능하다.

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## SEE ALSO

<tt>[[clock_gettime(2)]]</tt>, <tt>[[timer_create(2)]]</tt>, <tt>[[timer_getoverrun(2)]]</tt>, <tt>[[timer_settime(2)]]</tt>, <tt>[[time(7)]]</tt>

----

2021-03-22
