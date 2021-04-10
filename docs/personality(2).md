## NAME

personality - 프로세스 실행 도메인 설정하기

## SYNOPSIS

```c
#include <sys/personality.h>

int personality(unsigned long persona);
```

## DESCRIPTION

리눅스는 각 프로세스마다 다른 실행 도메인, 즉 인격을 지원한다. 실행 도메인의 여러 역할 중에는 시그널 번호를 시그널 동작으로 사상할 방식을 리눅스에게 알려주는 것이 있다. 실행 도메인 시스템을 통해 리눅스가 다른 유닉스 계열 운영 체제에서 컴파일 한 바이너리를 제한적으로나마 지원할 수 있다.

`persona`가 0xffffffff가 아니면 `personality()`는 호출자의 실행 도메인을 `persona`로 지정한 값으로 설정한다. `persona`를 0xffffffff로 지정하여 현재 페르소나를 변경 없이 얻어올 수 있다.

사용 가능한 실행 도메인의 목록을 `<sys/personality.h>`에서 찾을 수 있다. 실행 도메인은 32비트 값이며 그 중 상위 세 바이트는 커널이 특정 시스템 호출의 동작 방식을 과거 내지 아키텍처에 한정된 특이성을 흉내내도록 변경하게끔 하는 플래그들을 위한 것이다. 최하위 바이트는 커널이 취해야 하는 인격을 규정하는 값이다. 플래그 값들은 다음과 같다.

`ADDR_COMPAT_LAYOUT` (리눅스 2.6.9부터)
:   이 플래그가 설정돼 있으면 구식 가상 주소 공간 레이아웃을 제공한다.

`ADDR_NO_RANDOMIZE` (리눅스 2.6.12부터)
:   이 플래그가 설정돼 있으면 주소 공간 배치 무작위화를 끈다.

`ADDR_LIMIT_32BIT` (리눅스 2.2부터)
:   주소 공간을 32비트로 제한한다.

`ADDR_LIMIT_3GB` (리눅스 2.4.0부터)
:   이 플래그가 설정돼 있으면 <tt>[[mmap(2)]]</tt>에서 가상 메모리 덩어리를 탐색하는 오프셋으로 0xc0000000을 사용한다. 아니면 0xffffe000을 사용한다.

`FDPIC_FUNCPTRS` (리눅스 2.6.11부터)
:   시그널 핸들러에 대한 사용자 공간 함수 포인터가 (특정 아키텍처에서) 디스크립터를 가리킨다.

`MMAP_PAGE_ZERO` (리눅스 2.4.0부터)
:   페이지 0을 읽기 전용으로 맵 한다. (SVr4의 이 동작 방식에 의존하는 바이너리를 지원하기 위해서.)

`READ_IMPLIES_EXEC` (리눅스 2.6.8부터)
:   이 플래그가 설정돼 있으면 <tt>[[mmap(2)]]</tt>에서 `PROT_READ`가 `PROT_EXEC`를 함의한다.

`SHORT_INODE` (리눅스 2.4.0부터)
:   효과 없음(?).

`STICKY_TIMEOUTS` (리눅스 1.2.0부터)
:   이 플래그가 설정돼 있으면 <tt>[[select(2)]]</tt>, <tt>[[pselect(2)]]</tt>, <tt>[[ppoll(2)]]</tt>이 시그널 핸들러에 의해 중단될 때 반환되는 타임아웃 인자를 변경하지 않는다.

`UNAME26` (리눅스 3.1부터)
:   <tt>[[uname(2)]]</tt>이 3.x 버전 번호 대신 2.6.40+ 버전 번호를 보고하게 한다. 2.6.x에서 3.x로의 커널 버전 부여 방식 변경을 처리할 수 없는 문제 응용들을 지원하기 위한 임시방편으로 추가되었다.

`WHOLE_SECONDS` (리눅스 1.2.0부터)
:   효과 없음(?).

사용 가능한 실행 도메인은 다음과 같다.

`PER_BSD` (리눅스 1.2.0부터)
:   BSD. (효과 없음.)

`PER_HPUX` (리눅스 2.4부터)
:   32비트 HP/UX 지원. 이 지원은 한번도 완벽한 적이 없다가 결국 중단되었고, 그래서 리눅스 4.0부터 이 값은 효과가 없다.

