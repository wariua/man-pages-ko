## NAME

bsearch - 정렬된 배열 이진 탐색하기

## SYNOPSIS

```c
#include <stdlib.h>

void *bsearch(const void *key, const void *base,
              size_t nmemb, size_t size,
              int (*compar)(const void *, const void *));
```

## DESCRIPTION

`bsearch()` 함수는 `base`가 첫 항목을 가리키는 `nmemb` 개 객체의 배열에서 `key`가 가리키는 객체와 일치하는 항목을 탐색한다. 배열의 각 항목의 크기를 `size`로 지정한다.

배열 내용물이 `compar`가 가리키는 비교 함수를 기준으로 오름차순으로 정렬돼 있어야 한다. `compar` 루틴에는 두 인자가 있어서 차례로 `key` 객체와 배열 항목을 가리키게 돼 있으며, `key` 객체가 배열 항목보다 작거나 같거나 클 때 각각 0보다 작거나 같거나 큰 정수를 반환해야 한다.

## RETURN VALUE

`bsearch()` 함수는 일치하는 배열 항목에 대한 포인터를 반환하며 일치 항목이 없으면 NULL을 반환한다. 키에 일치하는 항목이 여러 개 있는 경우 어느 항목을 반환하는지는 명세돼 있지 않다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `bsearch()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, C89, C99, SVr4, 4.3BSD.

## EXAMPLE

아래 예에서는 먼저 `qsort(3)`를 써서 구조체 배열을 정렬하고서 `bsearch()`를 써서 원하는 항목을 얻는다.

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

struct mi {
    int nr;
    char *name;
} months[] = {
    { 1, "jan" }, { 2, "feb" }, { 3, "mar" }, { 4, "apr" },
    { 5, "may" }, { 6, "jun" }, { 7, "jul" }, { 8, "aug" },
    { 9, "sep" }, {10, "oct" }, {11, "nov" }, {12, "dec" }
};

#define nr_of_months (sizeof(months)/sizeof(months[0]))

static int
compmi(const void *m1, const void *m2)
{
    struct mi *mi1 = (struct mi *) m1;
    struct mi *mi2 = (struct mi *) m2;
    return strcmp(mi1->name, mi2->name);
}

int
main(int argc, char **argv)
{
    int i;

    qsort(months, nr_of_months, sizeof(struct mi), compmi);
    for (i = 1; i < argc; i++) {
        struct mi key, *res;
        key.name = argv[i];
        res = bsearch(&key, months, nr_of_months,
                      sizeof(struct mi), compmi);
        if (ret == NULL)
            printf("'%s': unknown month\n", argv[i]);
        else
            printf("%s: month #%d\n", res->name, res->nr);
    }
    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[hsearch(3)]]</tt>, <tt>[[lsearch(3)]]</tt>, `qsort(3)`, <tt>[[tsearch(3)]]</tt>

----

2017-09-15
