## NAME

mallinfo, mallinfo2 - 메모리 할당 정보 얻기

## SYNOPSIS

```c
#include <malloc.h>

struct mallinfo mallinfo(void);
struct mallinfo2 mallinfo2(void);
```

## DESCRIPTION

이 함수들은 <tt>[[malloc(3)]]</tt> 및 관련 함수들이 수행하는 메모리 할당에 대한 정보를 담은 구조체 사본을 반환한다. 각 함수가 반환하는 구조체에는 같은 필드들이 들어 있다. 하지만 구식 함수인 `mallinfo()`는 필드에 쓰인 타입이 너무 작기 때문에 제거 예정이다. (BUGS 참고.)

이 함수들에서 모든 할당이 보이는 건 아니라는 점에 유의하라. BUGS를 참고하고 <tt>[[malloc_info(3)]]</tt> 사용을 검토해 보라.

`mallinfo2()`가 반환하는 `mallinfo2` 구조체는 다음처럼 정의돼 있다.

```c
struct mallinfo2 {
    size_t arena;     /* 비 mmap 할당 공간 (바이트) */
    size_t ordblks;   /* 유휴 청크 수 */
    size_t smblks;    /* 유휴 패스트빈 블록 수 */
    size_t hblks;     /* mmap 영역 수 */
    size_t hblkhd;    /* mmap 영역들의 할당 공간 (바이트) */
    size_t usmblks;   /* 총 할당 공간 최대치 (바이트) */
    size_t fsmblks;   /* 유휴 패스트빈 블록들의 공간 (바이트) */
    size_t uordblks;  /* 총 할당 공간 (바이트) */
    size_t fordblks;  /* 총 유휴 공간 (바이트) */
    size_t keepcost;  /* 최상단의 해제 가능 공간 (바이트) */
};
```

구식 `mallinfo()` 함수에서 반환하는 `mallinfo` 구조체는 필드들의 타입이 `int`라는 점을 빼고 위와 동일하다.

구조체의 필드들은 다음 정보를 담고 있다.

`arena`
:   <tt>[[mmap(2)]]</tt> 외의 방법으로 할당된 메모리의 (즉 힙에 할당된 메모리의) 총량. 이 수치에는 사용 중인 블록과 유휴 목록의 블록이 모두 포함된다.

`ordblks`
:   평범한 (즉 패스트빈 아닌) 유휴 블록 개수.

`smblks`
:   패스트빈 유휴 블록 개수. (<tt>[[mallopt(3)]]</tt> 참고.)

`hblks`
:   현재 <tt>[[mmap(2)]]</tt>으로 할당된 블록 개수. (<tt>[[mallopt(3)]]</tt>의 `M_MMAP_THRESHOLD` 설명 참고.)

`hblkhd`
:   현재 <tt>[[mmap(2)]]</tt>으로 할당된 블록들의 바이트 수.

`usmblks`
:   이 필드는 쓰지 않으며 항상 0이다. 역사적으로는 할당 공간의 "최고 수위 선", 즉 지금까지 할당했던 공간의 (바이트 단위) 최대치였다. 스레드를 쓰지 않는 환경에서만 이 필드를 유지했다.

`fsmblks`
:   패스트빈 유휴 블록들의 총 바이트 수.

`uordblks`
:   사용 중인 할당들에 쓰는 총 바이트 수.

`fordblks`
:   유휴 블록들의 총 바이트 수.

`keepcost`
:   힙 상단의 해제 가능한 유휴 공간의 총량. 다른 제약이 없다면 (즉 페이지 정렬 제약 등을 무시할 때) <tt>[[malloc_trim(3)]]</tt>으로 해제 가능한 최대 바이트 수이다.

## VERSIONS

glibc 2.33에서 `mallinfo2()` 함수가 추가되었다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `mallinfo()`, `mallinfo2()` | 스레드 안전성 | MT-Unsafe init const:mallopt |

`mallinfo()`/`mallinfo2()`는 몇몇 전역 내부 객체들에 접근하게 된다. 비원자적 변경 시 모순적인 결과를 얻을 수도 있다. `const:mallopt`의 식별자 `mallopt`가 뜻하는 것은 `mallopt()`는 전역 내부 객체를 원자적으로 변경하므로 `mallinfo()`/`mallinfo2()`가 충분히 안전할 것이고, 비원자적으로 변경하는 다른 함수들은 그렇지 않을 수 있다는 것이다.

## CONFORMING TO

이 함수들은 POSIX나 C 표준에 명세되어 있지 않다. 여러 시스템 V 파생 시스템에 `mallinfo()` 함수가 존재하며 SVID에 명세되었다.

## BUGS

**주 메모리 할당 영역에 대한 정보만 반환한다.** 다른 아레나에서의 할당은 빠져 있다. 다른 아레나에 대한 정보를 포함하는 대안으로는 <tt>[[malloc_stats(3)]]</tt>와 <tt>[[malloc_info(3)]]</tt>를 보라.

구식 `mallinfo()` 함수가 반환하는 `mallinfo` 구조체의 필드들이 `int` 타입으로 되어 있다. 하지만 내부의 몇몇 상태 관리 값들이 `long` 타입일 수도 있기 때문에 얻어 온 값이 잘려서 부정확할 수도 있다.

## EXAMPLES

