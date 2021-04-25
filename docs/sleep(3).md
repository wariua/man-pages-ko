## NAME

sleep - 지정한 초 동안 잠들기

## SYNOPSIS

```c
#include <unistd.h>

unsigned int sleep(unsigned int seconds);
```

## DESCRIPTION

`sleep()`은 실제 시간으로 `seconds`에 지정한 초가 지날 때까지, 또는 무시 안 되는 시그널을 받을 때까지 호출 스레드가 잠들게 만든다.

## RETURN VALUE

요청한 시간이 지났으면 0을 반환한다. 시그널 핸들러 때문에 호출이 중단됐으면 남은 초 수를 반환한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `sleep()` | 스레드 안전성 | MT-Unsafe sig:SIGCHLD/linux |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## NOTES

리눅스에서는 <tt>[[nanosleep(2)]]</tt>을 통해 `sleep()`이 구현돼 있다. <tt>[[nanosleep(2)]]</tt> 맨 페이지의 사용 클럭에 대한 논의를 보라.

### 이식성 관련 사항

일부 시스템에서는 <tt>[[alarm(2)]]</tt>과 `SIGALRM`을 사용해 `sleep()`이 구현돼 있을 수 있다. (POSIX.1에서 이를 허용한다.) <tt>[[alarm(2)]]</tt>과 `sleep()`을 섞어서 호출하는 건 안 좋은 생각이다.

잠이 들어 있는 동안 시그널 핸들러에서 <tt>[[longjmp(3)]]</tt>를 쓰거나 `SIGALRM` 처리 방식을 변경하면 규정 안 된 결과가 일어난다.

## SEE ALSO

`sleep(1)`, <tt>[[alarm(2)]]</tt>, <tt>[[nanosleep(2)]]</tt>, <tt>[[signal(2)]]</tt>, <tt>[[signal(7)]]</tt>

----

2021-03-22
