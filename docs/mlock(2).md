## NAME

mlock, mlock2, munlock, mlockall, munlockall - 메모리 고정하고 풀기

## SYNOPSIS

```c
#include <sys/mman.h>

int mlock(const void *addr, size_t len);
int mlock2(const void *addr, size_t len, unsigned int flags);
int munlock(const void *addr, size_t len);

int mlockall(int flags);
int munlockall(void);
```

## DESCRIPTION

`mlock()`, `mlock2()`, `mlockall()`은 호출 프로세스 가상 주소 공간의 일부 내지 전체를 램에 고정하여 그 메모리가 스왑 영역으로 빠지지 않도록 한다.

`munlock()`과 `munlockall()`은 반대 동작을 수행한다. 즉, 호출 프로세스 가상 주소 공간의 일부 내지 전체의 고정을 풀어서 해당 주소 공간 범위의 페이지가 다시 커널 메모리 관리자 요구 시 스왑으로 나갈 수 있도록 한다.

메모리 고정과 해제는 페이지 단위로 이뤄진다.

### `mlock()`, `mlock2()`, `munlock()`

`mlock()`은 `addr`에서 시작해서 `len` 바이트만큼 이어지는 주소 범위의 페이지들을 고정한다. 호출이 성공 반환할 때 지정한 주소 범위를 일부라도 포함하는 모든 페이지들이 램에 상주해 있다고 보장된다. 그리고 이후 풀기 전까지는 그 페이지들이 램 내에 있다고 보장된다.

`mlock2()`도 `addr`에서 시작해서 `len` 바이트만큼 이어지는 지정한 범위의 페이지들을 고정한다. 하지만 호출이 성공 반환한 후 그 범위에 포함된 페이지들의 상태가 `flags` 인자 값에 따라 달라지게 된다.

`flags` 인자는 0 또는 다음 상수일 수 있다.

`MLOCK_ONFAULT`
:   현재 상주 중인 페이지들은 고정하고 범위 전체에 표시를 해서 나머지 비상주 페이지들이 페이지 폴트에 의해 채워질 때 고정되도록 한다.

`flags`가 0이면 `mlock2()`는 `mlock()`과 동일하게 동작한다.

`munlock()`은 `addr`에서 시작해서 `len` 바이트만큼 이어지는 주소 범위의 페이지들을 고정 해제한다. 이 호출 후에는 지정한 메모리 범위를 일부라도 포함하는 모든 페이지들을 다시 커널이 외부 스왑 공간으로 옮길 수 있게 된다.

### `mlockall()`, `munlockall()`

`mlockall()`은 호출 프로세스 주소 공간의 맵 된 페이지들을 모두 고정한다. 여기에는 코드와 데이터, 스택 세그먼트의 페이지들뿐 아니라 공유 라이브러리, 사용자 공간 커널 데이터, 공유 메모리, 메모리 맵 파일도 포함된다. 호출이 성공 반환할 때 맵 된 페이지들이 모두 램에 상주해 있다고 보장된다. 그리고 이후 풀기 전까지는 그 페이지들이 램 내에 있다고 보장된다.

`flags` 인자는 다음 상수를 1개 이상 비트 OR 해서 구성한다.

`MCL_CURRENT`
:   프로세스 주소 공간에 현재 맵 되어 있는 모든 페이지들을 고정한다.

`MCL_FUTURE`
:   프로세스 주소 공간에 향후 맵 되는 모든 페이지들을 고정한다. 예를 들어 힙과 스택을 키우는 데 필요한 새 페이지나 새로운 메모리 맵 파일 내지 공유 메모리 영역이 해당될 수 있을 것이다.

`MCL_ONFAULT` (리눅스 4.4부터)
:   `MCL_CURRENT`나 `MCL_FUTURE`, 또는 둘 모두와 함께 사용한다. 현재(`MCL_CURRENT`) 내지 향후(`MCL_FUTURE`) 매핑 모두에 폴트로 들어온 페이지를 고정하게 표시한다. `MCL_CURRENT`와 사용하면 현재 페이지들을 모두 고정하되 `mlockall()`이 부재 페이지를 폴트로 들이지는 않는다. `MCL_FUTURE`와 사용하면 향후 매핑 모두에 페이지가 폴트로 들어올 때 고정하게 표시하되 매핑 생성 시 고정으로 인해 페이지들이 채워지지는 않는다. `MCL_ONFAULT`는 `MCL_CURRENT`나 `MCL_FUTURE`, 또는 둘 모두와 함께 써야 한다.

