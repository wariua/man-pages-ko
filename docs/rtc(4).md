## NAME

rtc - 실제 시간 클럭

## SYNOPSIS

```c
#include <linux/rtc.h>

int ioctl(fd, RTC_request, param);
```

## DESCRIPTION

실제 시간 클럭(RTC) 드라이버에 대한 인터페이스이다.

대부분의 컴퓨터에는 현재 "벽시계" 시간을 기록하는 하드웨어 클럭이 한 개 이상 있다. 그걸 "실제 시간 클럭(RTC, Real Time Clock)"이라고 한다. 보통 그 중 하나에는 전지로 된 백업 전원이 있어서 컴퓨터가 꺼져 있는 동안에도 시간을 따라간다. RTC에서 알람 및 기타 인터럽트를 제공하는 경우가 많다.

모든 i386 PC, 그리고 ACPI 기반 시스템에는 원래 PC/AT에 있던 모토롤라 MC146818 칩과 호환되는 RTC가 있다. 요즘 그 RTC는 보통 메인보드 칩셋(사우스 브리지)에 통합돼 있으며 교체 가능한 단추형 백업 전지를 쓴다.

단일 칩 시스템(SoC) 프로세서로 만든 임베디드 시스템 같은 PC 외 시스템에서는 다른 구현체들을 쓴다. 일반적으로 PC/AT의 RTC와 같은 기능성을 제공하지 않는다.

### RTC 대 시스템 클럭

RTC와 시스템 클럭을 혼동해선 안 된다. 시스템 클럭은 커널에서 유지하는 소프트웨어 클럭으로 <tt>[[gettimeofday(2)]]</tt>, <tt>[[time(2)]]</tt>, 그리고 파일 타임스탬프 설정 등에 사용한다. 시스템 클럭은 어떤 시작 시점, 즉 1970-01-01 00:00:00 +0000 (UTC)으로 정의된 POSIX 에포크 이후 지난 초와 마이크로초를 알려준다. (흔히 볼 수 있는 구현 방식은 100Hz나 250Hz, 1000Hz 빈도로 "지피(jiffy)"당 한 번씩 타이머 인터럽트를 세는 것이다.) 즉 벽시계 시간을 알려주기 위한 것이고, 그게 RTC의 역할이기도 하다.

RTC와 시스템 클럭의 핵심 차이는 시스템이 저전력 상태(꺼진 상태 포함)일 때도 RTC는 돌지만 시스템 클럭은 그럴 수 없다는 점이다. 초기화 전까지 시스템 클럭은 POSIX 에포크 이후가 아니라 시스템 부팅 이후의 시간을 알려줄 수 있을 뿐이다. 그래서 부팅 때, 그리고 시스템 저전력 상태에서 돌아온 후에 RTC를 써서 시스템 클럭을 현재 벽시계 시간으로 맞추는 경우가 많다. RTC가 없는 시스템은 다른 클럭을 이용해 시스템 클럭을 맞춰야 하는데, 네트워크를 통해서일 수도 있고 데이터를 직접 입력해 주는 방식일 수도 있다.

### RTC의 기능

`hwclock(8)`을 통해서, 또는 아래 나열하는 ioctl 요청으로 직접 RTC에 읽고 쓸 수 있다.

날짜와 시간을 따라가는 것 외에 여러 RTC들은 다음과 같이 인터럽트를 생성할 수 있다.

* 클럭 갱신 때마다 (즉 초당 한 번씩)

* 2Hz에서 8192Hz까지 범위에서 2의 제곱 수로 설정할 수 있는 간격으로 주기적으로

* 앞서 지정한 알람 시간이 됐을 때

이런 인터럽트 원천 각각을 따로 켜거나 끌 수 있다. 여러 시스템에서는 알람 인터럽트를 시스템 깨우기 이벤트로 설정할 수 있어서 Suspend-to-RAM(SRT, ACPI 시스템에서 S3)나 하이버네이션(ACPI 시스템에서 S4), 심지어 "꺼짐"(ACPI 시스템에서 S5) 같은 저전력 상태의 시스템을 복원시킬 수 있다. 어떤 시스템에서는 전지 기반 RTC가 인터럽트를 생성할 수 없지만 다른 시스템에서는 가능하다.

`/dev/rtc` (또는 `/dev/rtc0`, `/dev/rtc1` 등) 장치는 동시에 한 프로세스에서만 열 수 있다. `read(2)`나 <tt>[[select(2)]]</tt>를 호출한 프로세스는 그 RTC에서 다음 번 인터럽트를 수신할 때까지 블록 된다. 인터럽트 이후 프로세스에서 long 정수 하나를 읽을 수 있는데, 최하위 바이트는 발생 인터럽트 종류를 나타내는 비트 마스크를 담고 있고 나머지 3바이트는 마지막 `read(2)` 이후 인터럽트 횟수를 담고 있다.

