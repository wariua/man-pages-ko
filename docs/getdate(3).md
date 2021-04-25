## NAME

getdate, getdate_r - 날짜-시간 문자열을 분할 시간으로 변환하기

## SYNOPSIS

```c
#include <time.h>

struct tm *getdate(const char *string);

extern int getdate_err;

#include <time.h>

int getdate_r(const char *restrict string, struct tm *restrict res);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`getdate()`:
:   `_XOPEN_SOURCE >= 500`

`getdate_r()`:
:   `_GNU_SOURCE`

## DESCRIPTION

`getdate()` 함수는 `string`이 가리키는 버퍼에 담긴 날짜 및 시간 문자열 표현을 분할 시간으로 변환한다. 분할 시간을 `tm` 구조체에 저장해서 그 구조체에 대한 포인터를 함수 결과로 반환한다. 그 `tm` 구조체는 정적 저장 공간에 할당돼 있으므로 이어서 `getdate()`를 또 호출하면 덮어 써지게 된다.

(`format` 인자가 있는) <tt>[[strptime(3)]]</tt>과 달리 `getdate()`에서는 파일에서 얻은 형식들을 사용한다. 그 파일의 전체 경로명을 환경 변수 `DATEMSK`로 받는다. 주어진 입력 문자열에 맞는 파일 내의 첫 행을 변환에 쓴다.

맞는지 확인할 때 대소문자를 무시한다. 패턴이나 변환할 문자열 내의 잉여 공백은 무시한다.

<tt>[[strptime(3)]]</tt>의 변환 지정 항목들이 패턴에 포함될 수 있다. POSIX.1-2001에는 변환 지정 항목 한 가지가 추가로 명세돼 있다.

| | |
| --- | --- |
| `%Z` | 시간대 이름. glibc에는 구현돼 있지 않음. |

`%Z`가 있으면 분할 시간을 담는 구조체를 주어진 시간대의 현재 시간에 해당하는 값들로 초기화 한다. 아니면 구조체를 (<tt>[[localtime(3)]]</tt>을 호출한 것처럼) 현재 지역 시간에 해당하는 분할 시간으로 초기화 한다.

주 중 날 번호만 있는 경우에는 그 날을 오늘이나 이후의 첫 해당 날로 받는다.

월만 있는 (년은 없는) 경우에는 그 달을 금월이나 이후의 첫 해당 달로 받는다.

시분초가 없는 경우에는 현재 시분초를 쓴다.

날짜가 없지만 시간은 아는 경우에는 그 시간을 현재 시간이나 이후의 첫 해당 시간으로 받는다.

`getdate_r()`은 `getdate()`의 재진입 가능 버전을 제공하는 GNU 확장이다. 전역 변수로 오류를 보고하고 정적 버퍼로 분할 시간을 반환하는 대신 함수 반환 값을 통해 오류를 반환하고 `res` 인자가 가리키는 호출자 할당 버퍼로 결과 분할 시간을 반환한다.

## RETURN VALUE

성공 시 `getdate()`는 `struct tm`에 대한 포인터를 반환한다. 아니면 NULL을 반환하며 전역 변수 `getdate_err`를 아래 있는 오류 번호들 중 하나로 설정한다. `errno` 변화에 대해선 명세돼 있지 않다.

성공 시 `getdate_r()`은 0을 반환한다. 오류 시 아래 있는 오류 번호들 중 하나를 반환한다.

## ERRORS

`getdate_err`를 통해서 (`getdate()`), 또는 함수 결과를 통해서 (`getdate_r()`) 다음 오류를 반환한다.

| | |
| --- | --- |
| 1 | `DATEMSK` 환경 변수가 정의돼 있지 않거나 그 값이 빈 문자열이다. |
| 2 | `DATEMSK`에 지정된 템플릿 파일을 읽기용으로 열 수 없다. |
| 3 | 파일 상태 정보를 얻는 데 실패했다. |
| 4 | 템플릿 파일이 정규 파일이 아니다. |
| 5 | 템플릿 파일을 읽는 중 오류를 만났다. |
| 6 | 메모리 할당에 실패했다. (가용 메모리가 충분치 않다.) |
| 7 | 입력에 일치하는 행이 파일에 없다. |
| 8 | 입력 지정 항목이 유효하지 않다. |

## ENVIRONMENT

`DATEMSK`
:   서식 패턴을 담은 파일.

`TZ`, `LC_TIME`
:   <tt>[[strptime(3)]]</tt>에서 쓰는 변수들.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `getdate()` | 스레드 안전성 | MT-Unsafe race:getdate env locale |
| `getdate_r()` | 스레드 안전성 | MT-Safe env locale |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## NOTES

POSIX.1의 <tt>[[strptime(3)]]</tt> 명세에는 `%E` 및 `%O` 수식자를 이용한 변환 지정 항목이 있지만 `getdate()`에는 그런 지정 항목이 없다. glibc에서는 <tt>[[getptime(3)]]</tt>으로 `getdate()`를 구현하고 있으며, 그래서 양쪽에서 정확히 같은 변환들을 지원한다.

## EXAMPLES

아래 프로그램에서는 명령행 인자 각각에 대해 `getdate()`를 호출하고 각 호출마다 반환된 `tm` 구조체의 필드 값들을 표시한다. 다음 셸 세션은 프로그램 사용 방식을 보여 준다.

```text
$ TFILE=$PWD/tfile
$ echo '%A' > $TFILE       # 주 중 날짜 이름
$ echo '%T' >> $TFILE      # ISO 날짜 (YYYY-MM-DD)
$ echo '%F' >> $TFILE      # 시간 (HH:MM:SS)
$ date
$ export DATEMSK=$TFILE
$ ./a.out Tuesday '2009-12-28' '12:22:33'
Sun Sep  7 06:03:36 CEST 2008
Call 1 ("Tuesday") succeeded:
    tm_sec   = 36
    tm_min   = 3
    tm_hour  = 6
    tm_mday  = 9
    tm_mon   = 8
    tm_year  = 108
    tm_wday  = 2
    tm_yday  = 252
    tm_isdst = 1