`MCL_FUTURE`를 지정해 두면 이후 시스템 호출(가령 <tt>[[mmap(2)]]</tt>, <tt>[[sbrk(2)]]</tt>, <tt>[[malloc(3)]]</tt>)로 인해 고정 바이트 수가 허용 최대치(아래 참고)를 초과하게 되는 경우 그 시스템 호출이 실패할 수 있다. 같은 상황에서 마찬가지로 스택 성장이 실패할 수도 있는데, 커널이 스택 확장을 거부하고 프로세스에게 `SIGSEGV` 시그널을 보내게 된다.

`munlockall()`은 호출 프로세스의 주소 공간으로 맵 되어 있는 페이지들을 모두 고정 해제한다.

## RETURN VALUE

성공 시 이 시스템 호출들은 0을 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다. 그리고 프로세스 주소 공간 내의 고정에 어떤 변화도 생기지 않는다.

## ERRORS

`ENOMEM`
:   (리눅스 2.6.9 및 이후) 호출자에게 0 아닌 `RLIMIT_MEMLOCK` 연성 자원 제한이 있는데 허용 한계보다 많은 메모리를 고정하려 했다. 프로세스에 특권(`CAP_IPC_LOCK`)이 있으면 이 제약이 적용되지 않는다.

`ENOMEM`
:   (리눅스 2.4 및 이전) 호출 프로세스가 램 절반을 넘게 고정하려 했다.

`EPERM`
:   호출자에게 특권이 없는데 요청 동작을 수행하기 위해 특권(`CAP_IPC_LOCK`)이 필요하다.

`mlock()`, `mlock2()`, `munlock()`:

`EAGAIN`
:   지정한 주소 범위의 일부 내지 전체를 고정할 수 없다.

`EINVAL`
:   덧셈 `addr+len`의 결과가 `addr`보다 작다. (가령 덧셈에서 오버플로가 일어났을 수도 있다.)

`EINVAL`
:   (리눅스 외) `addr`이 페이지 크기의 배수가 아니다.

`ENOMEM`
:   지정한 주소 범위의 일부가 프로세스 주소 공간 내의 맵 된 페이지에 대응하지 않는다.

`ENOMEM`
:   영역을 고정하거나 풀면 상이한 속성(가령 고정과 비고정)의 매핑 총개수가 허용 최대치를 초과하게 된다. (예를 들어 현재 고정된 매핑 중간의 어느 범위를 풀면 양쪽의 고정된 매핑과 가운데의 풀린 페이지를 합쳐서 세 개 매핑이 생긴다.)

`mlock2()`:

`EINVAL`
:   모르는 `flags`를 지정했다.

`mlockall()`:

`EINVAL`
:   모르는 `flags`를 지정했다. 또는 `MCL_FUTURE`와 `MCL_CURRENT` 어느 쪽도 없이 `MCL_ONFAULT`를 지정했다.

`munlockall()`:

`EPERM`
:   (리눅스 2.6.8 및 이전) 호출자에게 특권(`CAP_IPC_LOCK`)이 없다.

## VERSIONS

리눅스 4.4부터 `mlock2()`가 사용 가능하다. glibc 버전 2.27에서 지원이 추가되었다.

## CONFORMING TO

`mlock()`, `munlock()`, `mlockall()`, `munlockall()`: POSIX.1-2001, POSIX.1-2008, SVr4.

`mlock2()`는 리눅스 전용이다.

## AVAILABILITY

`mlock()` 및 `munlock()`이 사용 가능한 POSIX 시스템에는 `<unistd.h>`에 `_POSIX_MEMLOCK_RANGE`가 정의되어 있으며, `<limits.h>`의 상수 `PAGESIZE`나 (정의돼 있지 않으면) `sysconf(_SC_PAGESIZE)` 호출로 페이지의 바이트 수를 알아낼 수 있다.

