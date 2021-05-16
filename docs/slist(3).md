## NAME

SLIST_EMPTY, SLIST_ENTRY, SLIST_FIRST, SLIST_FOREACH, SLIST_HEAD, SLIST_HEAD_INITIALIZER, SLIST_INIT, SLIST_INSERT_AFTER, SLIST_INSERT_HEAD, SLIST_NEXT, SLIST_REMOVE, SLIST_REMOVE_HEAD - 단일 연결 리스트 구현

## SYNOPSIS

```c
#include <sys/queue.h>

SLIST_ENTRY(TYPE);

SLIST_HEAD(HEADNAME, TYPE);
SLIST_HEAD SLIST_HEAD_INITIALIZER(SLIST_HEAD head);
void SLIST_INIT(SLIST_HEAD *head);

int SLIST_EMPTY(SLIST_HEAD *head);

void SLIST_INSERT_HEAD(SLIST_HEAD *head,
                        struct TYPE *elm, SLIST_ENTRY NAME);
void SLIST_INSERT_AFTER(struct TYPE *listelm,
                        struct TYPE *elm, SLIST_ENTRY NAME);

struct TYPE *SLIST_FIRST(SLIST_HEAD *head);
struct TYPE *SLIST_NEXT(struct TYPE *elm, SLIST_ENTRY NAME);

SLIST_FOREACH(struct TYPE *var, SLIST_HEAD *head, SLIST_ENTRY NAME);

void SLIST_REMOVE(SLIST_HEAD *head, struct TYPE *elm,
                        SLIST_ENTRY NAME);
void SLIST_REMOVE_HEAD(SLIST_HEAD *head,
                        SLIST_ENTRY NAME);
```

## DESCRIPTION

이 매크로들은 단일 연결 리스트를 정의하고 조작한다.

매크로 정의에서 `TYPE`은 사용자 정의 구조체의 이름이며, 그 구조체에는 타입이 `SLIST_ENTRY`이고 이름이 `NAME`인 필드가 있어야 한다. `HEADNAME` 인자는 `SLIST_HEAD()` 매크로를 써서 선언해야 하는 사용자 정의 구조체의 이름이다.

### 생성

단일 연결 리스트의 머리는 `SLIST_HEAD()` 매크로로 정의한 구조체이다. 이 구조체에는 리스트 첫 번째 항목에 대한 포인터 하나가 들어 있다. 공간 및 포인터 조작 오버헤드를 최소화하기 위해 단일 연결로 되어 있으며, 대신 임의 항목 제거에 O(n) 비용이 든다. 기존 항목 뒤나 리스트 머리에 새 항목을 추가할 수 있다. `SLIST_HEAD` 구조체를 다음처럼 선언한다.

```c
SLIST_HEAD(HEADNAME, TYPE) head;
```

`struct HEADNAME`이 정의하려는 구조체의 이름이고 `struct TYPE`이 리스트로 연결할 항목들의 타입이다. 이후 다음처럼 리스트 머리의 포인터를 선언할 수 있다.

```c
struct HEADNAME *headp;
```

(이름 `head`와 `headp`는 사용자가 정할 수 있다.)

`SLIST_ENTRY()`는 리스트에서 항목들을 연결하는 구조체를 선언한다.

`SLIST_HEAD_INITIALIZER()`는 리스트 `head`를 위한 초기화 값으로 평가된다.

`SLIST_INIT()`은 `head`가 가리키는 리스트를 초기화한다.

`SLIST_EMPTY()`는 리스트에 아무 항목도 없으면 참으로 평가된다.

### 삽입

`SLIST_INSERT_HEAD()`는 새 항목 `elm`을 리스트의 머리에 삽입한다.

`SLIST_INSERT_AFTER()`는 새 항목 `elm`을 항목 `listelm` 뒤에 삽입한다.

### 순회

`SLIST_FIRST()`는 리스트의 첫 번째 항목을 반환하며 리스트가 비어 있으면 NULL을 반환한다.

`SLIST_NEXT()`는 리스트 내의 다음 항목을 반환한다.

`SLIST_FOREACH()`는 `head`가 가리키는 리스트를 순방향으로 순회하면서 각 항목을 차례로 `var`에 할당한다.

### 제거

`SLIST_REMOVE()`는 항목 `elm`을 리스트에서 제거한다.

`SLIST_REMOVE_HEAD()`는 항목 `elm`을 리스트의 머리에서 제거한다. 최고의 효율성을 위해선 리스트의 머리에서 항목을 제거할 때 범용 `SLIST_REMOVE()` 매크로 대신 명시적으로 이 매크로를 쓰는 게 좋다.

## RETURN VALUE

`SLIST_EMPTY()`는 리스트가 비어 있으면 0 아닌 값을 반환하고, 리스트에 한 항목이라도 있으면 0을 반환한다.

`SLIST_FIRST()`와 `SLIST_NEXT()`는 각각 처음 또는 다음 `TYPE` 구조체의 포인터를 반환한다.

`SLIST_HEAD_INITIALIZER()`는 리스트 `head`에 할당할 수 있는 초기화 값을 반환한다.

## CONFORMING TO

POSIX.1, POSIX.1-2001, POSIX.1-2008에 없다. BSD들에 있다. (4.4BSD에서 SLIST 매크로들이 처음 등장했다.)

## BUGS

`SLIST_FOREACH()`에서 `var`를 루프 안에서 제거하거나 해제하는 것이 허용되지 않는다. 순회를 방해하게 되기 때문이다. BSD들에 있지만 glibc에는 없는 `SLIST_FOREACH_SAFE()`에서 이 제약을 수정해서, 순회를 방해하지 않고 루프 안에서 `var`를 안전하게 리스트에서 제거하고 해제하는 것이 허용된다.

### 단일 연결 리스트 예시

```c
#include <stddef.h>
#include <stdio.h>
#include <stdlib.h>
#include <sys/queue.h>

struct entry {
    int data;
    SLIST_ENTRY(entry) entries;             /* 단일 연결 리스트 */
};

SLIST_HEAD(slisthead, entry);

int
main(void)
{
    struct entry *n1, *n2, *n3, *np;
    struct slisthead head;                  /* 단일 연결 리스트 머리 */

    SLIST_INIT(&head);                      /* 리스트 초기화 */

    n1 = malloc(sizeof(struct entry));      /* 머리에 삽입 */
    SLIST_INSERT_HEAD(&head, n1, entries);

    n2 = malloc(sizeof(struct entry));      /* 바로 뒤에 삽입 */
    SLIST_INSERT_AFTER(n1, n2, entries);

    SLIST_REMOVE(&head, n2, entry, entries);/* 삭제 */
    free(n2);

    n3 = SLIST_FIRST(&head);
    SLIST_REMOVE_HEAD(&head, entries);      /* 머리에서 삭제 */
    free(n3);

    for (int i = 0; i < 5; i++) {
        n1 = malloc(sizeof(struct entry));
        SLIST_INSERT_HEAD(&head, n1, entries);
        n1->data = i;
    }

                                            /* 순방향 순회 */
    SLIST_FOREACH(np, &head, entries)
        printf("%i\n", np->data);

    while (!SLIST_EMPTY(&head)) {           /* 리스트 삭제 */
        n1 = SLIST_FIRST(&head);
        SLIST_REMOVE_HEAD(&head, entries);
        free(n1);
    }
    SLIST_INIT(&head);

    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[insque(3)]]</tt>, <tt>[[queue(7)]]</tt>

----

2021-03-22
