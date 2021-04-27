## NAME

dladdr, dladdr1 - 주소를 심볼 정보로 변환하기

## SYNOPSIS

```c
#define _GNU_SOURCE
#include <dlfcn.h>

int dladdr(void *addr, Dl_info *info);
int dladdr1(void *addr, Dl_info *info, void **extra_info, int flags);
```

`-ldl`로 링크.

## DESCRIPTION

`dladdr()` 함수는 `addr`에 지정한 주소가 호출 응용에서 적재한 공유 오브젝트에 위치해 있는지 확인한다. 그런 경우에 `dladdr()`은 그 공유 오브젝트 및 `addr`에 걸리는 심볼에 대한 정보를 반환한다. `DL_info` 구조체로 그 정보가 반환된다.

```c
typedef struct {
    const char *dli_fname;  /* 주소가 포함된 공유
                               오브젝트의 경로명 */
    void       *dli_fbase;  /* 공유 오브젝트가 적재된
                               기준 주소 */
    const char *dli_sname;  /* addr에 정의가 걸리는
                               심볼의 이름 */
    void       *dli_saddr;  /* dli_sname에 나와 있는
                               심볼의 정확한 주소 */
} Dl_info;
```

`addr`에 대응하는 심볼을 찾을 수 없는 경우에는 `dli_sname`과 `dli_saddr`이 NULL로 설정된다.

`dladdr1()` 함수는 `dladdr()`과 비슷하되 `extra_info` 인자를 통해 추가 정보를 반환한다. 반환되는 정보는 `flags`에 지정한 값에 따라 달라지는데, 다음 값들 중 하나를 지정할 수 있다.

`RTLD_DL_LINKMAP`
:   걸린 파일의 링크 맵의 포인터를 반환한다. `extra_info` 인자가 `link_map` 구조체 포인터를 가리킨다. (즉 `struct link_map **`이다.) 그 구조체는 `<link.h>`에 다음과 같이 정의돼 있다.

        struct link_map {
            ElfW(Addr) l_addr;  /* ELF 파일 내 주소와
                                   메모리 내 주소의 차이 */
            char      *l_name;  /* 오브젝트를 찾은
                                   절대 경로명 */
            ElfW(Dyn) *l_ld;    /* 공유 오브젝트의
                                   dynamic 섹션 */
            struct link_map *l_next, *l_prev;
                                /* 적재된 오브젝트들을 연결 */

            /* 더해서 구현 내부용 필드들이
               추가로 있음 */
        };

