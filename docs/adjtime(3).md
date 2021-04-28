## NAME

adjtime - 시스템 클럭 동기화를 위해 시간 정정하기

## SYNOPSIS

```c
#include <sys/time.h>

int adjtime(const struct timeval *delta, struct timeval *olddelta);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`adjtime()`:
:   glibc 2.19부터:
    :   `_DEFAULT_SOURCE`

    glibc 2.19 및 이전:
    :   `_BSD_SOURCE`

## DESCRIPTION

`adjtime()` 함수는 (<tt>[[gettimeofday(2)]]</tt>가 반환하는) 시스템 클럭을 서서히 조정한다. 클럭 시간을 얼마나 조정할지를 `delta`가 가리키는 구조체로 지정한다. 그 구조체는 다음 형태다.

```c
struct timeval {
    time_t      tv_sec;     /* 초 */
    suseconds_t tv_usec;    /* 마이크로초 */
};
```

`delta`의 조정치가 양수면 조정이 완료될 때까지 시스템 클럭 속도를 어떤 작은 비율만큼 올린다. (즉 매초마다 클럭 값에 작은 양의 시간을 더한다.) `delta`의 조정치가 음수면 비슷한 방식으로 클럭 속도를 내린다.

앞선 `adjtime()` 호출의 클럭 조정이 이미 진행 중인 상태에서 `adjtime()` 호출이 이뤄지면서 뒤쪽 호출의 `delta`가 NULL이 아니면 앞선 조정이 중단된다. 하지만 이미 완료된 부분을 되돌리지는 않는다.

`olddelta`가 NULL이 아니면 가리키는 버퍼를 이용해 앞선 조정에서 아직 완료되지 않고 남은 시간 양을 반환한다.

## RETURN VALUE

성공 시 `adjtime()`은 0을 반환한다. 실패 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EINVAL`
:   `delta`의 조정치가 허용 범위 밖이다.

`EPERM`
:   호출자에게 시간을 조정할 만한 특권이 없다. 리눅스에서는 `CAP_SYS_TIME` 역능이 필요하다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `adjtime()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

4.3BSD, 시스템 V.

## NOTES

`adjtime()`을 통해 클럭에 하는 조정은 클럭이 항상 단조 증가하게 하는 방식으로 이뤄진다. `adjtime()`을 써서 조정을 하면 시스템 시간이 앞이나 뒤로 갑자기 건너뛰면서 특정 응용들(가령 `make(1)`)에 발생할 수 있는 문제들이 방지된다.

`adjtime()`은 시스템 시간을 조금 조정하는 데 쓰는 것이다. 대부분의 시스템에서는 `delta`에 지정할 수 있는 조정치에 제한을 둔다. glibc 구현에서는 `delta`가 (INT_MAX / 1000000 - 2) 이하이고 (INT_MIN / 1000000 + 2) 이상이어야 한다. (i386에서 각각 2145초와 -2145초다.)

## BUGS

오래된 버그로 인해 `delta`를 NULL로 지정한 경우에 남아 있는 클럭 조정치에 대한 유효한 정보를 `olddelta`로 반환하지 않았다. (이 경우에 원래 `adjtime()`은 남아 있는 클럭 조정치를 변경 없이 반환해야 한다.) glibc 2.8 및 이후 버전과 리눅스 커널 2.6.26 및 이후 버전을 쓰는 시스템에서는 이 버그가 수정돼 있다.

## SEE ALSO

<tt>[[adjtimex(2)]]</tt>, <tt>[[gettimeofday(2)]]</tt>, <tt>[[time(7)]]</tt>

----

2021-03-22
