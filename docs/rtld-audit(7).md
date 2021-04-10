## NAME

rtld-audit - 동적 링커 감사 API

## SYNOPSIS

```c
#define _GNU_SOURCE             /* feature_test_macros(7) 참고 */
#include <link.h>
```

## DESCRIPTION

GNU 동적 링커(런타임 링커)에서 감사 API를 제공하는데 이를 통해 다양한 동적 링크 이벤트 발생 시에 응용에서 알림을 받을 수 있다. 이 API는 솔라리스 런타임 링커에서 제공하는 감사 인터페이스와 매우 비슷하다. `<link.h>`를 포함시켜서 필요한 상수 및 원형들을 정의한다.

이 인터페이스를 사용하려는 프로그래머는 표준 이름의 함수들을 구현한 공유 라이브러리를 만들게 된다. 모든 함수를 구현해야 하는 건 아니다. 대부분의 경우 특정 이벤트 감사 유형에 대해 프로그래머가 관심이 없으면 대응하는 감사 함수 구현을 제공할 필요가 없다.

감사 인터페이스를 사용하려면 환경 변수 `LD_AUDIT`을 콤마로 구분된 공유 라이브러리 목록을 담도록 정의해야 한다. 그 라이브러리들 각각에서 감사 API(의 일부)를 구현할 수 있다. 감사 대상 이벤트가 발생하면 라이브러리 나열 순서 대로 각 라이브러리의 대응 함수가 호출된다.

### `la_version()`

```c
unsigned int la_version(unsigned int version);
```

감사 라이브러리에 *꼭 정의돼 있어야 하는* 유일한 함수이며 동적 링커와 감사 라이브러리 간의 첫인사를 수행한다. 동적 링커가 이 함수를 호출할 때 링커에서 지원하는 가장 높은 감사 인터페이스 버전을 `version`으로 전달한다. 필요 시 감사 라이브러리에서 자기 요건에 그 버전이 충분한지 확인할 수 있다.

함수 결과로 이 함수는 이 감사 라이브러리에서 사용을 기대하는 감사 인터페이스 버전을 반환해야 한다. (`version`을 반환하는 것도 가능하다.) 반환 값이 0이거나 동적 링커가 지원하는 것보다 큰 버전이면 그 감사 라이브러리를 무시한다.

### `la_objsearch()`

```c
char *la_objsearch(const char *name, uintptr_t *cookie,
                   unsigned int flag);
```

동적 링커에서 이 함수를 호출해서 공유 오브젝트 탐색을 하려 한다는 걸 감사 라이브러리에게 알린다. `name` 인자는 탐색할 파일명이나 경로명이다. `cookie`는 탐색을 유발한 공유 오브젝트를 식별할 수 있게 해 준다. `flag`는 다음 값들 중 하나로 설정돼 있다.

`LA_SER_ORIG`
:   `name`이 탐색하려는 원래 이름이다. 보통 ELF `DT_NEEDED` 항목에서 온 이름이거나 <tt>[[dlopen(3)]]</tt>의 `filename` 인자이다.

`LA_SER_LIBPATH`
:   `LD_LIBRARY_PATH`에 지정된 디렉터리를 이용해 `name`을 만들었다.

`LA_SER_RUNPATH`
:   ELF `DT_RPATH` 내지 `DT_RUNPATH` 목록에 지정된 디렉터리를 이용해 `name`을 만들었다.

`LA_SER_CONFIG`
:   `ldconfig(8)` 캐시(`/etc/ld.so.cache`)를 통해 `name`을 찾았다.

`LA_SER_DEFAULT`
:   기본 디렉터리들 중 하나를 탐색해서 `name`을 찾았다.

`LA_SER_SECURE`
:   `name`이 보안 오브젝트로 한정돼 있다. (리눅스에서는 쓰지 않음.)

함수 결과로 `la_objsearch()`는 동적 링커가 이후 처리에 사용해야 할 경로명을 반환한다. NULL을 반환하면 이 경로명을 이후 처리에서 무시한다. 감사 라이브러리에서 탐색 경로를 감시만 하려는 거라면 `name`을 반환하면 된다.

### `la_activity()`

```c
void la_activity(uintptr_t *cookie, unsigned int flag);
```

동적 링커가 이 함수를 호출해서 링크 맵 동작이 있어나고 있다는 걸 감사 라이브러리에게 알린다. `cookie`는 링크 맵의 머리에 있는 오브젝트를 식별할 수 있게 해 준다. 동적 링커가 이 함수를 호출할 때 `flag`를 다음 값들 중 하나로 설정한다.

`LA_ACT_ADD`
:   링크 맵에 새 오브젝트들을 추가하려 한다.

