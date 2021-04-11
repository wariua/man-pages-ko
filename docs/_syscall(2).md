## NAME

\_syscall - 라이브러리 지원 없이 시스템 호출 부르기 (**구식**)

## SYNOPSIS

```c
#include <linux/unistd.h>

_syscall 매크로

원하는 시스템 호출
```

## DESCRIPTION

시스템 호출에 대해 알아야 할 중요한 것이 그 원형이다. 인자가 몇 개고, 타입이 무엇이고, 함수 반환 타입은 뭔지 알아야 한다. 시스템으로의 실제 호출을 쉽게 만들어 주는 매크로가 일곱 가지 있는데, 다음과 같은 형태이다.

```text
_syscallX(type,name,type1,arg1,type2,arg2,...)
```

여기서,

* `X`는 0-6이며 시스템 호출이 받는 인자 개수이다.

* `type`은 시스템 호출의 반환 타입이다.

* `name`은 시스템 호출의 이름이다.

* `typeN`은 N 번째 인자의 타입이다.

* `argN`은 N 번째 인자의 이름이다.

이 매크로들은 지정한 인자들로 `name`이라는 함수를 만든다. 소스 코드에서 그 \_syscall()을 포함시키고 나서 `name`으로 시스템 호출을 부른다.

## FILES

`/usr/include/linux/unistd.h`

## CONFORMING TO

이 매크로를 쓰는 방식은 리눅스 전용이며, 제거 예정이다.

## NOTES

커널 2.6.18 정도부터 사용자 공간에 제공되는 헤더 파일에서 \_syscall 매크로가 제거되었다. 대신 <tt>[[syscall(2)]]</tt>을 사용하라. (일부 아키텍처들, 특히 ia64에서는 한번도 \_syscall 매크로를 제공하지 않았다. 그런 아키텍처에서는 항상 <tt>[[syscall(2)]]</tt>이 필요했다.)

\_syscall() 매크로는 원형을 만들어 주지 *않는다*. 특히 C++ 사용자들을 위해서 직접 만들어 줘야 할 수도 있다.

시스템 호출들이 반드시 양수이거나 음수인 오류 코드만 반환해야 하는 것이 아니다. 오류를 어떻게 반환하는지 확실히 알려면 소스를 읽어 보아야 한다. 일반적으로는 표준 오류 코드의 음수 값, 예를 들어 `-EPERM`이다. \_syscall() 매크로들은 시스템 호출의 결과 `r`이 음수가 아닐 때는 `r`을 반환하고, `r`이 음수일 때는 -1을 반환하고 변수 `errno`를 -`r`로 설정하게 된다. 오류 코드들에 대해선 <tt>[[errno(3)]]</tt>를 보라.

시스템 호출을 정의할 때 인자 타입들은 값 전달이나 (구조체 같은 집합 타입의 경우) 포인터 전달이어야 한다.

## EXAMPLE

```c
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>
#include <linux/unistd.h>       /* _syscallX 매크로/관련 사항들 */
#include <linux/kernel.h>       /* struct sysinfo */

_syscall1(int, sysinfo, struct sysinfo *, info);

int
main(void)
{
    struct sysinfo s_info;
    int error;

    error = sysinfo(&s_info);
    printf("code error = %d\n", error);
    printf("Uptime = %lds\nLoad: 1 min %lu / 5 min %lu / 15 min %lu\n"
           "RAM: total %lu / free %lu / shared %lu\n"
           "Memory in buffers = %lu\nSwap: total %lu / free %lu\n"
           "Number of processes = %d\n",
           s_info.uptime, s_info.loads[0],
           s_info.loads[1], s_info.loads[2],
           s_info.totalram, s_info.freeram,
           s_info.sharedram, s_info.bufferram,
           s_info.totalswap, s_info.freeswap,
           s_info.procs);
    exit(EXIT_SUCCESS);
}
```

### 출력 예시

```text
code error = 0
uptime = 502034s
Load: 1 min 13376 / 5 min 5504 / 15 min 1152
RAM: total 15343616 / free 827392 / shared 8237056
Memory in buffers = 5066752
Swap: total 27881472 / free 24698880
Number of processes = 40
```

## SEE ALSO

`intro(2)`, <tt>[[syscall(2)]]</tt>, <tt>[[errno(3)]]</tt>

----

2019-03-06
