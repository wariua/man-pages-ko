## NAME

dl_iterate_phdr - 공유 오브젝트 리스트 순회하기

## SYNOPSIS

```c
#define _GNU_SOURCE         /* feature_test_macros(7) 참고 */
#include <link.h>

int dl_iterate_phdr(
          int (*callback) (struct dl_phdr_info *info,
                           size_t size, void *data),
          void *data);
```

## DESCRIPTION

`dl_iterate_phdr()` 함수를 이용해 응용에서 어떤 공유 오브젝트들을 적재했고 어떤 순서로 적재됐는지 런타임에 질의할 수 있다.

`dl_iterate_phdr()` 함수는 응용의 공유 오브젝트 목록을 순회하면서 각 오브젝트마다 함수 `callback`을 호출한다. 모든 공유 오브젝트를 처리했으면, 또는 `callback`이 0 아닌 값을 반환하면 멈춘다.

`callback` 호출 각각은 세 개 인자를 받는다. `info`는 공유 오브젝트에 대한 정보를 담은 구조체에 대한 포인터이고 `size`는 `info`가 가리키는 구조체의 크기이다. `data`는 호출 프로그램에서 `dl_iterate_phdr()` 호출 때 (역시 이름이 `data`인) 두 번째 인자로 준 값의 사본이다.

`info` 인자는 다음 타입의 구조체이다.

```c
struct dl_phdr_info {
    ElfW(Addr)        dlpi_addr;  /* 오브젝트의 기준 주소 */
    const char       *dlpi_name;  /* 오브젝트의 (널 종료) 이름 */
    const ElfW(Phdr) *dlpi_phdr;  /* 이 오브젝트의 ELF 프로그램
                                     헤더 배열에 대한 포인터 */
    ElfW(Half)        dlpi_phnum; /* dlpi_phdr의 항목 개수 */

    /* 다음 필드들은 이 구조체의 첫 번째 버전이 공개된 후에
       glibc 2.4에서 추가된 것들이다. dl_iterate_phdr 콜백에
       전달되는 size 인자를 사용해서 이후 멤버 각각이 사용
       가능한지 여부를 확인해야 한다. */

    unsigned long long int dlpi_adds;
                    /* 새 오브젝트가 추가됐을 수도
		       있을 때 증가됨 */
    unsigned long long int dlpi_subs;
                    /* 오브젝트가 제거됐을 수도
		       있을 때 증가됨 */
    size_t dlpi_tls_modid;
                    /* PT_TLS 세그먼트가 있는 경우는 TLS 재배치에
		       쓰인 그 모듈 ID, 아니면 0 */
    void  *dlpi_tls_data;
                    /* 이 모듈의 PT_TLS 세그먼트 인스턴스가
		       하나 있고 호출 스레드에 할당돼 있으면
		       그 주소, 아니면 널 포인터 */
};
```

(`ElfW()` 매크로는 그 인자를 하드웨어 아키텍처에 맞는 ELF 데이터 타입 이름으로 바꿔 준다. 예를 들어 32비트 플랫폼에서 `ElfW(Addr)`은 데이터 타입 이름 `Elf32_Addr`을 내놓는다. 이런 타입들에 대한 추가 정보를 헤더 파일 `<elf.h>` 및 `<link.h>`에서 볼 수 있다.)

`dlpi_addr` 필드는 공유 오브젝트의 기준 주소를 나타낸다. (즉 공유 오브젝트의 가상 메모리 주소와 그 오브젝트를 적재한 파일 내에서의 오프셋 간의 차이이다.) `dlpi_name` 필드는 그 공유 오브젝트를 적재한 경로명을 알려 주는 널 종료 문자열이다.

`dlpi_phdr` 및 `dlpi_phnum` 필드의 의미를 이해하려면 ELF 공유 오브젝트가 여러 세그먼트로 이뤄져 있고 각각에 그 세그먼트를 기술하는 프로그램 헤더가 있다는 점을 알아야 한다. `dlpi_phdr` 필드는 이 공유 오브젝트의 프로그램 헤더들의 배열에 대한 포인터이다. `dlpi_phnum` 필드는 그 배열의 크기를 나타낸다.

그 프로그램 헤더들은 다음 형태의 구조체이다.

