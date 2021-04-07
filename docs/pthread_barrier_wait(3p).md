## PROLOG

이 매뉴얼 페이지는 POSIX 프로그래머 매뉴얼의 일부이다. 이 인터페이스의 리눅스 구현에 차이가 있을 수 있으며 (상세한 리눅스 동작 방식은 해당 리눅스 매뉴얼 페이지 참고) 리눅스에서 이 인터페이스가 구현되어 있지 않을 수도 있다.

## NAME

pthread_barrier_wait - 배리어에서 동기화

## SYNOPSIS

```c
#include <pthread.h>

int pthread_barrier_wait(pthread_barrier_t *barrier);
```

## DESCRIPTION

`pthread_barrier_wait()` 함수는 `barrier`가 가리키는 배리어에서 참여 스레드들을 동기화 시킨다. 지정한 수의 스레드가 그 배리어를 지정해서 `pthread_barrier_wait()`을 호출할 때까지는 호출 스레드가 블록 하게 된다.

지정한 수의 스레드가 그 배리어를 지정해서 `pthread_barrier_wait()`을 호출했을 때 한 불특정 스레드에게는 상수 `PTHREAD_BARRIER_SERIAL_THREAD`가 반환되고 나머지 스레드 각각에게는 0이 반환된다. 그 시점에 배리어는 그 배리어를 참조한 가장 최근 `pthread_barrier_init()` 함수 호출 결과와 같은 상태로 재설정 된다.

상수 `PTHREAD_BARRIER_SERIAL_THREAD`가 `<pthread.h>`에 정의되어 있으며 그 값은 `pthread_barrier_wait()`이 반환하는 다른 어떤 값과도 다르다.

초기화 안 된 배리어로 이 함수를 호출하는 경우의 결과는 규정되어 있지 않다.

배리어에 블록 되어 있는 스레드에게 시그널이 전달되는 경우 시그널 핸들러 반환 시에 배리어 대기가 완료되지 않았으면 (즉 시그널 핸들러 실행 동안에 지정한 수의 스레드가 배리어에 도달하지 못했으면) 스레드가 계속해서 배리어에서 대기한다. 아니라면 그 완료된 배리어 대기로부터 스레드가 정상적으로 실행을 이어 간다. 스레드가 시그널 핸들러에서 반환하기 전에 모든 스레드가 배리어에 도달했을 때 다른 스레드가 배리어를 지나 진행할 수 있는지 여부는 명세되어 있지 않다.

배리어에서 블록 하는 스레드가 같은 프로세싱 자원을 사용할 자격이 있는 어느 논블록 스레드에 대해서도 그 실행이 최종적으로 전진하지 못하게 막지 않는다. 프로세싱 자원에 대한 자격은 스케줄링 정책에 따라 정해진다.

## RETURN VALUE

성공 완료 시 `pthread_barrier_wait()` 함수는 배리어에 동기화 된 (임의의) 한 스레드에게 `PTHREAD_BARRIER_SERIAL_THREAD`를 반환하고 다른 스레드들 각각에게 0을 반환한다. 아니면 오류를 나타내는 오류 번호를 반환한다.

## ERRORS

이 함수는 오류 코드 `[EINTR]`을 반환하지 않는다.

<em>이하는 규범적이지 않은 내용이다.</em>

## EXAMPLES

없음.

## APPLICATION USAGE

이 함수를 쓰는 응용에서 POSIX.1-2008 Base Definitions 권의 <em>3.287절 Priority Inversion</em>에서 논의하는 우선순위 역전을 겪을 수 있다.

## RATIONALE

`pthread_barrier_wait()`의 `barrier` 인자로 지정한 값이 초기화 된 배리어 객체를 가리키고 있지 않음을 구현에서 감지하는 경우 함수를 실패 처리하고 `[EINVAL]` 오류를 보고하기를 권장한다.

## FUTURE DIRECTIONS

없음.

## SEE ALSO

<tt>[[pthread_barrier_destroy()|pthread_barrier_destroy(3p)]]</tt>

POSIX.1-2008 Base Definitions 권, <em>3.287절 Priority Inversion</em>, <em>4.11절 Memory Synchronization</em>, `<pthread.h>`

## COPYRIGHT

Portions of this text are reprinted and reproduced in electronic form from IEEE Std 1003.1, 2013 Edition, Standard for Information Technology -- Portable Operating System Interface (POSIX), The Open Group Base Specifications Issue 7, Copyright (C) 2013 by the Institute of Electrical and Electronics Engineers, Inc and The Open Group. (This is POSIX.1-2008 with the 2013 Technical Corrigendum 1 applied.) In the event of any discrepancy between this version and the original IEEE and The Open Group Standard, the original IEEE and The Open Group Standard is the referee document. The original Standard can be obtained online at http://www.unix.org/online.html .

Any typographical or formatting errors that appear in this page are most likely to have been introduced during the conversion of the source files to man page format. To report such errors, see https://www.kernel.org/doc/man-pages/reporting_bugs.html .

----

2013
