## NAME

pthread_attr_setguardsize, pthread_attr_getguardsize - 스레드 속성 객체의 방호 구역 크기 설정하기/얻기

## SYNOPSIS

```c
#include <pthread.h>

int pthread_attr_setguardsize(pthread_attr_t *attr, size_t guardsize);
int pthread_attr_getguardsize(const pthread_attr_t *restrict attr,
                              size_t *restrict guardsize);
```

`-pthread`로 컴파일 및 링크.

## DESCRIPTION

`pthread_attr_setguardsize()` 함수는 `attr`이 가리키는 스레드 속성 객체의 방호 구역 크기 속성을 `guardsize`에 지정한 값으로 설정한다.

`guardsize`가 0보다 크면 `attr`을 이용해 생성하는 새 스레드 각각에 대해 시스템에서 스레드 스택 끝에 최소 `guardsize` 바이트의 영역을 추가로 할당해서 스택의 방호 구역 역할을 하게 한다. (하지만 BUGS 참고.)

`guardsize`가 0이면 `attr`로 생성하는 새 스레드에 방호 구역이 없게 된다.

기본 방호 구역 크기는 시스템 페이지 크기와 같다.

`attr`에 (<tt>[[pthread_attr_setstack(3)]]</tt>이나 <tt>[[pthread_attr_setstackaddr(3)]]</tt>로) 스택 주소 속성이 설정되어 스레드의 스택을 호출자가 할당하는 경우에는 방호 구역 크기 속성을 무시한다. (즉 시스템에서 어떤 방호 구역도 만들지 않는다.) 응용이 스택 오버플로우 처리를 책임진다. (<tt>[[mprotect(2)]]</tt>를 이용해 할당한 스택 끝에 방호 구역을 수동으로 지정할 수 있을 것이다.)

`pthread_attr_getguardsize()` 함수는 `attr`이 가리키는 스레드 속성 객체의 방호 구역 크기 속성을 `guardsize`가 가리키는 버퍼로 반환한다.

## RETURN VALUE

성공 시 이 함수들은 0을 반환한다. 오류 시 0 아닌 오류 번호를 반환한다.

## ERRORS

POSIX.1에서는 `attr`이나 `guardsize`가 유효하지 않은 경우 `EINVAL` 오류를 적고 있다. 리눅스에서는 이 함수들이 항상 성공한다. (그렇기는 하지만 이식 가능하고 미래를 대비하는 응용에서는 가능한 오류 반환을 처리해야 할 것이다.)

## VERSIONS

glibc 버전 2.1부터 이 함수들을 제공한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `pthread_attr_setguardsize()`,<br>`pthread_attr_getguardsize()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## NOTES

방호 구역은 읽기 및 쓰기 접근을 막도록 보호된 가상 메모리 페이지들로 이뤄진다. 스레드가 그 방호 구역으로 스택을 넘치게 하면 대부분의 하드웨어 아키텍처에서 오버플로우를 알리는 `SIGSEGV` 시그널을 받는다. 방호 구역은 페이지 경계에서 시작하며 스레드 생성 시에 내부적으로 방호 구역 크기를 시스템 페이지 크기로 올림 한다. (그렇게 해도 `pthread_attr_getguardsize()`는 `pthread_attr_setguardsize()`로 설정했던 방호 구역 크기를 반환한다.)

스레드를 많이 생성하며 스택 오버플로우가 절대 발생하지 않는 걸 아는 응용에서는 방호 구역 크기를 0으로 설정하는 게 메모리 절약에 도움이 될 수 있다.

스레드가 스택에 큰 자료 구조를 할당하는 경우 스택 오버플로우 탐지를 위해 기본보다 큰 방호 구역 크기를 택하는 것이 필요할 수 있다.

## BUGS

glibc 2.8 기준으로 NPTL 스레딩 구현에서는 POSIX.1의 요구대로 스택 끝에 추가 공간을 할당하지 않고 스택 할당 크기에 방호 구역 크기를 포함한다. (그래서 방호 구역 크기 값이 아주 커서 실제 스택을 위한 공간이 없는 경우 <tt>[[pthread_create(3)]]</tt>에서 `EINVAL` 오류가 발생할 수 있다.)

구식 LinuxThreads 구현에서는 올바로 스택 끝에 방호 구역을 위한 추가 공간을 할당한다.

## EXAMPLES

<tt>[[pthread_getattr_np(3)]]</tt> 참고.

## SEE ALSO

<tt>[[mmap(2)]]</tt>, <tt>[[mprotect(2)]]</tt>, <tt>[[pthread_attr_init(3)]]</tt>, <tt>[[pthread_attr_setstack(3)]]</tt>, <tt>[[pthread_attr_setstacksize(3)]]</tt>, <tt>[[pthread_create(3)]]</tt>, <tt>[[pthreads(7)]]</tt>

----

2021-03-22
