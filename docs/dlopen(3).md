## NAME

dlclose, dlopen, dlmopen - 공유 오브젝트 열고 닫기

## SYNOPSIS

```c
#include <dlfcn.h>

void *dlopen(const char *filename, int flags);

int dlclose(void *handle);

#define _GNU_SOURCE
#include <dlfcn.h>

void *dlmopen (Lmid_t lmid, const char *filename, int flags);
```

`-ldl`로 링크.

## DESCRIPTION

### `dlopen()`

`dlopen()` 함수는 널 종료 문자열 `filename`이 가리키는 동적 공유 오브젝트(공유 라이브러리)를 적재하고 적재된 오브젝트에 대한 불투명한 "핸들"을 반환한다. 그 핸들을 dlopen API의 <tt>[[dlsym(3)]]</tt>, <tt>[[dladdr(3)]]</tt>, <tt>[[dlinfo(3)]]</tt>, `dlclose()` 같은 다른 함수들에 쓴다.

`filename`이 NULL이면 주 프로그램에 대한 핸들을 반환한다. `filename`에 슬래시("/")가 있으면 (상대 또는 절대) 경로명으로 해석한다. 아니면 동적 링커가 다음처럼 오브젝트를 탐색한다. (더 자세한 내용은 <tt>[[ld.so(8)]]</tt> 참고.)

* (ELF 한정) 호출하는 프로그램의 실행 파일에 DT_RPATH 태그가 있으면서 DT_RUNPATH 태그는 없으면 그 DT_RPATH 태그에 나열된 디렉터리들을 탐색한다.

* 프로그램 시작 시점에 콤마 구분 디렉터리 목록을 담은 환경 변수 `LD_LIBRARY_PATH`가 정의돼 있었으면 그 디렉터리들을 탐색한다. (보안을 위해 set-user-ID 및 set-group-ID 프로그램에 대해선 이 변수를 무시한다.)

* (ELF 한정) 호출하는 프로그램의 실행 파일에 DT_RUNPATH가 있으면 그 태그에 나열된 디렉터리들을 탐색한다.

* (`ldconfig(8)`로 관리하는) 캐시 파일 `/etc/ld.so.cache`를 확인해서 `filename`에 대한 항목이 있는지 본다.

* `/lib` 및 `/usr/lib` 디렉터리를 (차례대로) 탐색한다.

`filename`으로 지정한 오브젝트에서 다른 공유 오브젝트에 의존하는 경우에는 동적 링커에서 같은 규칙으로 그 오브젝트들을 자동으로 적재한다. (그 오브젝트들에 다시 의존 관계가 있고 하면 이 과정이 재귀적으로 이뤄질 수도 있다.)

`flags`에 다음 중 한 가지 값이 들어가야 한다.

`RTLD_LAZY`
:   게으른 바이딩을 수행한다. 참조하는 코드가 실행될 때에서야 심볼을 결정한다. 심볼 참조가  전혀 없으면 아예 결정을 하지 않는다. (함수 참조에만 게으른 바인딩을 수행한다. 변수에 대한 참조는 항상 공유 오브젝트 적재 때 즉시 바인드 한다.) glibc 2.1.1부터 환경 변수 `LD_BIND_NOW` 효력이 이 플래그보다 우선시된다.

`RTLD_NOW`
:   이 값을 지정하거나 환경 변수 `LD_BIND_NOW`가 비어 있지 않은 문자열로 설정돼 있으면 공유 오브젝트 내의 정의 안 된 심볼들을 모두 결정한 다음에 `dlopen()`이 반환한다. 그럴 수 없으면 오류를 반환한다.

`flags`에 다음 값을 0개 이상 OR 할 수도 있다.

`RTLD_GLOBAL`
:   이 공유 오브젝트에서 정의하는 심볼들이 이후 적재되는 공유 오브젝트들의 심볼 결정에 쓰일 수 있게 한다.

`RTLD_LOCAL`
:   `RTLD_GLOBAL`의 반대이며 어느 쪽 플래그도 지정하지 않은 경우의 기본값이다. 이 공유 오브젝트에서 정의하는 심볼들이 이후 적재되는 공유 오브젝트들의 심볼 찾기에 쓰이지 않는다.

