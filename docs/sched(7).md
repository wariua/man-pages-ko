## NAME

sched - CPU 스케줄링 개요

## DESCRIPTION

리눅스 2.6.23부터는 기본 스케줄러가 CFS, 즉 "Completely Fair Scheduler"이다. CFS 스케줄러가 이전의 "O(1)" 스케줄러를 대체하였다.

### API 요약

리눅스에서는 CPU 스케줄링 동작 방식과 정책, 프로세스 (더 정확히는 스레드) 우선순위를 제어할 수 있는 다음 시스템 호출들을 제공한다.

<tt>[[nice(2)]]</tt>
:   호출 스레드의 나이스 값을 새로 설정하고 새 나이스 값을 반환.

<tt>[[getpriority(2)]]</tt>
:   스레드나 프로세스 그룹, 또는 지정한 사용자가 소유한 스레드들의 나이스 값 반환.

<tt>[[setpriority(2)]]</tt>
:   스레드나 프로세스 그룹, 또는 지정한 사용자가 소유한 스레드들의 나이스 값 설정.

<tt>[[sched_setscheduler(2)]]</tt>
:   지정한 스레드의 스케줄링 정책과 매개변수 설정.

<tt>[[sched_getscheduler(2)]]</tt>
:   지정한 스레드의 스케줄링 정책 반환.

<tt>[[sched_setparam(2)]]</tt>
:   지정한 스레드의 스케줄링 매개변수 설정.

<tt>[[sched_getparam(2)]]</tt>
:   지정한 스레드의 스케줄링 매개변수 가져오기.

<tt>[[sched_get_priority_max(2)]]</tt>
:   지정한 스케줄링 정책에 사용 가능한 우선순위 최댓값 반환.

<tt>[[sched_get_priority_min(2)]]</tt>
:   지정한 스케줄링 정책에 사용 가능한 우선순위 최솟값 반환.

<tt>[[sched_rr_get_interval(2)]]</tt>
:   "라운드 로빈" 스케줄링 정책으로 스케줄 하는 스레드들에 쓰는 단위 시간 가져오기.

<tt>[[sched_yield(2)]]</tt>
:   호출자가 CPU를 포기해서 어떤 다른 스레드가 실행되게 하기.

<tt>[[sched_setaffinity(2)]]</tt>
:   (리눅스 전용) 지정한 스레드의 CPU 친화성 설정.

<tt>[[sched_getaffinity(2)]]</tt>
:   (리눅스 전용) 지정한 스레드의 CPU 친화성 얻기.

<tt>[[sched_setattr(2)]]</tt>
:   지정한 스레드의 스케줄링 정책과 매개변수 설정. 이 (리눅스 전용) 시스템 호출은 <tt>[[sched_setscheduler(2)]]</tt>와 <tt>[[sched_setparam(2)]]</tt>을 포괄하는 기능성을 제공한다.

<tt>[[sched_getattr(2)]]</tt>
:   지정한 스레드의 스케줄링 정책과 매개변수 가져오기. 이 (리눅스 전용) 시스템 호출은 <tt>[[sched_getscheduler(2)]]</tt>와 <tt>[[sched_getparam(2)]]</tt>을 포괄하는 기능성을 제공한다.

### 스케줄링 정책

스케줄러는 실행 가능 스레드들 중 CPU가 다음으로 무엇을 실행할지 결정하는 커널 구성 요소이다. 각 스레드에는 스케줄링 정책과 *고정(static)* 스케줄링 우선순위인 `sched_priority`가 연계돼 있다. 스케줄러는 시스템의 모든 스레드들의 스케줄링 정책과 고정 우선순위에 기반하여 판단을 내린다.

일반 스케줄링 정책들(`SCHED_OTHER`, `SCHED_IDLE`, `SCHED_BATCH`) 중 하나로 스케줄 되는 스레드에 대해선 스케줄링 판단에 `sched_priority`를 사용하지 않는다. (0으로 지정되어 있어야 한다.)

실시간 정책들(`SCHED_FIFO`, `SCHED_RR`) 중 하나로 스케줄링 되는 프로세스는 1(낮음)에서 99(높음)까지 범위로 `sched_priority` 값을 가지고 있다. (숫자가 암시하듯 실시간 스레드는 항상 일반 스레드보다 우선순위가 높다.) 참고: POSIX.1에서는 구현체가 실시간 정책에 최소 32개의 구별되는 우선순위를 지원할 것을 요구하며, 어떤 시스템들은 이 최소한만 지원한다. 이식 가능한 프로그램에서는 <tt>[[sched_get_priority_min(2)]]</tt>과 <tt>[[sched_get_priority_max(2)]]</tt>를 사용해 특정 정책에 지원되는 우선순위 범위를 알아내야 할 것이다.

개념적으로 스케줄러는 가능한 각 `sched_priority` 값마다 실행 가능 스레드들의 목록을 유지한다. 다음으로 실행할 스레드를 정해야 하면 스케줄러는 비어 있지 않은 가장 높은 고정 우선순위의 목록을 찾아서 그 목록 선두에 있는 스레드를 선택한다.

