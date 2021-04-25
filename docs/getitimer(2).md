## NAME

getitimer, setitimer - 간격 타이머 값 얻거나 설정하기

## SYNOPSIS

```c
#include <sys/time.h>

int getitimer(int which, struct itimerval *curr_value);
int setitimer(int which, const struct itimerval *restrict new_value,
              struct itimerval *restrict old_value);
```

## DESCRIPTION

이 시스템 호출들은 간격 타이머, 즉 미래 어느 시점에 최초 만료되고 (선택적으로) 이후 주기적 간격으로 만료되는 타이머들을 사용할 수 있게 해 준다. 타이머가 만료될 때 호출 프로세스에 시그널이 생성되고 (간격이 0이 아니면) 타이머가 지정 간격으로 재설정된다.

세 가지 종류의 타이머가 있어서 `which` 인자를 통해 지정한다. 각각은 서로 다른 클럭에 대해 계산을 하며 타이머 만료 시 다른 시그널을 생성한다.

`ITIMER_REAL`
:   실제 시간(즉 벽시계 시간)으로 타이머를 카운트다운 한다. 만료 때마다 `SIGALRM` 시그널이 생성된다.

`ITIMER_VIRTUAL`
:   프로세스가 소모한 사용자 모드 CPU 시간에 대해 타이머를 카운트다운 한다. (프로세스 내의 모든 스레드들이 소모한 CPU 시간이 측정에 포함된다.) 만료 때마다 `SIGVTALRM` 시그널이 생성된다.

`ITIMER_PROF`
:   프로세스가 소모한 CPU 시간(즉 사용자 시간과 시스템 시간 모두)에 대해 타이머를 카운트다운 한다. (프로세스 내의 모든 스레드들이 소모한 CPU 시간이 측정에 포함된다.) 만료 때마다 `SIGPROF` 시그널이 생성된다.

    이 타이머를 `ITIMER_VIRTUAL`과 함께 사용하여 프로세스가 소모한 사용자 및 시스템 CPU 시간을 프로파일 할 수 있다.

프로세스는 세 가지 타이머 종류별로 하나씩을 가지고 있다.

다음 구조체들로 타이머 값을 지정한다.

```c
struct itimerval {
    struct timeval it_interval; /* 주기 타이머의 간격 */
    struct timeval it_value;    /* 다음 만료까지의 시간 */
};

struct timeval {
    time_t      tv_sec;         /* 초 */
    suseconds_t tv_usec;        /* 마이크로초 */
};
```

### `getitimer()`

`getitimer()` 함수는 `which`로 지정한 타이머의 현재 값을 `curr_value`가 가리키는 버퍼에 넣는다.

`it_value` 하위 구조체에는 지정한 타이머의 다음 만료 때까지 남은 시간이 채워진다. 이 값은 타이머가 카운트다운 하면서 바뀌게 되며 타이머가 만료될 때 `it_interval`로 재설정된다. `it_value`의 두 필드가 모두 0이면 이 타이머는 현재 해제된 (비활성) 상태이다.

`it_interval` 하위 구조체에는 타이머 간격이 채워진다. `it_interval`의 두 필드가 모두 0이면 이 타이머는 단발성이다. (즉, 한 번만 만료된다.)

### `setitimer()`

`setitimer()` 함수는 `new_value`로 지정한 값으로 타이머를 설정하여 `which`로 지정한 타이머를 장전하거나 해제한다. `old_value`가 NULL이 아니면 그 버퍼를 이용해 타이머의 이전 값을 (즉, `getitimer()`가 반환하는 것과 같은 정보를) 반환한다.

`new_value.it_value`의 어느 한 필드라도 0이 아니면 그 지정 시간에 최초 만료되도록 타이머를 장전한다. `new_value.it_value`의 두 필드가 모두 0이면 타이머를 해제한다.

`new_value.it_interval` 필드가 타이머의 새 간격을 지정한다. 그 하위 필드가 둘 모두 0이면 타이머가 단발성이다.

## RETURN VALUE

