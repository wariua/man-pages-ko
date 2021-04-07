## NAME

mtrace, muntrace - malloc 추적

## SYNOPSIS

```c
#include <mcheck.h>

void mtrace(void);

void muntrace(void);
```

## DESCRIPTION

`mtrace()` 함수는 메모리 할당 함수들(<tt>[[malloc(3)]]</tt>, <tt>[[realloc(3)]]</tt>, <tt>[[memalign(3)]]</tt>, <tt>[[free(3)]]</tt>)에 훅 함수를 설치한다. 그 훅 함수들에서는 메모리 할당과 해제에 대한 추적 정보를 기록한다. 그 추적 정보를 이용해 프로그램 내의 메모리 누수와 비할당 메모리 해제 시도를 찾아낼 수 있다.

`muntrace()` 함수는 `mtrace()`로 설치한 훅 함수들을 비활성화해서 메모리 할당 함수들에 대한 추적 정보를 더 이상 기록하지 않게 한다. `mtrace()`로 설치했던 훅 함수가 없으면 `muntrace()`는 아무것도 하지 않는다.

`mtrace()`를 호출하면 환경 변수 `MALLOC_TRACE`의 값을 확인하는데, 추적 정보를 기록할 파일의 경로명을 담고 있어야 한다. 그 경로명이 성공적으로 열리면 파일을 길이 0으로 잘라낸다.

`MALLOC_TRACE`가 설정돼 있지 않거나, 지정한 경로명이 유효하지 않거나 쓰기 가능하지 않으면 훅 함수를 설치하지 않으며 `mtrace()`가 아무것도 하지 않는다. set-user-ID 및 set-group-ID 프로그램에서는 `MALLOC_TRACE`를 무시하며 `mtrace()`가 아무것도 하지 않는다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `mtrace()`, `muntrace()` | 스레드 안전성 | MT-Unsafe |

## CONFORMING TO

이 함수들은 GNU 확장이다.

## NOTES

일반적인 사용 방식에서는 프로그램 실행 시작 때 `mtrace()`를 한 번 호출하고 `muntrace()`는 전혀 호출하지 않는다.

`mtrace()` 호출 후 나오는 추적 출력은 텍스트지만 사람이 읽도록 만들어진 건 아니다. GNU C 라이브러리에서 제공하는 펄 스크립트 `mtrace(1)`가 그 추적 로그를 해석해서 사람이 읽을 수 있는 출력을 내놓는다. 보기 좋은 결과를 위해선 추적 대상 프로그램을 컴파일 할 때 디버깅을 켜서 실행 파일에 행 번호 정보를 기록하게 하는 게 좋다.

`mtrace()`로 수행하는 추적은 (`MALLOC_TRACE`가 유효하고 쓰기 가능한 경로명을 가리킨다면) 성능 저하를 유발한다.

## BUGS

`mtrace(1)`가 내놓는 행 번호 정보가 항상 정확하지는 않다. 행 번호가 소스 코드에서 앞이나 뒤의 (비어 있지 않은) 행을 가리킬 수도 있다.

## EXAMPLE

아래의 셸 세션은 두 군데에 메모리 누수가 있는 프로그램으로 `mtrace()` 함수와 `mtrace(1)` 명령 사용 방식을 보여 준다. 다음 프로그램을 사용한다.

```
$ cat t_mtrace.c
#include <mcheck.h>
#include <stdlib.h>
#include <stdio.h>

int
main(int argc, char *argv[])
{
    int j;

    mtrace();

    for (j = 0; j < 2; j++)
        malloc(100);            /* free 안 함 - 메모리 누수 */

    calloc(16, 16);             /* free 안 함 - 메모리 누수 */
    exit(EXIT_SUCCESS);
}
```

다음과 같이 프로그램을 돌리면 `mtrace()`가 프로그램 내의 두 곳에서 메모리 누수를 진단하는 것을 볼 수 있다.

```
$ cc -g t_mtrace.c -o t_mtrace
$ export MALLOC_TRACE=/tmp/t
$ ./t_mtrace
$ mtrace ./t_mtrace $MALLOC_TRACE
Memory not freed:
-----------------
   Address     Size     Caller
0x084c9378     0x64  at /home/cecilia/t_mtrace.c:12
0x084c93e0     0x64  at /home/cecilia/t_mtrace.c:12
0x084c9448    0x100  at /home/cecilia/t_mtrace.c:16
```

처음 두 메시지는 `for` 루프 안의 <tt>[[malloc(3)]]</tt> 호출 두 번에 해당하는 해제 안 된 메모리에 대한 것이다. 마지막 메시지는 <tt>[[calloc(3)]]</tt> 호출(내부에서 다시 <tt>[[malloc(3)]]</tt> 호출함)에 해당한다.

## SEE ALSO

`mtrace(1)`, <tt>[[malloc(3)]]</tt>, <tt>[[malloc_hook(3)]]</tt>, <tt>[[mcheck(3)]]</tt>

----

2017-09-15
