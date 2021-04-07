## NAME

hcreate, hdestroy, hsearch, hcreate_r, hdestroy_r, hsearch_r - 해시 테이블 관리

## SYNOPSIS

```c
#include <search.h>

int hcreate(size_t nel);

ENTRY *hsearch(ENTRY item, ACTION action);

void hdestroy(void);

#define _GNU_SOURCE         /* feature_test_macros(7) 참고 */
#include <search.h>

int hcreate_r(size_t nel, struct hsearch_data *htab);

int hsearch_r(ENTRY item, ACTION action, ENTRY **retval,
              struct hsearch_data *htab);

void hdestroy_r(struct hsearch_data *htab);
```

## DESCRIPTION

세 함수 `hcreate()`, `hsearch()`, `hdestroy()`를 통해 호출자가 키(문자열)와 연계 데이터로 이뤄진 항목들을 담은 해시 탐색 테이블을 만들고 관리할 수 있다. 이 함수들을 쓰면 한번에 해시 테이블 한 개만 사용할 수 있다.

세 함수 `hcreate_r()`, `hsearch_r()`, `hdestroy_r()`은 재진입 가능 버전이며 프로그램에서 한번에 여러 개의 해시 탐색 테이블을 쓸 수 있게 해 준다. 마지막 인자 `htab`가 가리키는 것이 함수가 동작하는 테이블을 기술하는 구조체이다. 프로그래머는 이 구조체를 불투명한 것으로 다뤄야 한다. (즉 이 구조체의 필드에 직접 접근하거나 변경하려 하지 않아야 한다.)

먼저 `hcreate()`로 해시 테이블을 만들어야 한다. 인자 `nel`은 테이블 내의 항목 최대 개수를 지정한다. (이후에 이 최댓값을 바꿀 수 없으므로 잘 선택해야 한다.) 결과 해시 테이블의 성능을 개선하기 위해 구현에서 이 값을 위로 조정할 수도 있다.

`hcreate_r()` 함수는 `hcreate()`와 같은 일을 수행하되 구조체 `*htab`로 기술한 테이블에 대해 그렇게 한다. 처음에 `hcreate_r()`을 호출하기 전에 `htab`가 가리키는 구조체를 0으로 채워야 한다.

`hdestroy()` 함수는 `hcreate()`로 만든 해시 테이블이 차지하는 메모리를 해제한다. `hdestroy()` 호출 후에 `hcreate()`로 새 해시 테이블을 만들 수 있다. `hdestroy_r()`은 유사한 일을 수행하되 앞서 `hcreate_r()`로 생성한 `*htab`로 기술한 해시 테이블에 대해 그렇게 한다.

`hsearch()` 함수는 해시 테이블에서 `item`과 같은 키를 가진 항목을 탐색한다. ("같은" 키인지 `strcmp(3)`로 판단한다.) 성공하면 그 항목에 대한 포인터를 반환한다.

`item` 인자는 `ENTRY` 타입인데, `<search.h>`에 다음처럼 정의돼 있다.

```c
typedef struct entry {
    char *key;
    void *data;
} ENTRY;
```

`key` 필드가 가리키는 널 종료 문자열은 탐색 키이다. `data` 필드는 그 키에 연계된 데이터를 가리킨다.

`action` 인자는 탐색 실패 후에 `hsearch()`가 무엇을 할지 결정한다. 이 인자는 `item` 사본을 삽입하라는 뜻의 `ENTER` 값이거나 (이 경우 새 해시 테이블 항목에 대한 포인터를 함수 결과로 반환함) NULL을 반환해야 한다는 뜻의 `FIND` 값이어야 한다. (`action`이 `FIND`인 경우 `data`는 무시된다.)

`hsearch_r()` 함수는 `hsearch()`와 비슷하되 `*htab`로 기술한 테이블에 대해 동작한다. 그리고 `hsearch_r()` 함수는 발견 항목에 대한 포인터를 함수 결과가 아니라 `*retval`로 반환한다는 점에서 `hsearch()`와 다르다.

## RETURN VALUE

`hcreate()`와 `hcreate_r()`은 성공 시 0 아닌 값을 반환한다. 오류 시 0을 반환하며 오류 원인을 나타내도록 `errno`를 설정한다.

성공 시 `hsearch()`는 해시 테이블 내 항목에 대한 포인터를 반환한다. `hsearch()`는 오류 시에, 즉 `action`이 `ENTER`인데 해시 테이블이 가득 찼거나 `action`이 `FIND`인데 해시 테이블에서 `item`을 찾을 수 없는 경우에 NULL을 반환한다. `hsearch_r()`은 성공 시 0 아닌 값을 반환하고 오류 시 0을 반환한다. 오류 시에 이 두 함수는 오류 원인을 나타내도록 `errno`를 설정한다.

