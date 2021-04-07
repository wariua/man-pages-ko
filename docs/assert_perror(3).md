## NAME

assert_perror - 오류 번호 검사해서 중단하기

## SYNOPSIS

```c
#define _GNU_SOURCE         /* feature_test_macros(7) 참고 */
#include <assert.h>

void assert_perror(int errnum);
```

## DESCRIPTION

`<assert.h>`를 마지막으로 포함하는 시점에 매크로 `NDEBUG`가 정의되어 있으면 `assert_perror()` 매크로가 아무 코드도 만들지 않고, 그래서 아무것도 하지 않는다. 그 외의 경우에는 `errnum`이 0이 아닌 경우에 `assert_perror()` 매크로가 표준 오류로 오류 메시지를 찍고 <tt>[[abort(3)]]</tt> 호출로 프로그램을 종료시킨다. 메시지에는 매크로 호출이 있는 파일명, 함수명, 행 번호, 그리고 `strerror(errnum)`의 출력이 담긴다.

## RETURN VALUE

아무 값도 반환하지 않는다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `assert_perror()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

GNU 확장이다.

## BUGS

assert 매크로의 목적은 프로그래머가 자기 프로그램에서 코딩 실수가 아니면 발생할 수 없는 종류의 버그들을 찾는 데 도움을 주는 것이다. 하지만 시스템 호출이나 라이브러리 호출에서는 상황이 좀 달라서, 오류 반환이 일어날 수 있고, 일어날 것이고, 그래서 검사를 해야 한다. `NDEBUG`가 정의되어 있으면 검사가 사라지는 assert를 통해서서가 아니라 올바른 오류 처리 코드를 통해서 말이다. 절대 이 매크로를 쓰지 마라.

## SEE ALSO

<tt>[[abort(3)]]</tt>, <tt>[[assert(3)]]</tt>, <tt>[[exit(3)]]</tt>, <tt>[[strerror(3)]]</tt>

----

2017-09-15
