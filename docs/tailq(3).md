## NAME

TAILQ_CONCAT, TAILQ_EMPTY, TAILQ_ENTRY, TAILQ_FIRST, TAILQ_FOREACH, TAILQ_FOREACH_REVERSE, TAILQ_HEAD, TAILQ_HEAD_INITIALIZER, TAILQ_INIT, TAILQ_INSERT_AFTER, TAILQ_INSERT_BEFORE, TAILQ_INSERT_HEAD, TAILQ_INSERT_TAIL, TAILQ_LAST, TAILQ_NEXT, TAILQ_PREV, TAILQ_REMOVE - 이중 연결 꼬리 큐 구현

## SYNOPSIS

```c
#include <sys/queue.h>

TAILQ_ENTRY(TYPE);

TAILQ_HEAD(HEADNAME, TYPE);
TAILQ_HEAD TAILQ_HEAD_INITIALIZER(TAILQ_HEAD head);
void TAILQ_INIT(TAILQ_HEAD *head);

int TAILQ_EMPTY(TAILQ_HEAD *head);

void TAILQ_INSERT_HEAD(TAILQ_HEAD *head,
                         struct TYPE *elm, TAILQ_ENTRY NAME);
void TAILQ_INSERT_TAIL(TAILQ_HEAD *head,
                         struct TYPE *elm, TAILQ_ENTRY NAME);
void TAILQ_INSERT_BEFORE(struct TYPE *listelm,
                         struct TYPE *elm, TAILQ_ENTRY NAME);
void TAILQ_INSERT_AFTER(TAILQ_HEAD *head, struct TYPE *listelm,
                         struct TYPE *elm, TAILQ_ENTRY NAME);

struct TYPE *TAILQ_FIRST(TAILQ_HEAD *head);
struct TYPE *TAILQ_LAST(TAILQ_HEAD *head, HEADNAME);
struct TYPE *TAILQ_PREV(struct TYPE *elm, HEADNAME, TAILQ_ENTRY NAME);
struct TYPE *TAILQ_NEXT(struct TYPE *elm, TAILQ_ENTRY NAME);

TAILQ_FOREACH(struct TYPE *var, TAILQ_HEAD *head,
                         TAILQ_ENTRY NAME);
TAILQ_FOREACH_REVERSE(struct TYPE *var, TAILQ_HEAD *head, HEADNAME,
                         TAILQ_ENTRY NAME);

void TAILQ_REMOVE(TAILQ_HEAD *head, struct TYPE *elm,
                         TAILQ_ENTRY NAME);

void TAILQ_CONCAT(TAILQ_HEAD *head1, TAILQ_HEAD *head2,
                         TAILQ_ENTRY NAME);
```

## DESCRIPTION

이 매크로들은 이중 연결 꼬리 큐를 정의하고 조작한다.

매크로 정의에서 `TYPE`은 사용자 정의 구조체의 이름이며, 그 구조체에는 타입이 `TAILQ_ENTRY`이고 이름이 `NAME`인 필드가 있어야 한다. `HEADNAME` 인자는 `TAILQ_HEAD()` 매크로를 써서 선언해야 하는 사용자 정의 구조체의 이름이다.

### 생성

꼬리 큐의 머리는 `TAILQ_HEAD()` 매크로로 정의한 구조체이다. 이 구조체에는 포인터 한 쌍이 있는데, 하나는 큐의 첫 번째 항목을 가리키고 다른 하나는 큐의 마지막 항목을 가리킨다. 항목들이 이중으로 연결되어 있으므로 큐 순회 없이 임의 항목을 제거할 수 있다. 기존 항목 뒤나 기존 항목 앞에, 큐 머리나 큐 끝에 새 항목을 추가할 수 있다. `TAILQ_HEAD` 구조체를 다음처럼 선언한다.

```c
TAILQ_HEAD(HEADNAME, TYPE) head;
```

`struct HEADNAME`은 정의하려는 구조체의 이름이고 `struct TYPE`은 큐로 연결할 항목들의 타입이다. 이후 다음처럼 큐 머리의 포인터를 선언할 수 있다.

```c
struct HEADNAME *headp;
```

(이름 `head`와 `headp`는 사용자가 정할 수 있다.)

`TAILQ_ENTRY()`는 큐에서 항목들을 연결하는 구조체를 선언한다.

`TAILQ_HEAD_INITIALIZER()`는 큐 `head`를 위한 초기화 값으로 평가된다.

`TAILQ_INIT()`은 `head`가 가리키는 큐를 초기화 한다.

`TAILQ_EMPTY()`는 큐에 아무 항목도 없으면 참으로 평가된다.

### 삽입

`TAILQ_INSERT_HEAD()`는 새 항목 `elm`을 큐의 머리에 삽입한다.

`TAILQ_INSERT_TAIL()`은 새 항목 `elm`을 큐의 끝에 삽입한다.

