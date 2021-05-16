## NAME

asctime, ctime, gmtime, localtime, mktime, asctime_r, ctime_r, gmtime_r, localtime_r - 날짜와 시간을 분할 시간이나 ASCII로 변환하기

## SYNOPSIS

```c
#include <time.h>

char *asctime(const struct tm *tm);
char *asctime_r(const struct tm *restrict tm, char *restrict buf);

char *ctime(const time_t *timep);
char *ctime_r(const time_t *restrict timep, char *restrict buf);

struct tm *gmtime(const time_t *timep);
struct tm *gmtime_r(const time_t *restrict timep,
                    struct tm *restrict result);

struct tm *localtime(const time_t *timep);
struct tm *localtime_r(const time_t *restrict timep,
                    struct tm *restrict result);

time_t mktime(struct tm *tm);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`asctime_r()`, `ctime_r()`, `gmtime_r()`, `localtime_r()`:
:   `_POSIX_C_SOURCE`<br>
    `    || /* glibc <= 2.19: */ _BSD_SOURCE || _SVID_SOURCE`

## DESCRIPTION

`ctime()`, `gmtime()`, `localtime()` 함수는 모두 달력 시간을 나타내는 데이터 타입인 `time_t` 인자를 받는다. 절대 시간 값으로 해석할 때 그 값은 에포크, 즉 1970-01-01 00:00:00 +0000 (UTC) 이후로 지난 초 수를 나타낸다.

`asctime()` 및 `mktime()` 함수는 모두 분할 시간, 즉 년, 월, 일 등으로 나눠서 표현한 시간 인자를 받는다.

분할 시간은 `tm` 구조체에 저장하는데, 그 구조체는 `<time.h>`에 다음처럼 정의돼 있다.

```c
struct tm {
    int tm_sec;    /* 초 (0-60) */
    int tm_min;    /* 분 (0-59) */
    int tm_hour;   /* 시간 (0-23) */
    int tm_mday;   /* 월 중 날 번호 (1-31) */
    int tm_mon;    /* 월 (0-11) */
    int tm_year;   /* 년 - 1900 */
    int tm_wday;   /* 주 중 날 번호 (0-6, 일요일 = 0) */
    int tm_yday;   /* 연 중 날 번호 (0-365, 1월 1일 = 0) */
    int tm_isdst;  /* 일광 절약 시간 */
};
```

`tm` 구조체의 멤버는 다음과 같다.

`tm_sec`
:   해당 분 이후의 초 수. 보통은 0에서 59까지 범위지만 윤초를 위해 60까지도 가능하다.

`tm_min`
:   해당 시 이후의 분 수. 0에서 59까지 범위.

`tm_hour`
:   자정 이후의 시간 수. 0에서 23까지 범위.

`tm_mday`
:   월 중 날짜. 1에서 31까지 범위.

`tm_mon`
:   1월 이후의 달 수. 0에서 11까지 범위.

`tm_year`
:   1900년 이후의 년 수.

`tm_wday`
:   일요일 이후의 날 수. 0에서 6까지 범위.

`tm_yday`
:   1월 1일 이후의 날 수. 0에서 365까지 범위.

`tm_isdst`
:   기술한 시간에 일광 절약 시간이 시행 중인지 여부를 나타내는 플래그. 일광 절약 시간이 시행 중이면 값이 양수이고, 아니면 0이며, 가용 정보가 없으면 음수이다.

`ctime(t)` 호출은 `asctime(localtime(t))`와 동등하다. 달력 시간 `t`를 다음 형태의 널 종료 문자열로 바꾼다.

```c
"Wed Jun 30 21:49:08 1993\n"
```

요일 이름 줄임말은 "Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat"이다. 월 이름 줄임말은 "Jan", "Feb", "Mar", "Apr", "May", "Jun", "Jul", "Aug", "Sep", "Oct", "Nov", "Dec"이다. 반환 값은 정적으로 할당된 문자열을 가리키며 이어지는 날짜 및 시간 함수 호출에서 그 문자열을 덮어 쓸 수도 있다. 함수에서는 또한 외부 변수 `tzname`, `timezone`, `daylight`(<tt>[[tzset(3)]]</tt> 참고)에 현재 시간대에 대한 정보를 설정한다. 재진입 가능 버전인 `ctime_r()`에서는 같은 동작을 하되 사용자가 제공한 버퍼에 문자열을 저장하는데, 그 버퍼에는 최소 26바이트의 공간이 있어야 한다. `tzname`, `timezone`, `daylight`은 설정하지 않을 수 있다.

`gmtime()` 함수는 달력 시간 `timep`를 협정 세계시(UTC)로 나타낸 분할 시간 표현으로 변환한다. 연도를 정수로 나타낼 수 없으면 NULL을 반환할 수도 있다. 반환 값은 정적으로 할당된 구조체를 가리키며 이어지는 날짜 밎 시간 함수 호출에서 그 구조체를 덮어 쓸 수도 있다. `gmtime_r()` 함수는 같은 동작을 하되 사용자가 제공한 구조체에 데이터를 저장한다.

`localtime()` 함수는 달력 시간 `timep`를 사용자 지정 시간대 기준으로 나타낸 분할 시간 표현으로 변환한다. 이 함수는 <tt>[[tzset(3)]]</tt>을 호출한 것처럼 작용한다. 즉 외부 변수 `tzname`에 현재 시간대에 대한 정보를 설정하고, `timezone`에 협정 세계시(UTC)와 지역 표준시의 초 단위 차이를 설정하며, 그 해의 일부 시기 동안 일광 절약 시간 규칙이 적용되는 경우 `daylight`에 0 아닌 값을 설정한다. 반환 값은 정적으로 할당된 구조체를 가리키며 이어지는 날짜 및 시간 함수 호출에서 그 구조체를 덮어 쓸 수도 있다. `localtime_r()` 함수는 같은 동작을 하되 사용자가 제공한 구조체에 데이터를 저장한다. `tzname`, `timezone`, `daylight`은 설정하지 않을 수 있다.

`asctime()` 함수는 분할 시간 값 `tm`을 `ctime()`과 같은 형식의 널 종료 문자열로 변환한다. 반환 값은 정적으로 할당된 문자열을 가리키며 이어지는 날짜 및 시간 함수 호출에서 그 문자열을 덮어 쓸 수도 있다. `asctime_r()` 함수는 같은 동작을 하되 사용자가 제공한 버퍼에 문자열을 저장하는데, 그 버퍼에는 최소 26바이트의 공간이 있어야 한다.

`mktime()` 함수는 지역 시간으로 나타낸 분할 시간 구조체를 달력 시간 표현으로 변환한다. 호출자가 `tm_wday` 및 `tm_yday` 필드로 제공한 값은 무시한다. `tm_isdst` 필드에 지정한 값은 그 `tm` 구조체로 제공하는 시간에 일광 절약 시간(DST)이 시행 중인지 여부를 `mktime()`에 알려 준다. 양수 값은 DST가 시행 중이라는 뜻이고, 0은 DST가 시행 중이 아니라는 뜻이다. 그리고 음수 값은 지정한 시간에 DST가 시행 중인지 여부를 `mktime()`에서 (시간대 정보와 시스템 데이터베이스를 이용해) 알아내려고 해야 한다는 뜻이다.

`mktime()` 함수에서 `tm` 구조체의 필드들을 변경한다. `tm_wday` 및 `tm_yday`를 다른 필드들의 내용으로 알아낸 값으로 설정한다. 구조체 멤버가 유효 구간 밖에 있으면 정규화한다. (그래서 가령 10월 40일은 11월 9일로 바뀐다.) 그리고 `tm_isdst`를 (원래 값과 상관없이) 지정한 시간에 DST가 시행 중인지 여부를 나타내도록 양수 값 또는 0으로 설정한다. 또한 `mktime()`에서는 외부 변수 `tzname`을 현재 시간대에 대한 정보로 설정한다.

지정한 분할 시간을 달력 시간(에포크 이후 초)으로 나타낼 수 없는 경우 `mktime()`은 `(time_t) -1`을 반환하며 분할 시간 구조체의 멤버들을 변경하지 않는다.

## RETURN VALUE

성공 시 `gmtime()` 및 `localtime()`은 `struct tm` 포인터를 반환한다.

성공 시 `gmtime_r()` 및 `localtime_r()`은 `result`가 가리키는 구조체의 주소를 반환한다.

성공 시 `asctime()` 및 `ctime()`은 문자열의 포인터를 반환한다.

성공 시 `asctime_r()` 및 `ctime_r()`은 `buf`가 가리키는 문자열의 포인터를 반환한다.

성공 시 `mktime()`은 `time_t` 타입 값으로 표현한 달력 시간(에포크 이후 초 수)을 반환한다.

오류 시 `mktime()`은 `(time_t) -1` 값을 반환한다. 나머지 함수들은 오류 시 NULL을 반환한다. 오류 시 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EOVERFLOW`
:   결과를 표현할 수 없다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `asctime()` | 스레드 안전성 | MT-Unsafe race:asctime locale |
| `asctime_r()` | 스레드 안전성 | MT-Safe locale |
| `ctime()` | 스레드 안전성 | MT-Unsafe race:tmbuf<br>race:asctime env locale |
| `ctime_r()`,<br>`gmtime_r()`,<br>`localtime_r()`,<br>`mktime()` | 스레드 안전성 | MT-Safe env locale |
| `gmtime()`, `localtime()` | 스레드 안전성 | MT-Unsafe race:tmbuf env locale |

