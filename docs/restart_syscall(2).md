## NAME

restart_syscall - 정지 시그널로 중단된 후 시스템 호출 재시작 하기

## SYNOPSIS

```c
long restart_syscall(void);
```

*주의*: 이 시스템 호출에 대한 glibc 래퍼가 없다. NOTES 참고.

## DESCRIPTION

시그널(가령 `SIGSTOP`이나 `SIGTSTP`)에 의해 정지됐던 프로세스가 이후 `SIGCONT` 시그널을 수신하여 재개된 후에 특정 시스템 호출들을 재시작하기 위해 `restart_syscall()`을 사용한다. 이 시스템 호출은 커널 내부 용도로만 설계된 것이다.

재시작 때 시간 관련 매개변수를 조정해야 하는 시스템 호출들을 재시작 하는 데에만 `restart_syscall()`을 사용한다. 말하자면 <tt>[[poll(2)]]</tt> (리눅스 2.6.24부터), <tt>[[nanosleep(2)]]</tt> (리눅스 2.6부터), <tt>[[clock_nanosleep(2)]]</tt> (리눅스 2.6부터), 그리고 <tt>[[futex(2)]]</tt>를 `FUTEX_WAIT`(리눅스 2.6.22부터) 및 `FUTEX_WAIT_BITSET`(리눅스 2.6.31부터)으로 쓸 때이다. `restart_syscall()`은 (시그널에 의해 프로세스가 정지되어 있던 시간를 포함해) 이미 지난 시간을 고려해 적절히 조정한 시간 인자를 가지고 중단됐던 시스템 호출을 재시작 한다. `restart_syscall()` 메커니즘이 없다면 프로세스가 실행을 속행했을 때 이 시스템 호출들을 재시작 하면서 이미 지난 시간을 정확히 제하지 못할 것이다.

## RETURN VALUE

`restart_syscall()`의 반환 값은 재시작 하는 그 시스템 호출의 반환 값이다.

## ERRORS

`restart_syscall()`로 재시작 하는 그 시스템 호출에서의 오류에 따라서 `errno`가 설정된다.

## VERSIONS

리눅스 2.6부터 `restart_syscall()` 시스템 호출이 존재한다.

## CONFORMING TO

이 시스템 호출은 리눅스 전용이다.

## NOTES

이 시스템 호출에 대한 glibc 래퍼가 없다. 커널에서만 쓰기 위한 것이고 응용에서는 절대 호출하지 말아야 하는 호출이기 때문이다.

프로세스가 시그널에 의해 정지되었다가 `SIGCONT`에 의해 재개된 후 시스템 호출을 재시작할 때 프로세스가 정지 상태에서 보낸 시간이 원래 시스템 호출에 지정했던 타임아웃 시간에서 빠지도록 하기 위해 커널에서 `restart_syscall()`을 사용한다. 시스템 호출에서 타임아웃 인자를 받으며 정지 시그널 및 `SIGCONT` 후에 자동으로 재시작 하지만 `restart_syscall()` 메커니즘을 내장하고 있지 않은 경우에는 프로세스가 정지 상태로 보낸 시간이 타임아웃 값에서 빠지지 *않는다*. 이런 문제를 겪는 주요 시스템 호출들로 <tt>[[ppoll(2)]]</tt>, <tt>[[select(2)]]</tt>, <tt>[[pselect(2)]]</tt>가 있다.

사용자 공간에게는 `restart_syscall()`의 동작이 거의 보이지 않는다. 재시작 되는 시스템 호출을 불렀던 프로세스에게는 시스템 호출이 평소처럼 실행되고 반환한 것처럼 보인다.

## SEE ALSO

<tt>[[sigaction(2)]]</tt>, <tt>[[sigreturn(2)]]</tt>, <tt>[[signal(7)]]</tt>

----

2021-03-22