`RTLD_DL_SYMENT`
:   대응하는 심볼의 ELF 심볼 테이블 항목의 포인터를 얻는다. `extra_info` 인자가 심볼 포인터의 포인터, 즉 `const ElfW(Sym) **`이다. `ElfW()` 매크로는 그 인자를 하드웨어 아키텍처에 맞는 ELF 데이터 타입 이름으로 바꿔 준다. 예를 들어 64비트 플랫폼에서 `ElfW(Sym)`은 데이터 타입 이름 `Elf64_Sym`을 내놓는데, 이는 `<elf.h>`에 다음처럼 정의돼 있다.

        typedef struct  {
            Elf64_Word    st_name;     /* 심볼 이름 */
            unsigned char st_info;     /* 심볼 타입 및 바인딩 */
            unsigned char st_other;    /* 심볼 가시성 */
            Elf64_Section st_shndx;    /* 섹션 인덱스 */
            Elf64_Addr    st_value;    /* 심볼 값 */
            Elf64_Xword   st_size;     /* 심볼 크기 */
        } Elf64_Sym;

    `st_name` 필드는 문자열 테이블의 인덱스이다.

    `st_info` 필드는 심볼의 타입과 바인딩을 담는다. 매크로 `ELF64_ST_TYPE(st_info)`으로 (32비트 플랫폼에서는 `ELF32_ST_TYPE()`으로) 타입을 뽑아낼 수 있으며 다음 값들 중 하나가 나온다.

    | 값              | 설명                                 |
    | --------------- | ------------------------------------ |
    | `STT_NOTYPE`    | 심볼 타입이 지정돼 있지 않음         |
    | `STT_OBJECT`    | 심볼이 데이터 오브젝트임             |
    | `STT_FUNC`      | 심볼이 코드 오브젝트임               |
    | `STT_SECTION`   | 섹션에 연계된 심볼                   |
    | `STT_FILE`      | 심볼 이름이 파일 이름임              |
    | `STT_COMMON`    | 심볼이 공용 데이터 오브젝트임        |
    | `STT_TLS`       | 심볼이 스레드 로컬 데이터 오브젝트임 |
    | `STT_GNU_IFUNC` | 심볼이 간접 코드 오브젝트임          |

    매크로 `ELF64_ST_BIND(st_info)`로 (32비트 플랫폼에서는 `ELF32_ST_BIND()`로) `st_info` 필드에서 심볼 바인딩을 뽑아낼 수 있으며 다음 값들 중 하나가 나온다.

    | 값               | 설명      |
    | ---------------- | --------- |
    | `STB_LOCAL`      | 지역 심볼 |
    | `STB_GLOBAL`     | 전역 심볼 |
    | `STB_WEAK`       | 약한 심볼 |
    | `STB_GNU_UNIQUE` | 유일 심볼 |

    `st_other` 필드는 심볼의 가시성을 담고 있는데 매크로 `ELF64_ST_VISIBILITY(st_info)`로 (32비트 플랫폼에서는 `ELF32_ST_VISIBILITY()`로) 뽑아낼 수 있으며 다음 값들 중 하나가 나온다.

    | 값              | 설명                             |
    | --------------- | -------------------------------- |
    | `STV_DEFAULT`   | 심볼 가시성 기본 규칙            |
    | `STV_INTERNAL`  | 프로세서별 숨기기 수준           |
    | `STV_HIDDEN`    | 다른 모듈에서 사용 불가능한 심볼 |
    | `STV_PROTECTED` | 선취 불가능, 내보이지 않음       |

## RETURN VALUE

성공 시 이 함수들은 0 아닌 값을 반환한다. `addr`에 지정한 주소가 어떤 공유 오브젝트에 걸리기는 하지만 그 공유 오브젝트 안의 심볼에 들어맞지는 않는 경우에는 `info->dli_sname` 및 `info->dli_saddr` 필드가 NULL으로 설정된다.

`addr`에 지정한 주소에 걸리는 공유 오브젝트를 찾을 수 없으면 이 함수들은 0을 반환한다. 이 경우에 <tt>[[dlerror(3)]]</tt>를 통해 오류 메시지를 얻을 수 *없다*.

## VERSIONS

glibc 2.0 및 이후에 `dladdr()`이 있다. glibc 2.3.3에서 `dladdr1()`이 처음 등장했다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `dladdr()`, `dladdr1()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

이 함수들은 비표준 GNU 확장이며 솔라리스에도 있다.

## BUGS

때때로 `dladdr()`에 함수 포인터를 줄 때 예상 외의 결과가 나올 수 있다. 일부 아키텍처(특히 i386과 x86-64)에서 인자로 준 함수가 분명 동적으로 링크 된 라이브러리에서 와야 하는 것인데 `dli_fname`과 `dli_fbase`가 `dladdr()`을 호출한 오브젝트를 가리키게 될 수도 있다.

함수 포인터 값 결정이 여전히 컴파일 시점이 이뤄지며 원래 오브젝트의 *plt*(Procedure Linkage Table) 섹션을 가리킬 뿐이라는 게 문제다. (호출 오브젝트에서 동적 링커에게 심볼 결정을 요청한 후에 그 호출을 처리한다.) 이걸 피하려면 코드를 위치 독립이 되게 컴파일 해 볼 수 있다. 그러면 컴파일러에서 컴파일 시점에 포인터를 미리 준비할 수 없게 되므로 *got*(Global Offset Table)에서 최종 심볼 주소를 가져온 다음에 `dladdr()`로 주는 코드를 `gcc(1)`가 만들어 내게 된다.

## SEE ALSO

<tt>[[dl_iterate_phdr(3)]]</tt>, <tt>[[dlinfo(3)]]</tt>, <tt>[[dlopen(3)]]</tt>, <tt>[[dlsym(3)]]</tt>, <tt>[[ld.so(8)]]</tt>

----

2021-03-22
