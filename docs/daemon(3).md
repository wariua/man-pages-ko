## NAME

daemon - 배경에서 돌기

## SYNOPSIS

```c
#include <unistd.h>

int daemon(int nochdir, int noclose);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`daemon()`:
:   glibc 2.21부터:
    :   `_DEFAULT_SOURCE`

    glibc 2.19 및 2.20:
    :   `_DEFAULT_SOURCE || (_XOPEN_SOURCE && _XOPEN_SOURCE < 500)`

    glibc 2.19까지:
    :   `_BSD_SOURCE || (_XOPEN_SOURCE && _XOPEN_SOURCE < 500)`

## DESCRIPTION

`daemon()`은 제어 터미널에서 분리돼서 배경에서 시스템 데몬으로 돌고 싶은 프로그램을 위한 함수다.

`nochdir`이 0이면 프로세스의 현재 작업 디렉터리를 루트 디렉터리("/")로 바꾼다. 아니면 현재 작업 디렉터리를 그대로 둔다.

`noclose`가 0이면 표준 입력, 표준 출력, 표준 오류가 `/dev/null`을 가리키게 만든다. 아니면 그 파일 디스크립터들에 아무 변화도 일으키지 않는다.

## RETURN VALUE

(이 함수에서 포크를 하고 <tt>[[fork(2)]]</tt> 성공 시 부모가 <tt>[[_exit(2)]]</tt>를 호출하므로 이후의 오류는 자식에게만 보인다.) 성공 시 `daemon()`이 0을 반환한다. 오류 시 `daemon()`이 -1을 반환하며 <tt>[[fork(2)]]</tt> 및 <tt>[[setsid(2)]]</tt>에 명세된 오류들 중 하나로 `errno`를 설정한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `daemon()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1에 없다. BSD에서 비슷한 함수가 보인다. 4.4BSD에서 `daemon()` 함수가 처음 등장했다.

## NOTES

glibc 구현에서는 `/dev/null`이 존재하지만 기대하는 주번호 및 부번호의 문자 장치가 아닌 경우에도 -1을 반환할 수 있다. 이 경우 `errno`가 설정돼 있지 않을 수 있다.

## BUGS

GNU C 라이브러리의 이 함수 구현은 BSD에서 가져온 것이며, 최종 데몬 프로세스가 세션 리더가 되지 않게 하기 위해 필요한 이중 포크 기법(<tt>[[fork(2)]]</tt>, <tt>[[setsid(2)]]</tt>, <tt>[[fork(2)]]</tt>)을 쓰지 않는다. 즉, 최종 데몬이 세션 리더가 된다. 그래서 시스템 V 동작 방식을 따르지 않는 시스템(예: 리눅스)에서는 다른 세션의 제어 터미널이 아닌 어떤 터미널을 데몬이 열면 그 터미널이 의도치 않게 데몬의 제어 터미널이 된다.

## SEE ALSO

<tt>[[fork(2)]]</tt>, <tt>[[setsid(2)]]</tt>, `daemon(7)`, `logrotate(8)`

----

2021-03-22