`LA_ACT_DELETE`
:   링크 맵에서 오브젝트들을 제거하려 한다.

`LA_ACT_CONSISTENT`
:   링크 맵 동작이 끝났다. 맵이 다시 모순 없는 상태이다.

### `la_objopen()`

```c
unsigned int la_objopen(struct link_map *map, Lmid_t lmid,
                        uintptr_t *cookie);
```

새 공유 오브젝트를 적재할 때 동적 링커가 이 함수를 호출한다. `map` 인자는 그 오브젝트를 기술하는 링크 맵 구조체에 대한 포인터이다. `lmid` 인자는 다음 값들 중 하나이다.

`LM_ID_BASE`
:   링크 맵이 최초 네임스페이스에 속한다.

`LM_ID_NEWLM`
:   링크 맵이 <tt>[[dlmopen(3)]]</tt>을 통해 요청된 새 네임스페이스에 속한다.

`cookie`는 이 오브젝트의 식별자에 대한 포인터이다. 이후의 감사 라이브러리 함수 호출에 이 식별자가 제공되므로 이 오브젝트를 식별할 수 있다. 이 식별자는 오브젝트의 링크 맵을 가리키도록 초기화 돼 있지만 감사 라이브러리에서 오브젝트 식별에 쓰기 편한 다른 어떤 값으로 바꿀 수 있다.

반환 값으로 `la_objopen()`은 다음 상수들을 0개 이상 OR 해서 만든 비트 마스크를 반환한다. 그래서 감사 라이브러리에서 `la_symbind*()`로 감시할 오브젝트를 선택할 수 있다.

`LA_FLG_BINDTO`
:   이 오브젝트로의 심볼 바인딩을 감사한다.

`LA_FLG_BINDFROM`
:   이 오브젝트로부터의 심볼 바인딩을 감사한다.

`la_objopen()`의 반환 값 0은 이 오브젝트에 대해 어떤 심볼 바인딩도 감사하지 않을 것임을 나타낸다.

### `la_objclose()`

```c
unsinged int la_objclose(uintptr_t *cookie);
```

오브젝트에 마무리 코드가 있으면 실행한 후이면서 그 오브젝트를 내리기는 전에 동적 링커가 이 함수를 호출한다. `cookie` 인자는 앞선 `la_objopen()` 호출에서 얻은 식별자이다.

현재 구현에서는 `la_objclose()`가 반환하는 값을 무시한다.

### `la_preinit()`

```c
void la_preinit(uintptr_t *cookie);
```

모든 공유 오브젝트들을 적재한 후이면서 응용으로 제어를 넘기기는 전에 (즉 `main()` 호출 전에) 동적 링커가 이 함수를 호출한다. 참고로 이후 `main()`에서도 <tt>[[dlopen(3)]]</tt>으로 오브젝트를 동적으로 적재할 수 있다.

### `la_symbind*()`

```c
uintptr_t la_symbind32(Elf32_Sym *sym, unsigned int ndx,
                       uintptr_t *refcook, uintptr_t *defcook,
                       unsigned int *flags, const char *symname);
uintptr_t la_symbind64(Elf64_Sym *sym, unsigned int ndx,
                       uintptr_t *refcook, uintptr_t *defcook,
                       unsigned int *flags, const char *symname);
```

`la_objopen()`에서 감사 알림을 받게 표시했던 두 공유 오브젝트 간에 심볼 바인딩이 이뤄질 때 동적 링커가 이 함수들 중 하나를 호출한다. `la_symbind32()` 함수는 32비트 플랫폼에서 쓰는 것이고 `la_symbind64()` 함수는 64비트 플랫폼에서 쓰는 것이다.

`sym` 인자는 바인드 하려는 심볼에 대한 정보를 제공하는 구조체에 대한 포인터이다. 구조체 정의는 `<elf.h>`에 있다. 이 구조체의 필드들 중에 `st_value`은 심볼이 바인드 되는 주소를 나타낸다.

`ndx` 인자는 바인드 대상 공유 오브젝트의 심볼 테이블에서 그 심볼의 인덱스이다.

`refcook` 인자는 심볼 참조를 하고 있는 공유 오브젝트를 나타낸다. `LA_FLG_BINDFROM`을 반환한 `la_objopen()` 함수에 제공된 것과 같은 식별자이다. `defcook` 인자는 참조되는 심볼을 정의하고 있는 공유 오브젝트를 나타낸다. `LA_FLG_BINDTO`를 반환한 `la_objopen()` 함수에 제공된 것과 같은 식별자이다.

`symname` 인자는 심볼 이름은 담은 문자열을 가리킨다.

