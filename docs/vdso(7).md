## NAME

vdso - 가상 ELF 동적 공유 오브젝트 소개

## SYNOPSIS

```c
#include <sys/auxv.h>

void *vdso = (uintptr_t) getauxval(AT_SYSINFO_EHDR);
```

## DESCRIPTION

"vDSO" (virtual dynamic shared object: 가상 동적 공유 오브젝트)는 커널이 모든 사용자 공간 응용 프로그램의 주소 공간으로 자동으로 맵 하는 작은 공유 라이브러리이다. 보통 vDSO를 호출하는 것은 C 라이브러리이므로 응용 프로그램에서는 일반적으로 자세한 내용에 신경쓸 필요가 없다. 표준 함수들을 사용하는 정상적인 방식으로 코딩 하면 C 라이브러리에서 알아서 vDSO를 통해 사용 가능한 기능들을 이용한다.

그런데 vDSO는 도대체 왜 존재하는 것일까? 커널이 제공하는 시스템 호출 중에는 사용자 공간 코드에서 워낙 빈번하게 사용해서 그 호출들이 전체 성능에 크게 영향을 줄 수 있는 것들이 있다. 이건 호출 빈도만이 아니라 사용자 공간을 빠져나와 커널로 들어가는 것으로 인한 문맥 전환 오버헤드 때문이기도 하다.

이 문서의 나머지 부분은 일반 개발자보다는 호기심 있는 이들 및/또는 C 라이브러리 작성자에게 맞춰져 있다. 만약 C 라이브러리를 사용하는 대신에 응용에서 vDSO를 호출하려 하고 있다면 아마도 분명히 잘못하고 있는 것이다.

### 배경 예시

시스템 호출을 하는 것은 느릴 수 있다. x86 32비트 시스템에서는 소프트웨어 인터럽트(`int $0x80`)를 유발시켜서 시스템 호출을 하고 싶다고 커널에게 알려줄 수 있다. 하지만 이 인스트럭션은 비용이 크다. 프로세서의 마이크로코드와 커널 내의 인터럽트 처리 경로 전체를 거친다. 더 최신 프로세서에는 시스템 호출을 개시할 수 있는 더 빠른 (하지만 하위 호환되지 않는) 인스트럭션이 있다. 런타임에 이 기능을 사용할 수 있는지를 C 라이브러리에서 알아내기는 요구하는 대신에 커널이 vDSO로 제공하는 함수들을 C 라이브러리가 사용하게 할 수 있다.

용어가 혼란스러울 수 있으니 주의하라. x86 시스템에서 우선적인 시스템 호출 방법을 알아내는 데 사용하는 vDSO 함수의 이름이 "\__kernel_vsyscall"인데, x86-64에서는 "vsyscall"이라는 용어가 지금이 몇 시이고 호출자가 어느 CPU에서 돌고 있는지 커널에게 물을 수 있는 구식 방법을 가리키기도 한다.

자주 쓰는 시스템 호출로 <tt>[[gettimeofday(2)]]</tt>가 있다. 이 시스템 호출을 사용자 공간 응용에서 직접 부르기도 하고 C 라이브러리에서 간접적으로 부르기도 한다. 타임스탬프나 시간 제어 루프, 폴링을 생각해 보라. 이 모두에서는 바로 지금 시간을 자주 알아내야 한다. 또한 이 정보는 비밀이 아니다. 어느 특권 모드의 어떤 응용에서도 (루트도, 다른 비특권 사용자도) 같은 답을 얻게 된다. 따라서 커널은 이 질문에 답하기 위해 필요한 정보를 프로세스가 접근할 수 있는 메모리 내에 위치시킨다. 그러면 시스템 호출이었던 <tt>[[gettimeofday(2)]]</tt>가 일반 함수 호출과 몇 번의 메모리 접근으로 바뀐다.

### vDSO 찾기

(존재하는 경우) vDSO의 기준 주소는 초기 보조 벡터(<tt>[[getauxval(3)]]</tt> 참고)의 `AT_SYSINFO_EHDR` 태그를 통해 커널이 각 프로그램에게 전달해 준다.

