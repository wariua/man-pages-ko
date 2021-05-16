## NAME

CIRCLEQ_EMPTY, CIRCLEQ_ENTRY, CIRCLEQ_FIRST, CIRCLEQ_FOREACH, CIRCLEQ_FOREACH_REVERSE, CIRCLEQ_HEAD, CIRCLEQ_HEAD_INITIALIZER, CIRCLEQ_INIT, CIRCLEQ_INSERT_AFTER, CIRCLEQ_INSERT_BEFORE, CIRCLEQ_INSERT_HEAD, CIRCLEQ_INSERT_TAIL, CIRCLEQ_LAST, CIRCLEQ_LOOP_NEXT, CIRCLEQ_LOOP_PREV, CIRCLEQ_NEXT, CIRCLEQ_PREV, CIRCLEQ_REMOVE - 이중 연결 원형 큐 구현

## SYNOPSIS

```c
#include <sys/queue.h>

CIRCLEQ_ENTRY(TYPE);

CIRCLEQ_HEAD(HEADNAME, TYPE);
CIRCLEQ_HEAD CIRCLEQ_HEAD_INITIALIZER(CIRCLEQ_HEAD head);
void CIRCLEQ_INIT(CIRCLEQ_HEAD *head);

int CIRCLEQ_EMPTY(CIRCLEQ_HEAD *head);

void CIRCLEQ_INSERT_HEAD(CIRCLEQ_HEAD *head,
                           struct TYPE *elm, CIRCLEQ_ENTRY NAME);
void CIRCLEQ_INSERT_TAIL(CIRCLEQ_HEAD *head,
                           struct TYPE *elm, CIRCLEQ_ENTRY NAME);
void CIRCLEQ_INSERT_BEFORE(CIRCLEQ_HEAD *head, struct TYPE *listelm,
                           struct TYPE *elm, CIRCLEQ_ENTRY NAME);
void CIRCLEQ_INSERT_AFTER(CIRCLEQ_HEAD *head, struct TYPE *listelm,
                           struct TYPE *elm, CIRCLEQ_ENTRY NAME);

struct TYPE *CIRCLEQ_FIRST(CIRCLEQ_HEAD *head);
struct TYPE *CIRCLEQ_LAST(CIRCLEQ_HEAD *head);
struct TYPE *CIRCLEQ_PREV(struct TYPE *elm, CIRCLEQ_ENTRY NAME);
struct TYPE *CIRCLEQ_NEXT(struct TYPE *elm, CIRCLEQ_ENTRY NAME);
struct TYPE *CIRCLEQ_LOOP_PREV(CIRCLEQ_HEAD *head,
                           struct TYPE *elm, CIRCLEQ_ENTRY NAME);
struct TYPE *CIRCLEQ_LOOP_NEXT(CIRCLEQ_HEAD *head,
                           struct TYPE *elm, CIRCLEQ_ENTRY NAME);

CIRCLEQ_FOREACH(struct TYPE *var, CIRCLEQ_HEAD *head,
                           CIRCLEQ_ENTRY NAME);
CIRCLEQ_FOREACH_REVERSE(struct TYPE *var, CIRCLEQ_HEAD *head,
                           CIRCLEQ_ENTRY NAME);

void CIRCLEQ_REMOVE(CIRCLEQ_HEAD *head, struct TYPE *elm,
                           CIRCLEQ_ENTRY NAME);
```

## DESCRIPTION

이 매크로들은 이중 연결 원형 큐를 정의하고 조작한다.

매크로 정의에서 `TYPE`은 사용자 정의 구조체의 이름이며, 그 구조체에는 타입이 `CIRCLEQ_ENTRY`이고 이름이 `NAME`인 필드가 있어야 한다. `HEADNAME` 인자는 `CIRCLEQ_HEAD()` 매크로를 써서 선언해야 하는 사용자 정의 구조체의 이름이다.

### 생성

원형 큐의 머리는 `CIRCLEQ_HEAD()` 매크로로 정의한 구조체이다. 이 구조체에는 포인터 한 쌍이 있는데, 하나는 큐의 첫 번째 항목을 가리키고 다른 하나는 큐의 마지막 항목을 가리킨다. 항목들이 이중으로 연결되어 있으므로 큐 순회 없이 임의 항목을 제거할 수 있다. 기존 항목 뒤나 기존 항목 앞에, 큐 머리나 큐 끝에 새 항목을 추가할 수 있다. `CIRCLEQ_HEAD` 구조체를 다음처럼 선언한다.

```c
CIRCLEQ_HEAD(HEADNAME, TYPE) head;
```

`struct HEADNAME`이 정의하려는 구조체의 이름이고 `struct TYPE`이 큐로 연결할 항목들의 타입이다. 이후 다음처럼 큐 머리의 포인터를 선언할 수 있다.

```c
struct HEADNAME *headp;
```

(이름 `head`와 `headp`는 사용자가 정할 수 있다.)

`CIRCLEQ_ENTRY()`는 큐에서 항목들을 연결하는 구조체를 선언한다.

`CIRCLEQ_HEAD_INITIALIZER()`는 큐 `head`를 위한 초기화 값으로 평가된다.

`CIRCLEQ_INIT()`은 `head`가 가리키는 큐를 초기화한다.

`CIRCLEQ_EMPTY()`는 큐에 아무 항목도 없으면 참으로 평가된다.

### 삽입

`CIRCLEQ_INSERT_HEAD()`는 새 항목 `elm`을 큐의 머리에 삽입한다.

