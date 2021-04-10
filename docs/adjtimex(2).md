## NAME

adjtimex, ntp_adjtime - 커널 클럭 조정

## SYNOPSIS

```c
#include <sys/timex.h>

int adjtimex(struct timex *buf);

int ntp_adjtime(struct timex *buf);
```

## DESCRIPTION

리눅스에서는 David L. Mills의 클럭 조정 알고리즘(RFC 5905)을 사용한다. 시스템 호출 `adjtimex()`는 이 알고리즘의 조정 매개변수들을 읽고 설정한다. `timex` 구조체 포인터를 받아서 (선택된) 필드 값들로 커널 매개변수를 갱신하고, 그 구조체를 현행 커널 값들로 갱신해서 반환한다. 구조체는 다음과 같이 선언돼 있다.

```c
struct timex {
    int  modes;      /* 동작 선택 */
    long offset;     /* 시간 오프셋. 상태 플래그 STA_NANO가
                        설정돼 있으면 나노초, 아니면
                        마이크로초 */
    long freq;       /* 진동수 오프셋. 단위는 NOTES 참고 */
    long maxerror;   /* 최대 오차 (마이크로초) */
    long esterror;   /* 추정 오차 (마이크로초) */
    int  status;     /* 클럭 명령/상태 */
    long constant;   /* 위상 동기 루프(PLL) 시간 상수 */
    long precision;  /* 클럭 정밀도
                        (마이크로초, 읽기 전용) */
    long tolerance;  /* 클럭 진동수 허용 오차 (읽기 전용).
                        단위는 NOTES 참고 */
    struct timeval time;
                     /* 현재 시간. (읽기 전용, ADJ_SETOFFSET에선
                        예외.) 반환 시 time.tv_usec에 담기는
                        값이 STA_NANO 상태 플래그가 설정돼
                        있으면 나노초, 아니면 마이크로초 */
    long tick;       /* 클럭 틱 간격 (마이크로초) */
    long ppsfreq;    /* 펄스 반복(PPS) 진동수
                        (읽기 전용). 단위는 NOTES 참고 */
    long jitter;     /* PPS 지터 (읽기 전용). 상태 플래그
                        STA_NANO가 설정돼 있으면 나노초,
                        아니면 마이크로초 */
    int  shift;      /* PPS 구간 길이
                        (초, 읽기 전용) */
    long stabil;     /* PPS 안정성 (읽기 전용).
                        단위는 NOTES 참고 */
    long jitcnt;     /* PPS 지터 제한 초과 발생 횟수
                        (읽기 전용) */
    long calcnt;     /* PPS 보정 구간 수 (읽기 전용) */
    long errcnt;     /* PPS 보정 오류 횟수 (읽기 전용) */
    long stbcnt;     /* PPS 안정성 제한 초과 발생 횟수
                        (읽기 전용) */
    int tai;         /* TAI 오프셋. 앞선 ADJ_TAI 동작에서
                        설정한 값. (초, 읽기 전용,
                        리눅스 2.6.26부터.) */
    /* 향후 확장을 위한 추가 패딩 바이트 */
};
```

`modes` 필드가 설정할 매개변수를 결정한다. (잠시 후 설명하겠지만 `ntp_adjtime()`에는 동등하지만 이름이 다른 상수들을 쓴다.) 그 필드는 다음 비트들을 0개 이상 비트 *or* 조합해서 담은 비트 마스크이다.

`ADJ_OFFSET`
:   `buf.offset`으로 시간 오프셋 설정. 리눅스 2.6.26부터는 제공된 값을 (-0.5s, +0.5s) 범위로 잘라낸다. 이전 커널에서는 제공된 값이 범위를 벗어나면 `EINVAL` 오류가 발생한다.

