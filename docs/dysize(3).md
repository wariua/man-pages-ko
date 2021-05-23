## NAME

dysize - 지정한 해의 날 수 얻기

## SYNOPSIS

```c
#include <time.h>

int dysize(int year);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`dysize()`:
:   glibc 2.19부터:
    :   `_DEFAULT_SOURCE`

    glibc 2.19 및 이전:
    :   `_BSD_SOURCE || _SVID_SOURCE`

## DESCRIPTION

평년에 대해 365를 반환하고 윤년에 대해 366을 반환한다. 다음 식에 따라 윤년을 계산한다.

```c
(year) %4 == 0 && ((year) %100 != 0 || (year) %400 == 0)
```

마찬가지로 `<time.h>`에 있는 `__isleap(year)` 매크로에 공식이 정의돼 있다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `dysize()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

SunOS 4.x에 이 함수가 존재한다.

## NOTES

호환성 함수일 뿐이다. 신규 프로그램에서 쓰지 말아야 한다.

## SEE ALSO

<tt>[[strftime(3)]]</tt>

----

2021-03-22
