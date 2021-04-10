## NAME

kexec_load, kexec_file_load - 후에 실행할 새 커널 적재하기

## SYNOPSIS

```c
#include <linux/kexec.h>

long kexec_load(unsigned long entry, unsigned long nr_segments,
                struct kexec_segment *segments, unsigned long flags);

long kexec_file_load(int kernel_fd, int initrd_fd,
                     unsigned long cmdline_len, const char *cmdline,
                     unsigned long flags);
```

*주의*: 이 시스템 호출에 대한 glibc 래퍼가 없다. NOTES 참고.

## DESCRIPTION

`kexec_load()` 시스템 호출은 후에 <tt>[[reboot(2)]]</tt>로 실행할 수 있는 새 커널을 적재한다.

`flags` 인자는 호출의 동작을 제어하는 비트 마스크이다. `flags`에 다음 값들을 지정할 수 있다.

`KEXEC_ON_CRASH` (리눅스 2.6.13부터)
:   시스템 크래시 발생 시 새 커널을 자동으로 실행한다. 이 "크래시 커널"을 부팅 때 커널 명령행 매개변수 `crashkernel`로 정한 예약된 메모리 영역으로 적재한다. 그 예약 메모리의 위치는 `/proc/iomem` 파일의 "Crash kernel"이라는 항목을 통해 사용자 공간으로 노출된다. 사용자 공간 응용에서 그 파일을 파싱 해서 예약된 메모리를 목적지로 하는 세그먼트 목록(아래 참고)을 준비할 수 있다. 이 플래그를 지정하면 커널에서는 `segments`에 지정한 대상 세그먼트들이 그 예약 영역 안에 있는지 확인한다.

`KEXEC_PRESERVE_CONTEXT` (리눅스 2.6.27부터)
:   새 커널을 실행하기 전에 시스템 하드웨어 및 소프트웨어 상태를 보존한다. 시스템 일시 정지에 쓸 수 있을 것이다. 커널이 `CONFIG_KEXEC_JUMP`로 구성된 경우에만 이 플래그를 쓸 수 있으며, `nr_segments`가 0보다 큰 경우에만 효력이 있다.

`flags`의 (마스크 0xffff0000에 해당하는) 상위 비트들은 실행할 커널의 아키텍처를 담는다. 현재 아키텍처를 쓰려면 상수 `KEXEC_ARCH_DEFAULT`를, 아니면 아키텍처 상수 `KEXEC_ARCH_386`, `KEXEC_ARCH_68K`, `KEXEC_ARCH_X86_64`, `KEXEC_ARCH_PPC`, `KEXEC_ARCH_PPC64`, `KEXEC_ARCH_IA_64`, `KEXEC_ARCH_ARM`, `KEXEC_ARCH_S390`, `KEXEC_ARCH_SH`, `KEXEC_ARCH_MIPS`, `KEXEC_ARCH_MIPS_LE` 중 하나를 지정(OR)하면 된다. 시스템의 CPU 상에서 실행 가능한 아키텍처여야 한다.

`entry` 인자는 커널 이미지 내 진입점의 물리 주소이다. `nr_segments` 인자는 `segments` 포인터가 가리키는 세그먼트들의 수이다. 커널에서 세그먼트 개수를 (임의적으로) 16개로 제한한다. `segments` 인자는 커널 배치를 규정하는 `kexec_segment` 구조체의 배열이다.

```c
struct kexec_segment {
    void   *buf;        /* 사용자 공간의 버퍼 */
    size_t  bufsz;      /* 사용자 공간의 버퍼 길이 */
    void   *mem;        /* 커널의 물리 주소 */
    size_t  memsz;      /* 물리 주소 길이 */
};
```

`segments`로 지정한 커널 이미지는 호출 프로세스로부터 커널의 정규 메모리나 (`KEXEC_ON_CRASH`가 설정돼 있으면) 예약 메모리로 복사된다. 커널에서는 먼저 `segments`로 받은 정보에 대해 다양한 검사를 수행한다. 그 검사들을 통과하면 세그먼트 데이터를 커널 메모리로 복사한다. `segments`에 지정한 각 세그먼트를 다음과 같이 복사한다.

* `buf` 및 `bufsz`는 복사 출발지이며 호출자 가상 주소 공간 내 메모리 영역을 나타낸다. `bufsz`의 값이 `memsz` 필드의 값을 초과할 수 없다.

* `mem` 및 `memsz`는 복사 목적지이며 물리 주소 범위를 나타낸다. 두 필드 모두 값이 시스템 페이지 크기의 배수여야 한다.

* 출발 버퍼에서 대상 커널 버퍼로 `bufsz` 바이트를 복사한다. `bufsz`가 `memsz`보다 작은 경우에는 커널 버퍼의 남는 바이트들을 0으로 채운다.

일반 kexec에서는 (즉 `KEXEC_ON_CRASH` 플래그가 설정돼 있지 않을 때는) 사용 가능한 적당한 메모리로 세그먼트 데이터를 적재했다가 kexec 재부팅 시점에 (즉 `kexec(8)` 명령을 `-e` 옵션으로 실행할 때) 최종 목적지로 옮긴다.