`ADJ_FREQUENCY`
:   `buf.freq`로 진동수 오프셋 설정. 리눅스 2.6.26부터는 제공된 값을 (-32768000, +32768000) 범위로 잘라낸다. 이전 커널에서는 제공된 값이 범위를 벗어나면 `EINVAL` 오류가 발생한다.

`ADJ_MAXERROR`
:   `buf.maxerror`로 최대 시간 오차 설정.

`ADJ_ESTERROR`
:   `buf.esterror`로 추정 시간 오차 설정.

`ADJ_STATUS`
:   `buf.status`로 클럭 상태 비트 설정. 이 비트들에 대해선 아래에서 설명한다.

`ADJ_TIMECONST`
:   `buf.constant`로 PLL 시간 상수 설정. `STA_NANO` 상태 플래그(아래 참고)가 해제돼 있으면 커널에서 이 값에 4를 더한다.

`ADJ_SETOFFSET` (리눅스 2.6.39부터)
:   현재 시간에 `buf.time` 더하기. `buf.status`에 `ADJ_NANO` 플래그가 포함돼 있으면 `buf.time.tv_usec`을 나노초 값으로 해석한다. 아니면 마이크로초로 해석한다.

`ADJ_MICRO` (리눅스 2.6.26부터)
:   마이크로초 정밀도 선택.

`ADJ_NANO` (리눅스 2.6.26부터)
:   나노초 정밀도 선택. `ADJ_MICRO`와 `ADJ_NANO` 중 하나만 지정해야 한다.

