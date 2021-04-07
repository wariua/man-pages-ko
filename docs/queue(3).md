## NAME

SLIST_EMPTY, SLIST_ENTRY, SLIST_FIRST, SLIST_FOREACH, SLIST_HEAD, SLIST_HEAD_INITIALIZER, SLIST_INIT, SLIST_INSERT_AFTER, SLIST_INSERT_HEAD, SLIST_NEXT, SLIST_REMOVE_HEAD, SLIST_REMOVE, STAILQ_CONCAT, STAILQ_EMPTY, STAILQ_ENTRY, STAILQ_FIRST, STAILQ_FOREACH, STAILQ_HEAD, STAILQ_HEAD_INITIALIZER, STAILQ_INIT, STAILQ_INSERT_AFTER, STAILQ_INSERT_HEAD, STAILQ_INSERT_TAIL, STAILQ_NEXT, STAILQ_REMOVE_HEAD, STAILQ_REMOVE, LIST_EMPTY, LIST_ENTRY, LIST_FIRST, LIST_FOREACH, LIST_HEAD, LIST_HEAD_INITIALIZER, LIST_INIT, LIST_INSERT_AFTER, LIST_INSERT_BEFORE, LIST_INSERT_HEAD, LIST_NEXT, LIST_REMOVE, TAILQ_CONCAT, TAILQ_EMPTY, TAILQ_ENTRY, TAILQ_FIRST, TAILQ_FOREACH, TAILQ_FOREACH_REVERSE, TAILQ_HEAD, TAILQ_HEAD_INITIALIZER, TAILQ_INIT, TAILQ_INSERT_AFTER, TAILQ_INSERT_BEFORE, TAILQ_INSERT_HEAD, TAILQ_INSERT_TAIL, TAILQ_LAST, TAILQ_NEXT, TAILQ_PREV, TAILQ_REMOVE, TAILQ_SWAP - 단일 연결 리스트, 단일 연결 꼬리 큐, 리스트, 꼬리 큐 구현

## SYNOPSIS