`PER_IRIX32` (리눅스 2.2부터)
:   IRIX 5 32비트. 한번도 완전하게 동작한 적이 없었고 리눅스 2.6.27에서 지원 중단됨. `STICKY_TIMEOUTS`를 함의함.

`PER_IRIX64` (리눅스 2.2부터)
:   IRIX 6 64비트. `STICKY_TIMEOUTS`를 함의함. 그 외에는 아무 효과 없음.

`PER_IRIXN32` (리눅스 2.2부터)
:   IRIX 6 신 32비트. `STICKY_TIMEOUTS`를 함의함. 그 외에는 아무 효과 없음.

`PER_ISCR4` (리눅스 1.2.0부터)
:   `STICKY_TIMEOUTS`를 함의함. 그 외에는 아무 효과 없음.

`PER_LINUX` (리눅스 1.2.0부터)
:   리눅스.

`PER_LINUX32` (리눅스 2.2부터)
:   [문서화 필요.]

`PER_LINUX32_3GB` (리눅스 2.4부터)
:   `ADDR_LIMIT_3GB`를 함의함.

`PER_LINUX_32BIT` (리눅스 2.0부터)
:   `ADDR_LIMIT_32BIT`를 함의함.

`PER_LINUX_FDPIC` (리눅스 2.6.11부터)
:   `FDPIC_FUNCPTRS`를 함의함.

`PER_OSF4` (리눅스 2.4부터)
:   OSF/1 v4. 알파에서, iov_len를 `int`로 정의했던 OSF/1 이전 버전들과의 호환성을 위해 사용자 버퍼에서 iov_len의 상위 32비트를 날린다.

`PER_OSR5` (리눅스 2.4부터)
:   `STICKY_TIMEOUTS`와 `WHOLE_SECONDS`를 함의함. 그 외에는 아무 효과 없음.

`PER_RISCOS` (리눅스 2.2부터)
:   [문서화 필요.]

`PER_SCOSVR3` (리눅스 1.2.0부터)
:   `STICKY_TIMEOUTS`와 `WHOLE_SECONDS`, `SHORT_INODE`를 함의함. 그 외에는 아무 효과 없음.

`PER_SOLARIS` (리눅스 2.4부터)
:   `STICKY_TIMEOUTS`를 함의함. 그 외에는 아무 효과 없음.

`PER_SUNOS` (리눅스 2.4.0부터)
:   `STICKY_TIMEOUTS`를 함의함. 라이브러리 및 동적 링커 탐색 위치를 `/usr/gnemul`로 바꾼다. 버그가 많고 전반적으로 관리가 안 되었고 거의 쓰이지 않았다. 리눅스 2.6.26에서 지원이 제거됐다.

`PER_SVR3` (리눅스 1.2.0부터)
:   `STICKY_TIMEOUTS`와 `SHORT_INODE`를 함의함. 그 외에는 아무 효과 없음.

`PER_SVR4` (리눅스 1.2.0부터)
:   `STICKY_TIMEOUTS`와 `MMAP_PAGE_ZERO`를 함의함. 그 외에는 아무 효과 없음.

`PER_UW7` (리눅스 2.4부터)
:   `STICKY_TIMEOUTS`와 `MMAP_PAGE_ZERO`를 함의함. 그 외에는 아무 효과 없음.

`PER_WYSEV386` (리눅스 1.2.0부터)
:   `STICKY_TIMEOUTS`와 `SHORT_INODE`를 함의함. 그 외에는 아무 효과 없음.

`PER_XENIX` (리눅스 1.2.0부터)
:   `STICKY_TIMEOUTS`와 `SHORT_INODE`를 함의함. 그 외에는 아무 효과 없음.

## RETURN VALUE

성공 시 이전 `persona`를 반환한다. 오류 시 -1을 반환하며 `errno`를 적절히 설정한다.

## ERRORS

`EINVAL`
:   커널에서 인격을 바꿀 수 없다.

## VERSIONS

리눅스 1.1.20에서 (따라서 안정 커널 릴리스로는 리눅스 1.2.0에서) 이 시스템 호출이 처음 등장했다. glibc 2.3에서 라이브러리 지원이 추가되었다.

## CONFORMING TO

`personality()`는 리눅스 전용이므로 이식성이 있어야 하는 프로그램에서는 사용하지 말아야 한다.

## SEE ALSO

`setarch(8)`

----

2017-09-15
