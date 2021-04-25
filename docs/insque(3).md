## NAME

insque, remque - 큐에 항목 삽입하기/제거하기

## SYNOPSIS

```c
#include <search.h>

void insque(void *elem, void *prev);
void remque(void *elem);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`insque()`, `remque()`:
:   `_XOPEN_SOURCE >= 500`<br>
    `    || /* Glibc 2.19부터: */ _DEFAULT_SOURCE`<br>
    `    || /* Glibc <= 2.19: */ _SVID_SOURCE`

## DESCRIPTION

`insque()` 및 `remque()` 함수는 이중 연결 리스트를 조작한다. 리스트의 각 항목은 처음 두 요소가 순방향 및 역방향 포인터인 구조체이다. 연결 리스트는 선형(즉 리스트 끝에서 순방향 포인터가 NULL이고 리스트 처음에서 역방향 포인터가 NULL)일 수도 있고 원형일 수도 있다.

`insque()` 함수는 `elem`이 가리키는 항목을 `prev`가 가리키는 항목 바로 다음에 삽입한다.

리스트가 선형이면 `insque(elem, NULL)` 호출을 써서 리스트 첫 항목을 삽입할 수 있으며, 그 호출에서 `elem`의 정방향 포인터와 역방향 포인터를 NULL로 설정한다.

리스트가 원형이면 호출자가 첫 항목의 순방향 포인터와 역방향 포인터가 그 항목을 가리키도록 해야 하며, `insque()`의 `prev` 인자도 그 항목을 가리켜야 한다.

`remque()` 함수는 `elem`이 가리키는 항목을 이중 연결 리스트에서 제거한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `insque()`, `remque()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## NOTES

아주 오래된 시스템에서는 이 함수들의 인자가 다음처럼 정의된 `struct qelem *` 타입이었다.

```c
struct qelem {
    struct qelem *q_forw;
    struct qelem *q_back;
    char          q_data[1];
};
```

아직도 `<search.h>` 포함 전에 `_GNU_SOURCE`가 정의되어 있으면 이 타입을 얻을 수 있다.

이 함수들 원형의 위치가 유닉스 버전에 따라 다르다. 위는 POSIX 버전이다. 어떤 시스템에서는 `<string.h>`에 있다.

## BUGS

glibc 2.4 및 이전에서는 `prev`에 NULL을 지정하는 게 불가능했다. 그래서 선형 리스트를 만들려면 호출자가 순방향 및 역방향 포인터를 적절히 초기화 한 리스트의 처음 두 항목으로 최초 호출을 해서 리스트를 만들어야 했다.

## EXAMPLES

아래 프로그램은 `insque()` 사용 방식을 보여 준다. 다음은 프로그램 실행 예이다.

```text
$ ./a.out -c a b c
Traversing completed list:
    a
    b
    c
That was a circular list
```

### 프로그램 소스

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <search.h>

struct element {
    struct element *forward;
    struct element *backward;
    char *name;
};

static struct element *
new_element(void)
{
    struct element *e = malloc(sizeof(*e));
    if (e == NULL) {
        fprintf(stderr, "malloc() failed\n");
        exit(EXIT_FAILURE);
    }

    return e;
}

int
main(int argc, char *argv[])
{
    struct element *first, *elem, *prev;
    int circular, opt, errfnd;

    /* "-c" 명령행 옵션을 사용해 리스트를 원형으로
       지정할 수 있다. */

    errfnd = 0;
    circular = 0;
    while ((opt = getopt(argc, argv, "c")) != -1) {
        switch (opt) {
        case 'c':
            circular = 1;
            break;
        default:
            errfnd = 1;
            break;
        }
    }

    if (errfnd || optind >= argc) {
        fprintf(stderr,  "Usage: %s [-c] string...\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    /* 첫 번째 항목을 만들어서 연결 리스트에 넣기. */

    elem = new_element();
    first = elem;

    elem->name = argv[optind];

    if (circular) {
        elem->forward = elem;
        elem->backward = elem;
        insque(elem, elem);
    } else {
        insque(elem, NULL);
    }

    /* 나머지 명령행 인자들을 리스트 항목으로 추가하기. */

    while (++optind < argc) {
        prev = elem;

        elem = new_element();
        elem->name = argv[optind];
        insque(elem, prev);
    }

    /* 리스트를 처음부터 순회하면서 항목 이름 찍기. */

    printf("Traversing completed list:\n");
    elem = first;
    do {
        printf("    %s\n", elem->name);
        elem = elem->forward;
    } while (elem != NULL && elem != first);

    if (elem == first)
        printf("That was a circular list\n");

    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[queue(7)]]</tt>

----

2021-03-22