Call 2 ("2009-12-28") succeeded:
    tm_sec   = 36
    tm_min   = 3
    tm_hour  = 6
    tm_mday  = 28
    tm_mon   = 11
    tm_year  = 109
    tm_wday  = 1
    tm_yday  = 361
    tm_isdst = 0
Call 3 ("12:22:33") succeeded:
    tm_sec   = 33
    tm_min   = 22
    tm_hour  = 12
    tm_mday  = 7
    tm_mon   = 8
    tm_year  = 108
    tm_wday  = 0
    tm_yday  = 250
    tm_isdst = 1
```

### 프로그램 소스

```c
#define _GNU_SOURCE
#include <time.h>
#include <stdio.h>
#include <stdlib.h>

int
main(int argc, char *argv[])
{
    struct tm *tmp;

    for (int j = 1; j < argc; j++) {
        tmp = getdate(argv[j]);

        if (tmp == NULL) {
            printf("Call %d failed; getdate_err = %d\n",
                   j, getdate_err);
            continue;
        }

        printf("Call %d (\"%s\") succeeded:\n", j, argv[j]);
        printf("    tm_sec   = %d\n", tmp->tm_sec);
        printf("    tm_min   = %d\n", tmp->tm_min);
        printf("    tm_hour  = %d\n", tmp->tm_hour);
        printf("    tm_mday  = %d\n", tmp->tm_mday);
        printf("    tm_mon   = %d\n", tmp->tm_mon);
        printf("    tm_year  = %d\n", tmp->tm_year);
        printf("    tm_wday  = %d\n", tmp->tm_wday);
        printf("    tm_yday  = %d\n", tmp->tm_yday);
        printf("    tm_isdst = %d\n", tmp->tm_isdst);
    }

    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[time(2)]]</tt>, <tt>[[localtime(3)]]</tt>, <tt>[[setlocale(3)]]</tt>, <tt>[[strftime(3)]]</tt>, <tt>[[strptime(3)]]</tt>

----

2021-03-22