`mlockall()` 및 `munlockall()`이 사용 가능한 POSIX 시스템에서는 `<unistd.h>`에 `_POSIX_MEMLOCK`이 0보다 큰 값으로 정의되어 있다. (<tt>[[sysconf(3)]]</tt>도 참고.)

## NOTES

메모리 고정의 주된 용처가 두 곳 있다. 실시간 알고리즘과 고수준 보안 데이터 처리이다. 실시간 응용에서는 시간이 예측 가능해야 하는데, 스케줄링에서도 그렇지만 페이징은 예상할 수 없는 프로그램 실행 지연의 주요 원인이다. 여기 더해 일반적으로 실시간 응용에서는 <tt>[[sched_setscheduler(2)]]</tt>로 실시간 스케줄러로 전환하게 된다. 암호 보안 소프트웨어에서는 종종 암호나 비밀키 같은 아주 중요한 바이트들을 자료 구조로 다룬다. 페이징 결과로 그 비밀값이 영속적인 스왑 저장 매체로 옮겨질 수 있을 것이고, 보안 소프트웨어가 램에서 그 비밀값을 삭제하고 종료한 후 오랫동안 공격자가 그 값에 접근 가능할 수도 있을 것이다. (하지만 메모리 고정과 상관없이 랩톱 및 일부 데스크톱의 절전 모드가 시스템 램의 사본을 디스크에 저장하게 된다는 점을 알고 있어야 한다.)

`mlockall()`을 이용해 페이지 폴트 지연을 방지하려는 실시간 프로세스에서는 시간이 중요한 영역에 진입하기 전에 스택 페이지를 충분하게 고정해서 함수 호출로 인해 페이지 폴트가 발생하지 않도록 하는 게 좋다. 충분히 큰 자동 변수(배열)를 할당해서 그 배열이 차지하는 메모리에 쓰기를 하여 스택 페이지들을 건드리는 함수를 호출하면 된다. 이렇게 하면 스택에 충분히 많은 페이지가 맵 되어서 램에 고정할 수 있다. 쓰기를 하는 것은 임계 구역에서 copy-on-write 페이지 폴트조차 일어나지 않게 하기 위해서이다.

<tt>[[fork(2)]]</tt>를 통해 생성되는 자식이 메모리 고정을 물려받지 않으며, <tt>[[execve(2)]]</tt> 과정 및 프로세스 종료 시에 자동으로 제거된다 (고정이 풀린다). <tt>[[fork(2)]]</tt>를 통해 생성되는 자식이 `mlockall()`의 `MCL_FUTURE` 및 `MCL_FUTURE | MCL_ONFAULT` 설정을 물려받지 않으며, <tt>[[execve(2)]]</tt> 과정에서 해제된다.

참고로 <tt>[[fork(2)]]</tt>에서는 주소 공간을 copy-on-write로 동작하게 만든다. 그 결과 이어지는 쓰기 접근이 페이지 폴트를 유발하게 되고, 이로 인해 실시간 프로세스에 높은 지연이 생길 수 있다. 따라서 `mlockall()`이나 `mlock()` 동작 후에 <tt>[[fork(2)]]</tt>를 호출하지 않는 게 매우 중요하다. 높은 우선순위로 도는 스레드와 같은 프로세스 내에서 낮은 우선순위로 도는 스레드에서 쓰기를 하는 경우도 마찬가지이다.

<tt>[[munmap(2)]]</tt>을 통해 주소 범위의 맵을 해제하면 주소 범위에 대한 메모리 고정이 자동으로 제거된다.

메모리 고정은 중첩되지 않는다. 즉, `mlock()`, `mlock2()`, `mlockall()` 호출로 여러 번 고정된 페이지가 해당 범위에 대한 `munlock()` 내지 `munlockall()` 호출 한 번으로 해제된다. 여러 위치에 맵 되어 있거나 여러 프로세스에 맵 되어 있는 페이지는 적어도 한 위치 내지 한 프로세스에 고정되어 있는 동안은 램에 고정되어 있는다.