`flags` 인자는 심볼에 대한 정보를 제공하면서 동시에 이 PLT(Procedure Linkage Table) 항목의 이후 감사 방식을 변경하는 데 쓸 수 있는 비트 마스크이다. 동적 링커가 이 인자에 다음 비트 값들을 제공할 수 있다.

`LA_SYMB_DLSYM`
:   <tt>[[dlsym(3)]]</tt> 호출로 인해 바인딩이 발생했다.

`LA_SYMB_ALTVALUE`
:   앞선 `la_symbind*()` 호출이 이 심볼에 대해 대체 값을 반환했다.

기본적으로 감사 라이브러리에서 `la_pltenter()` 및 `la_pltexit()` 함수(아래 참고)를 구현하고 있으면 `la_symbind()` 후에 심볼이 참조될 때마다 PLT 항목에 대해 그 함수들이 호출된다. `*flags`에 다음 플래그들을 OR 해서 그 기본 동작을 바꿀 수 있다.

`LA_SYMB_NOPLTENTER`
:   이 심볼에 `la_pltenter()`를 호출하지 말 것.

`LA_SYMB_NOPLTEXIT`
:   이 심볼에 `la_pltexit()`을 호출하지 말 것.

`la_symbind32()` 및 `la_symbind64()`의 반환 값은 함수 반환 후 제어가 넘어가야 할 주소이다. 감사 라이브러리에서 심볼 바인딩을 감시만 하려는 거라면 `sym->st_value`를 반환하면 된다. 라이브러리에서 제어를 다른 방향으로 바꾸고 싶다면 다른 값을 반환할 수 있다.

### `la_pltenter()`

이 함수의 정확한 이름과 인자 타입은 하드웨어 플랫폼에 따라 다르다. (`<link.h>`에 절적한 정의가 있다.) 다음은 x86-32용 정의이다.

```c
Elf32_Addr la_i86_gnu_pltenter(Elf32_Sym *sym, unsigned int ndx,
                 uintptr_t *refcook, uintptr_t *defcook,
                 La_i86_regs *regs, unsigned int *flags,
                 const char *symname, long int *framesizep);
```

바인딩 알림을 받게 표시된 두 공유 오브젝트 간에서 PLT 항목이 호출되기 바로 전에 이 함수가 호출된다.

`sym`, `ndx`, `refcook`, `defcook`, `symname`은 `la_symbind*()`에서와 같다.

`regs` 인자는 이 PLT 항목 호출에 사용될 레지스터들의 값을 담은 구조체(`<link.h>`에 정의돼 있음)를 가리킨다.

`flags` 인자는 `la_symbind*()`에서처럼 이 PLT 항목에 대한 정보를 담고 있고 이후 감사 방식 변경에 이용할 수 있는 비트 마스크를 가리킨다.

`framesizep` 인자는 `long int` 버퍼를 가리키는데 이를 이용해 이 PLT 항목 호출에 사용하는 프레임 크기를 명확히 설정할 수 있다. 이 심볼에 대해 여러 `la_pltenter()`에서 다른 값들을 반환하면 가장 큰 반환 값을 쓴다. 이 버퍼가 명확하게 적절한 값으로 설정된 경우에만 `la_pltexit()` 함수가 호출된다.

`la_pltenter()`의 반환 값은 `la_symbind*()`에서와 같다.

### `la_pltexit()`

이 함수의 정확한 이름과 인자 타입은 하드웨어 플랫폼에 따라 다르다. (`<link.h>`에 절적한 정의가 있다.) 다음은 x86-32용 정의이다.

```c
unsigned int la_i86_gnu_pltexit(Elf32_Sym *sym, unsigned int ndx,
                 uintptr_t *refcook, uintptr_t *defcook,
                 const La_i86_regs *inregs, La_i86_retval *outregs,
                 const char *symname);
```

바인딩 알림을 받게 표시된 두 공유 오브젝트 간에서 PLT 항목이 반환할 때 이 함수가 호출된다. PLT 항목 호출자에게 제어가 돌아가기 바로 전에 함수가 호출된다.

`sym`, `ndx`, `refcook`, `defcook`, `symname`은 `la_symbind*()`에서와 같다.

`inregs` 인자는 이 PLT 항목 호출에 사용된 레지스터들의 값을 담은 구조체(`<link.h>`에 정의돼 있음)를 가리킨다. `outregs` 인자는 이 PLT 항목 호출의 반환 값을 담은 구조체(`<link.h>`에 정의돼 있음)를 가리킨다. 이 값들을 호출자가 변경할 수 있으며 그 변경 내용이 PLT 항목 호출자에게 보이게 된다.

현재 GNU 구현에서는 `la_pltexit()`의 반환 값을 무시한다.