## ERRORS

`hcreate_r()`과 `hdestroy_r()`이 다음 이유로 실패할 수 있다.

<dl>
<dt><code>EINVAL</code></dt>
<dd><code>htab</code>이 NULL이다.</dd>
</dl>

`hsearch()`와 `hsearch_r()`이 다음 이유로 실패할 수 있다.

<dl>
<dt><code>ENOMEM</code></dt>
<dd><code>action</code>이 <code>ENTER</code>였는데 테이블에서 <code>key</code>를 찾지 못했고 테이블에 새 항목을 추가할 공간이 없다.</dd>
<dt><code>ESRCH</code></dt>
<dd><code>action</code>이 <code>FIND</code>였는데 테이블에서 <code>key</code>를 찾지 못했다.</dd>
</dl>

POSIX.1에서는 `ENOMEM` 오류만 명세하고 있다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `hcreate()`, `hsearch()`,<br>`hdestroy()` | 스레드 안전성 | MT-Unsafe race:hsearch |
| `hcreate_r()`, `hsearch_r()`<br>`hdestroy_r()` | 스레드 안전성 | MT-Safe race:htab |

## CONFORMING TO

`hcreate()`, `hsearch()`, `hdestroy()` 함수는 SVr4에서 왔으며 POSIX.1-2001 및 POSIX.1-2008에 기술돼 있다.

`hcreate_r()`, `hsearch_r()`, `hdestroy_r()` 함수는 GNU 확장이다.

## NOTES

일반적으로 테이블에 충분한 여유 공간이 있어서 충돌이 최소화될 때 해시 테이블 구현이 더 효율적이다. 보통은 `nel` 값이 테이블에 저장할 거라 예상하는 항목 최대 개수보다 최소 25% 정도 큰 게 좋다는 뜻이다.

`hdestroy()`와 `hdestroy_r()` 함수에서 해시 테이블 항목들의 `key` 및 `data` 필드가 가리키는 버퍼들을 해제하지 않는다. (그 버퍼들이 동적으로 할당되었는지 여부를 알 수 없으므로 그럴 수가 없다.) 그 버퍼들을 해제해야 한다면 (가령 프로그램에서 실행 시간 내내 쓰는 해시 테이블 하나만 만드는 게 아니라 반복해서 테이블을 만들고 없앤다면) 프로그램에서 버퍼를 추적하는 자료 구조를 유지해야 한다.

## BUGS

SVr4와 POSIX.1-2001에서는 탐색 실패 시에만 `action`이 의미가 있으며 그래서 탐색 성공 시에는 `ENTER`가 아무것도 하지 않아야 한다고 명세한다. libc 및 (버전 2.3 전의) glibc 구현에서는 그 명세를 위반해서 그 경우에 해당 `key`의 `data`를 갱신한다.

개별 해시 테이블 항목들을 추가할 수 있지만 삭제할 수는 없다.

## EXAMPLE

다음 프로그램에서는 해시 테이블에 24개 항목을 삽입하고서 그 중 일부를 찍는다.

```c
#include <stdio.h>
#include <stdlib.h>
#include <search.h>

static char *data[] = { "alpha", "bravo", "charlie", "delta",
     "echo", "foxtrot", "golf", "hotel", "india", "juliet",
     "kilo", "lima", "mike", "movember", "oscar", "papa",
     "quebec", "romeo", "sierra", "tango", "uniform",
     "victor", "whisky", "x-ray", "yankee", "zulu"
};

int
main(void)
{
    ENTRY e, *ep;
    int i;

    hcreate(30);

    for (i = 0; i < 24; i++) {
        e.key = data[i];
        /* data는 뭔가에 대한 포인터가 아니라
           그냥 정수임 */
        e.data = (void *) i;
        ep = hsearch(e, ENTER);
        /* 실패하지 않아야 함 */
        if (ep == NULL) {
            fprintf(stderr, "entry failed\n");
            exit(EXIT_FAILURE);
        }
    }

    for (i = 22; i < 26; i++) {
        /* 테이블에 있는 항목 두 개를 찍고,
           테이블에 없는 항목 두 개 보이기 */
        e.key = data[i];
        ep = hsearch(e, FIND);
        printf("%9.9s -> %9.9s:%d\n", e.key,
               ep ? ep->key : "NULL", ep ? (int)(ep->data) : 0);
    }
    hdestroy();
    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[bsearch(3)]]</tt>, <tt>[[lsearch(3)]]</tt>, <tt>[[malloc(3)]]</tt>, <tt>[[tsearch(3)]]</tt>

----

2019-03-06