```c
#include <sys/queue.h>

SLIST_EMPTY(SLIST_HEAD *head);

SLIST_ENTRY(TYPE);

SLIST_FIRST(SLIST_HEAD *head);

SLIST_FOREACH(TYPE *var, SLIST_HEAD *head, SLIST_ENTRY NAME);

SLIST_HEAD(HEADNAME, TYPE);

SLIST_HEAD_INITIALIZER(SLIST_HEAD head);

SLIST_INIT(SLIST_HEAD *head);

SLIST_INSERT_AFTER(TYPE *listelm, TYPE *elm, SLIST_ENTRY NAME);

SLIST_INSERT_HEAD(SLIST_HEAD *head, TYPE *elm, SLIST_ENTRY NAME);

SLIST_NEXT(TYPE *elm, SLIST_ENTRY NAME);

SLIST_REMOVE_HEAD(SLIST_HEAD *head, SLIST_ENTRY NAME);

SLIST_REMOVE(SLIST_HEAD *head, TYPE *elm, TYPE, SLIST_ENTRY NAME);

STAILQ_CONCAT(STAILQ_HEAD *head1, STAILQ_HEAD *head2);

STAILQ_EMPTY(STAILQ_HEAD *head);

STAILQ_ENTRY(TYPE);

STAILQ_FIRST(STAILQ_HEAD *head);

STAILQ_FOREACH(TYPE *var, STAILQ_HEAD *head, STAILQ_ENTRY NAME);

STAILQ_HEAD(HEADNAME, TYPE);

STAILQ_HEAD_INITIALIZER(STAILQ_HEAD head);

STIALQ_INIT(STAILQ_HEAD *head);

STAILQ_INSERT_AFTER(STAILQ_HEAD *head, TYPE *listelm, TYPE *elm,
    STAILQ_ENTRY NAME);

STAILQ_INSERT_HEAD(STAILQ_HEAD *head, TYPE *elm, STAILQ_ENTRY NAME);

STAILQ_INSERT_TAIL(STAILQ_HEAD *head, TYPE *elm, STAILQ_ENTRY NAME);

STAILQ_NEXT(TYPE *elm, STAILQ_ENTRY NAME);

STAILQ_REMOVE_HEAD(STAILQ_HEAD *head, STAILQ_ENTRY NAME);

STAILQ_REMOVE(STAILQ_HEAD *head, TYPE *elm, TYPE, STAILQ_ENTRY NAME);

LIST_EMPTY(LIST_HEAD *head);

LIST_ENTRY(TYPE);

LIST_FIRST(LIST_HEAD *head);

LIST_FOREACH(TYPE *var, LIST_HEAD *head, LIST_ENTRY NAME);

LIST_HEAD(HEADNAME, TYPE);

LIST_HEAD_INITIALIZER(LIST_HEAD head);

LIST_INIT(LIST_HEAD *head);

LIST_INSERT_AFTER(TYPE *listelm, TYPE *elm, LIST_ENTRY NAME);

LIST_INSERT_BEFORE(TYPE *listelm, TYPE *elm, LIST_ENTRY NAME);

LIST_INSERT_HEAD(LIST_HEAD *head, TYPE *elm, LIST_ENTRY NAME);

LIST_NEXT(TYPE *elm, LIST_ENTRY NAME);

LIST_REMOVE(TYPE *elm, LIST_ENTRY NAME);

LIST_SWAP(LIST_HEAD *head1, LIST_HEAD *head2, TYPE, LIST_ENTRY NAME);

TAILQ_CONCAT(TAILQ_HEAD *head1, TAILQ_HEAD *head2, TAILQ_ENTRY NAME);

TAILQ_EMPTY(TAILQ_HEAD *head);

TAILQ_ENTRY(TYPE);

TAILQ_FIRST(TAILQ_HEAD *head);

TAILQ_FOREACH(TYPE *var, TAILQ_HEAD *head, TAILQ_ENTRY NAME);

TAILQ_FOREACH_REVERSE(TYPE *var, TAILQ_HEAD *head, HEADNAME, TAILQ_ENTRY NAME);

TAILQ_HEAD(HEADNAME, TYPE);

TAILQ_HEAD_INITIALIZER(TAILQ_HEAD head);

TAILQ_INIT(TAILQ_HEAD *head);

TAILQ_INSERT_AFTER(TAILQ_HEAD *head, TYPE *listelm, TYPE *elm),
    TAILQ_ENTRY NAME);

TAILQ_INSERT_BEFORE(TYPE *listelm, TYPE *elm, TAILQ_ENTRY NAME);

TAILQ_INSERT_HEAD(TAILQ_HEAD *head, TYPE *elm, TAILQ_ENTRY NAME);

TAILQ_INSERT_TAIL(TAILQ_HEAD *head, TYPE *elm, TAILQ_ENTRY NAME);

TAILQ_LAST(TAILQ_HEAD *head, HEADNAME);

TAILQ_NEXT(TYPE *elm, TAILQ_ENTRY NAME);

TAILQ_PREV(TYPE *elm, HEADNAME, TAILQ_ENTRY NAME);

TAILQ_REMOVE(TAILQ_HEAD *head, TYPE *elm, TAILQ_ENTRY NAME);

TAILQ_SWAP(TAILQ_HEAD *head1, TAILQ_HEAD *head2, TYPE, TAILQ_ENTRY NAME);
```

## DESCRIPTION

이 매크로들은 네 가지 자료 구조(단일 연결 리스트, 단일 연결 꼬리 큐, 리스트, 꼬리 큐)에 대해 정의되어 동작한다. 네 가지 구조 모두 다음 기능성을 지원한다.

 1. 리스트 머리에 새 항목 삽입하기.
 2. 리스트 내 임의 항목 뒤에 새 항목 삽입하기.
 3. 리스트 머리에서 O(1)으로 항목 제거하기.
 4. 순방향으로 리스트 순회하기.
 5. 두 리스트의 내용 서로 바꾸기.