스레드의 스케줄링 정책은 같은 고정 우선순위의 스레드 목록 내에서 어디로 스레드가 삽입되고 그 목록 내에서 어떻게 이동하게 되는지를 결정한다.

모든 스케줄링은 선점적이다. 그래서 더 높은 고정 우선순위의 스레드가 실행 준비가 되면 현재 실행 중인 스레드가 선점되어 자기 고정 우선순위의 대기 목록으로 돌아가게 된다. 스케줄링 정책은 같은 고정 우선순위를 가진 실행 가능 스레드들의 목록 내에서 순서를 결정할 뿐이다.

### `SCHED_FIFO`: 선입선출 스케줄링

`SCHED_FIFO`는 0보다 높은 고정 우선순위로만 사용 가능하다. 따라서 `SCHED_FIFO` 스레드가 실행 가능해지면 항상 현재 실행 중인 `SCHED_OTHER`, `SCHED_BATCH`, `SCHED_IDLE` 스레드를 즉시 선점하게 된다. `SCHED_FIFO`는 시간 분할 없는 단순한 스케줄링 알고리듬이다. `SCHED_FIFO` 정책으로 스케줄링 되는 스레드에게 다음 규칙들이 적용된다.

1) 실행 중인 `SCHED_FIFO` 스레드가 더 높은 우선순위의 다른 스레드에 의해 선점되면 자기 우선순위의 목록 선두에 머물게 된다. 그리고 더 높은 우선순위의 스레드들이 모두 다시 블록 되자마자 실행을 재개한다.

2) 블록 된 `SCHED_FIFO` 스레드가 실행 가능해지면 자기 우선순위의 목록 끝에 삽입된다.

3) <tt>[[sched_setscheduler(2)]]</tt>, <tt>[[sched_setparam(2)]]</tt>, <tt>[[sched_setattr(2)]]</tt>, <tt>[[pthread_setschedparam(3)]]</tt>, <tt>[[pthread_setschedprio(3)]]</tt> 호출 때문에 `pid`가 가리키는 실행 중이거나 실행 가능인 `SCHED_FIFO` 스레드의 우선순위가 바뀌는 경우 스레드의 목록 내 위치에 대한 영향은 스레드 우선순위가 바뀌는 방향에 따라 달라진다.

   - 스레드의 우선순위가 올라가는 경우 그 새 우선순위의 목록 끝에 들어간다. 그로 인해 현재 실행 중인 같은 우선순위의 스레드를 선점할 수도 있다.

   - 스레드의 우선순위가 바뀌지 않는 경우 실행 목록에서의 위치가 바뀌지 않는다.

   - 스레드의 우선순위가 내려가는 경우 그 새 우선순위의 목록 선두에 들어간다.

   POSIX.1-2008에 따르면 <tt>[[pthread_setschedprio(3)]]</tt> 외의 메커니즘을 이용해 스레드 우선순위를 (또는 정책을) 바꾸면 스레드가 그 우선순위의 목록 끝에 들어가야 한다.

4) <tt>[[sched_yield(2)]]</tt>를 호출한 스레드가 목록 끝으로 간다.

이 외의 어떤 이벤트도 같은 고정 우선순위의 실행 가능 스레드 대기 목록 내에서 `SCHED_FIFO` 정책으로 스케줄 된 스레드를 옮기지 않을 것이다.

`SCHED_FIFO` 스레드는 I/O 요청 때문에 블록 되거나, 더 높은 우선순위의 스레드에게 선점되거나, <tt>[[sched_yield(2)]]</tt>를 호출할 때까지 돈다.

### `SCHED_RR`: 라운드 로빈 스케줄링

`SCHED_RR`은 `SCHED_FIFO`를 간단히 개선한 것이다. 위에서 `SCHED_FIFO`에 대해 기술한 내용이 모두 `SCHED_RR`에 적용된다. 단, 각 스레드가 최대로 어떤 단위 시간(quantum) 동안만 돌 수 있다. `SCHED_RR` 스레드가 그 단위 시간 이상의 기간 동안 동작했으면 자기 우선순위의 목록 끝으로 가게 된다. `SCHED_RR` 스레드가 더 높은 우선순위의 스레드에게 선점되었다가 그 뒤에 실행을 재개하면 자기 라운드 로빈 단위 시간에서 만료 안 된 부분을 채우게 된다. <tt>[[sched_rr_get_interval(2)]]</tt>을 사용해 단위 시간 길이를 얻어올 수 있다.

### `SCHED_DEADLINE`: 산발 태스크 모델 마감 스케줄링

버전 3.14부터 리눅스에서는 마감 스케줄링 정책(`SCHED_DEADLINE`)을 지원한다. 이 정책은 현재 GEDF(Global Earliest Deadline First; 전역 최단 마감 우선)에 CBS(Constant Bandwidth Server; 일정 대역폭 서버)를 결합해서 구현되어 있다. 이 정책과 관련 속성들을 설정하거나 가져오려면 리눅스 전용인 <tt>[[sched_setattr(2)]]</tt> 및 <tt>[[sched_getattr(2)]]</tt> 시스템 호출을 사용해야 한다.

