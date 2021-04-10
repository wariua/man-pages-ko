## NAME

drand48, erand48, lrand48, nrand48, mrand48, jrand48, srand48, seed48, lcong48 - 균일하게 분포하는 유사 난수 생성하기

## SYNOPSIS

```c
#include <stdlib.h>

double drand48(void);

double erand48(unsigned short xsubi[3]);

long int lrand48(void);

long int nrand48(unsigned short xsubi[3]);

long int mrand48(void);

long int jrand48(unsigned short xsubi[3]);

void srand48(long int seedval);

unsigned short *seed48(unsigned short seed16v[3]);

void lcong48(unsigned short param[7]);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

위 함수들 모두:
:   `_XOPEN_SOURCE`<br>
    `    || /* glibc 2.19부터: */ _DEFAULT_SOURCE`<br>
    `    || /* glibc 버전 <= 2.19: */ _SVID_SOURCE`

## DESCRIPTION

이 함수들은 선형 합동 알고리듬과 48비트 정수 연산을 이용해 유사 난수를 생성한다.

`drand48()` 및 `erand48()` 함수는 [0.0, 1.0) 구간에 균일하게 분포하는 음수 아닌 배정밀도 부동소수점 값을 반환한다.

`lrand48()` 및 `nrand48()` 함수는 [0, 2^31) 구간에 균일하게 분포하는 음수 아닌 긴 정수를 반환한다.

`mrand48()` 및 `jrand48()` 함수는 [-2^31, 2^31) 구간에 균일하게 분포하는 부호 있는 긴 정부를 반환한다.

`srand48()`, `seed48()`, `lcong48()` 함수는 초기화 함수이며 `drand48()`, `lrand48()`, `mrand48()` 사용 전에 호출해야 한다. `erand48()`, `nrand48()`, `jrand48()` 함수에는 초기화 함수를 먼저 호출할 필요가 없다.

모든 함수들은 다음 선형 합동 식에 따라 48비트 정수 `Xi`의 열을 생성하여 동작한다.

```
Xn+1 = (aXn + c) mod m, 여기서 n >= 0
```

매개변수 `m` = 2^48이며, 그래서 48비트 정수 연산을 수행한다. `lcong48()`을 호출한 경우가 아니면 `a`와 `c`에 다음 값을 준다.

```
a = 0x5DEECE66D
c = 0xB
```

함수 `drand48()`, `erand48()`, `lrand48()`, `nrand48()`, `mrand48()`, `jrand48()` 중 뭔가로 반환하는 값의 계산은 먼저 열의 다음 48비트 `Xi`를 생성하는 것으로 이뤄진다. 그러고서 반환할 데이터 항목의 타입에 따라 적절한 수의 비트를 `Xi`의 상위 비트들로부터 복사하고 이를 반환 값으로 변환한다.

`drand48()`, `lrand48()`, `mrand48()` 함수에서는 마지막으로 생성한 48비트 `Xi`를 내부 버퍼에 저장해 둔다. `erand48()`, `nrand48()`, `jrand48()` 함수에서는 연속되는 `Xi` 값을 위한 저장 공간을 호출 프로그램이 `xsubi` 배열 인자로 제공해야 한다. 처음 함수를 호출하기 전에 그 배열에 `Xi`의 초기값을 넣어서 그 함수들을 초기화 한다.

초기화 함수 `srand48()`은 `Xi`의 상위 32비트를 인자 `seedval`로 설정한다. 하위 16비트는 임의로 정한 0x330E 값으로 설정한다.

초기화 함수 `seed48()`은 `Xi`의 값을 배열 인자 `seed16v`에 지정된 48비트 값으로 설정한다. `Xi`의 이전 값이 어떤 내부 버퍼로 복사되고 이 버퍼에 대한 포인터를 `seed48()`이 반환한다.

초기화 함수 `lcong48()`은 사용자가 `Xi`, `a`, `c`의 초기값을 지정할 수 있게 해 준다. 배열 인자의 `param[0-2]` 항목들이 `Xi`를 지정하고, `param[3-5]`가 `a`를 지정하고, `param[6]`이 `c`를 지정한다. `lcong48()`을 호출한 다음에 `srand48()`이나 `seed48()`을 호출하면 `a`와 `c`의 표준 값이 다시 살아나게 된다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `drand48()`, `erand48()`,<br>`lrand48()`, `nrand48()`,<br>`mrand48()`, `jrand48()`,<br>`srand48()`, `seed48()`,<br>`lcong48()` | 스레드 안전성 | MT-Unsafe race:drand48 |

위 함수들은 난수 생성기를 위한 전역 상태 정보를 기록하며, 따라서 스레드 안전하지 않다.

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, SVr4.

## SEE ALSO

<tt>[[rand(3)]]</tt>, <tt>[[random(3)]]</tt>

----

2017-09-15
