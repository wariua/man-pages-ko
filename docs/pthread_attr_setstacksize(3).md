## NAME

pthread_attr_setstacksize, pthread_attr_getstacksize - 스레드 속성 객체의 스택 크기 속성 설정하기/얻기

## SYNOPSIS

```c
#include <pthread.h>

int pthread_attr_setstacksize(pthread_attr_t *attr, size_t stacksize);
int pthread_attr_getstacksize(const pthread_attr_t *attr, size_t *stacksize);
```

`-pthread`로 컴파일 및 링크.

## DESCRIPTION

`pthread_attr_setstacksize()` 함수는 `attr`이 가리키는 스레드 속성 객체의 스택 크기 속성을 `stacksize`에 지정한 값으로 설정한다.

스택 크기 속성은 스레드 속성 객체 `attr`을 이용해 생성하는 스레드에 할당될 (바이트 단위) 최소 크기를 결정한다.

`pthread_attr_getstacksize()` 함수는 `attr`이 가리키는 스레드 속성 객체의 스택 크기 속성을 `stacksize`가 가리키는 버퍼로 반환한다.

## RETURN VALUE

성공 시 이 함수들은 0을 반환한다. 오류 시 0 아닌 오류 번호를 반환한다.

## ERRORS

`pthread_attr_setstacksize()`가 다음 오류로 실패할 수 있다.

<dl>
<dt><code>EINVAL</code></dt>
<dd>스택 크기가 <code>PTHREAD_STACK_MIN</code>(16384)바이트보다 작다.</dd>
</dl>

일부 시스템에서는 <code>stacksize</code>가 시스템 페이지 크기의 배수가 아닌 경우에 `pthread_attr_setstacksize()`가 `EINVAL` 오류로 실패할 수 있다.

## VERSIONS

glibc 버전 2.1부터 이 함수들을 제공한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `pthread_attr_setstacksize()`,<br>`pthread_attr_getstacksize()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## NOTES

새 스레드의 기본 스택 크기에 대한 자세한 내용은 <tt>[[pthread_create(3)]]</tt>를 보라.

스레드의 스택 크기는 스레드 생성 시점에 고정된다. 메인 스레드만 자기 스택을 동적으로 늘일 수 있다.

응용에서 <tt>[[pthread_attr_setstack(3)]]</tt> 함수를 이용하면 스레드가 사용할 호출자 할당 스택의 크기와 위치 모두를 설정할 수 있다.

## BUGS

glibc 2.8 기준으로 지정한 `stacksize`가 `STACK_ALIGN`(대부분 아키텍처에서 16바이트)의 배수가 아니면 크기가 <em>내림</em> 될 수 있다. 이는 할당되는 스택이 최소 `stacksize` 바이트가 된다고 하는 POSIX.1을 위반하는 것이다.

## EXAMPLE

<tt>[[pthread_create(3)]]</tt> 참고.

## SEE ALSO

<tt>[[getrlimit(2)]]</tt>, <tt>[[pthread_attr_init(3)]]</tt>, <tt>[[pthread_attr_setguardsize(3)]]</tt>, <tt>[[pthread_attr_setstack(3)]]</tt>, <tt>[[pthread_create(3)]]</tt>, <tt>[[pthreads(7)]]</tt>

----

2017-09-15
