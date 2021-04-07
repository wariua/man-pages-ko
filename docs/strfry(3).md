## NAME

strfry - 문자열 난수화 하기

## SYNOPSIS

```c
#define _GNU_SOURCE         /* feature_test_macros(7) 참고 */
#include <string.h>

char *strfry(char *string);
```

## DESCRIPTION

`strfry()` 함수는 문자열 안의 글자들을 임의로 교환하여 `string`의 내용물을 난수화 한다. 결과는 `string`의 애너그램이다.

## RETURN VALUE

`strfry()` 함수는 난수화 된 그 문자열에 대한 포인터를 반환한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `strfry()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

`strfry()`는 GNU C 라이브러리에 고유한 함수이다.

## SEE ALSO

<tt>[[memfrob(3)]]</tt>, <tt>[[string(3)]]</tt>

----

2019-03-06
