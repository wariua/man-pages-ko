## NAME

malloc_get_state, malloc_set_state - malloc 구현의 상태를 기록하고 복원하기

## SYNOPSIS

```c
#include <malloc.h>

void *malloc_get_state(void);
int malloc_set_state(void *state);
```

## DESCRIPTION

*주의*: 이 함수들은 glibc 버전 2.25에서 제거되었다.

`malloc_get_state()` 함수는 <tt>[[malloc(3)]]</tt> 내부 상태 관리 변수들 모두의 현재 상태를 (힙의 실제 내용이나 <tt>[[malloc_hook(3)]]</tt> 함수 포인터들의 상태는 제외) 기록한다. <tt>[[malloc(3)]]</tt>을 통해 동적으로 할당한 시스템 의존적인 불투명 자료 구조에 그 상태를 기록하고 그 자료 구조에 대한 포인터를 함수 결과로 반환한다. (그 메모리를 <tt>[[free(3)]]</tt> 하는 것은 호출자의 책임이다.)

`malloc_set_state()` 함수는 <tt>[[malloc(3)]]</tt> 내부 상태 관리 변수들 모두의 상태를 `state`가 가리키는 불투명 자료 구조에 기록된 값들로 복원한다.

## RETURN VALUE

성공 시 `malloc_get_state()`는 새로 할당된 불투명 자료 구조에 대한 포인터를 반환한다. 오류 시 (가령 그 자료 구조를 위한 메모리를 할당할 수 없으면) `malloc_get_state()`는 NULL을 반환한다.

성공 시 `malloc_set_state()`는 0을 반환한다. `state`가 올바른 형식의 자료 구조를 가리키고 있지 않다고 구현에서 탐지한 경우 `malloc_set_state()`가 -1을 반환한다. `state`가 가리키는 자료 구조의 버전이 구현에서 인식하는 것보다 최신이라고 탐지한 경우 `malloc_set_state()`가 -2를 반환한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `malloc_get_state()`,<br>`malloc_set_state()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

이 함수들은 GNU 확장이다.

## NOTES

<tt>[[malloc(3)]]</tt> 구현을 어느 공유 라이브러리의 구성 요소로 사용하며 힙 내용을 어떤 다른 방법으로 저장/복원하는 경우에 이 함수들이 유용하다. GNU 이맥스에서 이 기법을 이용해 "덤프" 기능을 구현한다.

이 함수들에서 절대 훅 함수 포인터들을 저장 내지 복원하지 않되 두 가지 예외가 있다. `malloc_get_state()`를 호출할 때 malloc 검사(<tt>[[mallopt(3)]]</tt> 참고)를 쓰고 있었으면 `malloc_set_state()`에서 가능한 경우 malloc 검사 훅들을 초기화 한다. 기록한 상태에서는 malloc 검사를 쓰고 있지 않았는데 호출자가 malloc 검사를 요청해 둔 경우에는 훅들이 0으로 초기화 된다.

## SEE ALSO

<tt>[[malloc(3)]]</tt>, <tt>[[mallopt(3)]]</tt>

----

2021-03-22