vDSO가 사용자 메모리 맵의 특정 위치에 사상되어 있다고 가정해서는 안 된다. 일반적으로 새 프로세스 이미지가 만들어질 때마다 (<tt>[[execve(2)]]</tt> 시점에) 기준 주소가 임의로 정해진다. 이렇게 하는 건 "return-to-libc" 공격을 막기 위한 보안적 이유 때문이다.

일부 아키텍처에는 `AT_SYSINFO` 태그도 있다. 이 태그는 vsyscall 진입점의 위치를 나타내기 위한 것으로, 많은 경우 빠져 있거나 0으로 설정되어 있다. (즉, 사용 가능하지 않다.) 이 태그는 초기 vDSO 작업의 유산이며 (아래의 *역사* 참고) 사용을 피하는 게 좋다.

### 파일 형식

vDSO는 완전한 형태의 ELF 이미지이므로 거기서 심볼 검색을 할 수 있다. 그래서 최신 커널 릴리스에서 새 심볼을 추가하는 것이 가능하며 여러 커널 버전들에서 동작하는 C 라이브러리가 런타임에 사용 가능한 기능을 탐지할 수 있다. 많은 경우에 C 라이브러리는 첫 번째 호출에서 탐지를 하고서 이후 호출을 위해 그 결과를 캐싱 한다.

모든 심볼에는 또한 버전이 있다. (GNU 버전 형식을 사용한다.) 그래서 커널이 하위 호환성을 깨지 않으면서 함수 시그너처를 갱신할 수 있다. 반환 값뿐 아니라 함수가 받는 인자를 바꾸는 것도 가능하다. 따라서 vDSO에서 심볼 검색을 할 때는 항상 기대하는 ABI에 맞는 버전을 포함시켜야 한다.

보통 vDSO는 다른 표준 심볼과 구별하기 위해 모든 심볼 앞에 "\__vdso\_"나 "\__kernel\_"을 붙이는 명명 관행을 따른다. 예를 들어 "gettimeofday" 함수의 이름이 "\__vdso_gettimeofday"가 된다.

이 함수들을 호출할 때는 표준 C 호출 방식을 사용한다. 이상한 레지스터 내지 스택 동작 방식에 대해 염려할 필요가 없다.

## NOTES

### 소스

커널을 컴파일 할 때 자동으로 vDSO 코드를 컴파일 및 링크 해 준다. 많은 경우 아키텍처별 디렉터리 안에 들어 있다.

```sh
find arch/$ARCH/ -name '*vdso*.so*' -o -name '*gate*.so*'
```

### vDSO 이름

vDSO의 이름이 아키텍처마다 다르다. glibc의 `ldd(1)` 출력 같은 것에 종종 등장하게 된다. 어떤 코드에도 그 정확한 이름이 중요해서는 안 되므로 이름을 하드코딩 해서는 안 된다.

| 사용자 ABI | vDSO 이름           |
| ---------- | ------------------- |
| aarch64    | `linux-vdso.so.1`   |
| arm        | `linux-vdso.so.1`   |
| ia64       | `linux-gate.so.1`   |
| mips       | `linux-vdso.so.1`   |
| ppc/32     | `linux-vdso32.so.1` |
| ppc/64     | `linux-vdso64.so.1` |
| riscv      | `linux-vdso.so.1`   |
| s390       | `linux-vdso32.so.1` |
| s390x      | `linux-vdso64.so.1` |
| sh         | `linux-gate.so.1`   |
| i386       | `linux-gate.so.1`   |
| x86-64     | `linux-vdso.so.1`   |
| x86/x32    | `linux-vdso.so.1`   |

### <tt>[[strace(1)]]</tt> 및 <tt>[[seccomp(2)]]</tt>와 vDSO

<tt>[[strace(1)]]</tt>로 시스템 호출을 추적할 때 vDSO가 내보이는 심볼들(시스템 호출들)은 추적 출력에 나오지 *않는다*. 유사하게 그런 시스템 호출은 <tt>[[seccomp(2)]]</tt> 필터에 보이지 않는다.

## ARCHITECTURE-SPECIFIC NOTES

아래의 부절들은 vDSO에 대한 아키텍처별 내용이다.