단일 연결 리스트는 네 가지 자료 구조 중 가장 단순하며 위 기능들만 지원한다. 단일 연결 리스트는 데이터 양이 많고 제거가 없는 응용에, 또는 LIFO 큐 구현에 잘 맞는다. 단일 연결 리스트에는 추가로 다음 기능성이 있다.

 1. 리스트 내 임의 항목을 O(n)으로 제거하기.

단일 연결 꼬리 큐에는 추가로 다음 기능성이 있다.

 1. 리스트 끝에 항목을 추가할 수 있다.
 2. 리스트 내 임의 항목을 O(n)으로 제거하기.
 3. 두 리스트를 이어 붙일 수 있다.

하지만

 1. 리스트 삽입 시 리스트 머리를 지정해야 한다.
 2. 각 머리 항목에 포인터가 한 개가 아니라 두 개 필요하다.
 3. 단일 연결 리스트와 비교할 때 코드 크기가 15% 정도 크고 동작이 20% 정도 느리다.

단일 연결 꼬리 큐는 데이터 양이 많고 삭제가 드물거나 없는 응용에, 또는 FIFO 큐 구현에 잘 맞는다.

이중 연결 방식 자료 구조 모두(리스트와 꼬리 큐)에서는 추가로 다음이 가능하다.

 1. 리스트 내 임의 항목 앞에 새 항목 삽입하기.
 2. 리스트 내 임의 항목을 O(1)으로 제거하기.

하지만

 1. 각 항목에 포인터가 한 개가 아니라 두 개 필요하다.
 2. 코드 크기와 (제거를 제외한) 실행 시간이 단일 연결 자료 구조의 두 배 정도이다.

연결 리스트는 가장 단순한 이중 연결 자료 구조이다. 위에다 추가로 다음 기능성이 있다.

 1. 역방향으로 순회할 수 있다.

하지만

 1. 역방향으로 순회하려면 순회를 시작할 항목과 그 항목을 담은 리스트를 지정해야 한다.

꼬리 큐에는 추가로 다음 기능성이 있다.

 1. 리스트 끝에 항목을 추가할 수 있다.
 2. 꼬리에서 머리로 역방향 순회를 할 수 있다.
 3. 두 리스트를 이어 붙일 수 있다.

하지만

 1. 리스트 삽입 및 제거 시 리스트 머리를 지정해야 한다.
 2. 각 머리 항목에 포인터가 한 개가 아니라 두 개 필요하다.
 3. 단일 연결 리스트와 비교할 때 코드 크기가 15% 정도 크고 동작이 20% 정도 느리다.

매크로 정의에서 `TYPE`은 사용자 정의 구조체의 이름이며 그 구조체에는 이름이 `NAME`이고 타입이 `SLIST_ENTRY`, `STAILQ_ENTRY`, `LIST_ENTRY`, `TAILQ_ENTRY` 중 하나인 필드가 있어야 한다. 인자 `HEADNAME`은 사용자 정의 구조체의 이름이며 매크로 `SLIST_HEAD`, `STAILQ_HEAD`, `LIST_HEAD`, `TAILQ_HEAD` 중 하나로 그 구조체를 선언해야 한다. 이 매크로 사용 방식에 대한 추가 설명은 아래의 예시들을 참고하라.

### 단일 연결 리스트

단일 연결 리스트의 머리는 `SLIST_HEAD` 매크로로 정의한 구조체이다. 이 구조체에는 리스트 첫 번째 항목에 대한 포인터 하나가 있다. 공간 및 포인터 조작 오버헤드를 최소화하기 위해 단일 연결로 되어 있으며, 대신 임의 항목 제거에 O(n) 비용이 든다. 기존 항목 뒤나 리스트 머리에 새 항목을 추가할 수 있다. `SLIST_HEAD` 구조체를 다음처럼 선언한다.

```c
SLIST_HEAD(HEADNAME, TYPE) head;
```

`HEADNAME`은 정의하려는 구조체의 이름이고 `TYPE`은 리스트로 연결할 항목들의 타입이다. 이후 다음처럼 리스트 머리에 대한 포인터를 선언할 수 있다.

```c
struct HEADNAME *headp;
```

(이름 `head`와 `headp`는 사용자가 정할 수 있다.)

