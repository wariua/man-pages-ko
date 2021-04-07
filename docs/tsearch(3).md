## NAME

tsearch, tfind, tdelete, twalk, tdestroy - 이진 탐색 트리 관리하기

## SYNOPSIS

```c
#include <search.h>

typedef enum { preorder, postorder, endorder, leaf } VISIT;

void *tsearch(const void *key, void **rootp,
                int (*compar)(const void *, const void *));

void *tfind(const void *key, void *const *rootp,
                int (*compar)(const void *, const void *));

void *tdelete(const void *key, void **rootp,
                int (*compar)(const void *, const void *));

void twalk(const void *root,
                void (*action)(const void *nodep, VISIT which,
                               int depth));

#define _GNU_SOURCE         /* feature_test_macros(7) 참고 */
#include <search.h>

void twalk_r(const void *root,
                void (*action)(const void *nodep, VISIT which,
                               void *closure),
                void *closure);

void tdestroy(void *root, void (*free_node)(void *nodep));
```

## DESCRIPTION

`tsearch()`, `tfind()`, `twalk()`, `tdelete()`는 이진 탐색 트리를 관리한다. Knuth의 (6.2.2) 알고리듬 T를 일반화 한 것이다. 각 트리 노드의 첫 필드가 대응하는 데이터 항목에 대한 포인터이다. (실제 데이터는 호출 프로그램에서 저장해야 한다.) `compar`는 비교 루틴을 가리키며 두 항목에 대한 포인터를 받는다. 첫 번째 항목이 두 번째 항목보다 작거나 같거나 크면 음수이거나 0이거나 양수인 정수를 반환해야 한다.

`tsearch()`는 트리에서 항목을 탐색한다. `key`는 찾을 항목을 가리킨다. `rootp`는 트리 루트를 가리키는 변수를 가리킨다. 트리가 비어 있으면 `rootp`가 가리키는 변수가 NULL로 설정돼 있어야 한다. 트리에서 항목을 찾은 경우에 `tsearch()`는 대응하는 트리 노드에 대한 포인터를 반환한다. (달리 말해 데이터 항목에 대한 포인터에 대한 포인터를 반환한다.) 항목을 찾지 못하면 항목을 추가하고서 대응하는 트리 노드에 대한 포인터를 반환한다.

`tfind()`는 `tsearch()`와 비슷하되 항목을 찾지 못한 경우에 NULL을 반환한다.

`tdelete()`는 트리에서 항목을 삭제한다. 인자들은 `tsearch()`와 동일하다.

`twalk()`는 깊이 우선으로 왼쪽에서 오른쪽으로 이진 트리 순회를 수행한다. `root`는 순회 시작 노드를 가리킨다. 그 노드가 루트가 아니면 트리의 일부만 방문하게 된다. `twalk()`에서 노드를 방문할 때마다 (즉 내부 노드는 세 번씩, 말단 노드는 한 번씩) 사용자 함수 `action`을 호출한다. 그리고 `action`은 인자 세 개를 받는다. 첫 번째 인자는 방문 중인 노드에 대한 포인터이다. 노드의 구조는 명세돼 있지 않지만 그 포인터를 항목 포인터에 대한 포인터로 캐스트 해서 노드에 저장된 항목에 접근하는 게 가능하다. 이 인자가 가리키는 자료 구조를 응용에서 변경해선 안 된다. 두 번째 인자는 정수인데 내부 노드에 대한 첫 번째, 두 번째, 세 번째 방문이면 `preorder`, `postorder`, `endorder` 값을, 말단 노드에 대한 한 번의 방문이면 `leaf` 값을 가진다. (이 심볼들은 `<search.h>`에 정의되어 있다.) 세 번째 인자는 노드의 깊이이다. 루트 노드의 깊이가 0이다.

(`preorder`, `postorder`, `endorder`를 더 흔히 쓰는 말로 하면 `preorder`, `inorder`, `postorder`이다. 즉, 자식 방문 전, 첫 번째 후 두 번째 전, 자식 방문 후이다. `postorder`라는 이름을 써서 좀 헛갈릴 수 있다.)

`twalk_r()`은 `twalk()`와 비슷하되 `depth` 인자 대신 `closure` 인자 포인터가 그대로 action 콜백 호출으로 전달된다. 그 포인터를 이용하면 전역 변수에 기대지 않고 스레드에 안전한 방식으로 콜백 함수와 정보를 주고 받을 수 있다.

