## NAME

fileno - stdio 스트림의 파일 디스크립터 얻기

## SYNOPSIS

```c
#include <stdio.h>

int fileno(FILE *stream);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`fileno()`:
:   `_POSIX_C_SOURCE`

## DESCRIPTION

`fileno()` 함수는 `stream` 인자를 확인해서 그 스트림 구현에 쓰인 정수 파일 디스크립터를 반환한다. 그 파일 디스크립터는 여전히 `stream`이 소유하고 있으며 <tt>[[fclose(3)]]</tt> 호출 시에 닫히게 된다. 디스크립터를 닫을 수도 있는 코드로 전달할 때는 먼저 <tt>[[dup(2)]]</tt>으로 복제해야 한다.

락킹 없는 대응 함수는 <tt>[[unlocked_stdio(3)]]</tt> 참고.

## RETURN VALUE

성공 시 `fileno()`는 `stream`에 연계된 파일 디스크립터를 반환한다. 실패 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EBADF`
:   `stream`이 파일에 연계돼 있지 않다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `fileno()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

`fileno()` 함수는 POSIX.1-2001 및 POSIX.1-2008을 준수한다.

## SEE ALSO

<tt>[[open(2)]]</tt>, <tt>[[fdopen(3)]]</tt>, <tt>[[stdio(3)]]</tt>, <tt>[[unlocked_stdio(3)]]</tt>

----

2021-03-22