산발 태스크(sporadic task)란 일련의 작업(job)이 있고 각 작업이 주기당 최대 한 번씩 활성화되는 태스크이다. 각 작업에는 *상대적 마감 시간*이 있어서 그 전에 실행이 끝나야 하며, 작업 실행에 필요한 CPU 시간인 *연산 시간*이 있다. 새 작업을 실행해야 해서 태스크가 깨어나는 때를 *도착 시간*이라고 한다. (요청 시간이나 출고(release) 시간이라고도 한다.) *시작 시간*은 태스크가 실행을 시작하는 시간이다. 그리고 도착 시간에 상대적 마감 시간을 더해서 *절대적 마감 시간*을 얻는다.

다음 도표가 이 용어들을 분명히 보여 준다.

```text
도착/깨어남                     절대적 마감 시간
     |    시작 시간                    |
     |        |                        |
     v        v                        v
-----x--------xoooooooooooooooo--------x--------x---
              |<- 연산 시간 ->|
     |<------- 상대적 마감 시간 ------>|
     |<--------------- 주기 ------------------->|
```

<tt>[[sched_setattr(2)]]</tt>을 이용해 스레드에 `SCHED_DEADLINE` 정책을 설정할 때 "런타임(Runtime)", "마감(Deadline)", "주기(Period)"라는 세 가지 매개변수를 지정할 수 있다. 이 매개변수들이 앞서 언급한 용어들과 꼭 대응하는 것은 아니다. 일반적 관행은 런타임을 평균 연산 시간보다 (경성 실시간 태스크에는 최악 실행 시간보다) 큰 어떤 값으로, 마감을 상대적 마감 시간으로, 주기를 태스크의 주기로 설정하는 것이다. 그래서 `SCHED_DEADLINE` 스케줄링에서 다음과 같이 된다.

```text
도착/깨어남                      절대적 마감 시간
     |    시작 시간                     |
     |        |                         |
     v        v                         v
-----x--------xooooooooooooooooo--------x--------x---
              |<--- 런타임 ------->|
     |<------------- 마감 ------------->|
     |<--------------- 주기 -------------------->|
```

세 가지 마감 스케줄링 매개변수들이 `sched_attr` 구조체의 `sched_runtime`, `sched_deadline`, `sched_period` 필드에 해당한다. <tt>[[sched_setattr(2)]]</tt>을 참고하라. 이 필드들에서는 나노초로 값을 나타낸다. `sched_period`를 0으로 지정하면 `sched_deadline`과 같게 된다.

커널에서는 다음을 요구한다.

```text
sched_runtime <= sched_deadline <= sched_period
```

더불어 현재 구현에서는 모든 매개변수 값이 최소 1024여야 (즉 구현체의 해상도이기도 한 1마이크로초 살짝 넘는 값은 되어야) 하고 2^63보다 작아야 한다. 이 검사들 중 하나라도 실패하면 <tt>[[sched_setattr(2)]]</tt>이 `EINVAL` 오류로 실패한다.

지정된 자기 런타임을 초과하려 하는 스레드들을 CBS에서 제어하여 태스크들 간에 간섭이 없도록 보장한다.

마감 스케줄링 보장을 위해 커널에서는 주어진 제약에서 `SCHED_DEADLINE` 스레드들의 집합이 실현 가능하지 않은 (스케줄 가능하지 않은) 상황을 방지해야 한다. 그래서 커널에서는 `SCHED_DEADLINE` 정책 및 속성을 설정 내지 변경할 때 승인 검사를 수행한다. 이 승인 검사(admission test)에서 변경이 실행 가능한지 계산하고, 그렇지 않으면 <tt>[[sched_setattr(2)]]</tt>이 `EBUSY` 오류로 실패한다.

예를 들면 활용률 총합이 사용 가능 CPU 총개수 이하여야 한다. (하지만 이것으로 꼭 충분한 것은 아니다.) 스레드가 주기마다 최대로 런타임만큼 돌 수 있으므로 스레드의 활용률은 런타임을 주기로 나눈 것이다.

`SCHED_DEADLINE` 정책에 스레드를 승인할 때 보장하는 사항들을 충족하기 위해 `SCHED_DEADLINE` 스레드는 시스템에서 (사용자가 제어 가능한) 가장 높은 우선순위의 스레드이다. 어떤 `SCHED_DEADLINE` 스레드가 실행 가능하면 다른 정책으로 스케줄 된 어떤 스레드도 선점하게 된다.

`SCHED_DEADLINE` 정책으로 스케줄 된 스레드의 <tt>[[fork(2)]]</tt> 호출은 포크 시 초기화 플래그(아래 참고)를 설정해 두지 않았으면 `EAGAIN` 오류로 실패한다.

`SCHED_DEADLINE` 스레드가 <tt>[[sched_yield(2)]]</tt>를 호출하면 현재 작업을 내놓고 새 주기가 시작되기를 기다린다.

### `SCHED_OTHER`: 리눅스 기본 시공유 스케줄링

`SCHED_OTHER`는 고정 우선순위 0에서만 사용할 수 있다. (즉, 실시간 정책 하의 스레드가 항상 `SCHED_OTHER` 프로세스보다 우선순위가 높다.) `SCHED_OTHER`는 특별한 실시간 메커니즘이 필요하지 않은 모든 스레드를 위한 리눅스의 표준 시공유 스케줄러이다.

