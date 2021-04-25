## NAME

reboot - 재부팅 하기 또는 Ctrl-Alt-Del 켜기/끄기

## SYNOPSIS

```c
/* 커널 버전 2.1.30부터 상수 심볼 이름 LINUX_REBOOT_*와 호출의
   네 번째 인자가 있다. */

#include <unistd.h>
#include <linux/reboot.h>

int reboot(int magic, int magic2, int cmd, void *arg);

/* glibc 및 대다수의 다른 libc들(uclibc, dietlibc, musl 등)에서는
   관련 상수 일부가 RB_*라는 심볼 이름을 가지고 있으며 라이브러리
   호출이 인자 1개짜리 시스템 호출 래퍼이다. */

#include <unistd.h>
#include <sys/reboot.h>

int reboot(int cmd);
```

## DESCRIPTION

`reboot()` 호출은 시스템을 재부팅 하거나, 재부팅 키 입력(기본적으로 Ctrl-Alt-Delete이므로 CAD로 줄임. `loadkeys(1)`로 바꿀 수 있음)을 켜거나 끈다.

`magic`이 `LINUX_REBOOT_MAGIC1`(즉 0xfee1dead)과 같고 `magic2`가 `LINUX_REBOOT_MAGIC2`(즉 672274793)와 같아야 한다. 아니면 이 시스템 호출이 (`EINVAL` 오류로) 실패한다. 하지만 `magic2`의 값으로 2.1.17부터 `LINUX_REBOOT_MAGIC2A`(즉 85072278)도, 2.1.97부터 `LINUX_REBOOT_MAGIC_2B`(즉 369367448)도, 2.5.71부터 `LINUX_REBOOT_MAGIC2C`(즉 537993216)도 허용한다. (이 상수들의 16진수 값에는 의미가 있다.)

`cmd` 인자는 다음 값을 가질 수 있다.

`LINUX_REBOOT_CMD_CAD_OFF`
:   (`RB_DISABLE_CAD`, 0). CAD를 끈다. 그러면 CAD 키 입력 시 init(1번 프로세스)에게 `SIGINT` 시그널이 가고, 그 프로세스에서 적절한 동작(아마도 모든 프로세스 죽이고, sync 하고, 재부팅 하기)을 정할 수 있다.

`LINUX_REBOOT_CMD_CAD_ON`
:   (`RB_ENABLE_CAD`, 0x89abcdef). CAD를 켠다. 그러면 CAD 키 입력 시 즉시 `LINUX_REBOOT_CMD_RESTART`에 대응하는 동작이 이뤄진다.

`LINUX_REBOOT_CMD_HALT`
:   (`RB_HALT_SYSTEM`, 0xcdef0123, 리눅스 1.1.76부터). "System halted." 메시지를 찍고서 시스템을 멈춘다. ROM 모니터가 있으면 그리로 제어가 간다. 미리 <tt>[[sync(2)]]</tt> 하지 않았으면 데이터가 유실된다.

`LINUX_REBOOT_CMD_KEXEC`
:   (`RB_KEXEC`, 0x45584543, 리눅스 2.6.13부터). <tt>[[kexec_load(2)]]</tt>로 미리 적재해 둔 커널을 실행한다. 커널이 `CONFIG_KEXEC`로 구성된 경우에만 이 옵션을 쓸 수 있다.

`LINUX_REBOOT_CMD_POWER_OFF`
:   (`RB_POWER_OFF`, 0x4321fedc, 리눅스 2.1.30부터). "Power down." 메시지를 찍고서 시스템을 멈추고, 가능한 경우 시스템의 전원을 모두 제거한다. 미리 <tt>[[sync(2)]]</tt> 하지 않았으면 데이터가 유실된다.

`LINUX_REBOOT_CMD_RESTART`
:   (`RB_AUTOBOOT`, 0x1234567). "Restarting system." 메시지를 찍고서 즉시 기본 재시작 동작을 수행한다. 미리 <tt>[[sync(2)]]</tt> 하지 않았으면 데이터가 유실된다.

