## NAME

times - 프로세스 시간 얻기

## SYNOPSIS

```c
#include <sys/times.h>

clock_t times(struct tms *buf);
```

## DESCRIPTION

`times()`는 현재 프로세스 시간들을 `buf`가 가리키는 `struct tms`에 저장한다. `struct tms`는 `<sys/times.h>`에 다음처럼 정의돼 있다.

```c
struct tms {
    clock_t tms_utime;  /* 사용자 시간 */
    clock_t tms_stime;  /* 시스템 시간 */
    clock_t tms_cutime; /* 자식들의 사용자 시간 */
    clock_t tms_cstime; /* 자식들의 시스템 시간 */
};
```

`tms_utime` 필드는 호출 프로세스의 인스트럭션을 실행하는 데 쓴 CPU 시간을 담는다. `tms_stime` 필드는 호출 프로세스 대신 작업을 수행하며 커널 내에서 실행에 쓴 CPU 시간을 담는다.

`tms_cutime` 필드는 `tms_utime`에다가 종료돼서 대기가 이뤄진 자식들 모두의 `tms_cutime`을 합친 값을 담는다. `tms_cstime` 필드는 `tms_stime`에다가 종료돼서 대기가 이뤄진 자식들 모두의 `tms_cstime`을 합친 값을 담는다.

종료된 자식들의 (그리고 그 후손들의) 시간은 <tt>[[wait(2)]]</tt> 내지 <tt>[[waitpid(2)]]</tt>에서 프로세스 ID를 반환하는 시점에 합쳐진다. 특히 자식이 대기하지 않은 손주의 시간은 절대 보이지 않게 된다.

모든 시간을 클럭 틱 단위로 보고한다.

## RETURN VALUE

`times()`는 과거 임의 시점 이후 경과한 클럭 틱 수를 반환한다. 반환 값이 `clock_t` 타입의 가능 범위를 넘을 수도 있다. 오류 시 `(clock_t) -1`을 반환하며 `errno`를 적절히 설정한다.

## ERRORS

`EFAULT`
:   `tms`가 프로세스의 주소 공간 밖을 가리키고 있다.

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, SVr4, 4.3BSD.

## NOTES

초당 클럭 틱 수는 다음 방법으로 얻을 수 있다.

```c
sysconf(_SC_CLK_TCK);
```

POSIX.1-1996에서는 (`<time.h>`에 정의된) `CLK_TCK` 심볼이 구식화되고 있다고 말한다. 현재 구식이 되었다.

리눅스 커널 버전 2.6.9 전에서는 `SIGCHLD` 처리 방식이 `SIG_IGN`으로 설정돼 있는 경우에 종료한 자식들의 시간이 자동으로 `tms_cstime` 및 `tms_cutime` 필드에 포함된다. 하지만 POSIX.1-2001에 따르면 호출 프로세스가 자식들에 <tt>[[wait(2)]]</tt> 하는 경우에만 그래야 한다. 리눅스 2.6.9 및 이후에서는 이 불일치가 시정돼 있다.

리눅스에서는 `buf` 인자를 NULL로 지정할 수 있으며 그러면 `times()`가 함수 결과만 반환한다. 하지만 POSIX에서는 이런 동작을 명세하고 있지 않으며 대다수의 다른 유닉스 구현에서는 `buf`에 NULL 아닌 값을 요구한다.

참고로 <tt>[[clock(3)]]</tt>도 `clock_t` 타입인 값을 반환하지만 그 값은 `times()`에서 쓰는 클럭 틱이 아니라 `CLOCKS_PER_SEC` 단위로 측정한 값이다.

리눅스에서 `times()`의 반환 값을 측정하는 기준인 "과거 임의 시점"은 커널 버전에 따라 달랐다. 리눅스 2.4 및 이전에서는 시스템이 부팅 된 순간이었다. 리눅스 2.6부터는 이 시점이 시스템 부팅 시각에서 `(2^32/HZ) - 300` 초 전이다. 커널 버전에 따른 (그리고 유닉스 구현들에 따른) 이런 다양성에다가 반환 값이 `clock_t`의 범위를 넘을 수 있다는 걸 생각하면 이식 가능한 응용에서는 이 값을 쓰지 않는 게 현명할 것이다. 경과 시간 변화를 측정하려면 <tt>[[clock_gettime(2)]]</tt>을 쓰면 된다.

### 이력

SVr1-3에서는 `long`을 반환하며 에포크 이후 초가 아니라 클럭 틱을 저장하는데도 구조체 멤버가 `time_t` 타입이다. V7에서는 구조체 멤버에 `long`을 썼는데, 그때는 `time_t` 타입이 없었기 때문이다.

## BUGS

일부 시스템(특히 i386)에서 리눅스 시스템 호출 규약의 한계 때문에 부팅 직후의 짧은 시간(41초) 동안 `times()`가 -1을 반환해서 오류가 발생한 것처럼 잘못 보이게 될 가능성이 있다. 반환 값이 `clock_t`에 저장할 수 있는 최댓값을 넘어갈 때에도 같은 문제가 발생할 수 있다.

## SEE ALSO

`time(1)`, <tt>[[getrusage(2)]]</tt>, <tt>[[wait(2)]]</tt>, <tt>[[clock(3)]]</tt>, <tt>[[sysconf(3)]]</tt>, <tt>[[time(7)]]</tt>

----

2017-09-15