`CIRCLEQ_INSERT_TAIL()`는 새 항목 `elm`을 큐의 끝에 삽입한다.

`CIRCLEQ_INSERT_BEFORE()`는 새 항목 `elm`을 항목 `listelm` 앞에 삽입한다.

`CIRCLEQ_INSERT_AFTER()`는 새 항목 `elm`을 항목 `listelm` 뒤에 삽입한다.

### 순회

`CIRCLEQ_FIRST()`는 큐의 첫 번째 항목을 반환한다.

`CIRCLEQ_LAST()`는 큐의 마지막 항목을 반환한다.

`CIRCLEQ_PREV()`는 큐 내의 이전 항목을 반환하며, 현 항목이 첫 번째이면 `&head`를 반환한다.

`CIRCLEQ_NEXT()`는 큐 내의 다음 항목을 반환하며, 현 항목이 마지막이면 `&head`를 반환한다.

`CIRCLEQ_LOOP_PREV()`는 큐 내의 이전 항목을 반환한다. `elm`이 큐의 첫 번째 항목이면 마지막 항목을 반환한다.

`CIRCLEQ_LOOP_NEXT()`는 큐 내의 다음 항목을 반환한다. `elm`이 큐의 마지막 항목이면 첫 번째 항목을 반환한다.

`CIRCLEQ_FOREACH()`는 `head`가 가리키는 큐를 순방향으로 순회하면서 각 항목을 차례로 `var`에 할당한다. 루프가 정상적으로 끝나거나 항목이 없는 경우에 `var`를 `&head`로 설정한다.

`CIRCLEQ_FOREACH_REVERSE()`는 `head`가 가리키는 큐를 역방향으로 순회하면서 각 항목을 차례로 `var`에 할당한다.

### 제거

`CIRCLEQ_REMOVE()`는 항목 `elm`을 큐에서 제거한다.

## RETURN VALUE

`CIRCLEQ_EMPTY()`는 큐가 비어 있으면 0 아닌 값을 반환하고, 큐에 한 항목이라도 있으면 0을 반환한다.

`CIRCLEQ_FIRST()`, `CIRCLEQ_LAST()`, `CIRCLEQ_LOOP_PREV()`, `CIRCLEQ_LOOP_NEXT()`는 각각 처음, 마지막, 이전, 다음 `TYPE` 구조체의 포인터를 반환한다.

`CIRCLEQ_PREV()`와 `CIRCLEQ_NEXT()`는 `CIRCLEQ_LOOP_*()` 짝과 비슷하되, 인자가 각각 첫 번째 또는 마지막 항목이면 `&head`를 반환한다.

`CIRCLEQ_HEAD_INITIALIZER()`는 큐 `head`에 할당할 수 있는 초기화 값을 반환한다.

## CONFORMING TO

POSIX.1, POSIX.1-2001, POSIX.1-2008에 없다. BSD들에 있다. (4.4BSD에서 CIRCLEQ 매크로들이 처음 등장했다.)

## BUGS

`CIRCLEQ_FOREACH()` 및 `CIRCLEQ_FOREACH_REVERSE()`에서 `var`를 루프 안에서 제거하거나 해제하는 것이 허용되지 않는다. 순회를 방해하게 되기 때문이다. BSD들에 있지만 glibc에는 없는 `CIRCLEQ_FOREACH_SAFE()` 및 `CIRCLEQ_FOREACH_REVERSE_SAFE()`에서 이 제약을 수정해서, 순회를 방해하지 않고 루프 안에서 `var`를 안전하게 리스트에서 제거하고 해제하는 것이 허용된다.

## EXAMPLES

```c
#include <stddef.h>
#include <stdio.h>
#include <stdlib.h>
#include <sys/queue.h>

struct entry {
    int data;
    CIRCLEQ_ENTRY(entry) entries;           /* 큐 */
};

CIRCLEQ_HEAD(circlehead, entry);

int
main(void)
{
    struct entry *n1, *n2, *n3, *np;
    struct circlehead head;                 /* 큐 머리 */
    int i;

    CIRCLEQ_INIT(&head);                    /* 큐 초기화 */

    n1 = malloc(sizeof(struct entry));      /* 머리에 삽입 */
    CIRCLEQ_INSERT_HEAD(&head, n1, entries);

    n1 = malloc(sizeof(struct entry));      /* 꼬리에 삽입 */
    CIRCLEQ_INSERT_TAIL(&head, n1, entries);

    n2 = malloc(sizeof(struct entry));      /* 바로 뒤에 삽입 */
    CIRCLEQ_INSERT_AFTER(&head, n1, n2, entries);

    n3 = malloc(sizeof(struct entry));      /* 바로 앞에 삽입 */
    CIRCLEQ_INSERT_BEFORE(&head, n2, n3, entries);

    CIRCLEQ_REMOVE(&head, n2, entries);     /* 삭제 */
    free(n2);
                                            /* 순방향 순회 */
    i = 0;
    CIRCLEQ_FOREACH(np, &head, entries)
        np->data = i++;
                                            /* 역방향 순회 */
    CIRCLEQ_FOREACH_REVERSE(np, &head, entries)
        printf("%i\n", np->data);
                                            /* 큐 삭제 */
    n1 = CIRCLEQ_FIRST(&head);
    while (n1 != (void *)&head) {
        n2 = CIRCLEQ_NEXT(n1, entries);
        free(n1);
        n1 = n2;
    }
    CIRCLEQ_INIT(&head);

    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[insque(3)]]</tt>, <tt>[[queue(7)]]</tt>

----

2021-03-22
