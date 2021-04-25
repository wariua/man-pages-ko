## NAME

abort - 비정상적 프로세스 종료 일으키기

## SYNOSIS

```c
#include <stdlib.h>

noreturn void abort(void);
```

## DESCRIPTION

`abort()` 함수는 먼저 `SIGABRT` 시그널을 차단 해제하고서 호출 프로세스에게 (<tt>[[raise(3)]]</tt>를 호출한 것처럼) 그 시그널을 발생시킨다. `SIGABRT` 시그널을 잡아서 시그널 핸들러가 반환하지 않는 (<tt>[[longjmp(3)]]</tt> 참고) 경우가 아니면 이로 인해 프로세스 비정상적 종료가 일어난다.

`SIGABRT` 시그널을 무시하거나 시그널을 잡은 핸들러가 반환하는 경우에도 `abort()` 함수가 프로세스를 종료시키게 된다. `SIGABRT`의 기본 처리 방식을 되살리고서 다시 그 시그널을 발생시키기 때문이다.

## RETURN VALUE

`abort()` 함수는 절대 반환하지 않는다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `abort()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

SVr4, POSIX.1-2001, POSIX.1-2008, 4.3BSD, C89, C99.

## NOTES

glibc 2.26까지에서는 `abort()` 함수 때문에 프로세스가 종료되면 열린 스트림들이 모두 (<tt>[[fclose(3)]]</tt> 한 것처럼) 닫히고 플러시 되었다. 하지만 어떤 경우에 그 때문에 교착과 데이터 오염이 발생할 수 있었다. 그래서 glibc 2.27부터는 `abort()`가 프로세스를 종료시키면서 스트림을 플러시 하지 않는다. POSIX.1에서는 어느 동작도 허용하는데, `abort()`가 "모든 열린 스트림에 <tt>[[fclose()]]</tt> 효과를 주려는 시도를 포함할 수도 있다"고 한다.

## SEE ALSO

<tt>[[gdb(1)]]</tt>, <tt>[[sigaction(2)]]</tt>, <tt>[[assert(3)]]</tt>, <tt>[[exit(3)]]</tt>, <tt>[[longjmp(3)]]</tt>, <tt>[[raise(3)]]</tt>

----

2021-03-22