```c
typedef struct {
    Elf32_Word  p_type;    /* 세그먼트 타입 */
    Elf32_Off   p_offset;  /* 세그먼트 파일 오프셋 */
    Elf32_Addr  p_vaddr;   /* 세그먼트 가상 주소 */
    Elf32_Addr  p_paddr;   /* 세그먼트 물리 주소 */
    Elf32_Word  p_filesz;  /* 파일 내 세그먼트 크기 */
    Elf32_Word  p_memsz;   /* 메모리 내 세그먼트 크기 */
    Elf32_Word  p_flags;   /* 세그먼트 플래그 */
    Elf32_Word  p_align;   /* 세그먼트 정렬 */
} Elf32_Phdr;
```

참고로 다음 식을 사용해 특정 프로그램 헤더 `x`의 가상 메모리 내 위치를 계산할 수 있다.

```c
addr == info->dlpi_addr + info->dlpi_phdr[x].p_vaddr;
```

`p_type`에 가능한 값들로 다음이 있다. (더 자세한 내용은 `<elf.h>` 참고.)

```c
#define PT_LOAD         1    /* 적재 가능 프로그램 세그먼트 */
#define PT_DYNAMIC      2    /* 동적 링크 정보 */
#define PT_INTERP       3    /* 프로그램 인터프리터 */
#define PT_NOTE         4    /* 보조 정보 */
#define PT_SHLIB        5    /* 예비 */
#define PT_PHDR         6    /* 헤더 테이블 자체를 위한 항목 */
#define PT_TLS          7    /* 스레드 로컬 저장 공간 세그먼트 */
#define PT_GNU_EH_FRAME 0x6474e550 /* GCC .eh_frame_hdr segment */
#define PT_GNU_STACK  0x6474e551 /* 스택 실행 가능 여부 표시 */
#define PT_GNU_RELRO  0x6474e552 /* 재배치 후에는 읽기 전용 */
```

## RETURN VALUE

`dl_iterate_phdr()` 함수는 마지막 `callback` 호출이 반환한 값을 그대로 반환한다.

## VERSIONS

glibc 버전 2.2.4부터 `dl_iterate_phdr()`를 지원한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `dl_iterate_phdr()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

`dl_iterate_phdr()` 함수는 어느 표준에도 명세돼 있지 않다. 다른 여러 시스템에서도 이 함수를 제공하지만 반환되는 `dl_phdr_info` 구조체의 세부 내용이 다르다. BSD와 솔라리스에서는 구조체에 `dlpi_addr`, `dlpi_name`, `dlpi_phdr`, `dlpi_phnum` 필드에 더해서 다른 구현별 필드들이 포함돼 있다.

## NOTES

C 라이브러리의 이후 버전에서 `dl_phdr_info` 구조체에 필드를 더 추가할 수도 있다. 그 경우 콜백 함수에서는 `size` 인자를 통해 추가 필드가 있는 시스템에서 동작 중인지 알아낼 수 있다.

`callback`이 방문하는 첫 번째 오브젝트는 메인 프로그램이다. 메인 프로그램에서 `dlpi_name` 필드는 빈 문자열이 된다.

## EXAMPLE

다음 프로그램에서는 적재한 공유 오브젝트들의 경로명 목록을 표시한다. 그리고 각 공유 오브젝트에 대해서 오브젝트의 각 ELF 세그먼트의 정보(가상 주소, 크기, 플래그, 타입)를 나열한다.

다음 셸 세션은 x86-64 시스템에서 프로그램이 내놓는 출력을 보여 준다. 출력 첫 번째의 (이름이 빈 문자열인) 공유 오브젝트는 메인 프로그램이다.

```
$ ./a.out
Name: "" (9 segments)
     0: [      0x400040; memsz:    1f8] flags: 0x5; PT_PHDR
     1: [      0x400238; memsz:     1c] flags: 0x4; PT_INTERP
     2: [      0x400000; memsz:    ac4] flags: 0x5; PT_LOAD
     3: [      0x600e10; memsz:    240] flags: 0x6; PT_LOAD
     4: [      0x600e28; memsz:    1d0] flags: 0x6; PT_DYNAMIC
     5: [      0x400254; memsz:     44] flags: 0x4; PT_NOTE
     6: [      0x400970; memsz:     3c] flags: 0x4; PT_GNU_EH_FRAME
     7: [         (nil); memsz:      0] flags: 0x6; PT_GNU_STACK
     8: [      0x600e10; memsz:    1f0] flags: 0x4; PT_GNU_RELRO
