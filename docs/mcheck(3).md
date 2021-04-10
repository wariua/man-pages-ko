## NAME

mcheck, mcheck_check_all, mcheck_pedantic, mprobe - 힙 일관성 검사

## SYNOPSIS

```c
#include <mcheck.h>

int mcheck(void (*abortfunc)(enum mcheck_status mstatus));

int mcheck_pedantic(void (*abortfunc)(enum mcheck_status mstatus));

void mcheck_check_all(void);

enum mcheck_status mprobe(void *ptr);
```

## DESCRIPTION

`mcheck()` 함수는 <tt>[[malloc(3)]]</tt> 계열 메모리 할당 함수들에 디버깅 훅을 설치한다. 그러면 힙 상태에 대한 무모순성 검사 몇 가지를 수행하게 된다. 그 검사를 통해 메모리 블록을 여러 번 해제하거나 할당 메모리 블록 바로 앞의 관리용 자료 구조를 오염시키는 것 같은 응용 오류를 탐지할 수 있다.

효과가 있으려면 <tt>[[malloc(3)]]</tt> 내지 관련 함수를 처음 호출하기 전에 `mcheck()` 함수를 호출해야 한다. 그렇게 하는 게 어려운 경우에는 `-lmcheck`로 프로그램을 링크 하면 메모리 할당 함수 최초 호출 앞에 묵시적인 `mcheck()` 호출이 (NULL 인자로) 들어간다.

`mcheck_pedantic()` 함수는 `mcheck()`와 비슷하되 메모리 할당 함수들 중 하나를 호출할 때마다 모든 할당 블록에 대해 검사를 수행한다. 아주 느릴 수 있다!

`mcheck_check_all()` 함수는 모든 할당 블록에 대해 즉시 검사를 수행하게 한다. 이 호출은 앞서 `mcheck()`를 호출했을 때만 효과가 있다.

힙에서 모순성을 탐지하면 `abortfunc`가 가리키는 호출자 제공 함수를 시스템에서 부른다. 유일한 인자 `mstatus`는 탐지한 모순성 종류를 나타낸다. `abortfunc`가 NULL인 경우 기본 함수에서는 `stderr`로 오류 메시지를 찍고 <tt>[[abort(3)]]</tt>를 호출한다.

`mprobe()` 함수는 `ptr`이 가리키는 할당 메모리 블록에 대해 무모순성 검사를 수행한다. 앞서 `mcheck()` 함수를 호출했어야 한다. (안 그러면 `mprobe()`가 `MCHECK_DISABLED`를 반환한다.)

다음은 `mprobe()`가 반환하거나 `abortfunc` 호출 시 `mstatus` 인자로 전달되는 값들에 대한 설명이다.

`MCHECK_DISABLED` (`mprobe()` 한정)
:   메모리 할당 함수를 처음 호출하기 전에 `mcheck()`를 호출하지 않았다. 무모순성 검사가 불가능하다.

`MCHECK_OK` (`mprobe()` 한정)
:   모순성을 탐지하지 못했다.

`MCHECK_HEAD`
:   할당 블록 앞의 메모리가 손상됐다.

`MCHECK_TAIL`
:   할당 블록 뒤의 메모리가 손상됐다.

`MCHECK_FREE`
:   메모리 블록을 두 번 해제했다.

## RETURN VALUE

`mcheck()`와 `mcheck_pedantic()`은 성공 시 0을 반환하고 오류 시 -1을 반환한다.

## VERSIONS

glibc 2.2부터 `mcheck_pedantic()` 및 `mcheck_check_all()` 함수가 사용 가능하다. 적어도 glibc 2.0부터 `mcheck()` 및 `mprobe()` 함수가 존재한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `mcheck()`, `mcheck_pedantic()`,<br>`mcheck_check_all()`, `mprobe()` | 스레드 안전성 | MT-Unsafe race:mcheck<br>const:malloc_hooks |

## CONFORMING TO

이 함수들은 GNU 확장이다.

## NOTES

프로그램을 `-lmcheck`로 링크 하는 방식과 `MALLOC_CHECK_` 환경 변수(<tt>[[mallopt(3)]]</tt>에서 설명)를 쓰는 방식에서 탐지하는 오류 종류는 같다. 하지만 `MALLOC_CHECK_` 방식에서는 응용을 다시 링크할 필요가 없다.

## EXAMPLE

아래 프로그램은 NULL 인자로 `mcheck()`를 호출하고서 같은 메모리 블록을 두 번 해제한다. 다음 셸 세션은 프로그램이 돌 때 어떻게 되는지 보여 준다.

```
$ ./a.out
About to free

About to free a second time
block freed twice
Aborted (core dumped)
```

### 프로그램 소스

```c
#include <stdlib.h>
#include <stdio.h>
#include <mcheck.h>

int
main(int argc, char *argv[])
{
    char *p;

    if (mcheck(NULL) != 0) {
        fprintf(stderr, "mcheck() failed\n");

        exit(EXIT_FAILURE);
    }

    p = malloc(1000);

    fprintf(stderr, "About to free\n");
    free(p);
    fprintf(stderr, "\nAbout to free a second time\n");
    free(p);

    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[malloc(3)]]</tt>, <tt>[[mallopt(3)]]</tt>, <tt>[[mtrace(3)]]</tt>

----

2019-03-06
