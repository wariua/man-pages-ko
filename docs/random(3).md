## NAME

random, srandom, initstate, setstate - 난수 생성기

## SYNOPSIS

```c
#include <stdlib.h>

long int random(void);

void srandom(unsigned int seed);

char *initstate(unsigned int seed, char *state, size_t n);

char *setstate(char *state);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

<dl>
<dt><code>random()</code>, <code>srandom()</code>, <code>initstate()</code>, <code>setstate()</code>:</dt>
<dd>
<code>_XOPEN_SOURCE >= 500</code><br>
<code>    || /* glibc 2.19부터: */ _DEFAULT_SOURCE</code><br>
<code>    || /* glibc 버전 <= 2.19: */ _SVID_SOURCE || _BSD_SOURCE</code>
</dd>
</dl>

## DESCRIPTION

`random()` 함수는 비선형 가산 피드백 난수 생성기를 사용하여 0에서 `RAND_MAX`까지 범위에서 연속으로 유사 난수를 반환한다. 생성기에서는 기본적으로 31개 정수 크기의 테이블을 사용한다. 이 난수 생성기의 주기는 아주 길다. 대략 `16 * ((2^31) - 1)`이다.

`srandom()` 함수는 그 인자를 `random()`이 새로 반환할 유사 난수 정수 열의 시드로 설정한다. 같은 시드 값으로 `srandom()`을 호출하여 그 정수 열을 반복할 수 있다. 시드 값을 제공하지 않으면 `random()` 함수에서 자동으로 1 값을 시드로 삼는다.

`initstate()` 함수를 이용하면 `random()`에서 사용하도록 상태 배열 `state`를 초기화 할 수 있다. `initstate()`에서는 상태 배열의 크기 `n`을 사용해 얼마나 복잡한 난수 생성기를 사용해야 할지 결정한다. 상태 배열이 클수록 난수가 더 좋아지게 된다. 상태 배열 크기 `n`에 대한 현재의 "최적" 값은 8, 32, 64, 128, 256바이트이다. 다른 양은 가장 가까운 정해진 양으로 내림 하게 된다. 8바이트보다 작게 사용하려 하면 오류가 발생한다. `seed`는 초기화를 위한 시드이다. 난수 열의 시작점을 나타내며 동일 지점에서 다시 시작할 수 있게 해 준다.

`setstate()` 함수는 `random()` 함수가 쓰는 상태 배열을 바꾼다. 다음 번 `initstate()` 내지 `setstate()` 호출 때까지 난수 생성에 상태 배열 `state`를 사용한다. `state`는 `initstate()`를 이용해 먼저 초기화 한 것이거나 이전 `setstate()` 호출의 결과여야 한다.

## RETURN VALUE

`random()` 함수는 0에서 `RAND_MAX` 사이의 값을 반환한다. `srandom()` 함수는 아무 값도 반환하지 않는다.

`initstate()` 함수는 이전 상태 배열에 대한 포인터를 반환한다. 오류 시 원인을 나타내도록 `errno`를 설정한다.

성공 시 `setstate()`는 이전 상태 배열에 대한 포인터를 반환한다. 오류 시 NULL을 반환하며 오류 원인을 나타내도록 `errno`를 설정한다.

## ERRORS

<dl>
<dt><code>EINVAL</code></dt>
<dd><code>setstate()</code>에 준 <code>state</code> 인자가 NULL이었다.</dd>
<dt><code>EINVAL</code></dt>
<dd><code>initstate()</code>에 8바이트보다 작은 상태 배열을 지정하였다.</dd>
</dl>

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `random()`, `srandom()`,<br>`initstate()`, `setstate()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, 4.3BSD.

## NOTES

동작이 재연 가능해야 하는 다중 스레드 프로그램에서는 `random()` 함수를 사용하지 말아야 한다. 그런 용도에는 <tt>[[random_r(3)]]</tt>을 사용하라.

난수 생성은 복잡한 주제이다. *Numerical Recipes in C: The Art of Scientific Computing* (William H. Press, Brian P. Flannery, Saul A. Teukolsky, William T. Vetterling; New York: Cambridge University Press, 2007, 3rd ed.) 7장 "Random Numbers"에서 난수 생성의 현실적 문제들을 탁월하게 논한다.

더 이론적인 논의와 함께 여러 현실적 사안들도 깊이 다루고 있는 곳이 Donald E. Knuth의 *The Art of Computer Programming* 2권(Seminumerical Algorithms) 2판(Reading, Massachusetts: Addison-Wesley Publishing Company, 1981)의 3장 "Random Numbers"이다.

## BUGS

POSIX에 따르면 `initstate()`가 오류 시 NULL을 반환해야 한다. glibc 구현에서는 (명세된 대로) 오류 시 `errno`를 설정하기는 하지만 함수가 NULL을 반환하지는 않는다.

## SEE ALSO

<tt>[[getrandom(2)]]</tt>, <tt>[[drand48(3)]]</tt>, <tt>[[rand(3)]]</tt>, <tt>[[random_r(3)]]</tt>, <tt>[[srand(3)]]</tt>

----

2019-03-06