참고로 사용하는 vDSO는 커널의 ABI가 아니라 사용자 공간 코드의 ABI를 기준으로 한다. 따라서 예를 들어 i386 32비트 ELF 바이너리를 돌린다면 이를 i386 32비트 커널에서 돌리든 x86-64 64비트 커널에서 돌리든 상관없이 같은 vDSO를 얻게 된다. 따라서 아래에서 관련 절을 찾을 때 사용자 공간 ABI의 이름을 사용해야 한다.

### ARM 함수

아래 표에 vDSO가 내보이는 심볼들이 나열되어 있다.

| 심볼                   | 버전                                |
| ---------------------- | ----------------------------------- |
| `__vdso_gettimeofday`  | `LINUX_2.6` (리눅스 4.1부터 내보임) |
| `__vdso_clock_gettime` | `LINUX_2.6` (리눅스 4.1부터 내보임) |

더불어 ARM 포트에는 유틸리티 함수들로 가득한 코드 페이지가 있다. 그냥 코드가 있는 페이지일 뿐이므로 심볼 검색이나 버전 관리를 위한 ELF 정보가 전혀 없다. 하지만 다른 식으로 버전을 지원한다.

이 코드 페이지에 대해 알려면 커널 문서를 참조하는 것이 최선이다. 아주 자세하며 알아야 할 모든 것들을 다루고 있다. `Documentation/arm/kernel_user_helpers.txt`이다.

### aarch64 함수

아래 표에 vDSO가 내보이는 심볼들이 나열되어 있다.

| 심볼                     | 버전           |
| ------------------------ | -------------- |
| `__kernel_rt_sigreturn`  | `LINUX_2.6.39` |
| `__kernel_gettimeofday`  | `LINUX_2.6.39` |
| `__kernel_clock_gettime` | `LINUX_2.6.39` |
| `__kernel_clock_getres`  | `LINUX_2.6.39` |

### bfin (Blackfin) 함수 (리눅스 4.17에서 포트 제거)

이 CPU에는 메모리 관리 유닛(MMU)이 없으므로 일반적 의미의 vDSO를 만들지 않는다. 대신 부팅 때 저수준 함수 몇 개를 메모리의 고정된 위치에 맵 한다. 그러면 사용자 공간 응용에서 그 영역을 바로 호출한다. 하위 호환성에 있어선 opcode를 훔쳐보는 것 이상의 방안이 없지만 임베디드 CPU이므로 문제가 안 된다. 심지어 실행하는 오브젝트 형식 중 일부는 ELF 기반도 아니다 (bFLT/FLAT).

이 코드 페이지에 대한 정보는 공개된 문서를 참조하는 것이 최선이다: <http://docs.blackfin.uclinux.org/doku.php?id=linux-kernel:fixed-code>

### mips 함수

아래 표에 vDSO가 내보이는 심볼들이 나열되어 있다.

| 심볼                     | 버전                                |
| ------------------------ | ----------------------------------- |
| `__kernel_gettimeofday`  | `LINUX_2.6` (리눅스 4.4부터 내보임) |
| `__kernel_clock_gettime` | `LINUX_2.6` (리눅스 4.4부터 내보임) |

### ia64 (Itanium) 함수

아래 표에 vDSO가 내보이는 심볼들이 나열되어 있다.

| 심볼                         | 버전        |
| ---------------------------- | ----------- |
| `__kernel_sigtramp`          | `LINUX_2.5` |
| `__kernel_syscall_via_break` | `LINUX_2.5` |
| `__kernel_syscall_via_epc`   | `LINUX_2.5` |

아이테니엄 포트는 좀 복잡하다. 위의 vDSO에 더해서 "경량 시스템 호출"("fast syscall" 내지 "fsys"라고도 함)이라는 것이 있으며 vDSO 헬퍼 `__kernel_syscall_via_epc`를 통해 호출할 수 있다. 여기 나열된 시스템 호출들의 동작 결과는 <tt>[[syscall(2)]]</tt>을 통해 직접 호출했을 때와 동일하므로 각각의 관련 문서를 참고하라. 아래 표에서 이 메커니즘을 통해 사용 가능한 함수들을 보여 준다.

| 함수              |
| ----------------- |
| `clock_gettime`   |
| `getcpu`          |
| `getpid`          |
| `getppid`         |
| `gettimeofday`    |
| `set_tid_address` |

