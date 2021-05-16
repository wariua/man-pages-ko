## NAME

SIMPLEQ_EMPTY, SIMPLEQ_ENTRY, SIMPLEQ_FIRST, SIMPLEQ_FOREACH, SIMPLEQ_HEAD, SIMPLEQ_HEAD_INITIALIZER, SIMPLEQ_INIT, SIMPLEQ_INSERT_AFTER, SIMPLEQ_INSERT_HEAD, SIMPLEQ_INSERT_TAIL, SIMPLEQ_NEXT, SIMPLEQ_REMOVE, SIMPLEQ_REMOVE_HEAD, STAILQ_CONCAT, STAILQ_EMPTY, STAILQ_ENTRY, STAILQ_FIRST, STAILQ_FOREACH, STAILQ_HEAD, STAILQ_HEAD_INITIALIZER, STAILQ_INIT, STAILQ_INSERT_AFTER, STAILQ_INSERT_HEAD, STAILQ_INSERT_TAIL, STAILQ_NEXT, STAILQ_REMOVE, STAILQ_REMOVE_HEAD - 단일 연결 꼬리 큐 구현

## SYNOPSIS

```c
#include <sys/queue.h>

STAILQ_ENTRY(TYPE);

STAILQ_HEAD(HEADNAME, TYPE);
STAILQ_HEAD STAILQ_HEAD_INITIALIZER(STAILQ_HEAD head);
void STAILQ_INIT(STAILQ_HEAD *head);

int STAILQ_EMPTY(STAILQ_HEAD *head);

void STAILQ_INSERT_HEAD(STAILQ_HEAD *head,
                         struct TYPE *elm, STAILQ_ENTRY NAME);
void STAILQ_INSERT_TAIL(STAILQ_HEAD *head,
                         struct TYPE *elm, STAILQ_ENTRY NAME);
void STAILQ_INSERT_AFTER(STAILQ_HEAD *head, struct TYPE *listelm,
                         struct TYPE *elm, STAILQ_ENTRY NAME);

struct TYPE *STAILQ_FIRST(STAILQ_HEAD *head);
struct TYPE *STAILQ_NEXT(struct TYPE *elm, STAILQ_ENTRY NAME);

STAILQ_FOREACH(struct TYPE *var, STAILQ_HEAD *head, STAILQ_ENTRY NAME);

void STAILQ_REMOVE(STAILQ_HEAD *head, struct TYPE *elm, TYPE,
                         STAILQ_ENTRY NAME);
void STAILQ_REMOVE_HEAD(STAILQ_HEAD *head,
                         STAILQ_ENTRY NAME);

void STAILQ_CONCAT(STAILQ_HEAD *head1, STAILQ_HEAD *head2);
```

*주의*: 앞에 STAILQ 대신 SIMPLEQ가 붙은 동일한 매크로들이 존재한다. NOTES 참고.

## DESCRIPTION

이 매크로들은 단일 연결 꼬리 큐를 정의하고 조작한다.

매크로 정의에서 `TYPE`은 사용자 정의 구조체의 이름이며, 그 구조체에는 타입이 `STAILQ_ENTRY`이고 이름이 `NAME`인 필드가 있어야 한다. `HEADNAME` 인자는 `STAILQ_HEAD()` 매크로를 써서 선언해야 하는 사용자 정의 구조체의 이름이다.

### 생성

단일 연결 꼬리 큐의 머리는 `STAILQ_HEAD()` 매크로로 정의한 구조체이다. 이 구조체에는 포인터 한 쌍이 있는데, 하나는 꼬리 큐의 첫 번째 항목을 가리키고 다른 하나는 꼬리 큐의 마지막 항목을 가리킨다. 공간 및 포인터 조작 오버헤드를 최소화하기 위해 단일 연결로 되어 있으며, 대신 임의 항목 제거에 O(n) 비용이 든다. 기존 항목 뒤에, 또는 꼬리 큐 머리나 꼬리 큐 끝에 새 항목을 추가할 수 있다. `STAILQ_HEAD` 구조체를 다음처럼 선언한다.

```c
STAILQ_HEAD(HEADNAME, TYPE) head;
```

`struct HEADNAME`이 정의하려는 구조체의 이름이고 `struct TYPE`이 꼬리 큐로 연결할 항목들의 타입이다. 이후 다음처럼 꼬리 큐 머리의 포인터를 선언할 수 있다.

```c
struct HEADNAME *headp;
```

(이름 `head`와 `headp`는 사용자가 정할 수 있다.)

`STAILQ_ENTRY()`는 꼬리 큐에서 항목들을 연결하는 구조체를 선언한다.

`STAILQ_HEAD_INITIALIZER()`는 꼬리 큐 `head`를 위한 초기화 값으로 평가된다.

`STAILQ_INIT()`은 `head`가 가리키는 꼬리 큐를 초기화한다.

`STAILQ_EMPTY()`는 꼬리 큐에 아무 항목도 없으면 참으로 평가된다.

### 삽입

`STAILQ_INSERT_HEAD()`는 새 항목 `elm`을 꼬리 큐의 머리에 삽입한다.

`STAILQ_INSERT_TAIL()`은 새 항목 `elm`을 꼬리 큐의 끝에 삽입한다.