Name: "linux-vdso.so.1" (4 segments)
     0: [0x7ffc6edd1000; memsz:    e89] flags: 0x5; PT_LOAD
     1: [0x7ffc6edd1360; memsz:    110] flags: 0x4; PT_DYNAMIC
     2: [0x7ffc6edd17b0; memsz:     3c] flags: 0x4; PT_NOTE
     3: [0x7ffc6edd17ec; memsz:     3c] flags: 0x4; PT_GNU_EH_FRAME
Name: "/lib64/libc.so.6" (10 segments)
     0: [0x7f55712ce040; memsz:    230] flags: 0x5; PT_PHDR
     1: [0x7f557145b980; memsz:     1c] flags: 0x4; PT_INTERP
     2: [0x7f55712ce000; memsz: 1b6a5c] flags: 0x5; PT_LOAD
     3: [0x7f55716857a0; memsz:   9240] flags: 0x6; PT_LOAD
     4: [0x7f5571688b80; memsz:    1f0] flags: 0x6; PT_DYNAMIC
     5: [0x7f55712ce270; memsz:     44] flags: 0x4; PT_NOTE
     6: [0x7f55716857a0; memsz:     78] flags: 0x4; PT_TLS
     7: [0x7f557145b99c; memsz:   544c] flags: 0x4; PT_GNU_EH_FRAME
     8: [0x7f55712ce000; memsz:      0] flags: 0x6; PT_GNU_STACK
     9: [0x7f55716857a0; memsz:   3860] flags: 0x4; PT_GNU_RELRO
Name: "/lib64/ld-linux-x86-64.so.2" (7 segments)
     0: [0x7f557168f000; memsz:  20828] flags: 0x5; PT_LOAD
     1: [0x7f55718afba0; memsz:   15a8] flags: 0x6; PT_LOAD
     2: [0x7f55718afe10; memsz:    190] flags: 0x6; PT_DYNAMIC
     3: [0x7f557168f1c8; memsz:     24] flags: 0x4; PT_NOTE
     4: [0x7f55716acec4; memsz:    604] flags: 0x4; PT_GNU_EH_FRAME
     5: [0x7f557168f000; memsz:      0] flags: 0x6; PT_GNU_STACK
     6: [0x7f55718afba0; memsz:    460] flags: 0x4; PT_GNU_RELRO
```

### 프로그램 소스

```c
#define _GNU_SOURCE
#include <link.h>
#include <stdlib.h>
#include <stdio.h>

static int
callback(struct dl_phdr_info *info, size_t size, void *data)
{
    char *type;
    int p_type, j;

    printf("Name: \"%s\" (%d segments)\n", info->dlpi_name,
               info->dlpi_phnum);

    for (j = 0; j < info->dlpi_phnum; j++) {
        p_type = info->dlpi_phdr[j].p_type;
        type =  (p_type == PT_LOAD) ? "PT_LOAD" :
                (p_type == PT_DYNAMIC) ? "PT_DYNAMIC" :
                (p_type == PT_INTERP) ? "PT_INTERP" :
                (p_type == PT_NOTE) ? "PT_NOTE" :
                (p_type == PT_INTERP) ? "PT_INTERP" :
                (p_type == PT_PHDR) ? "PT_PHDR" :
                (p_type == PT_TLS) ? "PT_TLS" :
                (p_type == PT_GNU_EH_FRAME) ? "PT_GNU_EH_FRAME" :
                (p_type == PT_GNU_STACK) ? "PT_GNU_STACK" :
                (p_type == PT_GNU_RELRO) ? "PT_GNU_RELRO" : NULL;

        printf("    %2d: [%14p; memsz:%7lx] flags: 0x%x; ", j,
                (void *) (info->dlpi_addr + info->dlpi_phdr[j].p_vaddr),
                info->dlpi_phdr[j].p_memsz,
                info->dlpi_phdr[j].p_flags);
        if (type != NULL)
            printf("%s\n", type);
        else
            printf("[other (0x%x)]\n", p_type);
    }

    return 0;
}

int
main(int argc, char *argv[])
{
    dl_iterate_phdr(callback, NULL);

    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

`ldd(1)`, <tt>[[objdump(1)]]</tt>, <tt>[[readelf(1)]]</tt>, <tt>[[dladdr(3)]]</tt>, <tt>[[dlopen(3)]]</tt>, <tt>[[elf(5)]]</tt>, <tt>[[ld.so(8)]]</tt>

*Executable and Linking Format Specification*, 온라인 여러 곳에서 구할 수 있음.

----

2019-03-06
