## NAME

memfrob - 메모리 영역 뒤섞기 (암호화하기)

## SYNOPSIS

```c
#define _GNU_SOURCE             /* feature_test_macros(7) 참고 */
#include <string.h>

void *memfrob(void *s, size_t n);
```

## DESCRIPTION

`memfrob()` 함수는 각 문자를 숫자 42와 XOR 하여 메모리 영역 `s`의 처음 `n`바이트를 암호화한다. 암호화된 메모리 영역에 `memfrob()`을 사용해서 그 효과를 되돌릴 수 있다.

참고로 이 함수는 XOR 상수가 고정되어 있으므로 올바른 암호화 루틴이 아니며 문자열을 감추는 정도에만 적합하다.

## RETURN VALUE

`memfrob()` 함수는 암호화된 그 메모리 영역에 대한 포인터를 반환한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `memfrob()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

`memfrob()`는 GNU C 라이브러리에 고유한 함수이다.

## SEE ALSO

<tt>[[bstring(3)]]</tt>, <tt>[[strfry(3)]]</tt>

----

2017-03-13