## CONFORMING TO

POSIX.1-2001. C89 및 C99에서 `asctime()`, `ctime()`, `gmtime()`, `localtime()`, `mktime()`을 명세하고 있다. POSIX.1-2008에서 `asctime()`, `asctime_r()`, `ctime()`, `ctime_r()`을 구식으로 표시하고 대신 <tt>[[strftime(3)]]</tt>을 쓰기를 권하고 있다.

POSIX에서는 `ctime_r()`의 매개변수가 `restrict`여야 한다고 명세하고 있지 않다. glibc 한정이다.

## NOTES

네 함수 `asctime()`, `ctime()`, `gmtime()`, `localtime()`은 정적 데이터의 포인터를 반환하므로 스레드 안전이 아니다. 스레드 안전 버전인 `asctime_r()`, `ctime_r()`, `gmtime_r()`, `localtime_r()`을 SUSv2에서 명세한다.

POSIX.1-2001: "`asctime()`, `ctime()`, `gmtime()`, `localtime()` 함수는 두 가지 정적 객체, 즉 분할 시간 구조체와 `char` 타입 배열 중 하나로 값을 반환해야 한다. 이 함수들 중 어느 것을 실행하든 다른 어느 함수에서 그 객체들 중 어느 것으로든 반환한 정보를 덮어 쓸 수 있다." glibc 구현에서도 그럴 수 있다.

