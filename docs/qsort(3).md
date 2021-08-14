## NAME

qsort, qsort_r - 배열 정렬하기

## SYNOPSIS

```c
#include <stdlib.h>

void qsort(void *base, size_t nmemb, size_t size,
           int (*compar)(const void *, const void *));
void qsort_r(void *base, size_t nmemb, size_t size,
           int (*compar)(const void *, const void *, void *),
           void *arg);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`qsort_r()`:
:   `_GNU_SOURCE`

## DESCRIPTION

`qsort()` 함수는 크기가 `size`인 항목이 `nmemb` 개 있는 배열을 정렬한다. `base` 인자가 배열 시작점을 가리킨다.

`compar`가 가리키는 비교 함수에 따라서 배열 내용물이 오름차순으로 정렬된다. 비교할 객체들을 가리키는 두 인자를 가지고 비교 함수가 호출된다.

비교 함수는 첫 번째 인자가 두 번째 인자보다 작거나 같거나 크다고 볼 때 각각 0보다 작거나 같거나 큰 정수를 반환해야 한다. 두 항목이 같게 비교되는 경우 정렬된 배열에서의 순서는 규정돼 있지 않다.

`qsort_r()` 함수는 비교 함수 `compar`가 세 번째 인자를 받는다는 점을 빼면 `qsort()`와 똑같다. `arg`를 통해 포인터를 비교 함수로 전달할 수 있다. 이렇게 하면 원하는 인자를 전달하기 위해 비교 함수에서 전역 변수를 쓸 필요가 없으며, 그래서 재진입 가능이고 스레드에서 안전하게 쓸 수 있다.

## RETURN VALUE

`qsort()`와 `qsort_r()` 함수는 아무 값도 반환하지 않는다.

## VERSIONS

glibc 버전 2.8에서 `qsort_r()`이 추가되었다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `qsort()`, `qsort_r()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

`qsort()`: POSIX.1-2001, POSIX.1-2008, C89, C99, SVr4, 4.3BSD.

## NOTES

C 문자열을 비교하려면 아래 예처럼 비교 함수에서 <tt>[[strcmp(3)]]</tt>를 호출할 수 있다.

## EXAMPLES

<tt>[[bsearch(3)]]</tt>에서 사용례를 볼 수 있다.

또 다른 예가 다음 프로그램인데, 명령행 인자로 받은 문자열들을 정렬한다.

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

static int
cmpstringp(const void *p1, const void *p2)
{
    /* 이 함수의 실제 인자는 "char 포인터에 대한 포인터"지만
       strcmp(3)의 인자는 "char 포인터"다. 따라서 다음처럼
       캐스팅 및 따라가기를 한다. */

    return strcmp(*(const char **) p1, *(const char **) p2);
}

int
main(int argc, char *argv[])
{
    if (argc < 2) {
        fprintf(stderr, "Usage: %s <string>...\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    qsort(&argv[1], argc - 1, sizeof(char *), cmpstringp);

    for (int j = 1; j < argc; j++)
        puts(argv[j]);
    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

`sort(1)`, <tt>[[alphasort(3)]]</tt>, <tt>[[strcmp(3)]]</tt>, <tt>[[versionsort(3)]]</tt>

----

2021-03-22
