## NAME

perror - 시스템 오류 메시지 찍기

## SYNOPSIS

```c
#include <stdio.h>

void perror(const char *s);

#include <errno.h>

const char * const sys_errlist[];
int sys_nerr;
int errno;       /* 실제 이렇게 선언돼 있지 않음. errno(3) 참고 */
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`sys_errlist`, `sys_nerr`:
:   glibc 2.19부터:
    :   `_DEFAULT_SOURCE`

    glibc 2.19 및 이전:
    :    `_BSD_SOURCE`

## DESCRIPTION

`perror()` 함수는 시스템 내지 라이브러리 함수 호출 중 만난 최종 오류를 설명하는 메시지를 표준 오류로 내놓는다.

먼저 (`s`가 NULL이 아니고 `*s`가 널 바이트('\0')가 아니면) 인자 문자열 `s`를 찍고 이어서 콜론과 공백을 찍는다. 다음으로 현재 `errno` 값에 해당하는 오류 메시지와 개행이 온다.

제대로 쓰려면 인자 문자열에 오류가 발생한 함수의 이름을 포함시키는 게 좋다.

전역 오류 목록 `sys_errlist[]`는 `errno`를 인덱스로 사용해서 개행 없는 오류 메시지를 얻을 수 있다. 이 테이블에서 제공하는 가장 높은 메시지 번호는 `sys_nerr-1`이다. 새 오류 값이 `sys_errlist[]`에 추가되지 않았을 수도 있으므로 이 목록에 직접 접근할 때는 조심해야 한다. 요즘은 `sys_errlist[]`를 쓰지 않는 게 좋다. 대신 <tt>[[strerror(3)]]</tt>를 쓰면 된다.

시스템 호출이 실패하면 보통 -1을 반환하면서 `errno`에다가 뭐가 잘못됐는지 설명하는 값을 설정한다. (이 값들을 `<errno.h>`에서 볼 수 있다.) 여러 라이브러리 함수들도 비슷하다. `perror()` 함수는 그 오류 코드를 사람이 읽을 수 있는 형태로 바꿔 주는 역할을 한다. 참고로 시스템 호출이나 라이브러리 함수 호출이 성공한 후의 `errno`는 규정돼 있지 않다. 성공하는 경우에도 호출에서 그 변수를 바꿀 수 있는데, 예를 들어 내부적으로 이용한 어떤 다른 라이브러리 함수가 실패해서일 수 있다. 따라서 실패한 호출 바로 다음에서 `perror()`를 호출하는 게 아닌 경우에는 `errno` 값을 저장해 두는 게 좋다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `perror()` | 스레드 안전성 | MT-Safe race:stderr |

## CONFORMING TO

`perror()`, `errno`: POSIX.1-2001, POSIX.1-2008, C89, C99, 4.3BSD.

외부 변수 `sys_nerr` 및 `sys_errlist`는 BSD에서 온 것이며 POSIX.1에는 명세돼 있지 않다.

## NOTES

glibc에 외부 변수 `sys_nerr` 및 `sys_errlist`가 정의돼 있기는 한데 `<stdio.h>`에 있다.

## SEE ALSO

<tt>[[err(3)]]</tt>, <tt>[[errno(3)]]</tt>, <tt>[[error(3)]]</tt>, <tt>[[strerror(3)]]</tt>

----

2019-03-06
