## NAME

addseverity - 새로운 심각도 수준 도입하기

## SYNOPSIS

```c
#include <fmtmsg.h>

int addseverity(int severity, const char *s);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`addseverity()`:
:   glibc 2.19부터:
    :   `_DEFAULT_SOURCE`

    glibc 2.19 및 이전:
    :   `_SVID_SOURCE`

## DESCRIPTION

이 함수를 이용하면 <tt>[[fmtmsg(3)]]</tt> 함수의 `severity` 인자로 지정할 수 있는 새로운 심각도 수준을 도입할 수 있다. 기본적으로 그 함수는 심각도 0~4에 대해서만 (문자열 (없음), `HALT`, `ERROR`, `WARNING`, `INFO`로) 메시지를 찍을 줄 안다. 이 호출은 주어진 값 `severity`에 주어진 문자열 `s`를 연결시킨다. `s`가 NULL이면 숫자 값이 `severity`인 심각도 수준을 제거한다. 기본 심각도 수준을 덮어쓰거나 제거하는 것은 불가능하다. 심각도 값은 음수가 아니어야 한다.

## RETURN VALUE

성공 시 `MM_OK` 값을 반환한다. 오류 시 반환 값은 `MM_NOTOK`이다. 가능한 오류들로 메모리 부족, 존재하지 않거나 기본으로 있는 심각도 수준 제거 시도 등이 있다.

## VERSIONS

glibc 버전 2.1부터 `addseverity()`를 제공한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `addseverity()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

X/Open Portability Guide에 <tt>[[fmtmsg(3)]]</tt> 함수는 명세되어 있지만 이 함수는 명세되어 있지 않다. 시스템 V 시스템에서 사용 가능하다.

## NOTES

환경 변수 `SEV_LEVEL`을 설정하는 것으로도 새로운 심각도 수준을 추가할 수 있다.

## SEE ALSO

<tt>[[fmtmsg(3)]]</tt>

----

2016-03-15