`RTLD_NODELETE` (glibc 2.2부터)
:   `dlclose()`에서 공유 오브젝트를 내리지 않는다. 그래서 이후 `dlopen()`으로 그 오브젝트를 재적재하는 경우에 오브젝트의 정적 변수 및 전역 변수들이 다시 초기화되지 않는다.

`RTLD_NOLOAD` (glibc 2.2부터)
:   공유 오브젝트를 적재하지 않는다. 그 오브젝트가 이미 올라가 있는지 확인하는 데 쓸 수 있다. (올라가 있지 않으면 `dlopen()`이 NULL을 반환하고 올라가 있으면 오브젝트 핸들을 반환한다.) 또 이미 적재된 공유 오브젝트의 플래그를 승격시키는 데에도 쓸 수 있다. 예를 들어 `RTLD_LOCAL`로 올라가 있는 공유 오브젝트를 `RTLD_NOLOAD | RTLD_GLOBAL`로 다시 열 수 있다.

`RTLD_DEEPBIND` (glibc 2.3.4부터)
:   이 공유 오브젝트 내 심볼들에 대한 탐색 기회를 전역보다 우선시한다. 즉 오브젝트 자체에 심볼 정의가 있다면 이미 적재된 다른 오브젝트에 담긴 같은 이름의 전역 심볼 대신 자기 심볼을 쓰게 된다는 것이다.

`filename`이 NULL이면 주 프로그램에 대한 핸들을 반환한다. 그 핸들을 <tt>[[dlsym(3)]]</tt>에 주면 먼저 메인 프로그램에서 심볼을 찾아보고, 다음은 프로그램 시작 때 적재한 공유 오브젝트들 전체에서, 그리고 다음으로 `RTLD_GLOBAL` 플래그를 써서 `dlopen()`으로 적재한 공유 오브젝트들 전체에서 탐색한다.

공유 오브젝트 내 심볼 참조 결정은 메인 프로그램 및 의존 대상들을 위해 적재한 오브젝트들의 링크 맵에 있는 심볼, 앞서 `RTLD_GLOBAL`을 쓴 `dlopen()`으로 연 공유 오브젝트(와 의존 대상들)의 심볼, 공유 오브젝트 자체(와 그 오브젝트를 위해 적재된 의존 대상들) 내의 정의를 차례로 이용해 이뤄진다.

실행 파일의 전역 심볼 중에 `ld(1)`에서 동적 심볼 테이블에 집어넣은 게 있으면 마찬가지로 동적 적재 공유 오브젝트의 참조 결정에 이용될 수 있다. 심볼이 동적 심볼 테이블에 들어가는 건 실행 파일을 "-rdynamic" (또는 같은 의미의 "--export-dynamic") 플래그로 링크해서일 수도 있고 (그 경우 실행 파일의 전역 심볼 모두가 동적 심볼 테이블에 들어간다.) `ld(1)`가 정적 링크 중에 다른 오브젝트에 있는 심볼에 대한 의존성에 주목해서일 수도 있다.

`dlopen()`으로 같은 공유 오브젝트를 다시 열면 같은 오브젝트 핸들이 반환된다. 동적 링커에서 오브젝트 핸들별로 참조 카운트를 관리해서 성공적으로 `dlopen()` 한 횟수만큼 `dlclose()`를 호출할 때까지는 동적 적재 공유 오브젝트가 해제되지 않도록 한다. 생성자(아래 참고)도 오브젝트가 실제 메모리로 적재될 때만 (즉 참조 카운트가 1로 증가할 때만) 호출된다.

후속 `dlopen()` 호출에서 같은 공유 오브젝트를 `RTLD_NOW`로 적재해서 앞서 `RTLD_LAZY`로 적재된 공유 오브젝트에 대한 심볼 결정을 강제할 수 있다. 그와 비슷하게 앞서 `RTLD_LOCAL`로 열린 오브젝트가 후속 `dlopen()`에서 `RTLD_GLOBAL`로 승격될 수 있다.