### `ioctl(2)` 인터페이스

RTC 장치에 연결된 파일 디스크립터에서 다음 `ioctl(2)` 요청들이 정의돼 있다.

<dl>
<dt><code>RTC_RD_TIME</code></dt>
<dd>

다음 구조체로 RTC의 시간을 반환한다.

```c
struct rtc_time {
    int tm_sec;
    int tm_min;
    int tm_hour;
    int tm_mday;
    int tm_mon;
    int tm_year;
    int tm_wday;     /* 사용 안 함 */
    int tm_yday;     /* 사용 안 함 */
    int tm_isdst;    /* 사용 안 함 */
};
```

이 구조체 필드들의 의미와 범위는 <tt>[[gmtime(3)]]</tt>에 설명된 <code>tm</code> 구조체와 같다. <code>ioctl(2)</code> 세 번째 인자로 이 구조체에 대한 포인터를 전달해야 한다.
</dd>

<dt><code>RTC_SET_TIME</code></dt>
<dd>이 RTC의 시간을 <code>ioctl(2)</code> 세 번째 인자가 가리키는 <code>rtc_time</code> 구조체가 나타내는 시간으로 설정한다. RTC 시간을 설정하려면 프로세스에게 특권이 있어야 한다. (즉 <code>CAP_SYS_TIME</code> 역능이 있어야 한다.)</dd>

<dt><code>RTC_ALM_READ</code>, <code>RTC_ALM_SET</code></dt>
<dd>알람을 지원하는 RTC에 대해서 알람 시간을 읽거나 설정한다. 알람 인터럽트를 켜거나 끄는 건 <code>RTC_AIE_ON</code> 및 <code>RTC_AIE_OFF</code> 요청을 써서 따로 해야 한다. <code>ioctl(2)</code> 세 번째 인자는 <code>rtc_time</code> 구조체에 대한 포인터이다. 그 구조체의 <code>tm_sec</code>, <code>tm_min</code>, <code>tm_hour</code> 필드만 쓴다.</dd>

<dt><code>RTC_IRQP_READ</code>, <code>RTC_IRQP_SET</code></dt>
<dd>주기적 인터럽트를 지원하는 RTC에 대해서 주기적 인터럽트의 빈도를 읽거나 설정한다. 주기적 인터럽트를 켜거나 끄는 건 <code>RTC_PIE_ON</code> 및 <code>RTC_PIE_OFF</code> 요청을 써서 따로 해야 한다. <code>ioctl(2)</code> 세 번째 인자는 각각 <code>unsigned long *</code>와 <code>unsigned long</code>이다. 그 값이 초당 인터럽트 빈도이다. 허용 가능한 빈도는 2에서 8192까지 범위 내의 2의 배수들이다. 특권 프로세스만 (즉 <code>CAP_SYS_RESOURCE</code> 역능을 가진 프로세스만) <code>/proc/sys/dev/rtc/max-user-freq</code>에 지정된 값보다 높게 빈도를 설정할 수 있다. (이 파일에 기본으로 들어 있는 값은 64이다.)</dd>

<dt><code>RTC_AIE_ON</code>, <code>RTC_AIE_OFF</code></dt>
<dd>알람을 지원하는 RTC에 대해서 알람 인터럽트를 켜거나 끈다. <code>ioctl(2)</code> 세 번째 인자는 무시한다.</dd>

<dt><code>RTC_UIE_ON</code>, <code>RTC_UIE_OFF</code></dt>
<dd>초당 일회 방식 인터럽트를 지원하는 RTC에 대해서 모든 클럭 갱신 인터럽트를 켜거나 끈다. <code>ioctl(2)</code> 세 번째 인자는 무시한다.</dd>

<dt><code>RTC_PIE_ON</code>, <code>RTC_PIE_OFF</code></dt>
<dd>주기적 인터럽트를 지원하는 RTC에 대해서 주기적 인터럽트를 켜거나 끈다. <code>ioctl(2)</code> 세 번째 인자는 무시한다. 현재 빈도가 <code>/proc/sys/dev/rtc/max-user-freq</code>에 지정된 값보다 높게 설정돼 있는 경우 특권 프로세스만 (즉 <code>CAP_SYS_RESOURCE</code> 역능을 가진 프로세스만) 주기적 인터럽트를 켤 수 있다.</dd>

