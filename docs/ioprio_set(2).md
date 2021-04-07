## NAME

ioprio_get, ioprio_set - I/O 스케줄링 클래스 및 우선순위 얻기/설정하기

## SYNOPSIS

```c
int ioprio_get(int which, int who);
int ioprio_set(int which, int who, int ioprio);
```

*주의*: 이 시스템 호출에 대한 glibc 래퍼가 없다. NOTES 참고.

## DESCRIPTION

`ioprio_get()` 및 `ioprio_set()` 시스템 호출은 한 개 또는 여러 스레드의 I/O 스케줄링 클래스와 우선순위를 얻고 설정한다.

`which`와 `who` 인자는 시스템 호출의 동작 대상이 되는 스레드(들)을 나타낸다. `which` 인자는 `who`를 해석하는 방식을 결정하며 다음 값들 중 하나이다.

<dl>
<dt><code>IOPRIO_WHO_PROCESS</code></dt>
<dd><code>who</code>가 프로세스 ID나 스레드 ID이며 단일 프로세스 내지 스레드를 나타낸다. <code>who</code>가 0이면 호출 스레드가 대상이다.</dd>

<dt><code>IOPRIO_WHO_PGRP</code></dt>
<dd><code>who</code>가 프로세스 그룹 ID이며 프로세스 그룹의 구성원 모두를 나타낸다. <code>who</code>가 0이면 호출자가 구성원인 프로세스 그룹이 대상이다.</dd>

<dt><code>IOPRIO_WHO_USER</code></dt>
<dd><code>who</code>가 사용자 ID이며 실제 UID가 일치하는 프로세스 모두를 나타낸다.</dd>
</dl>

`ioprio_get()` 호출 시에 `which`가 `IOPRIO_WHO_PGRP`이나 `IOPRIO_WHO_USER`이고 `who`에 여러 프로세스가 걸리는 경우에는 걸리는 프로세스 전체에서 가장 높은 우선순위를 반환하게 된다. 어떤 우선순위가 더 높다는 것은 더 높은 우선순위 클래스에 속하거나 (`IOPRIO_CLASS_RT`가 가장 높은 우선순위 클래스이고 `IOPRIO_CLASS_IDLE`이 가장 낮음) 아니면 같은 우선순위 클래스에 속하면서 우선순위 단계가 더 높은 것이다 (낮은 우선순위 수가 높은 우선순위 단계를 뜻함).

`ioprio_set()`에 주는 `ioprio` 인자는 대상 프로세스(들)에 부여할 스케줄링 클래스와 우선순위 모두를 나타내는 비트 마스크이다. 다음 매크로를 사용해 `ioprio` 값을 합치거나 분해한다.

<dl>
<dt><code>IOPRIO_PRIO_VALUE(class, data)</code></dt>
<dd>이 매크로는 스케줄링 클래스(<code>class</code>)와 우선순위(<code>data</code>)를 받아서 두 값을 합친 <code>ioprio</code> 값을 만들어 매크로 결과로 반환한다.</dd>

<dt><code>IOPRIO_PRIO_CLASS(mask)</code></dt>
<dd>이 매크로는 <code>mask</code>(<code>ioprio</code> 값)를 받아서 I/O 클래스 요소를 반환한다. 즉 <code>IOPRIO_CLASS_RT</code>, <code>IOPRIO_CLASS_BE</code>, <code>IOPRIO_CLASS_IDLE</code> 중 하나를 반환한다.</dd>

<dt><code>IOPRIO_PRIO_DATA(mask)</code></dt>
<dd>이 매크로는 <code>mask</code>(<code>ioprio</code> 값)를 받아서 우선순위(<code>data</code>) 요소를 반환한다.</dd>
</dl>

스케줄링 클래스와 우선순위에 대한 추가 정보와 `ioprio`를 0으로 지정했을 때의 의미에 대해선 NOTES 절을 참고하라.

읽기와 동기적인 (`O_DIRECT`, `O_SYNC`) 쓰기에서 I/O 우선순위를 지원한다. 비동기 쓰기에는 I/O 우선순위를 지원하지 않는데, 메모리를 변경하는 프로그램의 맥락 밖에서 개시되므로 프로그램별 우선순위가 적용되지 않기 때문이다.

## RETURN VALUE