어떤 이유로든 `dlopen()`이 실패하면 NULL을 반환한다.

### `dlmopen()`

이 함수는 아래에 적은 차이점들을 제외하면 `dlopen()`과 같은 작업을 수행하며 `filename`과 `flags` 인자뿐 아니라 반환 값이 동일하다.

`dlmopen()` 함수가 `dlopen()`과 다른 점은 추가 인자 `lmid`가 있어서 공유 오브젝트를 적재해야 할 링크 맵 리스트(*네임스페이스*라고도 부름)를 지정한다는 것이다. (반면 `dlopen()`에서는 동적으로 적재한 공유 오브젝트를 `dlopen()` 호출을 한 공유 오브젝트와 같은 네임스페이스에 추가한다.) `Lmid_t` 타입은 네임스페이스를 가리키는 불투명한 핸들이다.

`lmid` 인자는 기존 네임스페이스의 ID(<tt>[[dlinfo(3)]]</tt> `RTLD_DI_DMID` 요청으로 얻을 수 있음)이거나 다음 특수 값들 중 하나이다.

`LM_ID_BASE`
:   최초 네임스페이스에 (즉 응용의 네임스페이스에) 공유 오브젝트를 적재한다.

`LM_ID_NEWLM`
:   새 네임스페이스를 만들고 그 네임스페이스에 공유 오브젝트를 적재한다. 새 네임스페이스가 처음에는 비어 있으므로 오브젝트에서 필요로 하는 다른 공유 오브젝트들을 모두 참조하도록 오브젝트가 제대로 링크 돼 있어야 한다.

`filename`이 NULL인 경우에는 `lmid`에 `LM_ID_BASE` 값만 허용된다.

### `dlclose()`

`dlclose()` 함수는 `handle`이 가리키는 동적 적재 공유 오브젝트의 참조 카운트를 줄인다.

오브젝트의 참조 카운트가 0으로 떨어지고 그 오브젝트의 어떤 심볼도 다른 오브젝트에서 필요로 하지 않으면 오브젝트에 정의된 소멸자가 있으면 호출한 다음 그 오브젝트를 내린다. (이 오브젝트의 심볼을 다른 오브젝트에서 필요로 할 수도 있는 경우로는 그 오브젝트가 `RTLD_GLOBAL` 플래그로 열렸고 그 심볼이 다른 오브젝트의 재배치를 유발했을 때가 있다.)

`handle`이 가리키는 오브젝트에 `dlopen()`을 호출했을 때 자동으로 적재됐던 공유 오브젝트들을 모두 재귀적으로 같은 방식으로 닫는다.

`dlclose()`가 성공을 반환하더라도 `handle`에 연계된 심볼이 호출자의 주소 공간에서 제거됐다고 보장되는 건 아니다. 명시적 `dlopen()` 호출로 참조되는 것에 더해서 다른 공유 오브젝트에서의 의존 관계 때문에 공유 오브젝트가 묵시적으로 적재됐을 수도 (참조 카운트가 늘었을 수도) 있다. 그 모든 참조가 풀린 후에야 공유 오브젝트를 주소 공간에서 제거할 수 있다.

## RETURN VALUE

성공 시 `dlopen()`과 `dlmopen()`은 적재된 오브젝트에 대한 NULL 아닌 핸들을 반환한다. 오류 시 (파일을 찾을 수 없거나, 읽을 수 없거나, 형식이 틀리거나, 적재 중 오류가 생겼으면) 이 함수들은 NULL을 반환한다.

성공 시 `dlclose()`는 0을 반환한다. 오류 시 0 아닌 값을 반환한다.

이 함수들이 반환한 오류를 <tt>[[dlerror(3)]]</tt>로 진단할 수 있다.

## VERSIONS

