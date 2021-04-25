## NAME

random_r, srandom_r, initstate_r, setstate_r - 재진입 가능한 난수 생성기

## SYNOPSIS

```c
#include <stdlib.h>

int random_r(struct random_data *restrict buf,
             int32_t *restrict result);
int srandom_r(unsigned int seed, struct random_data *buf);

int initstate_r(unsigned int seed, char *restrict statebuf,
             size_t statelen, struct random_data *restrict buf);
int setstate_r(char *restrict statebuf,
             struct random_data *restrict buf);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`random_r()`, `srandom_r()`, `initstate_r()`, `setstate_r()`:
:   `/* glibc 2.19부터: */ _DEFAULT_SOURCE`<br>
    `    || /* glibc <= 2.19: */ _SVID_SOURCE || _BSD_SOURCE`

## DESCRIPTION

이 함수들은 <tt>[[random(3)]]</tt>에서 기술하는 함수들의 재진입 가능 버전이다. 각 스레드에서 독립적이고 재연 가능한 난수 열을 얻어야 하는 다중 스레드 프로그램에서 쓰기에 적합하다.

`random_r()` 함수는 <tt>[[random(3)]]</tt>과 유사하되 전역 변수로 유지하는 상태 정보를 사용하는 것이 아니라 `buf`가 가리키는 인자에 있는 상태 정보를 사용한다. `buf`는 `initstate_r()`로 미리 초기화 되어 있어야 한다. 생성한 난수를 인자 `result`로 반환한다.

`srandom_r()` 함수는 <tt>[[srandom(3)]]</tt>과 유사하되 전역 상태 변수에 연계된 시드가 아니라 `buf`가 가리키는 객체에 상태를 유지하는 난수 생성기의 시드를 초기화 한다. `buf`는 `initstate_r()`로 미리 초기화 되어 있어야 한다.

`initstate_r()` 함수는 <tt>[[initstate(3)]]</tt>와 유사하되 전역 상태 변수를 초기화 하는 것이 아니라 `buf`가 가리키는 객체 내의 상태를 초기화 한다. 이 함수를 호출하기 전에 `buf.state` 필드를 NULL로 초기화 해야 한다. `initstate_r()` 함수는 `statebuf` 인자에 대한 포인터를 `buf`가 가리키는 구조체 내에 저장한다. 그래서 `buf`를 사용하는 동안은 `statebuf` 할당을 해제해서는 안 된다. (따라서 보통 `statebuf`를 정적 변수로 할당하거나 <tt>[[malloc(3)]]</tt> 등을 써서 힙에 할당해야 한다.)

`setstate_r()` 함수는 <tt>[[setstate(3)]]</tt>과 유사하되 전역 상태 변수를 변경하는 것이 아니라 `buf`가 가리키는 객체 내의 상태를 변경한다. `state`는 `initstate_r()`을 이용해 먼저 초기화 한 것이거나 이전 `setstate_r()` 호출의 결과여야 한다.

## RETURN VALUE

이 함수들은 모두 성공 시 0을 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EINVAL`
:   `initstate_r()`에 8바이트보다 작은 상태 배열을 지정했다.

`EINVAL`
:   `setstate_r()`에 대한 `statebuf` 인자나 `buf` 인자가 NULL이었다.

`EINVAL`
:   `random_r()`에 대한 `buf` 인자나 `result` 인자가 NULL이었다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `random_r()`, `srandom_r()`,<br>`initstate_r()`, `setstate_r()` | 스레드 안전성 | MT-Safe race:buf |

## CONFORMING TO

이 함수들은 비표준 glibc 확장이다.

## BUGS

`initstate_r()` 인터페이스가 혼란스럽다. `random_data` 타입을 불투명한 것으로 하려는 것 같은데 구현에서는 사용자가 호출 전에 `buf.state` 필드를 NULL로 초기화 하거나 구조체 전체를 0으로 채우기를 요구한다.

## SEE ALSO

<tt>[[drand48(3)]]</tt>, <tt>[[rand(3)]]</tt>, <tt>[[random(3)]]</tt>

----

2021-03-22
