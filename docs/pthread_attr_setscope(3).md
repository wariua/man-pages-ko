## NAME

pthread_attr_setscope, pthread_attr_getscope - 스레드 속성 객체의 경합 범위 속성 설정하기/얻기

## SYNOPSIS

```c
#include <pthread.h>

int pthread_attr_setscope(pthread_attr_t *attr, int scope);
int pthread_attr_getscope(const pthread_attr_t *attr, int *scope);
```

`-pthread`로 컴파일 및 링크.

## DESCRIPTION

`pthread_attr_setscope()` 함수는 `attr`이 가리키는 스레드 속성 객체의 경합 범위 속성을 `scope`에 지정한 값으로 설정한다. 경합 범위 속성은 스레드가 CPU 같은 자원을 두고 경쟁하는 스레드들의 집합을 규정한다. POSIX.1에서 `scope`에 가능한 값을 두 가지 명세한다.

<dl>
<dt><code>PTHREAD_SCOPE_SYSTEM</code></dt>
<dd>
스레드가 같은 스케줄링 도메인(한 개 이상 프로세서들의 그룹)에 있는 시스템의 모든 프로세스의 다른 모든 스레드들과 자원을 두고 경쟁한다. <code>PTHREAD_SCOPE_SYSTEM</code> 스레드들은 각자의 스케줄링 정책과 우선순위에 따라 서로에 대해서 스케줄 된다.
</dd>

<dt><code>PTHREAD_SCOPE_PROCESS</code></dt>
<dd>
스레드가 마찬가지로 경합 범위 <code>PTHREAD_SCOPE_PROCESS</code>로 생성한 동일 프로세스의 다른 모든 스레드들과 자원을 두고 경쟁한다. <code>PTHREAD_SCOPE_PROCESS</code> 스레드들은 각자의 스케줄링 정책과 우선순위에 따라 프로세스의 다른 스레드들에 대해서 스케줄 된다. POSIX.1에서는 이 스레드들이 시스템의 다른 프로세스의 스레드들이나 같은 프로세스에 있지만 경합 범위 <code>PTHREAD_SCOPE_SYSTEM</code>으로 생성한 스레드들과 어떻게 경쟁하는지를 명세 안 된 것으로 남겨 둔다.
</dd>
</dl>

POSIX.1에서는 구현체가 이 경합 범위들 중 최소 한 가지를 지원하기를 요구한다. 리눅스는 `PTHREAD_SCOPE_SYSTEM`을 지원하며 `PTHREAD_SCOPE_PROCESS`는 지원하지 않는다.

여러 경합 범위를 지원하는 시스템에서 `pthread_attr_setscope()`로 설정한 매개변수가 <tt>[[pthread_create(3)]]</tt> 호출 때 효과가 있으려면 호출자가 <tt>[[pthread_attr_setinheritsched(3)]]</tt>을 이용해 속성 객체 `attr`의 스케줄러 상속 속성을 `PTHREAD_EXPLICIT_SCHED`로 설정해야 한다.

`pthread_attr_getscope()` 함수는 `attr`이 가리키는 스레드 속성 객체의 경합 범위 속성을 `scope`가 가리키는 버퍼로 반환한다.

## RETURN VALUE

성공 시 이 함수들은 0을 반환한다. 오류 시 0 아닌 오류 번호를 반환한다.

## ERRORS

`pthread_attr_setscope()`가 다음 오류로 실패할 수 있다.

<dl>
<dt><code>EINVAL</code></dt>
<dd><code>scope</code>에 유효하지 않은 값을 지정했다.</dd>
<dt><code>ENOTSUP</code></dt>
<dd><code>scope</code>에 리눅스에서 지원하지 않는 <code>PTHREAD_SCOPE_PROCESS</code> 값을 지정했다.</dd>
</dl>

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `pthread_attr_setscope()`,<br>`pthread_attr_getscope()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## NOTES

경합 범위 `PTHREAD_SCOPE_SYSTEM`은 보통 사용자 공간 스레드가 한 개의 커널 스케줄링 개체에 직접 결합되어 있음을 나타낸다. 리눅스가 그런 경우인데, 구식이 된 LinuxThreads 구현과 최신 NPTL 구현 모두가 1:1 스레딩 구현이다.

POSIX.1에서는 기본 경합 범위를 구현에서 정의하는 것으로 명세한다.

## SEE ALSO

<tt>[[pthread_attr_init(3)]]</tt>, <tt>[[pthread_attr_setaffinity_np(3)]]</tt>, <tt>[[pthread_attr_setinheritsched(3)]]</tt>, <tt>[[pthread_attr_setschedparam(3)]]</tt>, <tt>[[pthread_attr_setschedpolicy(3)]]</tt>, <tt>[[pthread_create(3)]]</tt>, <tt>[[pthreads(7)]]</tt>

----

2017-09-15
