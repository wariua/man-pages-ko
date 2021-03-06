## NAME

bdflush - 변경 버퍼 플러시 데몬을 시작하고, 플러시하고, 조정하기

## SYNOPSIS

```c
#include <sys/kdaemon.h>

int bdflush(int func, long *address);
int bdflush(int func, long data);
```

*주의*: 이 시스템 호출에 대한 glibc 래퍼가 없다. VERSIONS 참고.

## DESCRIPTION

*주의*: 리눅스 2.6부터 이 시스템 호출이 제거 예정으로 표시되었으며 아무것도 하지 않는다. 향후 커널 릴리스에서 완전히 사라질 것 같다. `bdflush()`가 수행하던 작업을 요즘은 커널 스레드 `pdflush`가 맡고 있다.

`bdflush()`는 변경 버퍼 플러시(buffer-dirty-flush) 데몬을 시작하고, 플러시하고, 조정한다. 특권이 있는 (`CAP_SYS_ADMIN` 역능이 있는) 프로세스만 `bdflush()`를 호출할 수 있다.

`func`가 음수이거나 0이고 데몬이 아직 시작되지 않았으면 `bdflush()`가 데몬 코드로 진입하고 절대 반환하지 않는다.

`func`가 1이면 변경 버퍼들 일부를 디스크로 기록한다.

`func`가 2 이상이고 짝수(하위 비트가 0)이면 `address`가 `long` 워드의 주소이고 그 주소를 통해 `(func-2)/2` 번 튜닝 매개변수가 호출자에게 반환된다.

`func`가 3 이상이고 홀수(하위 비트가 1)이면 `data`가 `long` 워드이고 그 값을 `(func-3)/2` 번 튜닝 매개변수에 설정한다.

매개변수들과 그 값들, 유효 범위가 리눅스 커널 소스 파일 `fs/buffer.c`에 정의돼 있다.

## RETURN VALUE

`func`가 음수이거나 0이고 데몬이 성공적으로 시작하는 경우 `bdflush()`가 절대 반환하지 않는다. 그 외 경우에, 성공 시에는 반환 값이 0이며 오류 시에는 -1이고 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EBUSY`
:   다른 프로세스에서 이미 데몬 코드에 진입한 후에 진입을 시도하였다.

`EFAULT`
:   `address`가 접근 가능한 주소 공간 밖을 가리킨다.

`EINVAL`
:   유효하지 않은 매개변수 번호로 읽거나 쓰려고 시도했거나, 매개변수에 유효하지 않은 값을 쓰려고 시도했다.

`EPERM`
:   호출자에게 `CAP_SYS_ADMIN` 역능이 없다.

## VERSIONS

glibc 버전 2.23부터 이 구식 시스템 호출을 더이상 지원하지 않는다.

## CONFORMING TO

`bdflush()`는 리눅스 전용이므로 이식성이 있어야 하는 프로그램에서는 사용하지 말아야 한다.

## SEE ALSO

`sync(1)`, <tt>[[fsync(2)]]</tt>, <tt>[[sync(2)]]</tt>

----

2021-03-22