`LINUX_REBOOT_CMD_RESTART2`
:   (0xa1b2c3d4, 리눅스 2.1.30부터). "Restarting system with command '%s'" 메시지를 찍고서 즉시 (`arg`에 준 명령 문자열을 사용해) 재시작을 수행한다. 미리 <tt>[[sync(2)]]</tt> 하지 않았으면 데이터가 유실된다.

`LINUX_REBOOT_CMD_SW_SUSPEND`
:   (`RB_SW_SUSPEND`, 0xd000fce1, 리눅스 2.5.18부터). 시스템을 디스크에 저장하고 중지(하이버네이션)한다. 커널이 `CONFIG_HIBERNATION`으로 구성된 경우에만 이 옵션을 쓸 수 있다.

수퍼유저만 `reboot()`를 호출할 수 있다.

위 동작들의 정확한 효과는 아키텍처에 따라 다를 수 있다. i386 아키텍처에서 추가 인자는 (2.1.122) 현재 아무 역할도 하지 않으며 커널 명령행 인자("reboot=...")로 재부팅이 웜과 콜드 중 어느 쪽이어야 하는지, 하드인지 BIOS를 거쳐야 하는지 정할 수 있다.

### PID 네임스페이스 내 동작 방식

리눅스 3.4부터는 기본 PID 네임스페이스 아닌 PID 네임스페이스에서 아래 나열한 `cmd` 값들 중 하나로 `reboot()`를 호출하면 그 네임스페이스의 "재부팅"을 수행한다. 즉 그 PID 네임스페이스의 "init" 프로세스가 즉시 종료되는데 <tt>[[pid_namespaces(7)]]</tt>에서 그 효과를 설명한다.

이 경우 `reboot()` 호출 시 `cmd`에 줄 수 있는 값들은 다음과 같다.

`LINUX_REBOOT_CMD_RESTART`, `LINUX_REBOOT_CMD_RESTART2`
:   "init" 프로세스가 종료된다. 부모 프로세스에서의 <tt>[[wait(2)]]</tt>은 자식이 `SIGHUP` 시그널로 죽었다고 보고한다.

`LINUX_REBOOT_CMD_POWER_OFF`, `LINUX_REBOOT_CMD_HALT`
:   "init" 프로세스가 종료된다. 부모 프로세스에서의 <tt>[[wait(2)]]</tt>은 자식이 `SIGINT` 시그널로 죽었다고 보고한다.

다른 `cmd` 값들에 대해선 `reboot()`가 -1을 반환하고 `errno`를 `EINVAL`로 설정한다.

## RETURN VALUE

시스템을 멈추거나 재시작하는 `cmd` 값들에서는 성공한 `reboot()` 호출이 반환하지 않는다. 다른 `cmd` 값들에서는 성공 시 0을 반환한다. 모든 경우에서 실패 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EFAULT`
:   `LINUX_REBOOT_CMD_RESTART2`에서 사용자 공간 데이터를 가져오는 데 문제 발생.

`EINVAL`
:   매직 넘버나 `cmd` 잘못됨.

`EPERM`
:   호출 프로세스가 `reboot()`를 호출할 충분한 특권을 가지고 있지 않음. 호출자가 자기 사용자 네임스페이스 내에서 `CAP_SYS_BOOT`를 가지고 있어야 한다.

## CONFORMING TO

`reboot()`는 리눅스 전용이므로 이식성이 있어야 하는 프로그램에서는 사용하지 말아야 한다.

## SEE ALSO

`systemctl(1)`, `systemd(1)`, <tt>[[kexec_load(2)]]</tt>, <tt>[[sync(2)]]</tt>, <tt>[[bootparam(7)]]</tt>, <tt>[[capabilities(7)]]</tt>, `ctrlaltdel(8)`, `halt(8)`, `shutdown(8)`

----

2021-03-22
