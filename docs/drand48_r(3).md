## NAME

drand48_r, erand48_r, lrand48_r, nrand48_r, mrand48_r, jrand48_r, srand48_r, seed48_r, lcong48_r - 균일하게 분포하는 유사 난수를 재진입 가능하게 생성하기

## SYNOPSIS

```c
#include <stdlib.h>

int drand48_r(struct drand48_data *buffer, double *result);

int erand48_r(unsigned short xsubi[3],
              struct drand48_data *buffer, double *result);

int lrand48_r(struct drand48_data *buffer, long int *result);

int nrand48_r(unsigned short int xsubi[3],
              struct drand48_data *buffer, long int *result);

int mrand48_r(struct drand48_data *buffer, long int *result);

int jrand48_r(unsigned short int xsubi[3],
              struct drand48_data *buffer, long int *result);

int srand48_r(long int seedval, struct drand48_data *buffer);

int seed48_r(unsigned short int seed16v[3],
             struct drand48_data *buffer);

int lcong48_r(unsigned short int param[7],
              struct drand48_data *buffer);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

위 함수들 모두:
:   `/* glibc 2.19부터: */ _DEFAULT_SOURCE`<br>
    `    || /* glibc 버전 <= 2.19: */ _SVID_SOURCE || _BSD_SOURCE`

## DESCRIPTION

이 함수들은 <tt>[[drand48(3)]]</tt>에서 기술하는 함수들의 재진입 가능 버전이다. 전역 난수 생성기 상태를 변경하는 대신 `buffer`로 제공받는 데이터 버퍼를 사용한다.

첫 사용 전에 이 구조체를 초기화 해야 한다. 예를 들어 0으로 채우거나 함수 `srand48_r()`, `seed48_r()`, `lcong48_r()` 중 하나를 호출하면 된다.

## RETURN VALUE

반환 값은 0이다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `drand48_r()`, `erand48_r()`,<br>`lrand48_r()`, `nrand48_r()`,<br>`mrand48_r()`, `jrand48_r()`,<br>`srand48_r()`, `seed48_r()`,<br>`lcong48_r()` | 스레드 안전성 | MT-Safe race:buffer |

## CONFORMING TO

이 함수들은 GNU 확장이며 이식성이 없다.

## SEE ALSO

<tt>[[drand48(3)]]</tt>, <tt>[[rand(3)]]</tt>, <tt>[[random(3)]]</tt>

----

2017-09-15