고정 우선순위 0의 목록에서 거기서만 정하는 *동적* 우선순위에 따라 실행할 스레드를 고른다. 동적 우선순위는 나이스 값(아래 참고)을 기반으로 하며 스레드가 실행 준비 상태인데 스케줄러에 의해 실행이 거절되는 시간 단위마다 증가한다. 이렇게 하면 `SCHED_OTHER` 스레드 전체 사이에 공정한 진행이 보장된다.

리눅스 커널 소스 코드에서는 `SCHED_OTHER` 정책을 `SCHED_NORMAL`이라고 쓴다.

### 나이스 값

나이스(nice) 값은 스케줄링 판단 때 CPU 스케줄러가 프로세스를 선호하거나 불호하도록 영향을 줄 수 있는 속성이다. `SCHED_OTHER` 및 `SCHED_BATCH`(아래 참고) 프로세스의 스케줄링에 영향을 준다. <tt>[[nice(2)]]</tt>, <tt>[[setpriority(2)]]</tt>, <tt>[[sched_setattr(2)]]</tt>을 이용해 나이스 값을 변경할 수 있다.

POSIX.1에 따르면 나이스 값은 프로세스별 속성이다. 즉, 프로세스 내의 스레드들이 한 나이스 값을 공유해야 한다. 하지만 리눅스에서 나이스 값은 스레스별 속성이다. 즉, 동일 프로세스 내의 스레드들이 서로 다른 나이스 값을 가질 수도 있다.

유닉스 시스템들 사이에서 나이스 값의 범위는 다양하다. 최신 리눅스에서 그 범위는 -20(높은 우선순위)에서 +19(낮은 우선순위)까지이다. 다른 어느 시스템에서는 그 범위가 -20..20이다. 아주 초기의 (리눅스 2.0 전의) 리눅스 커널에서는 범위가 -무한..15였다.

나이스 값이 `SCHED_OTHER` 프로세스들 사이의 상대적 스케줄링에 영향을 주는 정도도 유닉스 시스템 종류와 리눅스 커널 버전에 따라 다양하다.

커널 2.6.23에 CFS 스케줄러가 등장하면서 리눅스에서는 나이스 값의 상대적 차이가 훨씬 큰 영향을 주게 되는 알고리듬을 채택했다. 현행 구현에서 두 프로세스의 나이스 값의 단위 차이는 스케줄러가 더 높은 우선순위 프로세스를 선호하는 정도에 1.25만큼의 인자가 된다. 이 때문에 아주 낮은 나이스 값(+19)은 시스템에 다른 더 높은 우선순위 작업이 있을 때 프로세스에게 정말 거의 CPU를 주지 않으며, 높은 나이스 값(-20)은 필요한 응용(가령 오디오 응용)에게 CPU를 대부분 준다.

리눅스에서는 `RLIMIT_NICE` 자원 제한을 사용해 비특권 프로세스의 나이스 값을 올릴 수 있는 한계를 규정할 수 있다. 자세한 내용은 <tt>[[setrlimit(2)]]</tt>을 보라.

아래의 autogroup 기능 및 그룹 스케줄링에 대한 절들에 나이스 값에 대한 더 자세한 내용이 있다.

### `SCHED_BATCH`: 배치 프로세스 스케줄링

(리눅스 2.6.16부터.) `SCHED_BATCH`는 고정 우선순위 0에서만 사용할 수 있다. 이 정책은 (나이스 값에 기반한) 동적 우선순위에 따라 스레드를 스케줄 한다는 면에서 `SCHED_OTHER`와 비슷하다. 차이는 스케줄러에서 항상 스레드가 CPU 집약적이라고 가정하게 만든다는 점이다. 그에 따라 스케줄러에서 깨어나기 동작에 대해 약간의 스케줄링 페널티를 주어서 스케줄링 결정 때 그 스레드를 약간 냉대하게 된다.

비대화형이지만 자기 나이스 값을 낮추고 싶지는 않은 작업들에, 그리고 (작업의 태스크들 사이에서) 대화성이 과도한 선점을 유발하지 않는 결정론적 스케줄링 정책을 원하는 작업들에 이 정책이 유용하다.

### `SCHED_IDLE`: 아주 낮은 우선순위 작업 스케줄링

(리눅스 2.6.23부터.) `SCHED_IDLE`은 고정 우선순위 0에서만 사용할 수 있다. 이 정책에서는 프로세스 나이스 값이 어떤 영향도 주지 않는다.

이 정책은 극도로 낮은 (`SCHED_OTHER` 내지 `SCHED_BATCH` 정책의 +19 나이스 값보다 더 낮은) 우선순위로 작업을 돌리기 위한 것이다.

### 자식 프로세스 스케줄링 정책 초기화

각 스레드에는 포크 시 초기화(reset-on-fork) 플래그가 있다. 이 플래그가 설정되어 있으면 <tt>[[fork(2)]]</tt>로 생성된 자식이 특권 스케줄링 정책을 물려받지 않는다. 다음 중 한 방법으로 포크 시 초기화 플래그를 설정할 수 있다.