`MCL_FUTURE`를 쓰는 `mlockall()` 호출 후에 그 플래그를 지정하지 않은 호출이 다시 있으면 `MCL_FUTURE` 호출에 의해 이뤄졌던 변경 사항들이 사라지게 된다.

`mlock2()`의 `MLOCK_ONFAULT` 플래그와 `mlockall()`의 `MCL_ONFAULT` 플래그를 이용하면 큰 메모리 매핑 중 (작은) 일부만 건드리는 응용에서 효율적인 메모리 고정이 가능하다. 그런 경우에 매핑 내 페이지 모두를 고정하는 것은 메모리 고정에 상당한 부담을 유발할 것이다.

### 리눅스 참고 사항

리눅스에서 `mlock()`, `mlock2()`, `munlock()`은 `addr`을 자동으로 가장 가까운 페이지 경계로 내림 한다. 하지만 `mlock()` 및 `munlock()`의 POSIX.1 명세에서는 `addr`이 페이지에 정렬되어 있기를 구현에서 요구할 수 있도록 한다. 따라서 이식 가능한 응용에서는 정렬되어 있게 해야 한다.

리눅스 한정인 `/proc/[pid]/status` 파일의 `VmLck` 필드는 ID가 `PID`인 프로세스의 메모리 중 몇 킬로바이트가 `mlock()`, `mlock2()`, `mlockall()`, 그리고 <tt>[[mmap(2)]]</tt> `MAP_LOCKED`에 의해 고정되어 있는지 보여 준다.

### 제한 및 권한

리눅스 2.6.8 및 이전에서는 메모리를 고정하려면 프로세스에게 특권(`CAP_IPC_LOCK`)이 있어야 하며 그 프로세스가 고정할 수 있는 메모리 양의 한계를 `RLIMIT_MEMLOCK` 연성 자원 제한이 규정한다.

리눅스 2.6.9부터는 특권 프로세스가 고정할 수 있는 메모리 양에 제한을 두지 않으며 `RLIMIT_MEMLOCK` 연성 자원 제한은 대신 비특권 프로세스가 고정할 수 있는 메모리 양의 한계를 규정한다.

## BUGS

리눅스 4.8 및 이전에서 비특권 (즉 `CAP_IPC_LOCK` 없는) 프로세스의 고정 메모리 양 계산에 버그가 있었다. 그래서 `addr`과 `len`으로 지정한 범위가 기존 고정과 겹치는 경우에 겹치는 영역의 이미 고정된 바이트들이 제한 검사 때 두 번 계산되었다. 그 이중 계산 때문에 프로세스의 "고정 메모리 총량" 값을 `RLIMIT_MEMLOCK` 제한을 넘도록 잘못 계산하여 성공했어야 할 `mlock()` 및 `mlock2()` 요청이 실패하는 결과를 낳을 수 있었다. 리눅스 4.9에서 버그가 고쳐졌다.

2.4.17까지의 2.4 시리즈 리눅스 커널에서 버그 때문에 <tt>[[fork(2)]]</tt>를 거칠 때 `mlockall()` `MCL_FUTURE` 플래그를 물려받았다. 커널 2.4.18에서 수정되었다.

커널 2.6.9부터는 특권 프로세스가 `mlockall(MCL_FUTURE)`을 호출하고 이후 특권을 버리는 (가령 실효 UID를 0 아닌 값으로 설정해서 `CAP_IPC_LOCK` 역능을 잃는) 경우에 `RLIMIT_MEMLOCK` 자원 제한에 도달해 있으면 이어지는 메모리 할당(가령 <tt>[[mmap(2)]]</tt>, <tt>[[brk(2)]]</tt>)이 실패하게 된다.

## SEE ALSO

<tt>[[mincore(2)]]</tt>, <tt>[[mmap(2)]]</tt>, <tt>[[setrlimit(2)]]</tt>, <tt>[[shmctl(2)]]</tt>, <tt>[[sysconf(3)]]</tt>, <tt>[[proc(5)]]</tt>, <tt>[[capabilities(7)]]</tt>

----

2021-03-22