매크로 `SLIST_HEAD_INITIALIZER`는 리스트 `head`를 위한 초기화 값으로 평가된다.

매크로 `SLIST_EMPTY`는 리스트에 항목이 없으면 참으로 평가된다.

매크로 `SLIST_ENTRY`는 리스트에서 항목들을 연결하는 구조체를 선언한다.

매크로 `SLIST_FIRST`는 리스트의 첫 번째 항목을 반환하며 리스트가 비어 있으면 NULL을 반환한다.

매크로 `SLIST_FOREACH`는 `head`가 가리키는 리스트를 순방향으로 순회하면서 각 항목을 차례로 `var`에 할당한다.

매크로 `SLIST_INIT`은 `head`가 가리키는 리스트를 초기화 한다.

매크로 `SLIST_INSERT_HEAD`는 새 항목 `elm`을 리스트의 머리에 삽입한다.

매크로 `SLIST_INSERT_AFTER`는 새 항목 `elm`을 항목 `listelm` 뒤에 삽입한다.

매크로 `SLIST_NEXT`는 리스트 내의 다음 항목을 반환한다.

매크로 `SLIST_REMOVE_HEAD`는 항목 `elm`을 리스트의 머리에서 제거한다. 최고의 효율성을 위해선 리스트의 머리에서 항목을 제거할 때 범용 `SLIST_REMOVE` 매크로 대신 명시적으로 이 매크로를 쓰는 게 좋다.

매크로 `SLIST_REMOVE`는 항목 `elm`을 리스트에서 제거한다.

### 단일 연결 리스트 예시

```c
SLIST_HEAD(slisthead, entry) head =
    SLIST_HEAD_INITIALIZER(head);
struct slisthead *headp;                /* 단일 연결 리스트 머리 */

struct entry {
        ...
        SLIST_ENTRY(entry) entries;     /* 단일 연결 리스트 */
        ...
} *n1, *n2, *n3, *np;

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

SLIST_FOREACH(np, &head, entries)
        np-> ...

while (!SLIST_EMPTY(&head)) {           /* 리스트 삭제 */
        n1 = SLIST_FIRST(&head);
        SLIST_REMOVE_HEAD(&head, entries);
        free(n1);
}
```

### 단일 연결 꼬리 큐

단일 연결 꼬리 큐의 머리는 `STAILQ_HEAD` 매크로로 정의한 구조체이다. 이 구조체에는 포인터 한 쌍이 있는데, 하나는 꼬리 큐의 첫 번째 항목을 가리키고 다른 하나는 꼬리 큐의 마지막 항목을 가리킨다. 공간 및 포인터 조작 오버헤드를 최소화하기 위해 단일 연결로 되어 있으며, 대신 임의 항목 제거에 O(n) 비용이 든다. 기존 항목 뒤에, 또는 꼬리 큐 머리나 꼬리 큐 끝에 새 항목을 추가할 수 있다. `STAILQ_HEAD` 구조체를 다음처럼 선언한다.

```c
STAILQ_HEAD(HEADNAME, TYPE) head;
```

`HEADNAME`은 정의하려는 구조체의 이름이고 `TYPE`은 꼬리 큐로 연결할 항목들의 타입이다. 이후 다음처럼 꼬리 큐 머리에 대한 포인터를 선언할 수 있다.

```c
struct HEADNAME *headp;
```

(이름 `head`와 `headp`는 사용자가 정할 수 있다.)

매크로 `STAILQ_HEAD_INITIALIZER`는 꼬리 큐 `head`를 위한 초기화 값으로 평가된다.

매크로 `STAILQ_CONCAT`은 `head2`가 가리키는 꼬리 큐를 `head1`이 가리키는 꼬리 큐 끝에 이어 붙이고 `head2`에서 모든 항목을 제거한다.

매크로 `STAILQ_EMPTY`는 꼬리 큐에 항목이 없으면 참으로 평가된다.

매크로 `STAILQ_ENTRY`는 꼬리 큐에서 항목들을 연결하는 구조체를 선언한다.