<dt><code>RTC_EPOCH_READ</code>, <code>RTC_EPOCH_SET</code></dt>
<dd>여러 RTC들에선 8비트 레지스터로 연도를 나타내고 8비트 이진 값으로 또는 BCD 값으로 해석한다. 어느 경우든 이 RTC의 에포크를 기준으로 그 값을 해석한다. 많은 시스템에선 RTC의 에포크가 1900으로 초기화 돼 있지만 Alpha나 MIPS에서는 RTC의 연도 레지스터 값에 따라 1952나 1980, 2000으로 초기화 돼 있을 수도 있다. 일부 RTC들에선 이 동작들을 사용해 RTC의 에포크를 읽거나 설정할 수 있다. <code>ioctl(2)</code> 세 번째 인자는 각각 <code>unsigned long *</code>와 <code>unsigned long</code>이며 반환(할당) 값이 에포크이다. RTC의 에포크를 설정하려면 프로세스에게 특권이 있어야 (즉 <code>CAP_SYS_TIME</code> 역능이 있어야) 한다.</dd>

<dt><code>RTC_WKALM_RD</code>, <code>RTC_WKALM_SET</code></dt>
<dd>

일부 RTC들은 더 강력한 알람 인터페이스를 제공하며, 이 ioctl을 이용해 그 RTC의 알람 시간을 다음 구조체로 읽거나 쓸 수 있다.

```c
struct rtc_wkalrm {
    unsigned char enabled;
    unsigned char pending;
    struct rtc_time time;
};
```

<code>enabled</code> 플래그는 알람 인터럽트를 켜고 끄거나 현재 상태를 읽는 데 쓴다. 이 호출 사용 시 <code>RTC_AIE_ON</code> 및 <code>RTC_AIE_OFF</code>는 쓰지 않는다. <code>pending</code> 플래그는 <code>RTC_WKALM_RD</code>에서 미처리 인터럽트를 알려 주기 위한 것이다. (그래서 EFI 펌웨어가 관리하는 RTC를 다룰 때를 빼면 리눅스에서는 쓸 일이 거의 없다.) <code>time</code> 필드는 <code>RTC_ALM_READ</code> 및 <code>RTC_ALM_SET</code>에서와 마찬가지이되 <code>tm_mday</code>, <code>tm_mon</code>, <code>tm_year</code> 필드도 유효하다. 이 구조체에 대한 포인터를 <code>ioctl(2)</code> 세 번째 인자로 전달해야 한다.
</dd>
</dl>

## FILES

<dl>
<dt><code>/dev/rtc</code>, <code>/dev/rtc0</code>, <code>/dev/rtc1</code> 등</dt>
<dd>RTC 특수 문자 장치 파일.</dd>

<dt><code>/proc/driver/rtc</code></dt>
<dd>(첫 번째) RTC의 상태.</dd>
</dl>

## NOTES

<tt>[[adjtimex(2)]]</tt>를 써서 커널 시스템 시간을 외부 참조 시간에 동기화하는 경우 11분마다 주기적으로 지정 RTC를 갱신하게 된다. 이를 위해 커널에서 주기 인터럽트를 잠시 꺼야 하는데, 이로 인해 그 RTC를 쓰는 프로그램들이 영향을 받을 수도 있다.

RTC의 에포크와 시스템 클럭에만 쓰이는 POSIX 에포크는 아무 상관이 없다.

RTC 에포크 및 연도 레지스터에 따른 연도가 1970보다 작으면 100년 후라고 본다. 즉 2000년에서 2069년까지 연도이다.

일부 RTC에서는 알람 필드에 "와일드카드" 값을 지원한다. 그래서 매시 15분에, 또는 매월 첫 번째 날에 주기적 알람을 받는 것 같은 시나리오를 지원한다. 하지만 그런 사용 방식은 이식성이 없다. 이식 가능한 사용자 공간 코드에서는 한 개의 알람 인터럽트만을 기대하며 알람을 수신한 후에는 끄거나 아니면 재초기화 한다.

일부 RTC에서는 1초를 쪼갠 간격이 아니라 1초 배수 주기의 주기 인터럽트, 여러 개 알람, 프로그래밍 가능한 출력 클럭 시그널, 비휘발성 메모리 등과 같이 현재 이 API에 없는 여러 하드웨어 기능을 제공한다.

## SEE ALSO

`date(1)`, <tt>[[adjtimex(2)]]</tt>, <tt>[[gettimeofday(2)]]</tt>, <tt>[[settimeofday(2)]]</tt>, <tt>[[stime(2)]]</tt>, <tt>[[time(2)]]</tt>, <tt>[[gmtime(3)]]</tt>, <tt>[[time(7)]]</tt>, `hwclock(8)`

리눅스 커널 소스 트리의 `Documentation/rtc.txt`

----

2017-09-15