### parisc (hppa) 함수

parisc 포트에는 유틸리티 함수들이 있는 게이트웨이 페이지라는 이름의 코드 페이지가 있다. 일반적인 ELF 보조 벡터 방식을 사용하는 대신 SR2 레지스터를 통해 페이지의 주소를 프로세스에게 전달한다. 페이지의 권한은 그 주소를 실행만 해도 자동으로 사용자 공간이 아닌 커널 특권으로 실행하게 되어 있다. HP-UX의 동작 방식과 일치하게 하기 위해서이다.

그냥 코드가 있는 페이지일 뿐이므로 심볼 검색이나 버전 관리를 위한 ELF 정보가 전혀 없다. 브랜치 인스트럭션을 통해 적절한 오프셋으로 호출해 들어가기만 하면 된다. 예:

```
ble <offset>(%sr2, %r0)
```

| 오프셋 | 함수                                  |
| ------ | ------------------------------------- |
| 00b0   | `lws_entry` (CAS 연산들)              |
| 00e0   | `set_thread_pointer` (glibc에서 사용) |
| 0100   | `linux_gateway_entry` (syscall)       |

### ppc/32 함수

아래 표에 vDSO가 내보이는 심볼들이 나열되어 있다. * 표시가 된 함수는 커널이 PowerPC64 (64비트) 커널일 때만 사용 가능하다.

| 심볼                       | 버전           |
| -------------------------- | -------------- |
| `__kernel_clock_getres`    | `LINUX_2.6.15` |
| `__kernel_clock_gettime`   | `LINUX_2.6.15` |
| `__kernel_datapage_offset` | `LINUX_2.6.15` |
| `__kernel_get_syscall_map` | `LINUX_2.6.15` |
| `__kernel_get_tbfreq`      | `LINUX_2.6.15` |
| `__kernel_getcpu` *        | `LINUX_2.6.15` |
| `__kernel_gettimeofday`    | `LINUX_2.6.15` |
| `__kernel_sigtramp_rt32`   | `LINUX_2.6.15` |
| `__kernel_sigtramp32`      | `LINUX_2.6.15` |
| `__kernel_sync_dicache`    | `LINUX_2.6.15` |
| `__kernel_sync_dicache_p5` | `LINUX_2.6.15` |

`CLOCK_REALTIME_COARSE` 및 `CLOCK_MONOTONIC_COARSE` 클럭은 `__kernel_clock_getres` 및 `__kernel_clock_gettime` 인터페이스에서 지원하지 *않는다*. 커널이 실제 시스템 호출로 돌린다.

### ppc/64 함수

아래 표에 vDSO가 내보이는 심볼들이 나열되어 있다.

| 심볼                       | 버전           |
| -------------------------- | -------------- |
| `__kernel_clock_getres`    | `LINUX_2.6.15` |
| `__kernel_clock_gettime`   | `LINUX_2.6.15` |
| `__kernel_datapage_offset` | `LINUX_2.6.15` |
| `__kernel_get_syscall_map` | `LINUX_2.6.15` |
| `__kernel_get_tbfreq`      | `LINUX_2.6.15` |
| `__kernel_getcpu`          | `LINUX_2.6.15` |
| `__kernel_gettimeofday`    | `LINUX_2.6.15` |
| `__kernel_sigtramp_rt64`   | `LINUX_2.6.15` |
| `__kernel_sync_dicache`    | `LINUX_2.6.15` |
| `__kernel_sync_dicache_p5` | `LINUX_2.6.15` |

`CLOCK_REALTIME_COARSE` 및 `CLOCK_MONOTONIC_COARSE` 클럭은 `__kernel_clock_getres` 및 `__kernel_clock_gettime` 인터페이스에서 지원하지 *않는다*. 커널이 실제 시스템 호출로 돌린다.

### riscv 함수

아래 표에 vDSO가 내보이는 심볼들이 나열되어 있다.