* <tt>[[sched_setscheduler(2)]]</tt> 호출 시 `policy` 인자에 `SCHED_RESET_ON_FORK`를 OR 하기. (리눅스 2.6.32부터)

* <tt>[[sched_setattr(2)]]</tt> 호출 시 `attr.sched_flags`에 `SCHED_FLAG_RESET_ON_FORK` 플래그 지정하기.

이 두 API에서 사용하는 상수 이름이 다른 것에 유의하라. 유사하게 <tt>[[sched_getscheduler(2)]]</tt> 및 <tt>[[sched_getattr(2)]]</tt>을 이용해 포크 시 초기화 플래그의 상태를 가져올 수 있다.

포크 시 초기화 기능은 미디어 재생 응용을 위한 것이다. 그리고 응용에서 여러 자식 프로세스를 생성해서 `RLIMIT_RTTIME` 자원 제한(<tt>[[getrlimit(2)]]</tt> 참고)을 회피하는 것을 막는 데 쓸 수 있다.

엄밀하게 말하자면 포크 시 초기화 플래그가 설정된 경우 이후 생성되는 자식에 대해 다음 규칙들이 적용된다.

* 호출 스레드의 스케줄링 정책이 `SCHED_FIFO`나 `SCHED_RR`이면 자식 프로세스들에서 정책이 `SCHED_OTHER`로 초기화 된다.

* 호출 프로세스가 음수 나이스 값을 가지고 있으면 자식 프로세스들에서 나이스 값이 0으로 초기화 된다.

포크 시 초기화 플래그를 켜고 나면 스레드가 `CAP_SYS_NICE` 역능을 가진 경우에만 되돌릴 수 있다. <tt>[[fork(2)]]</tt>로 생성된 자식 프로세스에서는 이 플래그가 꺼져 있다.

### 특권과 자원 제한

리눅스 커널 2.6.12 전에서는 특권(`CAP_SYS_NICE`) 스레드만 0 아닌 고정 우선순위를 설정할 (즉 실시간 스케줄링 정책을 설정할) 수 있다. 비특권 스레드가 할 수 있는 유일한 변경은 `SCHED_OTHER` 정책을 설정하는 것이고, 호출자의 실효 사용자 ID가 정책 변경 대상 스레드(즉 `pid`로 지정한 스레드)의 실제 사용자 ID나 실효 사용자 ID와 일치하는 경우에만 가능하다.

`SCHED_DEADLINE` 정책을 설정하거나 변경하려면 스레드에게 특권(`CAP_SYS_NICE`)이 있어야 한다.

리눅스 2.6.12부터 `RLIMIT_RTPRIO` 자원 제한이 비특권 스레드의 `SCHED_RR` 및 `SCHED_FIFO` 정책을 위한 고정 우선순위의 상한을 규정한다. 스케줄링 정책 및 우선순위 변경에 대한 규칙은 다음과 같다.

* 비특권 스레드에게 `RLIMIT_RTPRIO` 연성 제한이 있으면 현재 우선순위와 `RLIMIT_RTPRIO` 연성 제한 중 높은 값 위로 우선순위를 설정할 수 없다는 제약 하에 자기 스케줄링 정책과 우선순위를 바꿀 수 있다.

* `RLIMIT_RTPRIO` 연성 제한이 0이면 우선순위를 낮추거나 비실시간 정책으로 전환하는 것만 허용된다.

* 같은 규칙들 하에서 다른 비특권 스레드가 이런 변경을 할 수도 있다. 단, 변경을 하려는 스레드의 실효 사용자 ID가 대상 스레드의 실제 사용자 ID나 실효 사용자 ID와 일치해야 한다.

* `SCHED_IDLE` 정책에는 특별한 규칙을 적용한다. 리눅스 커널 2.6.39 전에서는 이 정책으로 동작 중인 비특권 스레드가 `RLIMIT_RTPRIO` 자원 제한값과 상관없이 자기 정책을 바꿀 수 없다. 리눅스 커널 2.6.39부터는 나이스 값이 `RLIMIT_NICE` 자원 제한(<tt>[[getrlimit(2)]]</tt> 참고)이 허용하는 범위 내에 들어가기만 하면 `SCHED_BATCH`나 `SCHED_OTHER` 정책으로 전환할 수 있다.

특권(`CAP_SYS_NICE`) 스레드는 `RLIMIT_RTPRIO` 제한을 무시한다. 이전 커널에서처럼 스케줄링 정책과 우선순위를 임의로 바꿀 수 있다. `RLIMIT_RTPRIO`에 대한 자세한 내용은 <tt>[[getrlimit(2)]]</tt>을 보라.

### 실시간 및 마감 프로세스의 CPU 사용량 제한하기

`SCHED_FIFO`나 `SCHED_RR`, `SCHED_DEADLINE` 정책으로 스케줄 된 스레드에 블록 없는 무한 루프가 있으면 다른 모든 스레드가 영원히 CPU에 접근할 수 없게 막을 가능성이 있다. 리눅스 2.6.25 전에서 폭주하는 실시간 프로세스가 시스템을 얼리는 걸 막는 유일한 방법은 테스트 하는 응용보다 높은 고정 우선순위로 스케줄 한 셸을 (콘솔에서) 실행해 두는 것이었다. 그러면 예상한 대로 블록 내지 정지하지 않는 실시간 응용을 비상 중단 시킬 수 있다.

