## NAME

\__malloc_hook, \__malloc_initialize_hook, \__memalign_hook, \__free_hook, \__realloc_hook, \__after_morecore_hook - malloc 디버깅 변수

## SYNOPSIS

```c
#include <malloc.h>

void *(*volatile __malloc_hook)(size_t size, const void *caller);

void *(*volatile __realloc_hook)(void *ptr, size_t size, const void *caller);

void *(*volatile __memalign_hook)(size_t alignment, size_t size,
                         const void *caller);

void (*volatile __free_hook)(void *ptr, const void *caller);

void (*__malloc_initialize_hook)(void);

void (*volatile __after_morecore_hook)(void);
```

## DESCRIPTION

GNU C 라이브러리 사용 시 적절한 훅 함수를 지정해서 <tt>[[malloc(3)]]</tt>, <tt>[[realloc(3)]]</tt>, <tt>[[free(3)]]</tt>의 동작 방식을 바꿀 수 있다. 예를 들어 동적 메모리 할당을 하는 프로그램을 디버깅 하는 데 이 훅들이 도움이 될 수 있다.

변수 `__malloc_initialize_hook`은 malloc 구현에서 초기화를 할 때 딱 한 번 호출되는 함수를 가리킨다. 약한 변수이므로 응용에서 다음과 같은 정의로 오버라이드 할 수 있다.

```c
void (*__malloc_initialize_hook)(void) = my_init_hook;
```

그러면 함수 `my_init_hook()`에서 모든 훅들을 초기화 할 수 있다.

`__malloc_hook`, `__realloc_hook`, `__memalign_hook`, `__free_hook`이 가리키는 네 함수는 마지막 인자 `caller`가 있는 걸 제외하면 각각 함수 <tt>[[malloc(3)]]</tt>, <tt>[[realloc(3)]]</tt>, <tt>[[memalign(3)]]</tt>, <tt>[[free(3)]]</tt>와 원형이 같다. `caller`는 <tt>[[malloc(3)]]</tt> 등의 호출자의 주소이다.

변수 `__after_morecore_hook`은 <tt>[[sbrk(2)]]</tt>에 추가 메모리 요청이 있으면 처리 후 호출되는 함수를 가리킨다.

## CONFORMING TO

이 함수들은 GNU 확장이다.

## NOTES

이 훅 함수들은 다중 스레드 프로그램에서 쓰기에 안전하지 않고, 그래서 현재 제거 예정 상태이다. glibc 2.24부터 API에서 `__malloc_initialize_hook`이 없어졌다. 프로그래머들은 대신 "malloc"이나 "free" 같은 함수를 정의해서 내보이는 방식으로 관련 함수 호출을 선점해야 한다.

## EXAMPLES

다음은 간단한 사용 예시이다.

```c
#include <stdio.h>
#include <malloc.h>

/* 훅 원형 */
static void my_init_hook(void);
static void *my_malloc_hook(size_t, const void *);

/* 원래 훅을 저장할 변수 */
static void *(*old_malloc_hook)(size_t, const void *);

/* C 라이브러리의 초기화 훅 오버라이드 */
void (*__malloc_initialize_hook)(void) = my_init_hook;

static void
my_init_hook(void)
{
    old_malloc_hook = __malloc_hook;
    __malloc_hook = my_malloc_hook;
}

static void *
my_malloc_hook(size_t size, const void *caller)
{
    void *result;

    /* 이전 훅 복원 */
    __malloc_hook = old_malloc_hook;

    /* 재귀적으로 호출 */
    result = malloc(size);

    /* 하위 훅 저장 */
    old_malloc_hook = __malloc_hook;

    /* printf()에서도 malloc()을 호출할 수도 있으므로 보호 필요 */
    printf("malloc(%zu) called from %p returns %p\n",
            size, caller, result);

    /* 우리 훅 복원 */
    __malloc_hook = my_malloc_hook;

    return result;
}
```

## SEE ALSO

<tt>[[mallinfo(3)]]</tt>, <tt>[[malloc(3)]]</tt>, <tt>[[mcheck(3)]]</tt>, <tt>[[mtrace(3)]]</tt>

----

2021-03-22
