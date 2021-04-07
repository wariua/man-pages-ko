## NAME

rand, rand_r, srand - 유사 난수 생성기

## SYNOPSIS

```c
#include <stdlib.h>

int rand(void);

int rand_r(unsigned int *seedp);

void srand(unsigned int seed);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

<dl>
<dt><code>rand_r()</code>:</dt>
<dd>
 <dl>
 <dt>glibc 2.24부터:</dt>
 <dd><code>_POSIX_C_SOURCE >= 199506L</code></dd>
 <dt>glibc 2.23 및 이전:</dt>
 <dd><code>_POSIX_C_SOURCE</code></dd>
 </dl>
</dd>
</dl>

## DESCRIPTION

`rand()` 함수는 0에서 `RAND_MAX`까지 범위의 (경계 포함) 유사 난수 정수를 반환한다. (즉, 수학적 범위가 [0, `RAND_MAX`]이다.)

`srand()` 함수는 그 인자를 `rand()`가 새로 반환할 유사 난수 정수 열의 시드로 설정한다. 같은 시드 값으로 `srand()`를 호출하여 그 정수 열을 반복할 수 있다.

시드 값을 제공하지 않으면 `rand()` 함수에서 자동으로 1 값을 시드로 삼는다.

`rand()` 함수는 재진입 가능하지 않다. 각 호출마다 바뀌는 숨겨진 상태를 사용하기 때문이다. 그 상태라는 것은 그저 다음 호출에서 사용할 시드 값일 수도 있고 더 정교한 무언가일 수도 있을 것이다. 스레드 사용 응용에서 재연 가능한 동작 방식을 얻기 위해선 이 상태를 명시적으로 만들어야 하는데, 재진입 가능 함수인 `rand_r()`을 사용하면 된다.

`rand()`와 마찬가지로 `rand_r()`은 [0, `RAND_MAX`] 범위 내의 유사 난수 정수를 반환한다. `seedp` 인자는 호출들 간에 상태 저장에 쓰이는 `unsigned int`에 대한 포인터이다. `seedp`가 가리키는 정수의 최초 값을 같게 해서 `rand_r()`을 호출하고 호출들 사이에서 그 값을 변경하지 않으면 동일한 유사 난수 열이 나오게 된다.

`rand_r()`의 `seedp` 인자가 가리키는 값은 아주 작은 양의 상태만 제공하므로 이 함수는 약한 유사 난수 생성기일 것이다. 대신 <tt>[[drand48_r(3)]]</tt>을 시도해 보라.

## RETURN VALUE

`rand()` 및 `rand_r()` 함수는 0에서 `RAND_MAX` 사이의 (경계 포함) 값을 반환한다. `srand()` 함수는 아무 값도 반환하지 않는다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `rand()`, `rand_r()`, `srand()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

`rand()` 및 `srand()` 함수는 SVr4, 4.3BSD, C89, C99, POSIX.1-2001을 준수한다. `rand_r()` 함수는 POSIX.1-2001에서 온 것이다. POSIX.1-2008에서 `rand_r()`을 구식으로 표시하고 있다.

## NOTES

리눅스 C 라이브러리에 있는 `rand()` 및 `srand()` 버전은 <tt>[[random(3)]]</tt> 및 <tt>[[srandom(3)]]</tt>과 같은 난수 생성기를 사용하므로 하위 비트들이 상위 비트들만큼 임의적일 것이다. 하지만 오래된 `rand()` 구현들에서는, 그리고 다른 시스템들의 현행 구현에서는 하위 비트들이 상위 비트들보다 훨씬 덜 임의적이다. 이식성을 의도한 응용에서 좋은 난수성이 필요할 때는 이 함수를 사용하지 말아야 한다. (대신 <tt>[[random(3)]]</tt>을 사용하라.)

## EXAMPLE

POSIX.1-2001에서 `rand()` 및 `srand()`의 구현 예시로 다음을 제시하고 있다. 서로 다른 머신에서 동일한 수열이 필요할 때 유용할 수도 있을 것이다.

```c
static unsigned long next = 1;

/* RAND_MAX가 32767이라고 가정 */
int myrand(void) {
    next = next * 1103515245 + 12345;
    return((unsigned)(next/65536) % 32768);
}

void mysrand(unsigned int seed) {
    next = seed;
}
```

다음 프로그램을 이용해 특정 시드가 주어졌을 때 `rand()`가 만들어 내는 유사 난수 열을 표시할 수 있다.

```c
#include <stdlib.h>
#include <stdio.h>

int
main(int argc, char *argv[])
{
    int j, r, nloops;
    unsigned int seed;

    if (argc != 3) {
        fprintf(stderr, "Usage: %s <seed> <nloops>\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    seed = atoi(argv[1]);
    nloops = atoi(argv[2]);

    srand(seed);
    for (j = 0; j < nloops; j++) {
        r =  rand();
        printf("%d\n", r);
    }

    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[drand48(3)]]</tt>, <tt>[[random(3)]]</tt>

----

2019-03-06
