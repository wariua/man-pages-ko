## NAME

swab - 이웃한 바이트 교환하기

## SYNOPSIS

```c
#define _XOPEN_SOURCE       /* feature_test_macros(7) 참고 */
#include <unistd.h>

void swab(const void *from, void *to, ssize_t n);
```

## DESCRIPTION

`swab()` 함수는 `from`이 가리키는 배열에서 `to`가 가리키는 배열로 `n` 바이트를 복사하면서 이웃한 짝수 번째 바이트와 홀수 번째 바이트를 교환한다. 하위/상위 바이트 순서가 다른 머신들 간에 데이터를 교환하는 데 이 함수를 쓸 수 있다.

`n`이 음수면 이 함수는 아무것도 하지 않는다. `n`이 양수이고 홀수면 `n-1` 바이트를 위와 같이 처리하며 마지막 바이트에는 명세되지 않은 뭔가를 한다. (달리 말해 `n`이 짝수여야 한다.)

## RETURN VALUE

`swab()` 함수는 아무 값도 반환하지 않는다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `swab()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, SVr4, 4.3BSD.

## SEE ALSO

<tt>[[bstring(3)]]</tt>

----

2015-08-08
