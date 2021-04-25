## NAME

malloc, free, calloc, realloc, reallocarray - 동적 메모리 할당하고 해제하기

## SYNOPSIS

```c
#include <stdlib.h>

void *malloc(size_t size);
void free(void *ptr);
void *calloc(size_t nmemb, size_t size);
void *realloc(void *ptr, size_t size);
void *reallocarray(void *ptr, size_t nmemb, size_t size);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`reallocarray()`:
:   glibc 2.29부터:
    :   `_DEFAULT_SOURCE`

    glibc 2.28 및 이전:
    :   `_GNU_SOURCE`

## DESCRIPTION

`malloc()` 함수는 `size` 바이트를 할당하여 할당한 메모리에 대한 포인터를 반환한다. *그 메모리는 초기화 되어 있지 않다.* `size`가 0이면 `malloc()`은 NULL을 반환하거나, 이후 `free()`에 무사히 전달할 수 있는 고유한 포인터 값을 반환한다.

`free()` 함수는 `ptr`이 가리키는 메모리 공간을 해제하는데 `ptr`은 이전의 `malloc()`, `calloc()`, `realloc()` 호출이 반환한 것이어야 한다. 그렇지 않으면, 또는 이미 `free(ptr)`을 호출했으면 규정되어 있지 않은 동작이 일어난다. `ptr`이 NULL이면 어떤 동작도 수행하지 않는다.

`calloc()` 함수는 각 `size` 바이트인 원소 `nmemb` 개의 배열을 위한 배열을 할당하여 할당한 메모리에 대한 포인터를 반환한다. 그 메모리는 0으로 채워져 있다. `nmemb`나 `size`가 0이면 `calloc()`은 NULL을 반환하거나, 이후 `free()`에 무사히 전달할 수 있는 고유한 포인터 값을 반환한다. `nmemb`와 `size`의 곱이 정수 오버플로우를 일으킬 경우 `calloc()`은 오류를 반환한다. 반면 다음 `malloc()` 호출은 정수 오버플로우를 탐지하지 않으며, 그래서 잘못된 크기의 블록이 할당되는 결과를 낳을 수 있다.

```c
malloc(nmemb * size);
```

`realloc()` 함수는 `ptr`이 가리키는 메모리 블록의 크기를 `size` 바이트로 바꾼다. 시작점부터 이전 크기와 새 크기 중 작은 값까지의 범위에서는 내용물이 바뀌지 않는다. 새 크기가 이전 크기보다 큰 경우 추가되는 메모리가 초기화 되지 *않는다*. `ptr`이 NULL이면 그 호출은 모든 `size` 값에 대해 `malloc(size)`와 동등하다. `size`가 0이고 `ptr`이 NULL이 아니면 그 호출은 `free(ptr)`과 동등하다. (이 동작은 이식성이 없다. NOTES 참고.) `ptr`이 NULL이 아니라면 그 값은 이전의 `malloc()`, `calloc()`, `realloc()` 호출이 반환한 것이어야 한다. 가리키는 영역이 옮겨지는 경우 `free(ptr)`를 한다.

`reallocarray()` 함수는 `ptr`이 가리키는 메모리 블록의 크기를 각 `size` 바이트인 원소 `nmemb` 개의 배열에 충분하도록 바꾼다. 다음 호출과 동등하다.

```c
realloc(ptr, nmemb * size);
```

하지만 `realloc()` 호출과 달리 `reallocarray()`는 곱셈이 넘치게 될 경우에 안전하게 실패한다. 그런 오버플로우가 일어나는 경우 `reallocarray()`는 NULL을 반환하고 `errno`를 `ENOMEM`으로 설정하며, 원래 메모리 블록을 그대로 남겨둔다.

## RETURN VALUE

`malloc()` 및 `calloc()` 함수는 할당한 메모리에 대한 포인터를 반환한다. 그 메모리는 어떤 내장 타입에도 적합하도록 정렬되어 있다. 오류 시 이 함수들은 NULL을 반환한다. `size`가 0인 `malloc()` 성공 호출이, 또는 `nmemb`나 `size`가 0인 `calloc()` 성공 호출이 NULL을 반환할 수도 있다.

`free()` 함수는 아무 값도 반환하지 않는다.

`realloc()` 함수는 새로 할당한 메모리에 대한 포인터를 반환하는데, 어떤 내장 타입에도 적합하도록 정렬되어 있다. 요청이 실패한 경우 NULL을 반환한다. 할당이 이동하지 않았다면 (가령 할당을 제자리에서 확장할 공간이 있었다면) 반환되는 포인터가 `ptr`과 같을 수도 있고, 할당이 다른 주소로 이동했다면 `ptr`과 다를 수도 있다. `size`가 0이면 NULL이나 `free()`에 전달하기 적합한 포인터를 반환한다. `realloc()`이 실패한 경우 원래 블록은 바뀌지 않고 그대로이다. 즉, 해제되거나 이동하지 않는다.

성공 시 `reallocarray()` 함수는 새로 할당한 메모리에 대한 포인터를 반환한다. 실패 시 NULL을 반환하며 원래 메모리 블록은 바뀌지 않는다.

## ERRORS

`calloc()`, `malloc()`, `realloc()`, `reallocarray()`가 다음 오류로 실패할 수 있다.

`ENOMEM`
:   메모리 부족. 아마 응용이 <tt>[[getrlimit(2)]]</tt>에 기술된 `RLIMIT_AS`나 `RLIMIT_DATA` 제한에 걸렸을 것이다.

## VERSIONS

glibc 버전 2.26에서 `reallocarray()`가 처음 등장했다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `malloc()`, `free()`, `calloc()`, `realloc()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

