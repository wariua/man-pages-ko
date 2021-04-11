## NAME

\_exit, \_Exit - 호출 프로세스 종료시키기

## SYNOPSIS

```c
#include <unistd.h>

void _exit(int status);

#include <stdlib.h>

void _Exit(int status);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`_Exit()`:
:   `_ISOC99_SOURCE || _POSIX_C_SOURCE >= 200112L`

## DESCRIPTION

`_exit()` 함수는 호출 프로세스를 "즉시" 종료시킨다. 그 프로세스에게 속한 열린 파일 디스크립터들이 있으면 모두 닫는다. 그 프로세스의 자식들이 있으면 `init(1)`에게 (또는 <tt>[[prctl(2)]]</tt> `PR_SET_CHILD_SUBREAPER` 동작을 통해 지정한 가장 가까운 "서브리퍼" 프로세스에게) 물려준다. 그 프로세스의 부모에게 `SIGCHLD` 시그널을 보낸다.

부모 프로세스에게 프로세스 종료 상태로 `status & 0377` 값을 반환하며, <tt>[[wait(2)]]</tt> 계열 함수들 중 하나를 이용해 그 값을 수집할 수 있다.

`_Exit()` 함수는 `_exit()`와 동등하다.

## RETURN VALUE

이 함수들은 반환하지 않는다.

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, SVr4, 4.3BSD. `_Exit()` 함수는 C99에서 도입되었다.

## NOTES

종료의 효과, 종료 상태의 전달, 좀비 프로세스, 전송 시그널 등에 대한 논의는 <tt>[[exit(3)]]</tt>을 보라.

`_exit()` 함수는 <tt>[[exit(3)]]</tt>과 비슷하되 <tt>[[atexit(3)]]</tt>이나 <tt>[[on_exit(3)]]</tt>으로 등록해 둔 함수들을 호출하지 않는다. 그리고 열린 <tt>[[stdio(3)]]</tt> 스트림들을 플러시 하지 않는다. 한편으로 `_exit()`는 열린 파일 디스크립터들은 확실히 닫는다. 이로 인해 미처리 출력이 끝나기를 기다리면서 알 수 없는 지연이 생길 수도 있다. 그런 지연을 원치 않는다면 `_exit()` 호출 전에 <tt>[[tcflush(3)]]</tt> 같은 함수를 호출해 볼 수도 있다. `_exit()`에서 미처리 I/O를 취소하는지, 그리고 어떤 미처리 I/O를 취소할 수 있는지는 구현에 따라 다르다.

### C 라이브러리/커널 차이

glibc 버전 2.3 전까지는 `_exit()` 래퍼 함수가 같은 이름의 커널 시스템 호출을 불렀다. glibc 2.3부터는 프로세스 내의 모든 스레드들을 종료시키도록 래퍼 함수가 <tt>[[exit_group(2)]]</tt>을 부른다.

## SEE ALSO

<tt>[[execve(2)]]</tt>, <tt>[[exit_group(2)]]</tt>, <tt>[[fork(2)]]</tt>, <tt>[[kill(2)]]</tt>, <tt>[[wait(2)]]</tt>, <tt>[[wait4(2)]]</tt>, <tt>[[waitpid(2)]]</tt>, <tt>[[atexit(3)]]</tt>, <tt>[[exit(3)]]</tt>, <tt>[[on_exit(3)]]</tt>, <tt>[[termios(3)]]</tt>

----

2017-05-03