매크로 `STAILQ_FIRST`는 꼬리 큐의 첫 번째 항목을 반환하며 꼬리 큐가 비어 있으면 NULL을 반환한다.

매크로 `STAILQ_FOREACH`는 `head`가 가리키는 꼬리 큐를 순방향으로 순회하면서 각 항목을 차례로 `var`에 할당한다.

매크로 `STAILQ_INIT`은 `head`가 가리키는 꼬리 큐를 초기화 한다.

매크로 `STAILQ_INSERT_HEAD`는 새 항목 `elm`을 꼬리 큐의 머리에 삽입한다.

매크로 `STAILQ_INSERT_TAIL`은 새 항목 `elm`을 꼬리 큐의 끝에 삽입한다.

매크로 `STAILQ_INSERT_AFTER`는 새 항목 `elm`을 항목 `listelm` 뒤에 삽입한다.

매크로 `STAILQ_NEXT`는 꼬리 큐 내의 다음 항목을 반환하며, 현 항목이 마지막이면 NULL을 반환한다.

매크로 `STAILQ_REMOVE_HEAD`는 항목 `elm`을 꼬리 큐의 머리에서 제거한다. 최고의 효율성을 위해선 꼬리 큐의 머리에서 항목을 제거할 때 범용 `STAILQ_REMOVE` 매크로 대신 명시적으로 이 매크로를 쓰는 게 좋다.

매크로 `STAILQ_REMOVE`는 항목 `elm`을 꼬리 큐에서 제거한다.

### 단일 연결 꼬리 큐 예시

```c
STAILQ_HEAD(stailhead, entry) head =
    STAILQ_HEAD_INITIALIZER(head);
struct stailhead *headp;                /* 단일 연결 꼬리 큐 머리 */
struct entry {
        ...
        STAILQ_ENTRY(entry) entries;    /* 꼬리 큐 */
        ...
} *n1, *n2, *n3, *np;

STAILQ_INIT(&head);                     /* 큐 초기화 */

n1 = malloc(sizeof(struct entry));      /* 머리에 삽입 */
STAILQ_INSERT_HEAD(&head, n1, entries);

n1 = malloc(sizeof(struct entry));      /* 꼬리에 삽입 */
STAILQ_INSERT_TAIL(&head, n1, entries);

n2 = malloc(sizeof(struct entry));      /* 바로 뒤에 삽입 */
STAILQ_INSERT_AFTER(&head, n1, n2, entries);
                                        /* 삭제 */
STAILQ_REMOVE(&head, n2, entry, entries);
free(n2);
                                        /* 머리에서 삭제 */
n3 = STAILQ_FIRST(&head);
STAILQ_REMOVE_HEAD(&head, entries);
free(n3);
                                        /* 순방향 순회 */
STAILQ_FOREACH(np, &head, entries)
        np-> ...
                                        /* 꼬리 큐 삭제 */
while (!STAILQ_EMPTY(&head)) {
        n1 = STAILQ_FIRST(&head);
        STAILQ_REMOVE_HEAD(&head, entries);
        free(n1);
}
                                        /* 더 빠른 꼬리 큐 삭제 */
n1 = STAILQ_FIRST(&head);
while (n1 != NULL) {
        n2 = STAILQ_NEXT(n1, entries);
        free(n1);
        n1 = n2;
}
STAILQ_INIT(&head);
```

### 리스트

리스트의 머리는 `LIST_HEAD` 매크로로 정의한 구조체이다. 이 구조체에는 리스트 첫 번째 항목에 대한 포인터 하나가 있다. 항목들이 이중으로 연결되어 있으므로 리스트 순회 없이 임의 항목을 제거할 수 있다. 기존 항목 뒤나 기존 항목 앞에, 또는 리스트 머리에 새 항목을 추가할 수 있다. `LIST_HEAD` 구조체를 다음처럼 선언한다.

```c
LIST_HEAD(HEADNAME, TYPE) head;
```

`HEADNAME`은 정의하려는 구조체의 이름이고 `TYPE`은 리스트로 연결할 항목들의 타입이다. 이후 다음처럼 리스트 머리에 대한 포인터를 선언할 수 있다.

