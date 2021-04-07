## NAME

bdflush - 버퍼 변경 기록 데몬을 시작하고, 기록시키고, 조정하기

## SYNOPSIS

```c
#include <sys/kdaemon.h>

int bdflush(int func, long *address);
int bdflush(int func, long data);
```

## DESCRIPTION

*주의*: 이 시스템 호출은 리눅스 2.6부터 제거 예정으로 표시되었으며 아무것도 하지 않는다. 향후 커널 릴리스에서 완전히 사라질 것 같다. `bdflush()`가 수행하던 작업을 요즘은 커널 스레드 `pdflush`가 맡고 있다.

`bdflush()`는 버퍼 변경 기록(buffer-dirty-flush) 데몬을 시작하고, 기록시키고, 조정한다. 특권이 있는 (`CAP_SYS_ADMIN` 역능이 있는) 프로세스만 `bdflush()`를 호출할 수 있다.

`func`가 음수이거나 0이고 데몬을 아직 시작하지 않았으면 `bdflush()`가 데몬 코드로 진입하고 절대 반환하지 않는다.

`func`가 1이면 변경된 버퍼들 일부를 디스크로 기록한다.

`func`가 2 이상이고 짝수(하위 비트가 0)이면 `address`가 `long` 워드의 주소이고 그 주소를 통해 `(func-2)/2`번 튜닝 매개변수를 호출자에게 반환한다.

`func`가 3 이상이고 홀수(하위 비트가 1)이면 `data`가 `long` 워드이고 그 값을 `(func-3)/2`번 튜닝 매개변수에 설정한다.

매개변수들과 그 값들, 유효 범위가 리눅스 커널 소스 파일 `fs/buffer.c`에 정의돼 있다.

## RETURN VALUE

`func`가 음수이거나 0이고 데몬이 성공적으로 시작하는 경우 `bdflush()`가 절대 반환하지 않는다. 그 외 경우에, 성공 시에는 반환 값이 0이며 오류 시에는 -1이고 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

<dl>
<dt><code>EBUSY</code></dt>
<dd>다른 프로세스에서 이미 진입한 후에 데몬 코드에 진입하려고 시도하였다.</dd>
<dt><code>EFAULT</code></dt>
<dd><code>address</code>가 접근 가능한 주소 공간 밖을 가리킨다.</dd>
<dt><code>EINVAL</code></dt>
<dd>유효하지 않은 매개변수 번호에 읽거나 쓰려고 시도했거나, 매개변수에 유효하지 않은 값을 쓰려고 시도했다.</dd>
<dt><code>EPERM</code></dt>
<dd>호출자에게 <code>CAP_SYS_ADMIN</code> 역능이 없다.</dd>
</dl>

## VERSIONS

glibc 버전 2.23부터 이 구식 시스템 호출을 더 이상 지원하지 않는다.

## CONFORMING TO

`bdflush()`는 리눅스 전용이므로 이식성이 있어야 하는 프로그램에서는 사용하지 말아야 한다.

## SEE ALSO

`sync(1)`, <tt>[[fsync(2)]]</tt>, <tt>[[sync(2)]]</tt>

----

2016-10-08