리눅스 2.6.25부터는 폭주하는 실시간 및 마감 프로세스를 다루기 위한 다른 기법들이 있다. 그 중 하나는 `RLIMIT_RTTIME` 자원 제한을 사용해 실시간 프로세스가 소모할 수 있는 CPU 시간에 상한을 설정하는 것이다. 자세한 내용은 <tt>[[setrlimit(2)]]</tt>을 보라.

리눅스 버전 2.6.25부터 제공하는 두 개의 `/proc` 파일을 사용하면 비실시간 프로세스들이 사용할 일정 양의 CPU 시간을 예약해 둘 수 있다. 이 방식으로 CPU 시간을 예약해 두면 (가령) 폭주 프로세스를 죽이는 데 이용할 루트 셸에게 CPU 시간이 좀 할당될 수 있다. 두 파일 모두 나노초 단위로 시간 값을 지정한다.

`/proc/sys/kernel/sched_rt_period_us`
:   이 파일은 CPU 대역폭 100%와 동등한 스케줄링 주기를 나타낸다. 이 파일의 값은 1에서 `INT_MAX`까지 범위일 수 있고, 그래서 1마이크로초에서 35분까지의 조작 범위가 나온다. 이 파일의 기본값은 1,000,000(1초)이다.

`/proc/sys/kernel/sched_rt_runtime_us`
:   이 파일의 값은 "주기" 시간 중 얼마만큼을 시스템의 실시간 및 마감 스케줄 프로세스들 전체가 사용할 수 있는지 나타낸다. 이 파일의 값은 -1에서 `INT_MAX`-1까지 범위일 수 있다. -1을 지정하면 런타임이 주기와 같게 된다. 즉, (커널 2.6.25 전의 리눅스 동작처럼) 비실시간 프로세스들을 위해 CPU 시간을 따로 떼어 두지 않는다. 이 파일의 기본값은 950,000(0.95초)이다. 즉 실시간 내지 마감 스케줄링 정책으로 돌고 있지 않은 프로세스들을 위해 CPU 시간 5%가 예약되어 있다.

### 응답 시간

I/O를 기다리며 블록 된 높은 우선순위 스레드가 다시 스케줄 되기까지는 어느 정도의 응답 시간이 있다. 장치 드라이버 작성자가 "느린" 인터럽트 핸들러를 사용함으로써 이 응답 시간을 크게 줄일 수 있다.

### 기타

<tt>[[fork(2)]]</tt>를 거치면서 자식 프로세스가 스케줄링 정책과 매개변수들을 물려받는다. <tt>[[execve(2)]]</tt>를 거치면서 스케줄링 정책과 매개변수들이 보존된다.

일반적으로 실시간 프로세스에서는 페이징 지연을 피하기 위해 메모리 잠그기가 필요하다. <tt>[[mlock(2)]]</tt>이나 <tt>[[mlockall(2)]]</tt>로 가능하다.

### autogroup 기능

리눅스 2.6.38부터 커널에서 자동 묶기(autogrouping)라는 기능을 제공한다. 많은 병렬 빌드 프로세스로 (즉 `make(1)`의 `-j` 플래그로) 리눅스 커널을 빌드 할 때처럼 여러 프로세스이고 CPU 집약적인 부하가 있을 때의 대화형 데스크톱 성능을 향상시킨다.

이 기능은 CFS 스케줄러와 함께 동작하며 커널이 `CONFIG_SCHED_AUTOGROUP`으로 구성돼 있어야 한다. 동작 중인 시스템에서 `/proc/sys/kernel/sched_autogroup_enabled` 파일을 통해 이 기능을 켜고 끈다. 0 값은 기능을 끄고 1 값은 켠다. `noautogroup` 매개변수로 커널을 부팅 한 경우가 아니면 이 파일의 기본값은 1이다.

<tt>[[setsid(2)]]</tt>를 통해 새 세션이 생겨날 때 새로운 autogroup이 생성된다. 예를 들면 새 터미널 창을 시작할 때 그렇게 된다. <tt>[[fork(2)]]</tt>로 생성된 새 프로세스는 부모의 autogroup 멤버십을 물려받는다. 따라서 세션 내의 프로세스 모두가 같은 autogroup의 구성원이다. 그러다 그룹 내의 마지막 프로세스가 종료될 때 autogroup이 자동으로 사라진다.

자동 묶기가 켜져 있을 때 autogroup의 구성원 모두가 동일한 커널 스케줄러 "태스크 그룹"에 들어간다. 그리고 CFS 스케줄러에서 사용하는 알고리듬이 태스크 그룹들 간에 CPU 사이클 분배를 균등하게 한다. 이게 대화형 데스크톱 성능에 주는 이득을 다음 예를 통해 알 수 있다.