`TAILQ_INSERT_BEFORE()`는 새 항목 `elm`을 항목 `listelm` 앞에 삽입한다.

`TAILQ_INSERT_AFTER()`는 새 항목 `elm`을 항목 `listelm` 뒤에 삽입한다.

### 순회

`TAILQ_FIRST()`는 큐의 첫 번째 항목을 반환하며 큐가 비어 있으면 NULL을 반환한다.

`TAILQ_LAST()`는 큐의 마지막 항목을 반환한다. 큐가 비어 있으면 반환 값이 NULL이다.

`TAILQ_PREV()`는 큐 내의 이전 항목을 반환하며, 현 항목이 첫 번째이면 NULL을 반환한다.

`TAILQ_NEXT()`는 큐 내의 다음 항목을 반환하며, 현 항목이 마지막이면 NULL을 반환한다.

`TAILQ_FOREACH()`는 `head`가 가리키는 큐를 순방향으로 순회하면서 각 항목을 차례로 `var`에 할당한다. 루프가 정상적으로 끝나거나 항목이 없는 경우에 `var`를 NULL로 설정한다.

`TAILQ_FOREACH_REVERSE()`는 `head`가 가리키는 큐를 역방향으로 순회하면서 각 항목을 차례로 `var`에 할당한다.

### 제거

`TAILQ_REMOVE()`는 항목 `elm`을 큐에서 제거한다.

### 기타

`TAILQ_CONCAT()`은 `head2`가 가리키는 큐를 `head1`이 가리키는 큐 끝에 이어 붙이고 `head2`에서 모든 항목을 제거한다.

## RETURN VALUE

`TAILQ_EMPTY()`는 큐가 비어 있으면 0 아닌 값을 반환하고, 큐에 한 항목이라도 있으면 0을 반환한다.

`TAILQ_FIRST()`, `TAILQ_LAST()`, `TAILQ_PREV()`, `TAILQ_NEXT()`는 각각 처음, 마지막, 이전, 다음 `TYPE` 구조체의 포인터를 반환한다.

`TAILQ_HEAD_INITIALIZER()`는 큐 `head`에 할당할 수 있는 초기화 값을 반환한다.

## CONFORMING TO

POSIX.1, POSIX.1-2001, POSIX.1-2008에 없다. BSD들에 있다. (4.4BSD에서 TAILQ 매크로들이 처음 등장했다.)

## BUGS

`TAILQ_FOREACH()` 및 `TAILQ_FOREACH_REVERSE()`에서 `var`를 루프 안에서 제거하거나 해제하는 것이 허용되지 않는다. 순회를 방해하게 되기 때문이다. BSD들에 있지만 glibc에는 없는 `TAILQ_FOREACH_SAFE()` 및 `TAILQ_FOREACH_REVERSE_SAFE()`에서 이 제약을 수정해서, 순회를 방해하지 않고 루프 안에서 `var`를 안전하게 리스트에서 제거하고 해제하는 것이 허용된다.

## EXAMPLES

```c
#include <stddef.h>
#include <stdio.h>
#include <stdlib.h>
#include <sys/queue.h>

struct entry {
    int data;
    TAILQ_ENTRY(entry) entries;             /* 꼬리 큐 */
};

TAILQ_HEAD(tailhead, entry);

int
main(void)
{
    struct entry *n1, *n2, *n3, *np;
    struct tailhead head;                   /* 꼬리 큐 머리 */
    int i;

    TAILQ_INIT(&head);                      /* 큐 초기화 */

    n1 = malloc(sizeof(struct entry));      /* 머리에 삽입 */
    TAILQ_INSERT_HEAD(&head, n1, entries);

    n1 = malloc(sizeof(struct entry));      /* 꼬리에 삽입 */
    TAILQ_INSERT_TAIL(&head, n1, entries);

    n2 = malloc(sizeof(struct entry));      /* 바로 뒤에 삽입 */
    TAILQ_INSERT_AFTER(&head, n1, n2, entries);

    n3 = malloc(sizeof(struct entry));      /* 바로 앞에 삽입 */
    TAILQ_INSERT_BEFORE(n2, n3, entries);

    TAILQ_REMOVE(&head, n2, entries);       /* 삭제 */
    free(n2);
                                            /* 순방향 순회 */
    i = 0;
    TAILQ_FOREACH(np, &head, entries)
        np->data = i++;
                                            /* 역방향 순회 */
    TAILQ_FOREACH_REVERSE(np, &head, tailhead, entries)
        printf("%i\n", np->data);
                                            /* 꼬리 큐 삭제 */
    n1 = TAILQ_FIRST(&head);
    while (n1 != NULL) {
        n2 = TAILQ_NEXT(n1, entries);
        free(n1);
        n1 = n2;
    }
    TAILQ_INIT(&head);

    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[insque(3)]]</tt>, <tt>[[queue(7)]]</tt>

----

2021-03-22