```c
struct HEADNAME *headp;
```

(이름 `head`와 `headp`는 사용자가 정할 수 있다.)

매크로 `LIST_HEAD_INITIALIZER`는 리스트 `head`를 위한 초기화 값으로 평가된다.

매크로 `LIST_EMPTY`는 리스트에 항목이 없으면 참으로 평가된다.

매크로 `LIST_ENTRY`는 리스트에서 항목들을 연결하는 구조체를 선언한다.

매크로 `LIST_FIRST`는 리스트의 첫 번째 항목을 반환하며 리스트가 비어 있으면 NULL을 반환한다.

매크로 `LIST_FOREACH`는 `head`가 가리키는 리스트를 순방향으로 순회하면서 각 항목을 차례로 `var`에 할당한다.

매크로 `LIST_INIT`은 `head`가 가리키는 리스트를 초기화 한다.

매크로 `LIST_INSERT_HEAD`는 새 항목 `elm`을 리스트의 머리에 삽입한다.

매크로 `LIST_INSERT_AFTER`는 새 항목 `elm`을 항목 `listelm` 뒤에 삽입한다.

매크로 `LIST_INSERT_BEFORE`는 새 항목 `elm`을 항목 `listelm` 앞에 삽입한다.

매크로 `LIST_NEXT`는 리스트 내의 다음 항목을 반환하며, 현 항목이 마지막이면 NULL을 반환한다.

매크로 `LIST_REMOVE`는 항목 `elm`을 리스트에서 제거한다.

### 리스트 예시

```c
LIST_HEAD(listhead, entry) head =
    LIST_HEAD_INITIALIZER(head);
struct listhead *headp;                 /* 리스트 머리 */
struct entry {
        ...
        LIST_ENTRY(entry) entries;      /* 리스트 */
        ...
} *n1, *n2, *n3, *np, *np_temp;

LIST_INIT(&head);                       /* 리스트 초기화 */

n1 = malloc(sizeof(struct entry));      /* 머리에 삽입 */
LIST_INSERT_HEAD(&head, n1, entries);

n2 = malloc(sizeof(struct entry));      /* 바로 뒤에 삽입 */
LIST_INSERT_AFTER(n1, n2, entries);

n3 = malloc(sizeof(struct entry));      /* 바로 앞에 삽입 */
LIST_INSERT_BEFORE(n2, n3, entries);

LIST_REMOVE(n2, entries);               /* 삭제 */
free(n2);
                                        /* 순방향 순회 */
LIST_FOREACH(np, &head, entries)
        np-> ...

while (!LIST_EMPTY(&head)) {            /* 리스트 삭제 */
        n1 = LIST_FIRST(&head);
        LIST_REMOVE(n1, entries);
        free(n1);
}

n1 = LIST_FIRST(&head);                 /* 더 빠른 리스트 삭제 */
while (n1 != NULL) {
        n2 = LIST_NEXT(n1, entries);
        free(n1);
        n1 = n2;
}
LIST_INIT(&head);
```

### 꼬리 큐

꼬리 큐의 머리는 `TAILQ_HEAD` 매크로로 정의한 구조체이다. 이 구조체에는 포인터 한 쌍이 있는데, 하나는 꼬리 큐의 첫 번째 항목을 가리키고 다른 하나는 꼬리 큐의 마지막 항목을 가리킨다. 항목들이 이중으로 연결되어 있으므로 꼬리 큐 순회 없이 임의 항목을 제거할 수 있다. 기존 항목 뒤나 기존 항목 앞에, 꼬리 큐 머리나 꼬리 큐 끝에 새 항목을 추가할 수 있다. `TAILQ_HEAD` 구조체를 다음처럼 선언한다.

```c
TAILQ_HEAD(HEADNAME, TYPE) head;
```

`HEADNAME`은 정의하려는 구조체의 이름이고 `TYPE`은 꼬리 큐로 연결할 항목들의 타입이다. 이후 다음처럼 꼬리 큐 머리에 대한 포인터를 선언할 수 있다.

```c
struct HEADNAME *headp;
```

(이름 `head`와 `headp`는 사용자가 정할 수 있다.)

