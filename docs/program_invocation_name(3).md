## NAME

program_invocation_name, program_invocation_short_name - 호출 프로그램 실행에 사용한 이름 얻기

## SYNOPSIS

```c
#define _GNU_SOURCE         /* feature_test_macros(7) 참고 */
#include <errno.h>

extern char *program_invocation_name;
extern char *program_invocation_short_name;
```

## DESCRIPTION

`program_invocation_name`은 호출 프로그램 실행에 사용했던 이름을 담고 있다. `main()`의 `argv[0]`의 값과 같은데, `program_invocation_name`은 스코프가 전역이라는 점이 다르다.

`program_invocation_short_name`은 호출 프로그램 실행에 사용했던 이름의 basename 부분을 담고 있다. 즉 `program_invocation_name`에서 마지막 슬래시(/)까지의 내용을 모두 없앤 것과 같은 값(슬래시가 없으면 그대로)이다.

glibc 런타임 시작 코드에서 이 변수들을 자동으로 초기화 한다.

## CONFORMING TO

이 변수들은 GNU 확장이므로 이식성이 있어야 하는 프로그램에서는 사용하지 말아야 한다.

## NOTES

리눅스 전용인 `/proc/[number]/cmdline` 파일을 통해 비슷한 정보에 접근할 수 있다.

## SEE ALSO

<tt>[[proc(5)]]</tt>

----

2017-09-15