성공 시 `ioprio_get()`은 `which`와 `who`로 지정한 기준에 맞는 프로세스들 중 가장 높은 I/O 우선순위를 가진 프로세스의 `ioprio` 값을 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

성공 시 `ioprio_set()`은 0을 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

<dl>
<dt><code>EINVAL</code></dt>
<dd><code>which</code>나 <code>ioprio</code>에 유효하지 않은 값. <code>ioprio</code>에 사용 가능한 스케줄러 클래스와 우선순위는 NOTES 절 참고.</dd>
<dt><code>EPERM</code></dt>
<dd>호출 프로세스에게 지정한 프로세스(들)에게 그 <code>ioprio</code>를 부여할 특권이 없다. <code>ioprio_set()</code>에 필요한 특권에 대한 자세한 내용은 NOTES 절 참고.</dd>
<dt><code>ESRCH</code></dt>
<dd><code>which</code>와 <code>who</code>의 지정 내용에 일치하는 프로세스를 찾을 수 없다.</dd>
</dl>

## VERSIONS

리눅스 커널 2.6.13부터 이 시스템 호출들이 사용 가능하다.

## CONFORMING TO

이 시스템 호출들은 리눅스 전용이다.

## NOTES

glibc에서 이 시스템 호출의 래퍼를 제공하지 않는다. <tt>[[syscall(2)]]</tt>을 이용해 호출해야 한다.

둘 이상의 프로세스 내지 스레드가 I/O 문맥을 공유할 수 있다. <tt>[[clone(2)]]</tt>에 `CLONE_IO` 플래그를 써서 호출한 경우가 그렇다. 하지만 기본적으로는 프로세스의 스레드들이 같은 I/O 문맥을 공유하지 *않는다*. 따라서 프로세스의 모든 스레드의 I/O 우선순위를 바꾸고 싶다면 스레드 각각에 `ioprio_set()`을 호출해야 할 수도 있다. 그때 필요한 스레드 ID는 <tt>[[gettid(2)]]</tt>나 <tt>[[clone(2)]]</tt>이 반환한 값이다.

I/O 우선순위를 지원하는 I/O 스케줄러를 사용할 때에만 이 시스템 호출들에 효력이 있다. 커널 2.6.17 현재 그런 유일한 스케줄러는 Completely Fair Queuing (CFQ) I/O 스케줄러이다.

스레드에 어떤 I/O 스케줄러도 설정하지 않았으면 기본적으로 I/O 우선순위가 CPU 나이스 값(<tt>[[setpriority(2)]]</tt>)을 따른다. 리눅스 커널 버전 2.6.24 전에서는 `ioprio_set()`으로 I/O 우선순위를 한번 설정하고 나면 I/O 스케줄링 동작을 기본으로 되돌릴 방법이 없었다. 리눅스 2.6.24부터는 `ioprio`를 0으로 지정해서 기본 I/O 스케줄링 동작으로 되돌릴 수 있다.

### I/O 스케줄러 선택

I/O 스케줄러는 특수 파일 `/sys/block/<device>/queue/scheduler`를 통해 장치별로 선택한다.

`/sys` 파일 시스템을 통해 현재 I/O 스케줄러를 볼 수 있다. 예를 들어 다음 명령은 현재 커널에 적재된 모든 스케줄러들의 목록을 보여 준다.

```
$ cat /sys/block/sda/queue/scheduler
noop anticipatory deadline [cfq]
```

꺽쇠괄호가 쳐진 게 그 장치(여기선 `sda`)에 실제 사용 중인 스케줄러이다. 다른 스케줄러를 선택하려면 이 파일에 새 스케줄러의 이름을 기록하면 된다. 예를 들어 다음 명령은 `sda` 장치의 스케줄러를 `cfq`로 설정한다.

```
$ su
Password:
# echo cfq > /sys/block/sda/queue/scheduler
```

### Completely Fair Queuing (CFQ) I/O 스케줄러

(CFQ Time Sliced라고도 하는) 버전 3부터 CFQ에서는 CPU 스케줄링과 비슷한 I/O 나이스 단계를 구현한다. 그 나이스 단계들을 세 가지 스케줄링 클래스로 묶으며, 각 클래스마다 한 개 이상의 우선순위 단계가 있다.

