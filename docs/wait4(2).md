## NAME

wait3, wait4 - 프로세스 상태 변경 기다리기, BSD 방식

## SYNOPSIS

```c
#include <sys/types.h>
#include <sys/time.h>
#include <sys/resource.h>
#include <sys/wait.h>

pid_t wait3(int *wstatus, int options, struct rusage *rusage);
pid_t wait4(pid_t pid, int *wstatus, int options,
            struct rusage *rusage);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`wait3()`:
:   glibc 2.26부터:
    :   `_DEFAULT_SOURCE`<br>
        `    || (_XOPEN_SOURCE >= 500 &&`<br>
        `        ! (_POSIX_C_SOURCE >= 200112L`<br>
        `           || _XOPEN_SOURCE >= 600))`

    glibc 2.19부터 2.25까지:
    :   `_DEFAULT_SOURCE || _XOPEN_SOURCE >= 500`

    glibc 2.19 및 이전:
    :   `_BSD_SOURCE || _XOPEN_SOURCE >= 500`

`wait4()`:
:   glibc 2.19부터:
    :   `_DEFAULT_SOURCE`

    glibc 2.19 및 이전:
    :   `_BSD_SOURCE`

## DESCRIPTION

이 함수들은 비표준이다. 새로 만드는 프로그램에서는 <tt>[[waitpid(2)]]</tt>나 <tt>[[waitid(2)]]</tt>를 사용하는 게 바람직하다.

`wait3()` 및 `wait4()` 시스템 호출은 <tt>[[waitpid(2)]]</tt>와 비슷하되 추가로 `rusage`가 가리키는 구조체에 자식에 대한 자원 사용 정보를 반환한다.

`rusage` 인자 사용을 제외하면 다음 `wait3()` 호출은

```c
wait3(wstatus, options, rusage);
```

다음과 동등하다.

```c
waitpid(-1, wstatus, options);
```

마찬가지로 다음 `wait4()` 호출은

```c
wait4(pid, wstatus, options, rusage);
```

다음과 동등하다.

```c
waitpid(pid, wstatus, options);
```

다시 말해 `wait3()`는 아무 자식이나 기다리는 반면 `wait4()`를 쓰면 기다릴 특정 자식 내지 자식들을 선택할 수 있다. 더 자세한 내용은 <tt>[[wait(2)]]</tt>을 보라.

`rusage`가 NULL이 아니면 그 인자가 가리키는 `struct rusage`를 자식에 대한 사용량 정보로 채우게 된다. 자세한 내용은 <tt>[[getrusage(2)]]</tt>를 보라.

## RETURN VALUE

<tt>[[waitpid(2)]]</tt>와 같음.

## ERRORS

<tt>[[waitpid(2)]]</tt>와 같음.

## CONFORMING TO

4.3BSD.

SUSv1에 `wait3()` 명세가 포함되었다. SUSv2에 `wait3()`가 포함되었지만 LEGACY로 표시되었다. SUSv3에서 제거되었다.

## NOTES

`<sys/time.h>`를 포함시키는 것이 요즘은 필요하지 않지만 이식성을 높여 준다. (실제로 `<sys/resource.h>`에서 정의하는 `rusage` 구조체에는 `<sys/time.h>`에 정의돼 있는  `struct timeval` 타입 필드들이 있다.)

### C 라이브러리/커널 차이

리눅스에서 `wait3()`는 `wait4()` 시스템 호출 위에서 구현한 라이브러리 함수이다.

## SEE ALSO

<tt>[[fork(2)]]</tt>, <tt>[[getrusage(2)]]</tt>, <tt>[[sigaction(2)]]</tt>, <tt>[[signal(2)]]</tt>, <tt>[[wait(2)]]</tt>, <tt>[[signal(7)]]</tt>

----

2021-03-22
