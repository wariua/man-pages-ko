## NAME

getauxval - 보조 벡터의 값 얻기

## SYNOPSIS

```c
#include <sys/auxv.h>

unsigned long getauxval(unsigned long type);
```

## DESCRIPTION

`getauxval()` 함수는 보조 벡터(auxiliary vector)에서 값을 얻어 온다. 보조 벡터는 커널의 ELF 바이너리 로더에서 프로그램을 실행할 때 특정 정보들을 사용자 공간으로 전달하는 메커니즘이다.

보조 벡터의 각 항목은 두 값으로 이뤄져 있는데, 그 항목이 뭔지 나타내는 타입과 그 값이다. `getauxval()`은 인자 `type`을 받아서 대응하는 값을 반환한다.

각 `type`별로 반환되는 값이 아래 목록에 나와 있다. 모든 아키텍처에 모든 `type` 값들이 있는 건 아니다.

`AT_BASE`
:   프로그램 인터프리터(일반적으로 동적 링커)의 기준 주소.

`AT_BASE_PLATFORM`
:   문자열에 대한 포인터. (PowerPC 및 MIPS 한정.) PowerPC에서는 실제 플랫폼을 나타내며 `AT_PLATFORM`과 다를 수도 있다. MIPS에서는 (리눅스 5.7부터) ISA 레벨을 나타낸다.

`AT_CLKTCK`
:   <tt>[[times(2)]]</tt>의 카운트 빈도. `sysconf(_SC_CLK_TCK)`로도 이 값을 얻을 수 있다.

`AT_DCACHEBSIZE`
:   데이터 캐시 블록 크기.

`AT_EGID`
:   스레드의 실효 그룹 ID.

`AT_ENTRY`
:   실행 파일의 진입점 주소.

`AT_EUID`
:   스레드의 실효 사용자 ID.

`AT_EXECFD`
:   프로그램의 파일 디스크립터.

`AT_EXECFN`
:   프로그램 실행에 쓰인 경로명을 담은 문자열에 대한 포인터.

`AT_FLAGS`
:   플래그 (사용 안 함).

`AT_FPUCW`
:   사용한 FPU 컨트롤 워드 (SuperH 아키텍처 한정). 커널에서 수행한 FPU 초기화에 대한 일부 정보를 준다.

`AT_GID`
:   스레드의 실제 그룹 ID.

`AT_HWCAP`
:   아키텍처 및 ABI에 따라 달라지는 비트 마스크이며 세부적인 프로세서 능력들을 나타낸다. 비트 마스크의 내용물은 하드웨어에 따라 다르다. (예를 들어 인텔 x86 아키텍처에 관한 세부 사항은 커널 소스 파일의 `arch/x86/include/asm/cpufeature.h` 참고. 거기 기술된 배열의 첫 번째 32비트 워드가 반환되는 값이다.) 사람이 읽을 수 있는 형태의 같은 정보를 `/proc/cpuinfo`에서 볼 수 있다.

`AT_HWCAP2` (glibc 2.18부터)
:   프로세서 기능에 대한 더 많은 머신별 힌트.

`AT_ICACHEBSIZE`
:   인스트럭션 캐시 블록 크기.

`AT_L1D_CACHEGEOMETRY`
:   L1 데이터 캐시 구조. 하위 16비트에 바이트 단위의 캐시 라인 크기가 들어가고 다음 16비트에 캐시 연관도가 들어간다. 그 16비트 연관도 값이 N이면 N-way 집합 연관인 캐시다.

`AT_L1D_CACHESIZE`
:   L1 데이터 캐시 크기.

`AT_L1I_CACHEGEOMETRY`
:   L1 인스트럭션 캐시 구조. `AT_L1D_CACHEGEOMETRY`처럼 인코딩 되어 있음.

`AT_L1I_CACHESIZE`
:   L1 인스트럭션 캐시 크기.

`AT_L2_CACHEGEOMETRY`
:   L2 캐시 구조. `AT_L1D_CACHEGEOMETRY`처럼 인코딩 되어 있음.

`AT_L2_CACHESIZE`
:   L2 캐시 크기.

`AT_L3_CACHEGEOMETRY`
:   L3 캐시 구조. `AT_L1D_CACHEGEOMETRY`처럼 인코딩 되어 있음.

