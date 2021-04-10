## NAME

ffs, ffsl, ffsll - 워드 내에 설정된 첫 비트 찾기

## SYNOPSIS

```c
#include <strings.h>

int ffs(int i);

#include <string.h>

int ffsl(long int i);

int ffsll(long long int i);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`ffs()`:
:   glibc 1.12부터:
    :   `    _XOPEN_SOURCE >= 700`<br>
        `    || ! (_POSIX_C_SOURCE >= 200809L)`<br>
        `    || /* Glibc 2.19부터: */ _DEFAULT_SOURCE`<br>
        `    || /* Glibc 버전 <= 2.19: */ _BSD_SOURCE || _SVID_SOURCE`

    glibc 1.12 전:
    :   없음

`ffsl()`, `ffsll()`:
:   glibc 2.27부터:
    :   `_DEFAULT_SOURCE`

    glibc 2.27 전:
    :   `_GNU_SOURCE`

## DESCRIPTION

`ffs()` 함수는 워드 `i` 내의 첫 번째 (최하위) 설정 비트의 위치를 반환한다. 최하위 비트가 1번 위치이고 최상위가 가령 32번이나 64번이다. `ffsll()` 및 `ffsl()` 함수는 같은 일을 하되 다른 크기일 수도 있는 인자를 받는다.

## RETURN VALUE

이 함수들은 첫 번째 설정 비트의 위치를 반환한다. `i`에 어떤 비트도 설정되어 있지 않으면 0을 반환한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `ffs()`, `ffsl()`, `ffsll()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

`ffs()`: POSIX.1-2001, POSIX.1-2008, 4.3BSD.

`ffsl()` 및 `ffsll()` 함수는 glibc 확장이다.

## NOTES

BSD 시스템에서는 `<string.h>`에 원형이 있다.

## SEE ALSO

<tt>[[memchr(3)]]</tt>

----

2017-09-15
