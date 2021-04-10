## NAME

pthread_setconcurrency, pthread_getconcurrency - 동시성 수준 설정하기/얻기

## SYNOPSIS

```c
#include <pthread.h>

int pthread_setconcurrency(int new_level);
int pthread_getconcurrency(void);
```

`-pthread`로 컴파일 및 링크.

## DESCRIPTION

`pthread_setconcurrency()` 함수는 `new_level`에 지정한 응용이 원하는 동시성 수준을 구현체에게 알린다. 구현체는 이를 힌트로만 받아들인다. POSIX.1에서는 `pthread_setconcurrency()` 호출의 결과로 제공되어야 하는 동시성 수준을 명세하고 있지 않다.

`new_value`를 0으로 지정하는 것은 구현체에게 적절하다 싶은 대로 동시성 수준을 관리하라고 지시하는 것이다.

`pthread_getconcurrency()`는 이 프로세스에 대한 현재의 동시성 수준 값을 반환한다.

## RETURN VALUE

성공 시 `pthread_setconcurrency()`는 0을 반환한다. 오류 시 0 아닌 오류 번호를 반환한다.

`pthread_getconcurrency()`는 항상 성공하며 앞선 `pthread_setconcurrency()` 호출로 설정한 동시성 수준을 반환한다. 앞서 `pthread_setconcurrency()`를 호출한 적이 없으면 0을 반환한다.

## ERRORS

`pthread_setconcurrency()`가 다음 오류로 실패할 수 있다.

`EINVAL`
:   `new_level`이 음수이다.

POSIX.1에서는 `EAGAIN` 오류("`new_value`로 지정한 값이 시스템 자원 초과를 일으키게 됨")도 적고 있다.

## VERSIONS

glibc 버전 2.1부터 이 함수들이 사용 가능하다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `pthread_setconcurrency()`,<br>`pthread_getconcurrency()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## NOTES

기본 동시성 수준은 0이다.

동시성 수준은 M:N 스레딩 구현에서만 의미가 있다. 그 방식에서는 한 시점에 프로세스의 사용자 수준 스레드들 중 일부가 더 적은 커널 스케줄링 개체들에 결속되어 있을 수 있다. 동시성 수준을 설정함으로써 효율적인 응용 실행을 위해 제공돼야 하는 커널 스케줄링 개체 수에 대한 힌트를 응용이 시스템에게 줄 수 있다.

LinuxThreads와 NPTL 모두 1:1 스레딩 구현이므로 동시성 수준 설정이 의미가 없다. 다시 말해 리눅스에서 이 함수들은 다른 시스템과의 호환성을 위해 있을 뿐이며 프로그램 실행에 어떤 영향도 주지 않는다.

## SEE ALSO

<tt>[[pthread_attr_setscope(3)]]</tt>, <tt>[[pthread(7)]]</tt>

----

2017-09-15
