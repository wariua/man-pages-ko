## NAME

assert - 진술이 거짓이면 프로그램 중단하기

## SYNOPSIS

```c
#include <assert.h>

void assert(scalar expression);
```

## DESCRIPTION

이 매크로는 제한된 디버깅 출력을 내놓는 크래시를 통해 프로그래머가 자기 프로그램의 버그를 찾거나 예외적 경우를 다루는 걸 돕는다.

`expression`이 거짓이면 (즉 0과 비교해서 같으면) 표준 오류로 오류 메시지를 찍고 <tt>[[abort(3)]]</tt> 호출로 프로그램을 종료시킨다. 오류 메시지에는 `assert()` 호출이 있는 파일과 함수의 이름, 호출의 소스 코드 행 번호, 인자 텍스트가 포함된다. 다음과 같은 식이다.

```text
prog: some_file.c:16: some_func: Assertion `val == 0' failed.
```

`<assert.h>`를 마지막으로 포함하는 시점에 매크로 `NDEBUG`가 정의돼 있으면 `assert()` 매크로가 아무 코드도 만들지 않고, 그래서 아무것도 하지 않는다. 오류 조건 탐지에 `assert()`를 쓰는 경우에는 `NDEBUG`를 정의하지 않기를 권한다. 소프트웨어가 비결정적으로 동작할 수 있기 때문이다.

## RETURN VALUE

아무 값도 반환하지 않는다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `assert()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, C89, C99. C89에서는 `expression`이 `int` 타입이어야 하고 아닌 경우 동작 결과가 규정되어 있지 않다. 하지만 C99에서는 어떤 스칼라 타입도 될 수 있다.

## BUGS

`assert()`가 매크로로 구현되어 있다. 검사하는 식에 부대 효과가 있으면 `NDEBUG`가 정의돼 있는지 여부에 따라 프로그램 동작이 달라질 수도 있다. 이 때문에 디버깅을 켜면 사라져 버리는 하이젠버그가 생길 수 있다.

## SEE ALSO

<tt>[[abort(3)]]</tt>, <tt>[[assert_perror(3)]]</tt>, <tt>[[exit(3)]]</tt>

----

2021-03-22