| 심볼                     | 버전         |
| ------------------------ | ------------ |
| `__kernel_rt_sigreturn`  | `LINUX_4.15` |
| `__kernel_gettimeofday`  | `LINUX_4.15` |
| `__kernel_clock_gettime` | `LINUX_4.15` |
| `__kernel_clock_getres`  | `LINUX_4.15` |
| `__kernel_getcpu`        | `LINUX_4.15` |
| `__kernel_flush_icache`  | `LINUX_4.15` |

### s390 함수

아래 표에 vDSO가 내보이는 심볼들이 나열되어 있다.

| 심볼                     | 버전           |
| ------------------------ | -------------- |
| `__kernel_clock_getres`  | `LINUX_2.6.29` |
| `__kernel_clock_gettime` | `LINUX_2.6.29` |
| `__kernel_gettimeofday`  | `LINUX_2.6.29` |

### x390x 함수

아래 표에 vDSO가 내보이는 심볼들이 나열되어 있다.

| 심볼                     | 버전           |
| ------------------------ | -------------- |
| `__kernel_clock_getres`  | `LINUX_2.6.29` |
| `__kernel_clock_gettime` | `LINUX_2.6.29` |
| `__kernel_gettimeofday`  | `LINUX_2.6.29` |

### sh (SuperH) 함수

아래 표에 vDSO가 내보이는 심볼들이 나열되어 있다.

| 심볼                    | 버전        |
| ----------------------- | ----------- |
| `__kernel_rt_sigreturn` | `LINUX_2.6` |
| `__kernel_sigreturn`    | `LINUX_2.6` |
| `__kernel_vsyscall`     | `LINUX_2.6` |

### i386 함수

아래 표에 vDSO가 내보이는 심볼들이 나열되어 있다.

| 심볼                    | 버전                                 |
| ----------------------- | ------------------------------------ |
| `__kernel_sigreturn`    | `LINUX_2.5`                          |
| `__kernel_rt_sigreturn` | `LINUX_2.5`                          |
| `__kernel_vsyscall`     | `LINUX_2.5`                          |
| `__vdso_clock_gettime`  | `LINUX_2.6` (리눅스 3.15부터 내보임) |
| `__vdso_gettimeofday`   | `LINUX_2.6` (리눅스 3.15부터 내보임) |
| `__vdso_time`           | `LINUX_2.6` (리눅스 3.15부터 내보임) |

### x86-64 함수

아래 표에 vDSO가 내보이는 심볼들이 나열되어 있다. 이 심볼들 모두 "\__vdso\_" 접두부가 없는 형태로도 있지만 그건 무시하고 아래 나열된 이름만 쓰는 게 좋다.

| 심볼                   | 버전        |
| ---------------------- | ----------- |
| `__vdso_clock_gettime` | `LINUX_2.6` |
| `__vdso_getcpu`        | `LINUX_2.6` |
| `__vdso_gettimeofday`  | `LINUX_2.6` |
| `__vdso_time`          | `LINUX_2.6` |

### x86/x32 함수

아래 표에 vDSO가 내보이는 심볼들이 나열되어 있다.

| 심볼                   | 버전        |
| ---------------------- | ----------- |
| `__vdso_clock_gettime` | `LINUX_2.6` |
| `__vdso_getcpu`        | `LINUX_2.6` |
| `__vdso_gettimeofday`  | `LINUX_2.6` |
| `__vdso_time`          | `LINUX_2.6` |

### 역사

vDSO는 원래 함수 하나, 즉 vsyscall이었다. 이전 커널들에서는 "vdso"가 아니라 프로세스의 메모리 맵에서 그 이름을 볼 수도 있었다. 시간이 지나면서 이 메커니즘이 사용자 공간으로 더 많은 기능을 전달할 멋진 방법이라는 걸 사람들이 알게 되었고, 그래서 현재 형식의 vDSO로 다시 생각하게 되었다.

## SEE ALSO

<tt>[[syscalls(2)]]</tt>, <tt>[[getauxval(3)]]</tt>, <tt>[[proc(5)]]</tt>

리눅스 소스 코드 트리의 문서, 예시, 소스 코드:
* `Documentation/ABI/stable/vdso`
* `Documentation/ia64/fsys.txt`
* `Documentation/vDSO/*` (vDSO 사용 예시 포함)

```sh
find arch/ -iname '*vdso*' -o -iname '*gate*'
```

----

2019-08-02