`ADJ_TAI` (리눅스 2.6.26부터)
:   `buf.tai`로 국제원자시(TAI) 오프셋 설정.

    `ADJ_TAI`를 `ADJ_TIMECONST`와 함께 쓰지 말아야 한다. 그 모드에서도 `buf.constant` 필드를 이용하기 때문이다.

    TAI가 무엇이고 TAI와 UTC의 차이가 뭔지에 대한 설명은 *BIPM*(<http://www.bipm.org/en/bipm/tai/tai.html>) 참고.

`ADJ_TICK`
:   `buf.tick`으로 틱 값을 설정.

또는 `modes`에 다음 (여러 비트로 된 마스크) 값들 중 하나를 지정할 수도 있으며, 그 경우 `modes`에 다른 비트들은 지정하지 말아야 한다.

`ADJ_OFFSET_SINGLESHOT`
:   구식 `adjtime()` 방식: `buf.offset`에 마이크로초 단위로 지정된 조정 값에 따라 시간을 (점진적으로) 조정한다.

`ADJ_OFFSET_SS_READ` (리눅스 2.6.28부터 동작)
:   앞선 `ADJ_OFFSET_SINGLESHOT` 동작 후에 남은 조정 시간 양을 (`buf.offset`으로) 반환한다. 이 기능은 리눅스 2.6.24에서 추가되었는데 리눅스 2.6.28까지는 올바로 동작하지 않았다.

일반 사용자는 `modes`에 0 또는 `ADJ_OFFSET_SS_READ` 값만 지정할 수 있다. 수퍼유저만 매개변수 설정을 할 수 있다.

`buf.status` 필드는 NTP 구현과 관련된 상태 비트를 설정 및/또는 조회하는 데 쓰는 비트 마스크이다. 마스크의 일부 비트는 읽기와 설정이 모두 가능하지만 나머지는 읽기 전용이다.

`STA_PLL` (읽기-쓰기)
:   `ADJ_OFFSET`을 통한 위상 동기 루프(PLL) 갱신 활성화.

`STA_PPSFREQ` (읽기-쓰기)
:   펄스 반복(PPS) 진동수 조정 활성화.

`STA_PPSTIME` (읽기-쓰기)
:   PPS 시간 조정 활성화.

`STA_FLL` (읽기-쓰기)
:   진동수 동기 루프(FLL) 모드 선택.

`STA_INS` (읽기-쓰기)
:   그 UTC 일의 마지막 초 다음에 윤초를 삽입한다. 그래서 그 날의 마지막 분을 1초만큼 늘인다. 이 플래그가 설정돼 있는 동안은 매일 윤초 삽입이 일어나게 된다.

`STA_DEL` (읽기-쓰기)
:   그 UTC 일의 마지막 초에서 윤초를 삭제한다. 이 플래그가 설정돼 있는 동안은 매일 윤초 삭제가 일어나게 된다.

`STA_UNSYNC` (읽기-쓰기)
:   클럭이 비동기 상태임.

`STA_FREQHOLD` (읽기-쓰기)
:   진동수 유지. 보통 `ADJ_OFFSET`을 통해 조정을 하면 진동수 감쇄 조정도 이뤄지게 된다. 그래서 호출 한 번으로는 현재 오프셋을 바로잡고, 같은 방향으로 오프셋 정정이 반복해서 이뤄지면 작은 진동수 조정이 누적돼서 장기적인 왜곡을 수정하게 된다.

    이 플래그는 `ADJ_OFFSET` 값으로 정정을 할 때 그 작은 진동수 조정이 이뤄지지 않게 한다.

`STA_PPSSIGNAL` (읽기 전용)
:   유효한 펄스 반복(PPS) 신호 있음.

`STA_PPSJITTER` (읽기 전용)
:   PPS 신호 지터 초과.

`STA_PPSWANDER` (읽기 전용)
:   PPS 신호 원더 초과.

`STA_PPSERROR` (읽기 전용)
:   PPS 신호 보정 오류.

`STA_CLOCKERR` (읽기 전용)
:   클럭 하드웨어 오동작.

`STA_NANO` (읽기 전용, 리눅스 2.6.26부터)
:   해상도 (0 = 마이크로초, 1 = 나노초). `ADJ_NANO`를 통해 설정하고 `ADJ_MICRO`를 통해 해제한다.

`STA_MODE` (리눅스 2.6.26부터)
:   모드 (0 = 위상 동기 루프, 1 = 진동수 동기 루프).

`STA_CLK` (읽기 전용, 리눅스 2.6.26부터)
:   클럭 원천 (0 = A, 1 = B). 현재 쓰지 않음.

읽기 전용인 `status` 비트를 설정하려고 시도하면 조용히 무시한다.

### `ntp_adjtime()`

(NTP "Kernel Application Program API", 즉 KAPI에 기술돼 있는) `ntp_adjtime()` 라이브러리 함수는 `adjtimex()`와 같은 일을 수행할 수 있는 더 이식성 좋은 인터페이스이다. 다음 사항들을 제외하면 `adjtime()`과 동일하다.

* `modes`에 쓰는 상수들이 "ADJ_" 대신 "MOD_"로 시작하고 뒷부분이 같다. (즉 `MOD_OFFSET`, `MOD_FREQUENCY` 등이다.) 단 아래 항목들은 예외이다.

* `ADJ_OFFSET_SINGLESHOT`의 동의어는 `MOD_CLKA`이다.

* `ADJ_TICK`의 동의어는 `MOD_CLKB`이다.

* KAPI에 기술돼 있지 않은 `ADJ_OFFSET_SS_READ`의 동의어는 없다.

## RETURN VALUE

성공 시 `adjtimex()` 및 `ntp_adjtime()`은 클럭 상태를 반환한다. 즉 다음 값들 중 하나를 반환한다.

`TIME_OK`
:   클럭이 동기화돼 있고 대기 중인 윤초 조정이 없다.

`TIME_INS`
:   그 UTC 일 끝에서 윤초가 추가될 것임을 나타낸다.

`TIME_DEL`
:   그 UTC 일 끝에서 윤초가 삭제될 것임을 나타낸다.

`TIME_OOP`
:   윤초 삽입이 진행 중이다.

`TIME_WAIT`
:   윤초 삽입 내지 삭제가 완료됐다. 다음 `ADJ_STATUS` 동작에서 `STA_INS` 및 `STA_DEL` 플래그를 해제할 때까지 이 값이 반환된다.

`TIME_ERROR`
:   시스템 클럭이 믿을 만한 서버에 동기화돼 있지 않다. 다음 중 하나라도 참일 때 이 값이 반환된다.

    * `STA_UNSYNC`나 `STA_CLOCKERR` 중 하나가 설정돼 있다.

    * `STA_PPSSIGNAL`이 해제돼 있으면서 `STA_PPSFREQ`나 `STA_PPSTIME`이 설정돼 있다.

    * `STA_PPSTIME`과 `STA_PPSJITTER`가 모두 설정돼 있다.

    * `STA_PPSFREQ`가 설정돼 있으면서 `STA_PPSWANDER`나 `STA_PPSJITTER`가 설정돼 있다.

    심볼 이름 `TIME_BAD`는 `TIME_ERROR`의 동의어이며 하위 호환성을 위해 제공된다.

참고로 리눅스 3.4부터는 호출이 비동기적으로 동작하므로 일반적으로 반환 값이 그 호출 자체로 인한 상태 변화를 반영하지 않게 된다.

실패 시 이 호출들은 -1을 반환하고 `errno`를 설정한다.

## ERRORS

`EFAULT`
:   `buf`가 쓰기 가능한 메모리를 가리키고 있지 않다.

`EINVAL` (리눅스 2.6.26 전 커널)
:   `buf.freq`를 범위 (-33554432, +33554432) 밖의 값으로 설정하려 했다.

`EINVAL` (리눅스 2.6.26 전 커널)
:   `buf.offset`을 허용 범위 밖의 값으로 설정하려 했다. 리눅스 2.0 전의 커널에서는 허용 범위가 (-131072, +131072)였다. 리눅스 2.0부터는 허용 범위가 (-512000, +512000)이다.

`EINVAL`
:   `buf.status`를 위에 나열된 것 외의 값으로 설정하려 했다.

`EINVAL`
:   `buf.tick`을 `900000/HZ`에서 `1100000/HZ`까지 범위 밖의 값으로 설정하려 했다. 여기서 `HZ`는 시스템 타이머 인터럽트 빈도이다.

`EPERM`
:   `buf.modes`가 0이나 `ADJ_OFFSET_SS_READ`가 아니며 호출자에게 충분한 특권이 없다. 리눅스에서는 `CAP_SYS_TIME` 역능이 필요하다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `ntp_adjtime()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

이 인터페이스들 중 어느 것도 POSIX.1에 기술돼 있지 않다.

`adjtimex()`는 리눅스 전용이므로 이식성이 있어야 하는 프로그램에서는 사용하지 말아야 한다.

NTP 데몬에게 적당한 API는 `ntp_adjtime()`이다.

## NOTES

`timex` 구조체에서 `freq`, `ppsfreq`, `stabil`은 백만분율(ppm)이고 16비트는 소수부이다. 즉 그 필드들에서 값 1은 2^-16 ppm을 뜻하고 2^16=65536이 1 ppm이다. 입력 값(`freq`)과 출력 값 모두에서 그렇다.

`STA_INS` 및 `STA_DEL`로 인한 윤초 처리는 커널 타이머 문맥에서 이뤄진다. 따라서 윤초를 추가 내지 삭제하려면 초당 한 틱이 걸리게 된다.

## SEE ALSO

<tt>[[settimeofday(2)]]</tt>, <tt>[[adjtime(3)]]</tt>, <tt>[[ntp_gettime(3)]]</tt>, <tt>[[capabilities(7)]]</tt>, <tt>[[time(7)]]</tt>, `adjtimex(8)`, `hwclock(8)`

NTP "Kernel Application Program Interface" (<http://www.slac.stanford.edu/comp/unix/package/rtems/src/ssrlApps/ntpNanoclock/api.htm>)

----

2019-03-06