성공 시 0을 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EFAULT`
:   `new_value`나 `old_value`, `curr_value`가 유효한 포인터가 아니다.

`EINVAL`
:   `which`가 `ITIMER_REAL`, `ITIMER_VIRTUAL`, `ITIMER_PROF` 중 하나가 아니다. 또는 (리눅스 2.6.22부터) `new_value`가 가리키는 구조체 내의 한 `tv_usec` 필드가 0에서 999999까지 범위 밖의 값을 담고 있다.

## CONFORMING TO

POSIX.1-2001, SVr4, 4.4BSD (4.2BSD에서 이 호출이 처음 등장). POSIX.1-2008에서는 `getitimer()`와 `setitimer()`를 구식으로 표시하고 대신 POSIX 타이머 API(<tt>[[timer_gettime(2)]]</tt>, <tt>[[timer_settime(2)]]</tt> 등) 사용을 권장한다.

## NOTES

타이머는 요청 시간 전에는 절대 만료되지 않지만 시스템 타이머 해상도와 시스템 부하에 따라서 약간의 (짧은) 시간 후에 만료될 수도 있다. <tt>[[time(7)]]</tt> 참고. (하지만 아래 BUGS를 보라.) 프로세스가 활동 중인 동안 타이머가 만료되면 (`ITIMER_VIRTUAL`에서는 항상 참이다) 시그널이 생성 즉시 전달될 것이다.

<tt>[[fork(2)]]</tt>로 생성한 자식이 부모의 간격 타이머들을 물려받지 않는다. <tt>[[execve(2)]]</tt>를 거칠 때 간격 타이머들이 유지된다.

POSIX.1에서는 `setitimer()`와 세 가지 인터페이스 <tt>[[alarm(2)]]</tt>, <tt>[[sleep(3)]]</tt>, <tt>[[usleep(3)]]</tt>과의 상호작용을 명세하지 않고 남겨두었다.

표준들에서 다음 호출의 의미에 대해 언급하지 않고 있다.

```c
setitimer(which, NULL, &old_value);
```

대부분의 (솔라리스, BSD, 그리고 아마 다른) 시스템들에서는 이를 다음과 동등하게 다룬다.

```c
getitimer(which, &old_value);
```

리눅스에서는 이를 `new_value`의 필드들이 0인 호출과 동등하게 다룬다. 즉, 타이머가 비활성화된다. *리눅스의 이 비기능을 이용해서는 안 된다.* 이식성이 없으며 필요한 것도 아니다.

## BUGS

시그널의 생성과 전달은 서로 별개이며 프로세스별로 위에 나열된 시그널들마다 한 인스턴스만 미처리 상태일 수 있다. 아주 높은 부하에서는 앞선 만료에서의 시그널이 전달되기 전에 `ITIMER_REAL` 타이머가 다시 만료될 수도 있다. 그런 경우 두 번째 시그널은 유실되게 된다.

리눅스 커널 2.6.16 전에서는 타이머 값들을 지피로 표현한다. 지피로 표현하면 (`include/linux/jiffies.h`에 정의된) `MAX_SEC_IN_JIFFIES`를 초과하는 값으로 타이머를 설정하도록 요청하면 조용하게 타이머를 그 상한 값으로 잘라낸다. (리눅스 2.6.13부터 기본 지피가 0.004초인) 리눅스/i386에서라면 타이머의 상한값이 약 99.42일이라는 뜻이다. 리눅스 2.6.16부터는 커널에서 시간에 대해 다른 내부 표현을 사용하므로 이런 상한이 없다.

특정 시스템들(i386 포함)에서 버전 2.6.12 전의 리눅스 커널에는 일부 상황에서 한 지피까지 이르게 타이머 만료가 일어나는 버그가 있다. 커널 2.6.12에서 이 버그가 수정되었다.

POSIX.1-2001에서는 `tv_usec` 값을 0에서 999999까지 범위를 벗어나게 지정한 경우 `setitimer()`가 실패해야 한다고 한다. 하지만 2.6.21까지 커널에서 리눅스는 오류를 주지 않고 그 대신 조용하게 타이머의 대응하는 초 값을 조정한다. 커널 2.6.22부터는 이런 비준수 사항이 고쳐져서 잘못된 `tv_usec` 값이 `EINVAL` 오류를 유발한다.

## SEE ALSO

<tt>[[gettimeofday(2)]]</tt>, <tt>[[sigaction(2)]]</tt>, <tt>[[signal(2)]]</tt>, <tt>[[timer_create(2)]]</tt>, <tt>[[timerfd_create(2)]]</tt>, <tt>[[time(7)]]</tt>

----

2021-03-22