`malloc()`, `free()`, `calloc()`, `realloc()`: POSIX.1-2001, POSIX.1-2008, C89, C99.

`reallocarray()`는 비표준 확장이며 OpenBSD 5.6과 FreeBSD 11.0에서 처음 등장했다.

## NOTES

기본적으로 리눅스는 낙천적 메모리 할당 전략을 따른다. 이는 `malloc()`이 NULL 아닌 결과를 반환할 때 그 메모리가 정말 사용 가능하다는 보장이 없다는 뜻이다. 시스템에 메모리가 부족하다고 판명되는 경우 OOM 킬러에 의해 한 개 이상의 프로세스가 죽게 된다. 더 많은 정보는 <tt>[[proc(5)]]</tt>의 `/proc/sys/vm/overcommit_memory` 및 `/proc/sys/vm/oom_adj` 설명과 리눅스 커널 소스 파일의 `Documentation/vm/overcommit-accounting.rst`를 보라.

보통은 `malloc()`이 힙에서 메모리를 할당하며 <tt>[[sbrk(2)]]</tt>를 이용해 필요한 만큼 힙의 크기를 조정한다. `MMAP_THRESHOLD`보다 큰 메모리 블록을 할당할 때 glibc의 `malloc()` 구현은 <tt>[[mmap(2)]]</tt>을 이용해 비공유 익명 매핑으로 메모리를 할당한다. `MMAP_THRESHOLD`는 기본적으로 128kB이고 <tt>[[mallopt(3)]]</tt>로 조정 가능하다. 리눅스 4.7 전에서는 <tt>[[mmap(2)]]</tt>으로 수행하는 할당이 `RLIMIT_DATA` 자원 제한의 영향을 받지 않았다. 리눅스 4.7부터는 <tt>[[mmap(2)]]</tt>으로 수행하는 할당에도 이 제한이 적용된다.

다중 스레드 응용에서의 오염을 막기 위해 내부적으로 뮤텍스를 사용해서 이 함수들이 이용하는 메모리 관리 자료 구조들을 보호한다. 여러 스레드가 동시에 메모리를 할당 및 해제하는 다중 스레드 응용에서는 이 뮤텍스들에 대한 경쟁이 있을 수 있을 것이다. 다중 스레드 응용에서의 메모리 할당을 확장성 있게 처리하기 위해 glibc에서는 뮤텍스 경쟁을 탐지하는 경우 *메모리 할당 아레나(arena)*를 추가로 만든다. 각 아레나는 시스템이 내부적으로 (<tt>[[brk(2)]]</tt>나 <tt>[[mmap(2)]]</tt>으로) 할당해서 별도의 뮤텍스로 관리하는 커다란 메모리 영역이다.

SUSv2에서는 `malloc()`, `calloc()`, `realloc()`이 실패 시에 `errno`를 `ENOMEM`으로 설정하기를 요구한다. glibc에서는 이를 가정한다. (그리고 이 루틴들의 glibc 버전은 그렇게 동작한다.) `errno`를 설정하지 않는 자체 malloc 구현을 사용하는 경우에 특정 라이브러리 루틴들이 실패하면서 `errno`에 이유를 주지 않을 수도 있다.

`malloc()`, `calloc()`, `realloc()`, `free()` 안에서 죽는 것은 거의 언제나 할당 영역을 넘겨 접근하거나 같은 포인터를 두 번 해제하는 것 같은 힙 오염과 관련돼 있다.

환경 변수를 통해 `malloc()` 구현을 튜닝 할 수 있다. 자세한 내용은 <tt>[[mallopt(3)]]</tt>를 보라.

### 이식성 없는 동작

`size`가 0과 같고 `ptr`이 NULL이 아닐 때 `realloc()`의 동작 방식은 glibc 한정이다. 다른 구현체에서는 NULL을 반환하고 `errno`를 설정할 수 있다. 이식 가능한 POSIX 프로그램에서는 이를 피해야 한다. `realloc(3p)` 참고.

## SEE ALSO

<tt>[[valgrind(1)]]</tt>, <tt>[[brk(2)]]</tt>, <tt>[[mmap(2)]]</tt>, <tt>[[alloca(3)]]</tt>, <tt>[[malloc_get_state(3)]]</tt>, <tt>[[malloc_info(3)]]</tt>, <tt>[[malloc_trim(3)]]</tt>, <tt>[[malloc_usable_size(3)]]</tt>, <tt>[[mallopt(3)]]</tt>, <tt>[[mcheck(3)]]</tt>, <tt>[[mtrace(3)]]</tt>, <tt>[[posix_memalign(3)]]</tt>

GNU C 라이브러리 구현에 대한 자세한 내용은 <https://sourceware.org/glibc/wiki/MallocInternals> 참고.

----

2021-03-22
