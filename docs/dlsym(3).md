## NAME

dlsym, dlvsym - 공유 오브젝트나 실행 파일 내 심볼의 주소 얻기

## SYNOPSIS

```c
#include <dlfcn.h>

void *dlsym(void *restrict handle, const char *restrict symbol);

#define _GNU_SOURCE
#include <dlfcn.h>

void *dlvsym(void *restrict handle, const char *restrict symbol,
             const char *restrict version);
```

`-ldl`로 링크.

## DESCRIPTION

`dlsym()` 함수는 <tt>[[dlopen(3)]]</tt>이 반환한 동적 적재 공유 오브젝트 "핸들"을 널 종료 심볼 이름과 함께 받아서 그 심볼이 메모리에 적재된 주소를 반환한다. 지정한 오브젝트에서, 그리고 그 오브젝트 적재 시 <tt>[[dlopen(3)]]</tt>에 의해 자동으로 적재된 공유 오브젝트들 어디서도 그 심볼을 찾지 못하면 `dlsym()`이 NULL을 반환한다. (`dlsym()`의 탐색은 그 공유 오브젝트들의 의존 관계 트리에서 너비 우선 방식으로 이뤄진다.)

드문 경우로 (아래 NOTES 참고) 심볼의 값이 실제로 NULL일 수도 있다. 따라서 `dlsym()`이 NULL을 반환한 게 꼭 오류를 나타내는 건 아니다. 값이 NULL인 심볼과 오류를 구별하는 정확한 방법은 <tt>[[dlerror(3)]]</tt> 호출로 이전 오류 조건을 지우고 `dlsym()`을 호출한 다음 다시 <tt>[[dlerror(3)]]</tt>를 호출해서 반환 값을 변수에 저장하고, 그 저장된 값이 NULL이 아닌지 확인하는 것이다.

`handle`에 지정할 수 있는 특수한 가상 핸들이 두 가지 있다.

`RTLD_DEFAULT`
:   기본 오브젝트 탐색 순서에 따라서 원하는 심볼의 첫 번째 위치를 찾는다. 실행 파일 및 그 의존 대상들의 전역 심볼에 더해서 `RTLD_GLOBAL` 플래그로 동적으로 적재된 공유 오브젝트의 심볼들이 탐색 대상에 포함된다.

`RTLD_NEXT`
:   탐색 순서에서 현재 오브젝트 다음부터 원하는 심볼의 다음 위치를 찾는다. 이를 이용해 다른 공유 오브젝트의 함수를 감싸는 래퍼를 제공할 수 있다. 예를 들어 사전 적재된 (<tt>[[ld.so(8)]]</tt>의 `LD_PRELOAD` 참고) 공유 오브젝트의 함수 정의에서 다른 공유 오브젝트에 있는 "진짜" 함수를 (또는 사전 적재가 여러 단계로 이뤄지는 경우에 "다음" 함수 정의를) 찾아서 호출할 수 있다.

`<dlfcn.h>`에서 `RTLD_DEFAULT` 및 `RTLD_NEXT` 정의를 얻으려면 기능 확인 매크로 `_GNU_SOURCE`가 정의돼 있어야 한다.

`dlvsym()` 함수는 `dlsym()`과 같은 동작을 하되 추가 인자로 버전 문자열을 받는다.

## RETURN VALUE

성공 시 이 함수들은 `symbol`에 연계된 주소를 반환한다. 실패 시 NULL을 반환한다. <tt>[[dlerror(3)]]</tt>로 오류 원인을 진단할 수 있다.

## VERSIONS

glibc 2.0 및 이후에 `dlsym()`이 있다. glibc 2.1에서 `dlvsym()`이 처음 등장했다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `dlsym()`, `dlvsym()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001에서 `dlsym()`을 기술한다. `dlvsym()` 함수는 GNU 확장이다.

## NOTES

전역 심볼의 주소가 NULL일 수 있는 경우가 여러 가지 있다. 예를 들어 링커 스크립트나 명령행 옵션 `--defsym`을 통해 링커가 주소 0에 심볼을 두게 할 수 있다. 또, 정의 안 된 약한 심볼도 NULL 값을 가진다. 그리고 GNU 간접 함수(IFUNC) 결정 함수가 결정 결과로 NULL을 반환해서 그게 심볼 값이 될 수도 있다. 마지막 경우에 `dlsym()`은 오류 없이 NULL을 반환한다. 하지만 앞의 두 경우에서 GNU 동적 링커의 동작은 비일관적이다. 재배치 과정이 성공하고 심볼이 NULL 값을 가지는 걸 관찰할 수 있지만 `dlsym()`은 실패하고 `dlerror()`는 검색 오류를 알린다.

### 역사

`dlsym()` 함수가 포함된 dlopen API는 SunOS에서 유래한 것이다. 그 시스템에 `dlvsym()`은 없다.

## EXAMPLES

<tt>[[dlopen(3)]]</tt> 참고.

## SEE ALSO

<tt>[[dl_iterate_phdr(3)]]</tt>, <tt>[[dladdr(3)]]</tt>, <tt>[[dlerror(3)]]</tt>, <tt>[[dlinfo(3)]]</tt>, <tt>[[dlopen(3)]]</tt>, <tt>[[dl.so(8)]]</tt>

----

2021-03-22