같은 CPU를 두고 경쟁하는 autogroup 두 개가 있다고 하자. (즉, 단일 CPU 시스템이거나 SMP에서 `taskset(1)`을 써서 모든 프로세스를 같은 CPU로 가뒀다고 하자.) 첫 번째 그룹에는 `make -j10`으로 커널 빌드를 시작해서 생긴 CPU 위주 프로세스 10개가 있다. 다른 그룹에는 CPU 위주 프로세스 1개가 있는데, 바로 동영상 재생기이다. 자동 묶기의 효과는 두 그룹이 각각 CPU 사이클의 절반씩 받게 되는 것이다. 즉, 동영상 재생기가 재생 품질 저하를 일으킬 만한 9%의 CPU 사이클만 받는 게 아니라 50%를 받게 된다. SMP에서는 상황이 더 복잡하지만 일반적 효과는 동일하다. 즉, 다수의 CPU 위주 프로세스들이 있는 autogroup이 시스템 상의 다른 작업들을 희생시키며 CPU 사이클을 독차지하게 되지 않게 스케줄러가 작업 그룹들 사이에 CPU 사이클을 분배한다.

`/proc/[pid]/autogroup` 파일을 통해 프로세스의 autogroup(태스크 그룹) 멤버십을 볼 수 있다.

```text
$ cat /proc/1/autogroup
/autogroup-1 nice 0
```

이 파일을 이용해 autogroup에 할당된 CPU 대역폭을 변경할 수도 있다. 파일에 "나이스" 범위 안의 수를 써넣어서 autogroup의 나이스 값을 설정하면 된다. 허용 범위는 +19(낮은 우선순위)에서 -20(높은 우선순위)까지이다. (이 범위 밖의 값을 써넣으면 `write(2)`가 `EINVAL` 오류로 실패하게 된다.)

autogroup 나이스 설정의 의미는 프로세스 나이스 값과 같되, autogroup들의 상대적 나이스 값에 기반한 autogroup 단위의 CPU 사이클 분배에 적용한다. autogroup 내의 프로세스가 받는 CPU 사이클은 autogroup의 (다른 autogroup들에 대한) 나이스 값과 프로세스의 (같은 autogroup 내 다른 프로세스들에 대한) 나이스 값에 따른 결과물이다.

<tt>[[cgroups(7)]]</tt> CPU 제어를 이용해 프로세스를 루트 CPU cgroup이 아닌 cgroup에 두면 자동 묶기의 효과를 무시하게 된다.

autogroup 기능은 비실시간 정책들(`SCHED_OTHER`, `SCHED_BATCH`, `SCHED_IDLE`)로 스케줄 된 프로세스들만 묶는다. 실시간 및 마감 정책으로 스케줄 된 프로세스들은 묶지 않는다. 그런 프로세스들은 앞서 기술한 규칙들에 따라 스케줄 된다.

### 나이스 값과 그룹 스케줄링

커널이 `CONFIG_FAIR_GROUP_SCHED` 옵션으로 구성되어 있으면 (보통 그렇다) 비실시간 프로세스들을 (즉 `SCHED_OTHER`, `SCHED_BATCH`, `SCHED_IDLE` 정책으로 스케줄 되는 프로세스들을) 스케줄 할 때 CFS 스케줄러에서 "그룹 스케줄링"이라는 기법을 사용한다.

그룹 스케줄링에서는 "태스크 그룹" 단위로 스레드들을 스케줄 한다. 태스크 그룹들에는 계층 관계가 있으며 최상위에는 "루트 태스크 그룹"이라는 시스템의 최초 태스크 그룹이 있다. 다음 경우에 태스크 그룹이 생긴다.

* CPU cgroup 내의 모든 스레드들이 태스크 그룹을 형성한다. 이 태스크 그룹의 부모는 대응하는 부모 cgroup의 태스크 그룹이다.

* 자동 묶기가 켜져 있는 경우에 (암묵적으로) autogroup에 (즉 <tt>[[setsid(2)]]</tt>로 생겨난 동일 세션에) 들어간 모든 스레드들이 태스크 그룹을 형성한다. 그래서 새 autogroup 각각은 별개의 태스크 그룹이다. 루트 태스크 그룹은 그런 autogroup들 모두의 부모이다.

* 자동 묶기가 켜져 있는 경우에 루트 태스크 그룹은 암묵적으로 새 autogroup에 들어가지 않은 루트 CPU cgroup 내의 모든 프로세스들로 이뤄진다.

* 자동 묶기가 꺼져 있는 경우에 루트 태스크 그룹은 루트 CPU cgroup 내의 모든 프로세스들로 이뤄진다.

* 그룹 스케줄링이 꺼져 있는 경우에는 (즉 커널이 `CONFIG_FAIR_GROUP_SCHED` 없이 구성되었으면) 개념적으로 시스템 상의 모든 프로세스들이 한 개의 태스크 그룹에 들어간다.

그룹 스케줄링 하에서 스레드의 나이스 값은 *같은 태스크 그룹의 다른 스레드들에 대해서만* 스케줄링 결정에 영향을 준다. 이로 인해 유닉스 시스템의 전통적 나이스 값 의미론 측면에서는 좀 의외인 경우들이 있다. 구체적으로 자동 묶기가 켜져 있는 경우에 (여러 배포판에서 기본으로 그렇다) 프로세스에 <tt>[[setpriority(2)]]</tt>나 `nice(1)`를 사용하면 같은 세션(보통은 같은 터미널 창) 내에서 실행되는 다른 프로세스들에 대한 상대적 스케줄링에만 영향을 준다.