`STAILQ_INSERT_AFTER()`는 새 항목 `elm`을 항목 `listelm` 뒤에 삽입한다.

### 순회

`STAILQ_FIRST()`는 꼬리 큐의 첫 번째 항목을 반환하며 꼬리 큐가 비어 있으면 NULL을 반환한다.

`STAILQ_NEXT()`는 꼬리 큐 내의 다음 항목을 반환하며, 현 항목이 마지막이면 NULL을 반환한다.

`STAILQ_FOREACH()`는 `head`가 가리키는 꼬리 큐를 순방향으로 순회하면서 각 항목을 차례로 `var`에 할당한다.

### 제거

`STAILQ_REMOVE()`는 항목 `elm`을 꼬리 큐에서 제거한다.

`STAILQ_REMOVE_HEAD()`는 꼬리 큐의 머리에 있는 항목을 제거한다. 최고의 효율성을 위해선 꼬리 큐의 머리에서 항목을 제거할 때 범용 `STAILQ_REMOVE()` 매크로 대신 명시적으로 이 매크로를 쓰는 게 좋다.

### 기타

`STAILQ_CONCAT()`은 `head2`가 가리키는 꼬리 큐를 `head1`이 가리키는 꼬리 큐 끝에 이어 붙이고 `head2`에서 모든 항목을 제거한다.

## RETURN VALUE

`STAILQ_EMPTY()`는 큐가 비어 있으면 0 아닌 값을 반환하고, 큐에 한 항목이라도 있으면 0을 반환한다.

`STAILQ_FIRST()`와 `STAILQ_NEXT()`는 각각 처음 또는 다음 `TYPE` 구조체의 포인터를 반환한다.

`STAILQ_HEAD_INITIALIZER()`는 큐 `head`에 할당할 수 있는 초기화 값을 반환한다.

## CONFORMING TO

POSIX.1, POSIX.1-2001, POSIX.1-2008에 없다. BSD들에 있다. (4.4BSD에서 STAILQ 매크로들이 처음 등장했다.)

## BUGS

`STAILQ_FOREACH()`에서 `var`를 루프 안에서 제거하거나 해제하는 것이 허용되지 않는다. 순회를 방해하게 되기 때문이다. BSD들에 있지만 glibc에는 없는 `STAILQ_FOREACH_SAFE()`에서 이 제약을 수정해서, 순회를 방해하지 않고 루프 안에서 `var`를 안전하게 리스트에서 제거하고 해제하는 것이 허용된다.

## NOTES

일부 BSD들에선 STAILQ 대신 SIMPLEQ를 제공한다. 둘은 동일하지만 역사적 이유 때문에 다른 이름이 붙었다. STAILQ는 FreeBSD에서 유래했고 SIMPLEQ는 NetBSD에서 유래했다. 호환성을 위해 어떤 시스템에선 두 매크로 세트를 모두 제공한다. glibc에서는 STAILQ와 SIMPLEQ를 모두 제공하는데, `STAILQ_CONCAT()`에 대응하는 SIMPLEQ 매크로가 없다는 점을 빼고 서로 동일하다.

## EXAMPLES

```c
#include <stddef.h>
#include <stdio.h>
#include <stdlib.h>
#include <sys/queue.h>

struct entry {
    int data;
    STAILQ_ENTRY(entry) entries;        /* 단일 연결 꼬리 큐 */
};

STAILQ_HEAD(stailhead, entry);

int
main(void)
{
    struct entry *n1, *n2, *n3, *np;
    struct stailhead head;                  /* 단일 연결 꼬리 큐 머리 */

    STAILQ_INIT(&head);                     /* 큐 초기화 */

    n1 = malloc(sizeof(struct entry));      /* 머리에 삽입 */
    STAILQ_INSERT_HEAD(&head, n1, entries);

    n1 = malloc(sizeof(struct entry));      /* 꼬리에 삽입 */
    STAILQ_INSERT_TAIL(&head, n1, entries);

    n2 = malloc(sizeof(struct entry));      /* 바로 뒤에 삽입 */
    STAILQ_INSERT_AFTER(&head, n1, n2, entries);

    STAILQ_REMOVE(&head, n2, entry, entries); /* 삭제 */
    free(n2);

    n3 = STAILQ_FIRST(&head);
    STAILQ_REMOVE_HEAD(&head, entries);     /* 머리에서 삭제 */
    free(n3);

    n1 = STAILQ_FIRST(&head);
    n1->data = 0;
    for (int i = 1; i < 5; i++) {
        n1 = malloc(sizeof(struct entry));
        STAILQ_INSERT_HEAD(&head, n1, entries);
        n1->data = i;
    }
                                            /* 순방향 순회 */
    STAILQ_FOREACH(np, &head, entries)
        printf("%i\n", np->data);
                                            /* 꼬리 큐 삭제 */
    n1 = STAILQ_FIRST(&head);
    while (n1 != NULL) {
        n2 = STAILQ_NEXT(n1, entries);
        free(n1);
        n1 = n2;
    }
    STAILQ_INIT(&head);

    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[insque(3)]]</tt>, <tt>[[queue(7)]]</tt>

----

2021-03-22