glibc를 포함한 여러 구현체에서는 `tm_mday`의 0 값을 전달 마지막 날을 뜻하는 것으로 해석한다.

`<time.h>`를 포함하기 전에 `_BSD_SOURCE`가 설정돼 있으면 glibc 버전 `struct tm`에는 다음 필드가 추가로 정의된다.

```c
const char *tm_zone;      /* 시간대 축약명 */
```

이는 4.3BSD-Reno에 존재하는 BSD 확장이다.

POSIX.1-2001에 따르면 `localtime()`은 <tt>[[tzset(3)]]</tt>이 호출된 것처럼 동작해야 하는 반면 `localtime_r()`에는 그런 요구 사항이 없다. 이식 가능한 코드에서는 `localtime_r()`에 앞서 <tt>[[tzset(3)]]</tt>을 호출하는 게 좋다.

## SEE ALSO

`date(1)`, <tt>[[gettimeofday(2)]]</tt>, <tt>[[time(2)]]</tt>, <tt>[[utime(2)]]</tt>, <tt>[[clock(3)]]</tt>, <tt>[[difftime(3)]]</tt>, <tt>[[strftime(3)]]</tt>, <tt>[[strptime(3)]]</tt>, <tt>[[timegm(3)]]</tt>, <tt>[[tzset(3)]]</tt>, <tt>[[time(7)]]</tt>

----

2021-03-22
