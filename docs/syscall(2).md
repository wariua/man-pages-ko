## NAME

syscall - 간접적 시스템 호출

## SYNOPSIS

```c
#include <unistd.h>
#include <sys/syscall.h>    /* SYS_xxx 정의 */

long syscall(long number, ...);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`syscall()`:
:   glibc 2.19부터:
    :   `_DEFAULT_SOURCE`

    glibc 2.19 전:
    :   `_BSD_SOURCE || _SVID_SOURCE`

## DESCRIPTION

`syscall()`은 작은 라이브러리 함수이며, 지정한 번호 `number`와 지정한 인자들을 가진 어셈블리 언어 인터페이스의 시스템 호출을 부른다. 예를 들어 C 라이브러리에 래퍼 함수가 없는 시스템 호출을 부를 때 `syscall()`이 유용하다.

`syscall()`은 시스템 호출을 하기 전에 CPU 레지스터들을 저장해 두고, 시스템 호출 반환 후 그 레지스터들을 복원하며, 시스템 호출이 오류를 반환했으면 그 오류를 <tt>[[errno(3)]]</tt>에 저장한다.

시스템 호출 번호 상수들을 헤더 파일 `<sys/syscall.h>`에서 찾을 수 있다.

## RETURN VALUE

부르려고 하는 시스템 호출이 반환 값을 규정한다. 일반적으로 반환 값 0은 성공을 나타낸다. 반환 값 -1은 오류를 나타내며 `errno`에 오류 번호가 저장된다.

## NOTES

4BSD에서 `syscall()`이 처음 등장했다.

### 아키텍처별 요구 사항

각각의 아키텍처 ABI마다 시스템 호출을 어떻게 커널로 전달하는지에 대한 나름의 요구 사항이 있다. glibc 래퍼가 있는 시스템 호출(즉, 대부분의 시스템 호출)에서는 아키텍처에 맞는 방식으로 인자를 레지스터로 복사하는 세부 사항들을 glibc가 다뤄 준다. 하지만 `syscall()`을 이용해 시스템 호출을 할 때는 호출자가 아키텍처별 세부 사항들을 다뤄야 할 수도 있다. 특정 32비트 아키텍처들에서 이런 요구 사항을 가장 흔히 만나게 된다.

예를 들어 ARM 아키텍처 임베디드 ABI(EABI)에서는 64비트 값(가령 `long long`)을 짝수 번 레지스터 쌍에 정렬시켜야 한다. 그래서 리틀 엔디언 모드 EABI ARM 아키텍처에서 glibc가 제공하는 래퍼 대신 `syscall()`을 사용한다면 <tt>[[readahead(2)]]</tt> 시스템 호출을 다음과 같이 부르게 될 것이다.

```c
syscall(SYS_readahead, fd, 0,
        (unsigned int) (offset & 0xFFFFFFFF),
        (unsigned int) (offset >> 32),
        count);
