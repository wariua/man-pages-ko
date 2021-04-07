## NAME

ntp_gettime, ntp_gettimex - 시간 매개변수 얻기 (NTP 데몬 인터페이스)

## SYNOPSIS

```c
#include <sys/timex.h>

int ntp_gettime(struct ntptimeval *ntv);

int ntp_gettimex(struct ntptimeval *ntv);
```

## DESCRIPTION

이 두 API는 모두 다음 구조체 타입인 `ntv` 인자를 통해 호출자에게 정보를 반환한다.

```c
struct ntptimeval {
    struct timeval time;        /* 현재 시간 */
    long int maxerror;          /* 최대 오차 */
    long int esterror;          /* 추정 오차 */
    long int tai;               /* TAI 오프셋 */

    /* 향후 확장을 위한 추가 패딩 바이트 */
};
```

이 구조체의 필드들은 다음과 같다.

<dl>
<dt><code>maxerror</code></dt>
<dd>최대 오차. 마이크로초 단위. <tt>[[ntp_adjtime(3)]]</tt>으로 이 값을 초기화할 수 있으며, 주기적으로 (리눅스에선 매초마다) 증가하되 상한(커널 상수 <code>NTP_PHASE_MAX</code>, 16,000)이 있다.</dd>

<dt><code>esterror</code></dt>
<dd>추정 오차. 마이크로초 단위. 시스템 클럭과 실제 시간의 추산 차이를 담도록 <tt>[[ntp_adjtime(3)]]</tt>으로 이 값을 설정할 수 있다. 이 값은 커널 내에선 쓰이지 않는다.</dd>

<dt><code>tai</code></dt>
<dd>TAI(국제원자시) 오프셋.</dd>
</dl>

`ntp_gettime()`은 `ntptimeval` 구조체의 `time`, `maxerror`, `esterror` 필드를 채워서 반환한다.

`ntp_gettimex()`는 `ntp_gettime()`과 같은 작업을 수행하되 `tai` 필드로도 정보를 반환한다.

## RETURN VALUE

`ntp_gettime()` 및 `ntp_gettimex()`의 반환 값은 <tt>[[adjtimex(2)]]</tt>에서와 같다. 포인터 인자가 올바르면 이 함수들은 항상 성공한다.

## VERSIONS

glibc 2.1부터 `ntp_gettime()` 함수가 사용 가능하다. glibc 2.12부터 `ntp_gettimex()` 함수가 사용 가능하다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `ntp_gettime()`, `ntp_gettimex()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

`ntp_gettime()`은 NTP Kernel Application Program Interface에 기술돼 있다. `ntp_gettimex()`는 GNU 확장이다.

## SEE ALSO

<tt>[[adjtimex(2)]]</tt>, <tt>[[ntp_adjtime(3)]]</tt>, <tt>[[time(7)]]</tt>

NTP "Kernel Application Program Interface" (http://www.slac.stanford.edu/comp/unix/package/rtems/src/ssrlApps/ntpNanoclock/api.htm)

----

2017-09-15
