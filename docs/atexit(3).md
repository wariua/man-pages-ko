## NAME

atexit - 정상적 프로세스 종료 때 호출되는 함수 등록하기

## SYNOPSIS

```c
#include <stdlib.h>

int atexit(void (*function)(void));
```

## DESCRIPTION

`atexit()` 함수는 <tt>[[exit(3)]]</tt>나 프로그램 `main()` 반환을 통한 정상적 프로세스 종료 때 호출하도록 주어진 함수 `function`을 등록한다. 그렇게 등록한 함수들은 등록 순서의 역순으로 호출된다. 어떤 인자도 전달되지 않는다.

같은 함수를 여러 번 등록할 수도 있다. 각 등록마다 한 번씩 호출된다.

POSIX.1에서는 구현에서 그런 함수들을 최소 `ATEXIT_MAX`(32)개 등록할 수 있어야 한다고 요구한다. 구현체에서 지원하는 실제 제한치를 <tt>[[sysconf(3)]]</tt>를 이용해 얻을 수 있다.

<tt>[[fork(2)]]</tt>를 통해 자식 프로세스를 생성하면 부모의 등록 내용 사본을 물려받는다. <tt>[[exec(3)]]</tt> 함수들 중 하나를 성공 호출 시 모든 등록 내용이 없어진다.

## RETURN VALUE

`atexit()` 함수는 성공 시 0 값을 반환한다. 그 외의 경우 0 아닌 값을 반환한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `atexit()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, C89, C99, SVr4, 4.3BSD.

## NOTES

시그널 전달로 인해 프로세스가 비정상 종료하는 경우 `atexit()`로 (그리고 <tt>[[on_exit(3)]]</tt>로) 등록한 함수들을 호출하지 않는다.

등록한 함수에서 <tt>[[_exit(2)]]</tt>를 호출하는 경우 남은 함수들을 호출하지 않으며 <tt>[[exit(3)]]</tt>가 수행하는 나머지 프로세스 종료 단계들을 수행하지 않는다.

POSIX.1에서는 <tt>[[exit(3)]]</tt> 중복 호출의 (즉 `atexit()`로 등록한 함수 안에서 <tt>[[exit(3)]]</tt>를 호출하는 것의) 결과가 규정되어 있지 않다고 한다. 어떤 시스템에서는 (리눅스에서는 아님) 이 때문에 무한 재귀가 발생할 수 있다. 이식 가능한 프로그램에서는 `atexit()`로 등록한 함수 안에서 <tt>[[exit(3)]]</tt>를 부르지 말아야 한다.

`atexit()` 함수와 <tt>[[on_exit(3)]]</tt> 함수는 같은 목록에 함수를 등록한다. 정상적 프로세스 종료 시 이 두 함수로 등록한 역순으로 등록 함수들을 호출한다.

POSIX.1에 따르면 `atexit()`로 등록한 함수의 실행을 <tt>[[longjmp(3)]]</tt>로 종료한 경우 그 결과가 규정되어 있지 않다.

### 리눅스 참고 사항

glibc 2.2.3부터 공유 라이브러리 내에서 `atexit()`를 (그리고 <tt>[[on_exit(3)]]</tt>를) 사용해 그 공유 라이브러리가 내려갈 때 호출될 함수를 설정할 수 있다.

## EXAMPLES

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

void
bye(void)
{
    printf("That was all, folks\n");
}

int
main(void)
{
    long a;
    int i;

    a = sysconf(_SC_ATEXIT_MAX);
    printf("ATEXIT_MAX = %ld\n", a);

    i = atexit(bye);
    if (i != 0) {
        fprintf(stderr, "cannot set exit function\n");
        exit(EXIT_FAILURE);
    }

    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[_exit(2)]]</tt>, <tt>[[dlopen(3)]]</tt>, <tt>[[exit(3)]]</tt>, <tt>[[on_exit(3)]]</tt>

----

2021-03-22