아래 프로그램에서는 `mallinfo2()`를 이용해 할당 전과 후, 그리고 일부 메모리 블록을 해제한 후에 메모리 할당 통계를 얻어 온다. 그 통계를 표준 출력으로 표시한다.

처음 두 개의 명령행 인자는 <tt>[[malloc(3)]]</tt>으로 할당할 블록의 수와 크기를 지정한다.

나머지 세 인자는 할당 블록들 중 어느 것을 <tt>[[free(3)]]</tt>로 해제할지 지정한다. 이 세 인자는 선택적이며, (차례대로) 블록 해제 루프 내에서 쓸 단계 크기 (기본은 1, 즉 범위 내 모든 블록을 해제), 해제할 첫 블록의 순번 (기본은 0, 즉 첫 번째 할당 블록), 해제할 마지막 블록의 순번에 1을 더한 것 (기본은 최대 블록 번호에 1을 더한 것)이다. 이 세 인자를 생략한 경우의 기본값은 모든 할당 블록들을 해제하게 한다.

다음 프로그램 실행 예에서는 100바이트 할당을 1000번 수행하고서 할당 블록 두 개마다 하나씩 해제한다.

```text
$ ./a.out 1000 100 2
============== Before allocating blocks ==============
Total non-mmapped bytes (arena):       0
# of free chunks (ordblks):            1
# of free fastbin blocks (smblks):     0
# of mapped regions (hblks):           0
Bytes in mapped regions (hblkhd):      0
Max. total allocated space (usmblks):  0
Free bytes held in fastbins (fsmblks): 0
Total allocated space (uordblks):      0
Total free space (fordblks):           0
Topmost releasable block (keepcost):   0

============== After allocating blocks ==============
Total non-mmapped bytes (arena):       135168
# of free chunks (ordblks):            1
# of free fastbin blocks (smblks):     0
# of mapped regions (hblks):           0
Bytes in mapped regions (hblkhd):      0
Max. total allocated space (usmblks):  0
Free bytes held in fastbins (fsmblks): 0
Total allocated space (uordblks):      104000
Total free space (fordblks):           31168
Topmost releasable block (keepcost):   31168

============== After freeing blocks ==============
Total non-mmapped bytes (arena):       135168
# of free chunks (ordblks):            501
# of free fastbin blocks (smblks):     0
# of mapped regions (hblks):           0
Bytes in mapped regions (hblkhd):      0
Max. total allocated space (usmblks):  0
Free bytes held in fastbins (fsmblks): 0
Total allocated space (uordblks):      52000
Total free space (fordblks):           83168
Topmost releasable block (keepcost):   31168
```

### 프로그램 소스

```c
#include <malloc.h>
#include <stdlib.h>
#include <string.h>

static void
display_mallinfo2(void)
{
    struct mallinfo2 mi;

    mi = mallinfo2();

    printf("Total non-mmapped bytes (arena):       %zu\n", mi.arena);
    printf("# of free chunks (ordblks):            %zu\n", mi.ordblks);
    printf("# of free fastbin blocks (smblks):     %zu\n", mi.smblks);
    printf("# of mapped regions (hblks):           %zu\n", mi.hblks);
    printf("Bytes in mapped regions (hblkhd):      %zu\n", mi.hblkhd);
    printf("Max. total allocated space (usmblks):  %zu\n", mi.usmblks);
    printf("Free bytes held in fastbins (fsmblks): %zu\n", mi.fsmblks);
    printf("Total allocated space (uordblks):      %zu\n", mi.uordblks);
    printf("Total free space (fordblks):           %zu\n", mi.fordblks);
    printf("Topmost releasable block (keepcost):   %zu\n", mi.keepcost);
}

int
main(int argc, char *argv[])
{
#define MAX_ALLOCS 2000000
    char *alloc[MAX_ALLOCS];
    int numBlocks, freeBegin, freeEnd, freeStep;
    size_t blockSize;

    if (argc < 3 || strcmp(argv[1], "--help") == 0) {
        fprintf(stderr, "%s num-blocks block-size [free-step "
                "[start-free [end-free]]]\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    numBlocks = atoi(argv[1]);
    blockSize = atoi(argv[2]);
    freeStep = (argc > 3) ? atoi(argv[3]) : 1;
    freeBegin = (argc > 4) ? atoi(argv[4]) : 0;
    freeEnd = (argc > 5) ? atoi(argv[5]) : numBlocks;

    printf("============== Before allocating blocks ==============\n");
    display_mallinfo2();

    for (int j = 0; j < numBlocks; j++) {
        if (numBlocks >= MAX_ALLOCS) {
            fprintf(stderr, "Too many allocations\n");
            exit(EXIT_FAILURE);
        }

        alloc[j] = malloc(blockSize);
        if (alloc[j] == NULL) {
            perror("malloc");
            exit(EXIT_FAILURE);
        }
    }

    printf("\n============== After allocating blocks ==============\n");
    display_mallinfo2();

    for (int j = freeBegin; j < freeEnd; j += freeStep)
        free(alloc[j]);

    printf("\n============== After freeing blocks ==============\n");
    display_mallinfo2();

    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[mmap(2)]]</tt>, <tt>[[malloc(3)]]</tt>, <tt>[[malloc_info(3)]]</tt>, <tt>[[malloc_stats(3)]]</tt>, <tt>[[malloc_trim(3)]]</tt>, <tt>[[mallopt(3)]]</tt>

----

2021-03-22