<dl>
<dt><code>IOPRIO_CLASS_RT</code> (1)</dt>
<dd>실시간 I/O 클래스이다. 이 스케줄링 클래스에는 다른 어떤 클래스보다 높은 우선순위를 준다. 즉 이 클래스의 프로세스들에 매번 최우선 디스크 접근권을 준다. 따라서 이 I/O 클래스는 조심해서 사용할 필요가 있다. I/O가 실시간인 프로세스 하나가 전체 시스템을 굶게 만들 수 있기 때문이다. 실시간 클래스 내에는 8단계의 클래스 데이터(우선순위)가 있어서 이 프로세스가 각 서비스마다 정확히 얼마나 오래 디스크를 필요로 하는지 결정한다. 가장 높은 실시간 우선순위 단계는 0이고 가장 낮은 단계는 7이다. 향후에는 원하는 데이터 속도를 전달해서 더 직접적으로 성능과 연결되도록 바뀔 수도 있다.</dd>

<dt><code>IOPRIO_CLASS_BE</code> (2)</dt>
<dd>최선 스케줄링 클래스이다. 특별히 I/O 우선순위를 설정하지 않은 프로세스들의 기본값이다. 클래스 데이터(우선순위)는 프로세스가 얼마나 많은 I/O 대역폭을 얻게 될지 결정한다. 최선 우선순위 단계들은 CPU 나이스 값과 유사하다. (<tt>[[getpriority(2)]]</tt> 참고.) 우선순위 단계는 최선 스케줄링 클래스 내의 다른 프로세스들에 대한 상대적 우선도를 결정한다. 우선순위 단계의 범위는 0(최고)에서 7(최저)까지이다.</dd>

<dt><code>IOPRIO_CLASS_IDLE</code> (3)</dt>
<dd>유휴 스케줄링 클래스이다. 이 단계에서 동작하는 프로세스는 디스크를 필요로 하는 다른 프로세스가 없을 때에만 I/O 시간을 얻는다. 유휴 클래스에는 클래스 데이터가 없다. 프로세스에 이 우선순위 클래스를 부여할 때는 주의할 필요가 있는데, 더 높은 우선도의 프로세스들이 계속해서 디스크에 접근하면 그 프로세스가 굶주리게 될 수 있기 때문이다.</dd>
</dl>

CFQ I/O 스케줄러에 대한 추가 내용과 예시 프로그램은 커널 소스 파일 `Documentation/block/ioprio.txt`를 참고하라.

### I/O 우선순위 설정에 필요한 권한

두 가지 기준에 따라 프로세스 우선순위 변경이 허가되거나 거부된다.

<dl>
<dt>프로세스 소유권</dt>
<dd>비특권 프로세스는 호출 프로세스의 실제 UID나 실효 UID와 일치하는 실제 UID를 가진 프로세스에 대해서만 I/O 우선순위를 설정할 수 있다. <code>CAP_SYS_NICE</code> 역능을 가진 프로세스는 모든 프로세스의 우선순위를 바꿀 수 있다.</dd>

<dt>원하는 우선순위</dt>
<dd>아주 높은 우선순위(<code>IOPRIO_CLASS_RT</code>)를 설정하려면 <code>CAP_SYS_ADMIN</code> 역능이 필요하다. 커널 버전 2.6.24까지에서는 아주 낮은 우선순위(<code>IOPRIO_CLASS_IDLE</code>)를 설정하는 데도 <code>CAP_SYS_ADMIN</code>이 필요했지만 리눅스 2.6.25부터는 필요하지 않다.</dd>
</dl>

`ioprio_set()` 호출 시 두 규칙 모두를 따라야 하며, 그렇지 않으면 호출이 `EPERM` 오류로 실패하게 된다.

## BUGS

이 페이지에서 기술하는 함수 원형과 매크로를 정의하는 적절한 헤더 파일을 glibc에서 아직 제공하지 않는다. `linux/ioprio.h`에서 적절한 정의들을 찾을 수 있다.

## SEE ALSO

`ionice(1)`, <tt>[[getpriority(2)]]</tt>, <tt>[[open(2)]]</tt>, <tt>[[capabilities(7)]]</tt>, <tt>[[cgroups(7)]]</tt>

리눅스 커널 소스 트리의 `Documentation/block/ioprio.txt`

----

2019-03-06