```

`offset` 인자가 64비트이고 첫 번째 인자(`fd`)를 `r0`로 전달하므로 호출자가 64비트 값을 직접 쪼개고 정렬해서 `r2`/`r3` 레지스터 쌍으로 전달되게 해야 한다. 즉 `r1`에 더미 값을 넣는다 (두 번째 인자 0). 쪼개기가 (플랫폼에 대한 C ABI에 따른) 엔디언 규약을 따르도록 하는 데에도 주의를 기울여야 한다.

O32 ABI의 MIPS, 32비트 ABI의 PowerPC 및 parisc, 그리고 Xtensa에서도 비슷한 문제가 발생할 수 있다.

참고로 parisc C ABI에서도 정렬된 레지스터 쌍을 사용하지만 중간 계층을 이용해 사용자 공간에게 문제를 감춰 준다.

영향 받는 시스템 호출은 <tt>[[fadvise64_64(2)]]</tt>, <tt>[[ftruncate64(2)]]</tt>, <tt>[[posix_fadvise(2)]]</tt>, <tt>[[pread64(2)]]</tt>, <tt>[[pwrite64(2)]]</tt>, <tt>[[readahead(2)]]</tt>, <tt>[[sync_file_range(2)]]</tt>, <tt>[[truncate64(2)]]</tt>이다.

<tt>[[_llseek(2)]]</tt>, <tt>[[preadv(2)]]</tt>, <tt>[[preadv2(2)]]</tt>, <tt>[[pwritev(2)]]</tt>, <tt>[[pwritev2(2)]]</tt>처럼 직접 64비트 값을 쪼개고 합치는 시스템 호출들은 영향을 받지 않는다. 역사적 잔재들의 멋진 세계에 온 것을 환영한다.

### 아키텍처 호출 규약

아키텍처마다 커널을 호출하고 인자를 전달하는 나름의 방식이 있다. 여러 아키텍처의 세부 사항들이 아래 두 표에 나열되어 있다.

첫 번째 표는 커널로 전환하는 데 쓰는 인스트럭션 (커널로 전환하는 최단 내지 최선의 방법이 아닐 수도 있으며, 그래서 <tt>[[vdso(7)]]</tt>를 참고해야 할 수도 있음), 시스템 호출 번호를 나타내는 데 쓰는 레지스터, 커널 호출 결과를 반환하는 데 쓰는 레지스터(들), 오류를 알리는 데 쓰는 레지스터를 보여 준다.

| 아키텍처/ABI | 인스트럭션             | 시스템<br>호출 # | 반환<br>값 | 반환<br>값 2 | 오류 | 참고 |
| ------------ | ---------------------- | ----- | ----- | ----- | ----------- | ---- |
| alpha        | `callsys`              | `v0`  | `v0`  | `a4`  | `a3`        | 1, 6 |
| arc          | `trap0`                | `r8`  | `r0`  | -     | -           |      |
| arm/OABI     | `swi NR`               | -     | `r0`  | -     | -           | 2    |
| arm/EABI     | `swi 0x0`              | `r7`  | `r0`  | `r1`  | -           |      |
| arm64        | `svc #0`               | `w8`  | `x0`  | `x1`  | -           |      |
| blackfin     | `excpt 0x0`            | `P0`  | `R0`  | -     | -           |      |
| i386         | `int $0x80`            | `eax` | `eax` | `edx` | -           |      |
| ia64         | `break 0x100000`       | `r15` | `r8`  | `r9`  | `r10`       | 1, 6 |
| m68k         | `trap #0`              | `d0`  | `d0`  | -     | -           |      |
| microblaze   | `brki r14,8`           | `r12` | `r3`  | -     | -           |      |
| mips         | `syscall`              | `v0`  | `v0`  | `v1`  | `a3`        | 1, 6 |
| nios2        | `trap`                 | `r2`  | `r2`  | -     | `r7`        |      |
| parisc       | `ble 0x100(%sr2, %r0)` | `r20` | `r28` | -     | -           |      |
| powerpc      | `sc`                   | `r0`  | `r3`  | -     | `r0`        | 1    |
| powerpc64    | `sc`                   | `r0`  | `r3`  | -     | `cr0.SO`    | 1    |
| riscv        | `ecall`                | `a7`  | `a0`  | `a1`  | -           |      |
| s390         | `svc 0`                | `r1`  | `r2`  | `r3`  | -           | 3    |
| s390x        | `svc 0`                | `r1`  | `r2`  | `r3`  | -           | 3    |
| superh       | `trapa #31`            | `r3`  | `r0`  | `r1`  | -           | 4, 6 |
| sparc/32     | `t 0x10`               | `g1`  | `o0`  | `o1`  | `psr`/`csr` | 1, 6 |
| sparc/64     | `t 0x6d`               | `g1`  | `o0`  | `o1`  | `psr`/`csr` | 1, 6 |
| tile         | `swint1`               | `R10` | `R00` | -     | `R01`       | 1    |
| x86-64       | `syscall`              | `rax` | `rax` | `rdx` | -           | 5    |
| x32          | `syscall`              | `rax` | `rax` | `rdx` | -           | 5    |
| xtensa       | `syscall`              | `a2`  | `a2`  | -     | -           |      |