`AT_L3_CACHESIZE`
:   L3 캐시 크기.

`AT_PAGESZ`
:   시스템 페이지 크기. (`sysconf(_SC_PAGESIZE)`가 반환하는 값과 동일.)

`AT_PHDR`
:   실행 파일의 프로그램 헤더들의 주소.

`AT_PHENT`
:   프로그램 헤더 항목 크기.

`AT_PHNUM`
:   프로그램 헤더 개수.

`AT_PLATFORM`
:   프로그램이 돌고 있는 하드웨어 플랫폼을 나타내는 문자열에 대한 포인터. 동적 링커에서 `rpath` 값 해석 시 이 값을 사용한다.

`AT_RANDOM`
:   난수 값을 담은 열여섯 바이트의 주소.

`AT_SECURE`
:   이 실행 파일이 안전하게 다뤄지고 있으면 0 아닌 값을 가진다. 0 아닌 값이 보통은 프로세스가 set-user-ID 내지 set-group-ID 바이너리를 실행하고 있음을 (그래서 실제 UID 내지 GID가 실효 UID 내지 GID와 다름을), 또는 역능을 가진 바이너리 파일을 실행해서 역능(<tt>[[capabilities(7)]]</tt> 참고)을 얻었음을 나타낸다. 또는 리눅스 보안 모듈에 의한 것일 수도 있다. 이 값이 0이 아닐 때 동적 링커에서는 특정 환경 변수들의 사용을 비활성화하며 (<tt>[[ld-linux.so(8)]]</tt> 참고) glibc에서도 여타 동작 방식들을 바꾼다. (<tt>[[secure_getenv(3)]]</tt> 참고.)

`AT_SYSINFO`
:   vDSO 내의 시스템 호출 함수 진입점. 모든 아키텍처에 있는/필요한 것은 아님 (가령 x86-64에는 없음).

`AT_SYSINFO_EHDR`
:   커널에서 특정 시스템 호출들의 빠른 구현을 제공하기 위해 생성하는 가상 동적 공유 오브젝트(vDSO)를 담은 페이지의 주소.

`AT_UCACHEBSIZE`
:   통합 캐시 블록 크기.

`AT_UID`
:   스레드의 실재 사용자 ID.

## RETURN VALUE

성공 시 `getauxval()`은 `type`에 대응하는 값을 반환한다. `type`을 찾을 수 없으면 0을 반환한다.

## ERRORS

`ENOENT` (glibc 2.19부터)
:   보조 벡터에서 `type`에 대응하는 항목을 찾을 수 없다.

## VERSIONS

glibc 버전 2.16에서 `getauxval()` 함수가 추가되었다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `getauxval()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

이 함수는 비표준 glibc 확장이다.

## NOTES

보조 벡터 내 정보의 주된 소비자는 동적 링커인 <tt>[[ld-linux.so(8)]]</tt>이다. 보조 벡터는 동적 링커에서 일반적으로나 상시적으로 필요한 일군의 표준 정보들을 커널이 전달해 줄 수 있는 편리하고 효율적인 방법이다. 일부 경우 시스템 호출로 같은 정보를 얻을 수도 있지만 보조 벡터를 쓰는 게 비용이 작다.

보조 벡터는 프로세스 주소 공간에서 인자 목록 및 환경 바로 위에 위치해 있다. 프로그램 실행 시 환경 변수 `LD_SHOW_AUXV`를 설정하면 프로그램에 제공된 보조 벡터를 볼 수 있다.

```text
$ LD_SHOW_AUXV=1 sleep 1
```

`/proc/[pid]/auxv`를 통해 임의 프로세스의 보조 벡터를 (파일 권한에 따라) 얻을 수 있다. 자세한 내용은 <tt>[[proc(5)]]</tt> 참고.

## BUGS

glibc 2.19에서 `ENOENT` 오류가 추가되기 전에는 `type`을 찾을 수 없는 경우와 `type`에 대응하는 값이 0인 경우를 명확하게 구별할 방법이 없었다.

## SEE ALSO

<tt>[[secure_getenv(3)]]</tt>, <tt>[[vdso(7)]]</tt>, <tt>[[ld-linux.so(8)]]</tt>

----

2021-03-22