## CONFORMING TO

이 API는 비표준이되 솔라리스 *Linker and Libraries Guide*의 *Runtime Linker Auditing Interface* 장에 기술된 솔라리스 API와 매우 비슷하다.

## NOTES

솔라리스 동적 링커 감사 API와의 다음 차이에 유의하라.

* 솔라리스의 `la_objfilter()` 인터페이스를 GNU 구현에서는 지원하지 않는다.

* 솔라리스의 `la_symbind32()` 및 `la_pltexit()` 함수에는 `symname` 인자가 없다.

* 솔라리스의 `la_pltexit()` 함수에는 `inregs` 및 `outregs` 인자가 없다. (하지만 `regval` 인자로 함수 반환 값은 제공한다.)

## BUGS

glibc 버전 2.9까지에서 `LD_AUDIT`에 감사 라이브러리를 여러 개 지정하면 런타임 크래시가 발생한다. glibc 2.10에서 고쳐졌다고 한다.

## EXAMPLE

```c
#include <link.h>
#include <stdio.h>

unsigned int
la_version(unsigned int version)
{
    printf("la_version(): %d\n", version);

    return version;
}

char *
la_objsearch(const char *name, uintptr_t *cookie, unsigned int flag)
{
    printf("la_objsearch(): name = %s; cookie = %p", name, cookie);
    printf("; flag = %s\n",
            (flag == LA_SER_ORIG) ?    "LA_SER_ORIG" :
            (flag == LA_SER_LIBPATH) ? "LA_SER_LIBPATH" :
            (flag == LA_SER_RUNPATH) ? "LA_SER_RUNPATH" :
            (flag == LA_SER_DEFAULT) ? "LA_SER_DEFAULT" :
            (flag == LA_SER_CONFIG) ?  "LA_SER_CONFIG" :
            (flag == LA_SER_SECURE) ?  "LA_SER_SECURE" :
            "???");

    return name;
}

void
la_activity (uintptr_t *cookie, unsigned int flag)
{
    printf("la_activity(): cookie = %p; flag = %s\n", cookie,
            (flag == LA_ACT_CONSISTENT) ? "LA_ACT_CONSISTENT" :
            (flag == LA_ACT_ADD) ?        "LA_ACT_ADD" :
            (flag == LA_ACT_DELETE) ?     "LA_ACT_DELETE" :
            "???");
}

unsigned int
la_objopen(struct link_map *map, Lmid_t lmid, uintptr_t *cookie)
{
    printf("la_objopen(): loading \"%s\"; lmid = %s; cookie=%p\n",
            map->l_name,
            (lmid == LM_ID_BASE) ?  "LM_ID_BASE" :
            (lmid == LM_ID_NEWLM) ? "LM_ID_NEWLM" :
            "???",
            cookie);

    return LA_FLG_BINDTO | LA_FLG_BINDFROM;
}

unsigned int
la_objclose (uintptr_t *cookie)
{
    printf("la_objclose(): %p\n", cookie);

    return 0;
}

void
la_preinit(uintptr_t *cookie)
{
    printf("la_preinit(): %p\n", cookie);
}

uintptr_t
la_symbind32(Elf32_Sym *sym, unsigned int ndx, uintptr_t *refcook,
        uintptr_t *defcook, unsigned int *flags, const char *symname)
{
    printf("la_symbind32(): symname = %s; sym->st_value = %p\n",
            symname, sym->st_value);
    printf("        ndx = %d; flags = 0x%x", ndx, *flags);
    printf("; refcook = %p; defcook = %p\n", refcook, defcook);

    return sym->st_value;
}

uintptr_t
la_symbind64(Elf64_Sym *sym, unsigned int ndx, uintptr_t *refcook,
        uintptr_t *defcook, unsigned int *flags, const char *symname)
{
    printf("la_symbind64(): symname = %s; sym->st_value = %p\n",
            symname, sym->st_value);
    printf("        ndx = %d; flags = 0x%x", ndx, *flags);
    printf("; refcook = %p; defcook = %p\n", refcook, defcook);

    return sym->st_value;
}

Elf32_Addr
la_i86_gnu_pltenter(Elf32_Sym *sym, unsigned int ndx,
        uintptr_t *refcook, uintptr_t *defcook, La_i86_regs *regs,
        unsigned int *flags, const char *symname, long int *framesizep)
{
    printf("la_i86_gnu_pltenter(): %s (%p)\n", symname, sym->st_value);

    return sym->st_value;
}
```

## SEE ALSO

`ldd(1)`, <tt>[[dlopen(3)]]</tt>, <tt>[[ld.so(8)]]</tt>, `ldconfig(8)`

----

2019-03-06