glibc 2.0 및 이후에 `dlopen()`과 `dlclose()`가 있다. glibc 2.3.4에서 `dlmopen()`이 처음 등장했다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `dlopen()`, `dlmopen()`, `dlclose()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001에서 `dlclose()`와 `dlopen()`을 기술한다. `dlmopen()` 함수는 GNU 확장이다.

`RTLD_NOLOAD`, `RTLD_NODELETE`, `RTLD_DEEPBIND` 플래그는 GNU 확장이다. 이 중 앞의 둘은 솔라리스에도 있다.

## NOTES

### `dlmopen()`과 네임스페이스

링크 맵(link-map) 리스트는 동적 링커의 심볼 결정에서 격리된 네임스페이스를 나타낸다. 네임스페이스 내에서는 의존하는 공유 오브젝트들을 일반적 규칙에 따라 묵시적으로 적재하고 심볼 참조를 일반적 규칙에 따라 마찬가지로 결정하되 그 결정이 네임스페이스 내로 (명시적으로 또는 묵시적으로) 적재된 오브젝트들에서 제공하는 정의들로 국한된다.

`dlmopen()` 함수를 통해 오브젝트 적재 격리가 가능하다. 즉 공유 오브젝트를 새 네임스페이스에 올려서 그 새 오브젝트에서 제공하는 심볼들을 응용의 나머지 부분에 노출시키지 않을 수 있다. 참고로 `RTLD_LOCAL` 플래그로는 충분치 않다. 그 공유 오브젝트의 심볼을 다른 *모든* 공유 오브젝트에서 사용하지 못하게 하기 때문이다. 어떤 경우에는 동적 적재 공유 오브젝트에서 제공하는 심볼들을 다른 공유 오브젝트들(의 일부)에서 쓸 수 있게 하면서도 그 심볼들을 응용 전체로 노출하고 싶지는 않을 수 있다. 별도 네임스페이스와 `RTLD_GLOBAL` 플래그를 쓰면 그렇게 할 수 있다.

`dlmopen()` 함수를 쓰면 `RTLD_LOCAL` 플래그보다 더 나은 격리 기능을 제공할 수도 있다. 특히 `RTLD_LOCAL`로 적재된 공유 오브젝트는 `RTLD_GLOBAL`로 적재된 다른 공유 오브젝트에서 의존하는 경우 `RTLD_GLOBAL`로 승격될 수 있다. 따라서 공유 오브젝트 의존성 전체를 명확히 통제할 수 있는 (흔치 않은) 경우를 제외하면 `RTLD_LOCAL`은 적재된 공유 오브젝트를 격리하기에 충분하지 않다.

`dlmopen()`을 쓸 수 있는 곳으로 플러그인이 있다. 플러그인 적재 프레임워크 작성자가 플러그인 작성자들을 신뢰할 수 없어서 플러그인 프레임워크 내의 정의 안 된 심볼이 플러그인 심볼로 결정되기를 원치 않는 경우이다. 또 다른 용도는 같은 오브젝트를 여러 번 적재할 때이다. `dlmopen()`을 쓰지 않는다면 공유 오브젝트 파일 사본들을 따로 만들어야 할 것이다. 하지만 `dlmopen()`을 쓰면 같은 공유 오브젝트 파일을 다른 네임스페이스로 올리기만 하면 된다.

glibc 구현에서는 네임스페이스를 최대 16개까지 지원한다.

### 초기화 및 마무리 함수

공유 오브젝트에서 함수 속성 `__attribute__((constructor))`와 `__attribute__((destructor))`를 써서 함수를 내보일 수 있다. 생성자 함수는 `dlopen()` 반환 전에 실행되고 소멸자 함수는 `dlclose()` 반환 전에 실행된다. 한 공유 오브젝트에서 생성자와 소멸자를 여러 개 내보일 수 있으며 각 함수에 우선순위를 연계해서 실행 순서를 정할 수도 있다. 더 자세한 내용은 `gcc` 인포 페이지("Function attributes")를 보라.

같은 결과를 (불완전하게) 얻는 구식 방법은 링커에서 인식하는 특수 심볼 `_init`과 `_fini`를 쓰는 것이었다. 동적 적재 공유 오브젝트에서 이름이 `_init()`인 루틴을 내보이면 공유 오브젝트 적재 후 `dlopen()` 반환 전에 그 코드가 실행된다. 또 공유 오브젝트에서 이름이 `_fini()`인 루틴을 내보이면 오브젝트를 내리기 바로 전에 그 루틴이 호출된다. 이 경우에 이 함수들의 기본 버전을 담고 있는 시스템 개시 파일들을 링크하지 않도록 해야 한다. `gcc(1)`의 명령행 옵션 `-nostartfiles`를 쓰면 된다.

`_init`과 `_fini`를 사용하는 방식은 현재 폐기 예정 상태이며 대신 앞서 언급한 생성자와 소멸자를 써야 한다. 여러 장점이 있으며 무엇보다 초기화 및 마무리 함수를 여러 개 정의할 수 있다.

glibc 2.2.3부터 <tt>[[atexit(3)]]</tt>를 이용해 공유 오브젝트가 내려갈 때 자동으로 호출되는 종료 핸들러를 등록할 수 있다.

### 역사

이 함수들이 포함된 dlopen API는 SunOS에서 유래한 것이다.

## BUGS

glibc 2.24 현재 `dlmopen()` 호출 시 `RTLD_GLOBAL` 플래그를 지정하면 오류가 발생한다. 또한 최초 네임스페이스 아닌 네임스페이스에 적재된 오브젝트에서 `dlopen()`을 호출하면서 `RTLD_GLOBAL`을 지정하면 프로그램 크래시(`SIGSEGV`)가 발생한다.

## EXAMPLE

아래 프로그램은 (glibc) 수학 라이브러리를 적재한 다음 `cos(3)` 함수 주소를 알아내서 코사인 2.0을 찍는다. 다음은 프로그램 빌드 및 실행 예이다.

```text
$ cc dlopen_demo.c -ldl
$ ./a.out
-0.416147
```

### 프로그램 소스

```c
#include <stdio.h>
#include <stdlib.h>
#include <dlfcn.h>
#include <gnu/lib-names.h>  /* LIBM_SO 정의 ("libm.so.6" 같은 문자열) */
int
main(void)
{
    void *handle;
    double (*cosine)(double);
    char *error;

    handle = dlopen(LIBM_SO, RTLD_LAZY);
    if (!handle) {
        fprintf(stderr, "%s\n", dlerror());
        exit(EXIT_FAILURE);
    }

    dlerror();    /* 이전 오류 지우기 */

    cosine = (double (*)(double)) dlsym(handle, "cos");

    /* ISO C 표준에 따르면 위처럼 함수 포인터와 'void *' 사이에서
       캐스팅을 해서 나오는 결과가 규정돼 있지 않다. POSIX.1-2003 및
       POSIX.1-2008에서는 그 상황을 받아들여서 다음 우회 방법을
       제안했다.

           *(void **) (&cosine) = dlsym(handle, "cos");

       이 (어색한) 캐스팅은 ISO C 표준을 준수하는 것이면서
       컴파일러 경고가 나오지 않게 해 준다.

       POSIX.1-2008의 2013년판 기술 정오표(소위 POSIX.1-2013)에서는
       준수 구현체가 'void *'와 함수 포인터 사이 캐스팅을 지원해야
       한다고 요구하는 것으로 상황을 개선했다. 그럼에도 불구하고
       일부 컴파일러들은 (가령 gcc에 '-pedantic' 옵션을 쓰면)
       이 프로그램에서 쓴 캐스팅에 대해 뭐라 뭐라 할 수도 있다. */

    error = dlerror();
    if (error != NULL) {
        fprintf(stderr, "%s\n", error);
        exit(EXIT_FAILURE);
    }

    printf("%f\n", (*cosine)(2.0));
    dlclose(handle);
    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

`ld(1)`, `ldd(1)`, `pldd(1)`, <tt>[[dl_iterate_phdr(3)]]</tt>, <tt>[[dladdr(3)]]</tt>, <tt>[[dlerror(3)]]</tt>, <tt>[[dlinfo(3)]]</tt>, <tt>[[dlsym(3)]]</tt>, <tt>[[rtld-audit(7)]]</tt>, <tt>[[ld.so(8)]]</tt>, `ldconfig(8)`

gcc info 페이지, ld info 페이지

----

2019-08-02