반대로 (예를 들어) 상이한 세션들에서 유일하게 CPU 위주 프로세스인 두 프로세스가 있을 때 (가령 서로 다른 터미널 창들이 있고 각각의 작업들이 서로 다른 autogroup에 결속돼 있을 때) 한 세션에 있는 프로세스의 나이스 값을 변경하는 것이 다른 세션의 프로세스에 대한 스케줄러의 결정 측면에서 *어떤 효과도 주지 않는다*. 이럴 때 유용할 수 있는 해결책은 다음과 같은 명령을 이용해 터미널 세션 내의 프로세스 *모두*에 대한 autogroup 나이스 값을 변경하는 것이다.

```text
$ echo 10 > /proc/self/autogroup
```

### 리눅스 주 커널의 실시간 기능

커널 버전 2.6.18부터 리눅스는 조금씩 실시간 역량을 갖추고 있다. 그 대부분은 이전의 `realtime-preempt` 패치 세트에서 온 것이다. 그 패치들이 주 커널로 완전히 병합될 때까지는 최고의 실시간 성능을 얻으려면 패치들을 설치해야 한다. 그 패치들은 <tt>patch-*커널버전*-rt*패치버전*</tt> 형태의 이름을 가지고 있으며 (<http://www.kernel.org/pub/linux/kernel/projects/rt/>)에서 내려받을 수 있다.

패치가 주 커널에 완전히 포함되기 전에 그 패치 없이는 커널 구성에 `CONFIG_PREEMPT_NONE`, `CONFIG_PREEMPT_VOLUNTARY`, `CONFIG_PREEMPT_DESKTOP`이라는 세 가지 선점 계층만 제공된다. 각각은 최악 스케줄링 지연을 전혀 줄여 주지 않거나, 조금 줄여 주거나, 상당히 줄여 준다.

패치가 적용되어 있거나 주 커널에 완전히 포함된 후에는 `CONFIG_PREEMPT_RT`라는 추가 구성 항목이 사용 가능해진다. 이를 선택하면 리눅스가 제대로 된 실시간 운영 체제로 바뀐다. 그러면 FIFO 및 RR 스케줄링 정책을 사용해 진짜 실시간 우선순위와 최소한의 최악 스케줄링 지연으로 스레드를 돌린다.

## NOTES

<tt>[[cgroups(7)]]</tt> CPU 제어를 사용해 프로세스 그룹의 CPU 소모를 제한할 수 있다.

원래 표준 리눅스는 배경 프로세스와 대화형 응용, 약한 실시간 응용(마감 시간을 일반적으로 지켜야 하는 응용)을 처리할 수 있는 범용 운영 체제였다. 리눅스 커널 2.6에서 커널 선점이 가능해졌고 새로 도입된 O(1) 스케줄러가 활성 태스크의 수와 상관없이 스케줄에 필요한 시간이 고정이고 결정론적임을 보장하기는 하지만 커널 버전 2.6.17까지는 진정한 실시간 컴퓨팅이 가능하지 않았다.

## SEE ALSO

`chcpu(1)`, `chrt(1)`, `lscpu(1)`, `ps(1)`, `taskset(1)`, `top(1)`, <tt>[[getpriority(2)]]</tt>, <tt>[[mlock(2)]]</tt>, <tt>[[mlockall(2)]]</tt>, <tt>[[munlock(2)]]</tt>, <tt>[[munlockall(2)]]</tt>, <tt>[[nice(2)]]</tt>, <tt>[[sched_get_priority_max(2)]]</tt>, <tt>[[sched_get_priority_min(2)]]</tt>, <tt>[[sched_getaffinity(2)]]</tt>, <tt>[[sched_getparam(2)]]</tt>, <tt>[[sched_getscheduler(2)]]</tt>, <tt>[[sched_rr_get_interval(2)]]</tt>, <tt>[[sched_setaffinity(2)]]</tt>, <tt>[[sched_setparam(2)]]</tt>, <tt>[[sched_setscheduler(2)]]</tt>, <tt>[[sched_yield(2)]]</tt>, <tt>[[setpriority(2)]]</tt>, <tt>[[pthread_getschedparam(3)]]</tt>, <tt>[[pthread_getaffinity_np(3)]]</tt>, <tt>[[pthread_setaffinity_np(3)]]</tt>, <tt>[[sched_getcpu(3)]]</tt>, <tt>[[capabilities(7)]]</tt>, <tt>[[cpuset(7)]]</tt>

*Programming for the real world - POSIX.4* by Bill O. Gallmeister, O'Reilly & Associates, Inc., ISBN 1-56592-074-0.

리눅스 커널 소스 파일 `Documentation/scheduler/sched-deadline.txt`, `Documentation/scheduler/sched-rt-group.txt`, `Documentation/scheduler/sched-design-CFS.txt`, `Documentation/scheduler/sched-nice-design.txt`

----

2019-08-02