`tdestroy()`는 `root`가 가리키는 트리 전체를 없애며 `tsearch()` 함수에서 할당한 자원을 모두 해제한다. 각 트리 노드의 데이터에 대해 `free_node` 함수를 호출한다. 함수 인자로 그 데이터에 대한 포인터가 전달된다. 그런 작업이 필요하지 않은 경우에 `free_node`는 아무것도 하지 않는 함수를 가리켜야 한다.

## RETURN VALUE

`tsearch()`는 트리 내의 일치 노드나 새로 추가한 노드에 대한 포인터를 반환한다. 항목을 추가하기 위한 메모리가 충분하지 않은 경우 NULL을 반환한다. `tfind()`는 노드에 대한 포인터를 반환한다. 일치 항목을 찾지 못한 경우 NULL을 반환한다. 키에 일치하는 항목이 여러 개 있는 경우에 어느 항목의 노드를 반환하는지는 명세돼 있지 않다.

`tdelete()`는 삭제한 노드의 부모에 대한 포인터를 반환한다. 항목을 찾지 못했으면 NULL을 반환한다. 삭제한 노드가 루트 노드이면 깨진 포인터를 반환하며 그 포인터에 접근해서는 안 된다.

`tsearch()`, `tfind()`, `tdelete()`는 또한 진입 시 `rootp`가 NULL이면 NULL을 반환한다.

## VERSIONS

glibc 버전 2.30부터 `tralk_r()`이 사용 가능하다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `tsearch()`, `tfind()`,<br>`tdelete()` | 스레드 안전성 | MT-Safe race:rootp |
| `twalk()` | 스레드 안전성 | MT-Safe race:root |
| `twalk_r()` | 스레드 안전성 | MT-Safe race:root |
| `tdestroy()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, SVr4. `tdestroy()` 및 `twalk_r()` 함수는 GNU 확장이다.

## NOTES

`twalk()`는 루트에 대한 포인터를 받는 반면 다른 함수들은 루트를 가지키는 변수에 대한 포인터를 받는다.

`tdelete()`는 트리에서 노드에 필요한 메모리를 해제한다. 대응하는 데이터를 위한 메모리를 해제하는 것은 사용자의 몫이다.

아래 예시 프로그램에서는 `twalk()`에서 `endorder` 내지 `leaf` 인자로 사용자 함수를 호출한 후에 더는 그 노드를 참조하지 않는다는 점에 기댄다. GNU 라이브러리 구현에서는 동작하지만 시스템 V 문서에서는 그렇지 않다.

## EXAMPLE

다음 프로그램에서는 중복을 없애며 이진 트리에 난수 열두 개를 삽입한 다음 그 수들을 차례로 찍는다.

```c
#define _GNU_SOURCE     /* tdestroy() 선언 드러내기 */
#include <search.h>
#include <stdlib.h>
#include <stdio.h>
#include <time.h>

static void *root = NULL;

static void *
xmalloc(unsigned n)
{
    void *p;
    p = malloc(n);
    if (p)
        return p;
    fprintf(stderr, "insufficient memory\n");
    exit(EXIT_FAILURE);
}

static int
compare(const void *pa, const void *pb)
{
    if (*(int *) pa < *(int *) pb)
        return -1;
    if (*(int *) pa > *(int *) pb)
        return 1;
    return 0;
}

static void
action(const void *nodep, VISIT which, int depth)
{
    int *datap;

    switch (which) {
    case preorder:
        break;
    case postorder:
        datap = *(int **) nodep;
        printf("%6d\n", *datap);
        break;
    case endorder:
        break;
    case leaf:
        datap = *(int **) nodep;
        printf("%6d\n", *datap);
        break;
    }
}

int
main(void)
{
    int i, *ptr;
    void *val;

    srand(time(NULL));
    for (i = 0; i < 12; i++) {
        ptr = xmalloc(sizeof(int));
        *ptr = rand() & 0xff;
        val = tsearch((void *) ptr, &root, compare);
        if (val == NULL)
            exit(EXIT_FAILURE);
        else if ((*(int **) val) != ptr)
            free(ptr);
    }
    twalk(root, action);
    tdestroy(root, free);
    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[bsearch(3)]]</tt>, <tt>[[hsearch(3)]]</tt>, <tt>[[lsearch(3)]]</tt>, `qsort(3)`

----

2019-05-09
