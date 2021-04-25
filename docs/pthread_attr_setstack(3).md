## NAME

pthread_attr_setstack, pthread_attr_getstack - 스레드 속성 객체의 스택 속성 설정하기/얻기

## SYNOPSIS

```c
#include <pthread.h>

int pthread_attr_setstack(pthread_attr_t *attr,
                          void *stackaddr, size_t stacksize);
int pthread_attr_getstack(const pthread_attr_t *restrict attr,
                          void **restrict stackaddr,
                          size_t *restrict stacksize);
```

`-pthread`로 컴파일 및 링크.

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`pthread_attr_getstack()`, `pthread_attr_setstack()`:
:   `_POSIX_C_SOURCE >= 200112L`

## DESCRIPTION

`pthread_attr_setstack()` 함수는 `attr`이 가리키는 스레드 속성 객체의 스택 주소 및 스택 크기 속성을 각각 `stackaddr` 및 `stacksize`에 지정한 값들로 설정한다. 이 속성들은 스레드 속성 객체 `attr`을 이용해 생성하는 스레드에서 사용할 스택의 위치와 크기를 나타낸다.

`stackaddr`은 호출자가 할당한 `stacksize` 바이트짜리 버퍼에서 가장 아래 주소의 바이트를 가리켜야 한다. 할당한 버퍼의 페이지들은 읽기와 쓰기가 모두 가능해야 한다.

`pthread_attr_getstack()` 함수는 `attr`이 가리키는 스레드 속성 객체의 스택 주소 및 스택 크기 속성을 각각 `stackaddr` 및 `stacksize`가 가리키는 버퍼로 반환한다.

## RETURN VALUE

성공 시 이 함수들은 0을 반환한다. 오류 시 0 아닌 오류 번호를 반환한다.

## ERRORS

`pthread_attr_setstack()`이 다음 오류로 실패할 수 있다.

`EINVAL`
:   `stacksize`가 `PTHREAD_STACK_MIN`(16384)바이트보다 작다. 일부 시스템에서는 `stackaddr`이나 `stackaddr + stacksize`가 올바로 정렬되어 있지 않은 경우에도 이 오류가 발생할 수 있다.

POSIX.1에서는 `stackaddr` 및 `stacksize`가 기술하는 스택이 호출자에게 읽기와 쓰기 모두 가능하지 않은 경우 `EACCES` 오류도 적고 있다.

## VERSIONS

glibc 버전 2.2부터 이 함수들을 제공한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `pthread_attr_setstack()`,<br>`pthread_attr_getstack()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## NOTES

이 함수들은 스레드의 스택이 특정 위치에 있도록 해야 하는 응용들을 위한 것이다. 대부분 응용에서는 그럴 필요가 없으므로 이 함수들을 쓰지 않는 게 좋다. (응용에서 필요한 게 기본과 다른 스택 크기일 뿐이라면 <tt>[[pthread_attr_setstacksize(3)]]</tt>를 쓰면 된다.)

응용에서 `pthread_attr_setstack()`을 이용할 때는 스택을 할당할 책임을 함께 가져간다. <tt>[[pthread_attr_setguardsize(3)]]</tt>로 설정한 방호 구역 크기 값이 무시된다. 필요시 스택 오버플로 가능성에 대처하기 위해 방호 구역(읽기 및 쓰기가 안 되게 보호된 한 개 이상의 페이지)을 할당하는 것은 응용의 몫이다.

`stackaddr`로 지정하는 주소가 적절히 정렬되어 있는 게 좋다. 완전한 이식성을 위해선 페이지 경계(`sysconf(_SC_PAGESIZE)`)에 맞춰 정렬하면 된다. 할당에 <tt>[[posix_memalign(3)]]</tt>이 도움이 될 수 있다. `stacksize` 역시도 시스템 페이지 크기의 배수로 하는 게 좋을 것이다.

`attr`을 사용해 여러 스레드를 생성하는 경우에 호출자는 <tt>[[pthread_create(3)]]</tt> 호출들 간에 스택 주소 속성을 바꿔 주어야 한다. 안 그러면 여러 스레드가 같은 메모리 구역을 스택으로 쓰려고 하면서 혼란이 뒤따를 것이다.

## EXAMPLES

<tt>[[pthread_attr_init(3)]]</tt> 참고.

## SEE ALSO

<tt>[[mmap(2)]]</tt>, <tt>[[mprotect(2)]]</tt>, <tt>[[posix_memalign(3)]]</tt>, <tt>[[pthread_attr_init(3)]]</tt>, <tt>[[pthread_attr_setguardsize(3)]]</tt>, <tt>[[pthread_attr_setstackaddr(3)]]</tt>, <tt>[[pthread_attr_setstacksize(3)]]</tt>, <tt>[[pthread_create(3)]]</tt>, <tt>[[pthreads(7)]]</tt>

----

2021-03-22