패닉 kexec에서는 (즉 `KEXEC_ON_CRASH` 플래그가 설정돼 있을 때는) 호출 시점에 예약 메모리로 세그먼트 데이터를 적재하며 크래시 후에 kexec 메커니즘에서 그대로 그 커널로 제어를 넘긴다.

커널이 `CONFIG_KEXEC`로 구성된 경우에만 `kexec_load()` 시스템 호출이 사용 가능하다.

### `kexec_file_load()`

`kexec_file_load()` 시스템 호출은 `kexec_load()`와 비슷하되 받는 인자들이 다르다. 파일 디스크립터 `kernel_fd`가 가리키는 파일에서 적재할 커널을 읽어 들이고 파일 디스크립터 `initrd_fd`가 가리키는 파일에서 적재할 initrd(최초 램 디스크)를 읽어 들인다. `cmdline` 인자는 새 커널을 위한 명령행을 담은 버퍼에 대한 포인터이다. `cmdline_len` 인자는 그 버퍼의 크기를 나타낸다. 버퍼의 마지막 바이트가 널 바이트('\0')여야 한다.

`flags` 인자는 호출의 동작 방식을 변경하는 비트 마스크이다. `flags`에 다음 값들을 지정할 수 있다.

`KEXEC_FILE_UNLOAD`
:   현재 적재된 커널을 내린다.

`KEXEC_FILE_ON_CRASH`
:   (`KEXEC_ON_CRASH`처럼) 크래시 커널을 위해 예약된 메모리 영역에 새 커널을 적재한다. 현재 돌고 있는 커널이 죽으면 이 커널로 부팅 한다.

`KEXEC_FILE_NO_INITRAMFS`
:   initrd/initramfs 적재는 선택적이다. initramfs를 적재하지 않을 경우 이 플래그를 지정하면 된다. 이 플래그가 설정돼 있으면 `initrd_fd` 값을 무시한다.

`kexec_file_load()` 시스템 호출은 서명된 커널들만 "kexec" 적재를 하도록 한정해야 하는 시스템을 위해 추가된 것이다. 커널이 `CONFIG_KEXEC_FILE`로 구성된 경우에만 이 시스템 호출이 사용 가능하다.

## RETURN VALUE

성공 시 이 시스템 호출들은 0을 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EADDRNOTAVAIL`
:   `KEXEC_ON_CRASH` 플래그를 지정했는데 `segments` 항목들 중 하나의 `mem` 및 `memsz` 필드로 지정한 영역이 크래시 커널을 위해 예약된 메모리 영역 밖에 있다.

`EADDRNOTAVAIL`
:   `segments` 항목들 중 하나에서 `mem`이나 `memsz` 필드 값이 시스템 페이지 크기의 배수가 아니다.

`EBADF`
:   `kernel_fd`나 `initrd_fd`가 유효한 파일 디스크립터가 아니다.

`EBUSY`
:   다른 크래시 커널이 이미 적재돼 있거나 이미 크래시 커널을 쓰고 있다.

`EINVAL`
:   `flags`가 유효하지 않다.

`EINVAL`
:   `segments` 항목들 중 하나에서 `bufsz` 필드의 값이 대응하는 `memsz` 필드 값을 초과한다.

`EINVAL`
:   `nr_segments`가 `KEXEC_SEGMENT_MAX`(16)를 초과한다.

`EINVAL`
:   둘 이상의 커널 대상 버퍼가 겹친다.

`EINVAL`
:   `cmdline[cmdline_len-1]`의 값이 '`\0`'이 아니다.

`EINVAL`
:   `kernel_fd`나 `initrd_fd`가 가리키는 파일이 비어 있다 (길이가 0이다).

`ENOEXEC`
:   `kernel_fd`가 열린 파일을 가리키고 있지 않거나 커널에서 그 파일을 적재할 수 없다. 현재 그 파일은 bzImage여야 하고 메모리의 4GiB 위에 적재 가능한 x86 커널을 담고 있어야 한다. (커널 소스 파일 `Documentation/x86/boot.txt` 참고.)

`ENOMEM`
:   메모리를 할당할 수 없다.

`EPERM`
:   호출자가 `CAP_SYS_BOOT` 역능을 가지고 있지 않다.

## VERSIONS

리눅스 2.6.13에서 `kexec_load()` 시스템 호출이 처음 등장했다. 리눅스 3.17에서 `kexec_file_load()` 시스템 호출이 처음 등장했다.

## CONFORMING TO

이 시스템 호출들은 리눅스 전용이다.

## NOTES

현재 이 시스템 호출들에 대한 glibc 지원이 없다. <tt>[[syscall(2)]]</tt>을 이용해 호출해야 한다.

## SEE ALSO

<tt>[[reboot(2)]]</tt>, <tt>[[syscall(2)]]</tt>, `kexec(8)`

커널 소스 파일 `Documentation/kdump/kdump.txt` 및 `Documentation/admin-guide/kernel-parameters.txt`

----

2019-03-06