참고:

[1] 몇몇 아키텍처에서는 레지스터 하나를 불리언으로 사용해 (0은 오류 없음, -1은 오류 나타냄) 시스템 호출이 실패했는지 알린다. 실제 오류 값은 마찬가지로 반환 레지스터에 담는다. sparc에서는 레지스터 전체 대신 프로세서 상태 레지스터(`psr`)의 캐리 비트(`csr`)를 사용한다. powerpc64에서는 조건 레지스터(`cr0`) 0번 필드의 summary overflow 비트(`SO`)를 사용한다.

[2] `NR`이 시스템 호출 번호이다.

[3] s390과 s390x에서는 `NR`(시스템 호출 번호)가 256보다 작으면 `svc NR`로 직접 전달할 수도 있다.

[4] SuperH에서는 역사적 이유로 다른 트랩 번호들을 추가로 지원하지만 권장하는 "통합" ABI는 `trapa`#31이다.

[5] x32 ABI는 x86-64 ABI와 시스템 호출 테이블을 공유하되 몇 가지 미묘한 차이가 있다.

* x32 ABI 하에서 시스템 호출이 이뤄지는 걸 나타내기 위해서 시스템 호출 번호에 추가로 `__X32_SYSCALL_BIT` 비트를 OR 한다. 프로세스에서 사용하는 ABI가 시그널 처리나 시스템 호출 재시작을 포함한 일부 프로세스 동작 방식에 영향을 준다.

* x32에서 `long` 및 포인터 타입의 크기가 다르기 때문에 일부 구조체의 (전부는 아니다. 예를 들어 `struct timeval`이나 `struct rlimit`은 64비트를 쓴다.) 배치가 다르다. 이를 다루기 위해 시스템 호출 테이블에 번호가 (`__X32_SYSCALL_BIT`는 제외하고) 512부터 시작하는 시스템 호출들이 추가돼 있다. 예를 들어 `__NR_readv`가 x86-64 ABI에는 19로 정의돼 있고 x32 ABI에는 `__X32_SYSCALL_BIT | 515`로 정의돼 있다. 이런 추가 시스템 호출 대부분은 사실 i386 호환성 제공을 위한 시스템 호출들과 동일하다. 하지만 <tt>[[preadv2(2)]]</tt>처럼 눈에 띄는 예외도 있는데, 4바이트 포인터 및 크기를 쓰는 `struct iovec` 항목(커널 용어로는 "compat_iovec")을 사용하면서도 여타 ABI에서처럼 8바이트 `pos` 인자를 둘이 아닌 한 레지스터로 전달한다.

[6] 일부 아키텍처(Alpha, IA-64, MIPS, SuperH, sparc/32, sparc/64)에서는 추가 레지스터(위 표의 "반환 값 2")를 사용해서 <tt>[[pipe(2)]]</tt> 시스템 호출의 두 번째 반환 값을 되돌려 준다. Alpha에선 아키텍처 한정 시스템 호출인 `getxpid(2)`, `getxuid(2)`, `getxgid(2)`에서도 이 기법을 쓴다. 다른 아키텍처들에선 System V ABI에 정의돼 있는 경우조차도 시스템 호출 인터페이스에서 두 번째 반환 값 레지스터를 쓰지 않는다.

두 번째 표는 시스템 호출 인자들을 전달하는 데 쓰는 레지스터들을 보여 준다.

