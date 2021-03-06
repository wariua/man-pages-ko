## NAME

tzset, tzname, timezone, daylight - 시간 변환 정보 초기화

## SYNOPSIS

```c
#include <time.h>

void tzset (void);

extern char *tzname[2];
extern long timezone;
extern int daylight;
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`tzset()`:
:   `_POSIX_C_SOURCE`

`tzname`:
:   `_POSIX_C_SOURCE`

`timezone`, `daylight`:
:   `_XOPEN_SOURCE`<br>
    `|| /* glibc 2.19부터: */ _DEFAULT_SOURCE`<br>
    `|| /* glibc <= 2.19: */ _SVID_SOURCE`

## DESCRIPTION

`tzset()` 함수는 `TZ` 환경 변수를 가지고 `tzname` 변수를 초기화 한다. 시간대에 의존하는 다른 시간 변환 함수들에서 이 함수를 자동으로 호출한다. 시스템 V 방식 환경에서는 변수 `timezone`(UTC 서쪽으로 몇 초) 및 `daylight`(시간대에 일광 절약 시간 규칙이 없으면 0으로, 일광 절약 시간이 적용되는 과거나 현재, 미래 시점이 있으면 0 아닌 값으로)까지 설정한다.

환경 내에 `TZ` 변수가 안 보이면 시스템 시간대를 쓴다. 시스템 시간대는 <tt>[[tzfile(5)]]</tt> 형식 파일을 `/etc/localtime`으로 복사하거나 링크 해서 설정한다. 시스템 시간대 디렉터리(아래 **FILES** 절 참고)에서 그런 파일들로 이뤄진 시간대 데이터베이스를 볼 수 있다.

환경 내에 `TZ` 변수가 있기는 한데 값이 비어 있거나 그 값을 아래 명세하는 형식들 중 어떤 것으로도 해석할 수 없으면 협정 세계시(UTC)를 쓴다.

`TZ`의 값은 두 가지 형식 중 하나로 돼 있을 수 있다. 첫 번째 형식은 사용할 시간대를 직접 표현하는 문자열이다.

```text
std offset[dst[offset][,start[/time],end[/time]]]
```

명세 문자열 내에 공백이 전혀 없다. `std` 문자열은 시간대 약자를 나타내며 세 글자 이상의 알파벳 문자여야 한다. 이상(<) 및 이상(>) 부호로 감싼 경우 더하기(+) 부호와 빼기(-) 부호, 숫자들까지 포함하도록 문자 집합이 확장된다. `offset` 문자열은 `std` 바로 다음에 오며 지역 시간에 더하면 협정 세계시(UTC)가 나오는 시간 양을 나타낸다. 지역 시간대가 본초 자오선 서쪽이면 `offset`이 양수고 동쪽이면 음수다. 시간은 0에서 24까지, 분과 초는 00에서 59까지여야 한다.

```text
[+|-]hh[:mm[:ss]]
```

`dst` 문자열 및 `offset`은 해당하는 일광 절약 시간대의 이름과 오프셋을 나타낸다. 오프셋이 생략돼 있으면 표준시보다 한 시간 앞서는 것으로 친다.

`start` 필드는 일광 절약 시간이 발효되는 때를 나타내고 `end` 필드는 다시 표준시로 바뀌는 때를 나타낸다. 이 두 필드는 다음 형식일 수 있다.

`Jn`
:   율리우스력 날짜를 1에서 365 사이의 `n`으로 지정한다. 윤날은 세지 않는다. 이 형식에서는 2월 29일을 표현할 수 없다. 즉 2월 28일이 59번 날이고 항상 3월 1일이 60번 날이다.

`n`
:   0이 기준인 율리우스력 날짜를 0에서 365 사이의 `n`으로 지정한다. 윤년의 2월 29일을 센다.

`Mm.w.d`
:   `m` 월(1 <= `m` <= 12)의 `w` 번째 주(1 <= `w` <= 5)의 `d` 번째 날(0 <= `d` <= 6)을 지정한다. 1번째 주는 `d` 번째 날이 있는 첫 번째 주이고 5번째 주는 `d` 번째 날이 있는 마지막 주이다. 0번째 날은 일요일이다.

`time` 필드는 현재 적용 중인 지역 시간으로 언제 그 다른 시간으로 바뀌는지를 나타낸다. 생략 시 기본값은 02:00:00이다.

다음은 뉴질랜드 사례이다. 표준 시간(NZST)이 UTC보다 12시간 앞서고, 일광 절약 시간(NZDT)이 UTC보다 13시간 앞서고 10월 첫째 일요일부터 3월 셋째 일요일까지 시행되며, 기본 시간 02:00:00에 전환이 이뤄진다.

```text
TZ="NZST-12:00:00NZDT-13:00:00,M10.1.0,M3.3.0"
```

두 번째 형식은 파일에서 시간대 정보를 읽어 들이도록 명시한다.

```text
:[filespec]
```

파일 명세 `filespec`이 생략돼 있거나 그 값을 해석할 수 없는 경우에는 협정 세계시(UTC)를 쓴다. `filespec`이 주어진 경우에는 시간대 정보를 읽어 들일 또 다른 <tt>[[tzfile(5)]]</tt> 형식 파일을 나타낸다. `filespec`이 '/'로 시작하지 않으면 시스템 시간대 디렉터리를 기준으로 파일을 지정하는 것이다. 콜론이 빠져 있으면 위의 `TZ` 형식들 각각을 시도해 보게 된다.

다시 뉴질랜드로 예를 들면 다음과 같다.

```text
TZ=":Pacific/Auckland"
```

## ENVIRONMENT

`TZ`
:   이 변수가 설정돼 있으면 그 값이 시스템 설정 시간대보다 우선한다.

`TZDIR`
:   이 변수가 설정돼 있으면 그 값이 시스템 설정 시간대 데이터베이스 디렉터리 경로보다 우선한다.

## FILES

`/etc/localtime`
:   시스템 시간대 파일.

`/usr/share/zoneinfo/`
:   시스템 시간대 데이터베이스 디렉터리.

`/usr/share/zoneinfo/posixrules`
:   TZ 문자열에 dst 시간대가 있고 그 뒤에 아무것도 없으면 이 파일을 써서 시작/끝 규칙을 얻는다. <tt>[[tzfile(5)]]</tt> 형식으로 돼 있다. 기본적으로 zoneinfo의 Makefile에서 tzfile `America/New York`에 대한 하드 링크로 만들어 둔다.

이상은 현행 표준 파일 위치이다. glibc를 컴파일 할 때 설정 가능하다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값
| --- | --- | --- |
| `tzset()` | 스레드 안전성 | MT-Safe env locale |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, SVr4, 4.3BSD.

## NOTES

4.3BSD에는 `char *timezone(zone, dst)` 함수가 있어서 첫 번째 인자(UTC 서쪽으로 몇 분)에 대응하는 시간대의 이름을 반환했다. 두 번째 인자가 0이면 표준 이름을 썼고 아니면 일광 절약 시간 버전을 썼다.

## SEE ALSO

`date(1)`, <tt>[[gettimeofday(2)]]</tt>, <tt>[[time(2)]]</tt>, <tt>[[ctime(3)]]</tt>, <tt>[[getenv(3)]]</tt>, <tt>[[tzfile(5)]]</tt>

----

2021-03-22
