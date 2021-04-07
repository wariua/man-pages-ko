## NAME

pthread_attr_setstackaddr, pthread_attr_getstackaddr - 스레드 속성 객체의 스택 주소 속성 설정하기/얻기

## SYNOPSIS

```c
#include <pthread.h>

int pthread_attr_setstackaddr(pthread_attr_t *attr, void *stackaddr);
int pthread_attr_getstackaddr(const pthread_attr_t *attr, void **stackaddr);
```

`-pthread`로 컴파일 및 링크.

## DESCRIPTION

이 함수들은 구식이다. <strong>쓰지 마라.</strong> 대신 <tt>[[pthread_attr_setstack(3)]]</tt> 및 <tt>[[pthread_attr_getstack(3)]]</tt>을 사용하라.

`pthread_attr_setstackaddr()` 함수는 `attr`이 가리키는 스레드 속성 객체의 스택 주소 속성을 `stackaddr`에 지정한 값으로 설정한다. 이 속성은 스레드 속성 객체 `attr`을 이용해 생성하는 스레드에서 사용할 스택의 위치를 나타낸다.

`stackaddr`은 호출자가 할당한 최소 `PTHREAD_STACK_MIN` 바이트짜리 버퍼를 가리켜야 한다. 할당한 버퍼의 페이지들은 읽기와 쓰기가 모두 가능해야 한다.

`pthread_attr_getstackaddr()` 함수는 `attr`이 가리키는 스레드 속성 객체의 스택 주소 속성을 `stackaddr`이 가리키는 버퍼로 반환한다.

## RETURN VALUE

성공 시 이 함수들은 0을 반환한다. 오류 시 0 아닌 오류 번호를 반환한다.

## ERRORS

아무 오류도 규정되어 있지 않다. (그렇기는 하지만 응용에서는 가능한 오류 반환을 처리해야 할 것이다.)

## VERSIONS

glibc 버전 2.1부터 이 함수들을 제공한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `pthread_attr_setstackaddr()`,<br>`pthread_attr_getstackaddr()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001에서 이 함수들을 명세하되 구식으로 표시하였다. POSIX.1-2008에서 이 함수들의 명세를 제거하였다.

## NOTES

<em>이 함수들을 쓰지 마라!</em> 스택 성장 방향이나 범위를 지정할 방법이 없기 때문에 이식성 있게 사용할 수 없다. 예를 들어 스택이 아래로 자라는 아키텍처에서 `stackaddr`은 할당한 스택 영역의 <em>최상위</em> 주소 다음 주소를 나타낸다. 하지만 스택이 위로 자라는 아키텍처에서 `stackaddr`은 할당한 스택 영역의 <em>최하위</em> 주소를 나타낸다. 반면 <tt>[[pthread_attr_setstack(3)]]</tt> 및 <tt>[[pthread_attr_getstack(3)]]</tt>에서 쓰는 `stackaddr`은 항상 할당한 스택 영역의 최하위 주소에 대한 포인터이다. (그리고 `stacksize` 인자가 스택의 범위를 지정한다.)

## SEE ALSO

<tt>[[pthread_attr_init(3)]]</tt>, <tt>[[pthread_attr_setstack(3)]]</tt>, <tt>[[pthread_attr_setstacksize(3)]]</tt>, <tt>[[pthread_create(3)]]</tt>, <tt>[[pthreads(7)]]</tt>

----

2017-09-15