| 아키텍처/ABI | arg1   | arg2   | arg3   | arg4   | arg5   | arg6   | arg7   | 참고 |
| ------------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ---- |
| alpha        | `a0`   | `a1`   | `a2`   | `a3`   | `a4`   | `a5`   | -      |      |
| arc          | `r0`   | `r1`   | `r2`   | `r3`   | `r4`   | `r5`   | -      |      |
| arm/OABI     | `r0`   | `r1`   | `r2`   | `r3`   | `r4`   | `r5`   | `r6`   |      |
| arm/EABI     | `r0`   | `r1`   | `r2`   | `r3`   | `r4`   | `r5`   | `r6`   |      |
| arm64        | `x0`   | `x1`   | `x2`   | `x3`   | `x4`   | `x5`   | -      |      |
| blackfin     | `R0`   | `R1`   | `R2`   | `R3`   | `R4`   | `R5`   | -      |      |
| i386         | `ebx`  | `ecx`  | `edx`  | `esi`  | `edi`  | `ebp`  | -      |      |
| ia64         | `out0` | `out1` | `out2` | `out3` | `out4` | `out5` | -      |      |
| m68k         | `d1`   | `d2`   | `d3`   | `d4`   | `d5`   | `a0`   | -      |      |
| microblaze   | `r5`   | `r6`   | `r7`   | `r8`   | `r9`   | `r10`  | -      |      |
| mips/o32     | `a0`   | `a1`   | `a2`   | `a3`   | -      | -      | -      | 1    |
| mips/n32,64  | `a0`   | `a1`   | `a2`   | `a3`   | `a4`   | `a5`   | -      |      |
| nios2        | `r4`   | `r5`   | `r6`   | `r7`   | `r8`   | `r9`   | -      |      |
| parisc       | `r26`  | `r25`  | `r24`  | `r23`  | `r22`  | `r21`  | -      |      |
| powerpc      | `r3`   | `r4`   | `r5`   | `r6`   | `r7`   | `r8`   | `r9`   |      |
| powerpc64    | `r3`   | `r4`   | `r5`   | `r6`   | `r7`   | `r8`   | -      |      |
| riscv        | `a0`   | `a1`   | `a2`   | `a3`   | `a4`   | `a5`   | -      |      |
| s390         | `r2`   | `r3`   | `r4`   | `r5`   | `r6`   | `r7`   | -      |      |
| s390x        | `r2`   | `r3`   | `r4`   | `r5`   | `r6`   | `r7`   | -      |      |
| superh       | `r4`   | `r5`   | `r6`   | `r7`   | `r0`   | `r1`   | `r2`   |      |
| sparc/32     | `o0`   | `o1`   | `o2`   | `o3`   | `o4`   | `o5`   | -      |      |
| sparc/64     | `o0`   | `o1`   | `o2`   | `o3`   | `o4`   | `o5`   | -      |      |
| tile         | `R00`  | `R01`  | `R02`  | `R03`  | `R04`  | `R05`  | -      |      |
| x86-64       | `rdi`  | `rsi`  | `rdx`  | `r10`  | `r8`   | `r9`   | -      |      |
| x32          | `rdi`  | `rsi`  | `rdx`  | `r10`  | `r8`   | `r9`   | -      |      |
| xtensa       | `a6`   | `a3`   | `a4`   | `a5`   | `a8`   | `a9`   | -      |      |

참고:

[1] mips/o32 시스템 호출 규약에서는 5번에서 8번까지 인자들을 사용자 스택으로 전달한다.

참고로 이 표들이 호출 규약 전체를 포괄하는 것은 아니다. 일부 아키텍처에서 여기 나열 안 된 다른 레지스터들을 마구 건드릴 수도 있다.

## EXAMPLES

```c
#define _GNU_SOURCE
#include <unistd.h>
#include <sys/syscall.h>
#include <sys/types.h>
#include <signal.h>

int
main(int argc, char *argv[])
{
    pid_t tid;

    tid = syscall(SYS_gettid);
    syscall(SYS_tgkill, getpid(), tid, SIGHUP);
}
```

## SEE ALSO

<tt>[[_syscall(2)]]</tt>, `intro(2)`, <tt>[[syscalls(2)]]</tt>, <tt>[[errno(3)]]</tt>, <tt>[[vdso(7)]]</tt>

----

2021-03-22
