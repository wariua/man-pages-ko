## NAME

backtrace, backtrace_symbols, backtrace_symbols_fd - 응용 셀프 디버깅 지원

## SYNOPSIS

```c
#include <execinfo.h>

int backtrace(void **buffer, int size);

char **backtrace_symbols(void *const *buffer, int size);
void backtrace_symbols_fd(void *const *buffer, int size, int fd);
```

## DESCRIPTION

`backtrace()`는 `buffer`가 가리키는 배열에 호출 프로그램의 백트레이스를 반환한다. 백트레이스란 프로그램에서 현재 활성인 함수 호출들의 열이다. `buffer`가 가리키는 배열의 각 항목은 `void *` 타입이며 대응하는 스택 프레임의 반환 주소이다. `size` 인자는 `buffer`에 저장할 수 있는 주소의 최대 개수를 나타낸다. 백트레이스가 `size`보다 큰 경우에는 가장 최근 `size` 개 함수 호출에 대응하는 주소들을 반환한다. 백트레이스 전체를 얻으려면 `buffer`와 `size`를 충분히 크게 해야 한다.

`backtrace()`가 반환한 주소 집합을 `backtrace_symbols()`에게 `buffer`로 주면 주소를 심볼로 나타낸 문자열들의 배열로 변환해 준다. `size` 인자는 `buffer`의 주소 개수를 나타낸다. 각 주소의 심볼 표현은 함수 이름 (알아낼 수 있는 경우), 16진수로 된 함수 내 오프셋, (16진수로 된) 실제 반환 주소로 이뤄져 있다. 문자열 포인터들의 배열의 주소를 `backtrace_symbols()`가 함수 결과로 반환한다. 그 배열은 `backtrace_symbols()`에서 <tt>[[malloc(3)]]</tt> 한 것이므로 호출자가 해제해야 한다. (그 포인터 배열이 가리키는 문자열들은 해제할 필요가 없으며 해서도 안 된다.)

`backtrace_symbols_fd()`는 `backtrace_symbols()`와 같은 `buffer` 및 `size` 인자를 받되 문자열 배열을 호출자에게 반환하는 대신 파일 디스크립터 `fd`에 그 문자열들을 한 행에 하나씩 쓴다. `backtrace_symbols_fd()`는 <tt>[[malloc(3)]]</tt>을 호출하지 않으므로 <tt>[[malloc(3)]]</tt>이 실패할 수도 있는 경우에 이용할 수 있다. 하지만 NOTES를 보라.

## RETURN VALUE

`backtrace()`는 `buffer`로 반환하는 주소들의 수를 반환하며 그 값은 `size`보다 크지 않다. 반환 값이 `size`보다 작은 경우에는 백트레이스 전체가 저장된 것이다. `size`와 같은 경우에는 일부가 잘렸을 수도 있으며, 그 경우 오래된 스택 프레임들의 주소가 반환되지 않는다.

성공 시 `backtrace_symbols()`는 호출에서 <tt>[[malloc(3)]]</tt> 한 배열의 포인터를 반환한다. 오류 시 NULL을 반환한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `backtrace()`, `backtrace_symbols()`,<br>`backtrace_symbols_fd()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

이 함수들은 GNU 확장이다.

## NOTES

이 함수들에서는 함수의 반환 주소가 스택에 저장되는 방식에 대해 몇 가지 가정을 한다. 다음에 유의하라.

* (`gcc(1)`의 0 아닌 최적화 단계들에 함의된) 프레임 포인터 생략으로 인해 그 가정들이 위배될 수 있다.

* 인라인 처리된 함수에는 스택 프레임이 없다.

* 꼬리 호출 최적화가 이뤄지면 한 스택 프레임이 다른 프레임을 대체한다.

* `backtrace()`와 `backtrace_symbols_fd()`에서 <tt>[[malloc(3)]]</tt>을 명시적으로 호출하지는 않지만 그 함수들이 포함된 `libgcc`가 최초 사용 때 동적으로 적재된다. 그리고 동적 적재는 일반적으로 <tt>[[malloc(3)]]</tt> 호출을 유발한다. 이 두 함수에 대한 특정 호출이 (가령 시그널 핸들러 안에서) 메모리 할당을 하지 않게 하려면 `libgcc`가 미리 적재돼 있도록 해야 한다.

특별한 링커 옵션을 쓰지 않으면 심볼 이름을 얻지 못할 수도 있다. GNU 링커를 사용하는 시스템에서는 `-rdynamic` 링커 옵션을 써야 한다. 참고로 "static" 함수의 이름은 노출되지 않으므로 백트레이스에 나오지 않는다.

## EXAMPLES

아래 프로그램은 `backtrace()` 및 `backtrace_symbols()`의 사용 방식을 보여 준다. 다음 셸 세션은 프로그램 실행 시 볼 수 있는 결과이다.

```text
$ cc -rdynamic prog.c -o prog
$ ./prog 3
backtrace() returned 8 addresses
./prog(myfunc3+0x5c) [0x80487f0]
./prog [0x8048871]
./prog(myfunc+0x21) [0x8048894]
./prog(myfunc+0x1a) [0x804888d]
./prog(myfunc+0x1a) [0x804888d]
./prog(main+0x65) [0x80488fb]
/lib/libc.so.6(__libc_start_main+0xdc) [0xb7e38f9c]
./prog [0x8048711]
```

### 프로그램 소스

```c
#include <execinfo.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

#define BT_BUF_SIZE 100

void
myfunc3(void)
{
    int nptrs;
    void *buffer[BT_BUF_SIZE];
    char **strings;

    nptrs = backtrace(buffer, BT_BUF_SIZE);
    printf("backtrace() returned %d addresses\n", nptrs);

    /* backtrace_symbols_fd(buffer, nptrs, STDOUT_FILENO) 호출 시
       다음 코드와 비슷한 출력이 나옴: */

    strings = backtrace_symbols(buffer, nptrs);
    if (strings == NULL) {
        perror("backtrace_symbols");
        exit(EXIT_FAILURE);
    }

    for (int j = 0; j < nptrs; j++)
        printf("%s\n", strings[j]);

    free(strings);
}

static void   /* "static"이므로 심볼 내보이지 않음... */
myfunc2(void)
{
    myfunc3();
}

void
myfunc(int ncalls)
{
    if (ncalls > 1)
        myfunc(ncalls - 1);
    else
        myfunc2();
}

int
main(int argc, char *argv[])
{
    if (argc != 2) {
        fprintf(stderr, "%s num-calls\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    myfunc(atoi(argv[1]));
    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[addr2line(1)]]</tt>, `gcc(1)`, <tt>[[gdb(1)]]</tt>, `ld(1)`, <tt>[[dlopen(3)]]</tt>, <tt>[[malloc(3)]]</tt>

----

2021-03-22
