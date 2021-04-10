## NAME

stpncpy - 고정 크기 문자열을 복사하고 그 끝에 대한 포인터 반환하기

## SYNOPSIS

```c
#include <string.h>

char *stpncpy(char *dest, const char *src, size_t n);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`stpncpy()`:
:   glibc 2.10부터:
    :   `_POSIX_C_SOURCE >= 200809L`

    glibc 2.10 전:
    :   `_GNU_SOURCE`

## DESCRIPTION

`stpncpy()` 함수는 `src`가 가리키는 문자열로부터 종료용 널 바이트(`'\0'`)를 포함해 최대 `n` 개 문자를 `dest`가 가리키는 배열로 복사한다. `dest`에 정확히 `n` 개 문자를 쓴다. 길이 `strlen(src)`가 `n`보다 짧으면 `dest`가 가리키는 배열의 나머지 문자들을 널 바이트(`'\0'`)로 채운다. 길이 `strlen(src)`가 `n`과 같거나 그보다 길면 `dest`가 가리키는 문자열이 널 종료가 아니게 된다.

두 문자열이 겹칠 수 없다.

`dest`에 최소 `n` 개 문자를 위한 공간이 있음을 프로그래머가 보장해야 한다.

## RETURN VALUE

`stpncpy()`는 `dest` 내의 종료용 널 바이트에 대한 포인터를 반환한다. `dest`가 널 종료가 아니면 `dest+n`을 반환한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `stpncpy()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

이 함수는 POSIX.1-2008에 추가되었다. 그 전에는 GNU 확장이었다. 1993년에 GNU C 라이브러리 버전 1.07에서 처음 등장했다.

## SEE ALSO

`strncpy(3)`, `wcpncpy(3)`

----

2019-03-06