매크로 `TAILQ_HEAD_INITIALIZER`는 꼬리 큐 `head`를 위한 초기화 값으로 평가된다.

매크로 `TAILQ_CONCAT`은 `head2`가 가리키는 꼬리 큐를 `head1`이 가리키는 꼬리 큐 끝에 이어 붙이고 `head2`에서 모든 항목을 제거한다.

매크로 `TAILQ_EMPTY`는 꼬리 큐에 항목이 없으면 참으로 평가된다.

매크로 `TAILQ_ENTRY`는 꼬리 큐에서 항목들을 연결하는 구조체를 선언한다.

매크로 `TAILQ_FIRST`는 꼬리 큐의 첫 번째 항목을 반환하며 꼬리 큐가 비어 있으면 NULL을 반환한다.

매크로 `TAILQ_FOREACH`는 `head`가 가리키는 꼬리 큐를 순방향으로 순회하면서 각 항목을 차례로 `var`에 할당한다. 루프가 정상적으로 끝나거나 항목이 없는 경우에 `var`를 NULL로 설정한다.

매크로 `TAILQ_FOREACH_REVERSE`는 `head`가 가리키는 꼬리 큐를 역방향으로 순회하면서 각 항목을 차례로 `var`에 할당한다.

매크로 `TAILQ_INIT`은 `head`가 가리키는 꼬리 큐를 초기화 한다.

매크로 `TAILQ_INSERT_HEAD`는 새 항목 `elm`을 꼬리 큐의 머리에 삽입한다.

매크로 `TAILQ_INSERT_TAIL`은 새 항목 `elm`을 꼬리 큐의 끝에 삽입한다.

매크로 `TAILQ_INSERT_AFTER`는 새 항목 `elm`을 항목 `listelm` 뒤에 삽입한다.

매크로 `TAILQ_INSERT_BEFORE`는 새 항목 `elm`을 항목 `listelm` 앞에 삽입한다.

매크로 `TAILQ_LAST`는 꼬리 큐의 마지막 항목을 반환한다. 꼬리 큐가 비어 있으면 반환 값이 NULL이다.

매크로 `TAILQ_NEXT`는 꼬리 큐 내의 다음 항목을 반환하며, 현 항목이 마지막이면 NULL을 반환한다.

매크로 `TAILQ_PREV`는 꼬리 큐 내의 이전 항목을 반환하며, 현 항목이 첫 번째이면 NULL을 반환한다.

매크로 `TAILQ_REMOVE`는 항목 `elm`을 꼬리 큐에서 제거한다.

매크로 `TAILQ_SWAP`은 `head1`과 `head2`의 내용을 서로 바꾼다.

### 꼬리 큐 예시

```c
TAILQ_HEAD(tailhead, entry) head =
    TAILQ_HEAD_INITIALIZER(head);
struct tailhead *headp;                 /* 꼬리 큐 머리 */
struct entry {
        ...
        TAILQ_ENTRY(entry) entries;     /* 꼬리 큐 */
        ...
} *n1, *n2, *n3, *np;

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
TAILQ_FOREACH(np, &head, entries)
        np-> ...
                                        /* 역방향 순회 */
TAILQ_FOREACH_REVERSE(np, &head, tailhead, entries)
        np-> ...
                                        /* 꼬리 큐 삭제 */
while (!TAILQ_EMPTY(&head)) {
        n1 = TAILQ_FIRST(&head);
        TAILQ_REMOVE(&head, n1, entries);
        free(n1);
}
                                        /* 더 빠른 꼬리 큐 삭제 */
n1 = TAILQ_FIRST(&head);
while (n1 != NULL) {
        n2 = TAILQ_NEXT(n1, entries);
        free(n1);
        n1 = n2;
}
TAILQ_INIT(&head);
```

## CONFORMING TO

POSIX.1, POSIX.1-2001, POSIX.1-2008에 없음. BSD에 있음. 4.4BSD에서 `queue` 함수들이 처음 등장했다.

## SEE ALSO

<tt>[[insque(3)]]</tt>

----

2015년 2월 7일
