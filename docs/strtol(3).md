## NAME

strtol, strtoll, strtoq - 문자열을 long 정수로 변환하기

## SYNOPSIS

```c
#include <stdlib.h>

long strtol(const char *restrict nptr,
            char **restrict endptr, int base);
long long strtoll(const char *restrict nptr,
            char **restrit endptr, int base);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`strtoll()`:
:   `_ISOC99_SOURCE`<br>
    `    || /* glibc <= 2.19: */ _SVID_SOURCE || _BSD_SOURCE`

## DESCRIPTION

`strtol()` 함수는 `nptr`에 있는 문자열의 처음 부분을 지정한 `base`에 따라 `long` 정수 값으로 변환한다. 기수는 2와 36 사이이거나 특수한 값 0이어야 한다.

문자열이 임의 개수의 공백(<tt>[[isspace(3)]]</tt>으로 판단)으로 시작할 수 있으며 그 다음에 선택적으로 '+' 내지 '-' 부호 한 개가 올 수 있다. `base`가 0이나 16이면 문자열에서 다음에 "0x" 내지 "0X" 접두부가 있을 수 있으며, 그러면 수를 16진수로 읽어 들인다. 그렇지 않고 다음 문자가 '0'이면 `base` 0을 8로 (8진수로) 받아들이며, 아니면 10으로 (10진수로) 받아들인다.

문자열 나머지를 명백한 방식으로 `long` 값으로 변환하며 해당 기수에서 유효 숫자가 아닌 첫 번째 문자에서 멈춘다. (기수가 10보다 큰 경우 대소문자 글자 'A'가 10을, 'B'가 11을 나타내며, 그런 식으로 'Z'가 35를 나타낸다.)

`endptr`이 NULL이 아니면 `strtol()`은 첫 번째 비유효 문자의 주소를 `*endptr`에 저장한다. 숫자가 전혀 없었으면 `strtol()`은 `nptr`의 원래 값을 `*endptr`에 저장한다. (그리고 0을 반환한다.) 특히 `*nptr`이 '\0'이 아닌데 반환 시 `**endptr`이 '\0'이면 문자열 전체가 유효한 것이다.

`strtoll()` 함수는 `strtol()` 함수처럼 동작하되 `long long` 정수 값을 반환한다.

## RETURN VALUE

값이 언더플로나 오버플로 되지 않으면 `strtol()` 함수는 변환 결과를 반환한다. 언더플로가 일어나면 `strtol()`은 `LONG_MIN`을 반환한다. 오버플로가 일어나면 `strtol()`은 `LONG_MAX`를 반환한다. 두 경우 모두에서 `errno`를 `ERANGE`로 설정한다. 같은 내용이 `strtoll()`에 (`LONG_MIN`과 `LONG_MAX` 대신 `LLONG_MIN`과 `LLONG_MAX`로) 적용된다.

## ERRORS

`EINVAL`
:   (C99에는 없음) 지정한 `base`가 지원하지 않는 값을 담고 있다.

`ERANGE`
:   결과 값이 범위를 벗어났다.

구현에서 변환을 전혀 수행하지 않은 경우에 (숫자 없음, 0 반환) `errno`를 `EINVAL`로 설정할 수도 있다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `strtol()`, `strtoll()`, `strtoq()` | 스레드 안전성 | MT-Safe locale |

## CONFORMING TO

`strtol()`: POSIX.1-2001, POSIX.1-2008, C89, C99, SVr4, 4.3BSD.

`strtoll()`: POSIX.1-2001, POSIX.1-2008, C99.

## NOTES

`strtol()`이 성공과 실패 어느 쪽에서도 적법하게 0이나 `LONG_MAX`, `LONG_MIN`을 (`strtoll()`에선 `LLONG_MAX`, `LLONG_MIN`) 반환할 수 있으므로 호출 프로그램에서는 호출 전에 `errno`를 0으로 설정하고서 호출 후에 `errno`가 0 아닌 값인지 검사해서 오류가 발생했는지 확인해야 한다.

POSIX.1에 따르면 "C" 및 "POSIX" 외의 로캘에서 이 함수들이 구현에서 정의하는 다른 숫자 문자열들을 받아들일 수도 있다.

BSD에는 완전히 유사하게 정의된 다음 함수가 있다.

```c
quad_t strtoq(const char *nptr, char **endptr, int base);
```

현재 아키텍처의 워드 크기에 따라 `strtoll()`이나 `strtol()`과 동등할 수 있다.

## EXAMPLES

아래 프로그램은 `strtol()` 사용 방식을 보여 준다. 첫 번째 명령행 인자는 `strtol()`이 수를 파싱 할 문자열을 지정한다. 두 번째 (선택적) 인자는 변환에 사용할 기수를 지정한다. (이 인자를 수로 변환하는 것은 <tt>[[atoi(3)]]</tt> 함수인데, 오류 검사를 하지 않으며 `strtol()`보다 인터페이스가 단순하다.) 다음은 이 프로그램이 내놓는 몇 가지 예시 결과들이다.

```text
$ ./a.out 123
strtol() returned 123
$ ./a.out '    123'
strtol() returned 123
$ ./a.out 123abc
strtol() returned 123
Further characters after number: "abc"
$ ./a.out 123abc 55
strtol: Invalid argument
$ ./a.out ''
No digits were found
$ ./a.out 4000000000
strtol: Numerical result out of range
```

### 프로그램 소스

```c
#include <stdlib.h>
#include <limits.h>
#include <stdio.h>
#include <errno.h>

int
main(int argc, char *argv[])
{
    int base;
    char *endptr, *str;
    long val;

    if (argc < 2) {
        fprintf(stderr, "Usage: %s str [base]\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    str = argv[1];
    base = (argc > 2) ? atoi(argv[2]) : 0;

    errno = 0;    /* 호출 후 성공/실패 구별을 위해 */
    val = strtol(str, &endptr, base);

    /* 가능한 여러 오류들 확인하기. */

    if (errno != 0) {
        perror("strtol");
        exit(EXIT_FAILURE);
    }

    if (endptr == str) {
        fprintf(stderr, "No digits were found\n");
        exit(EXIT_FAILURE);
    }

    /* 여기까지 왔다면 strtol()에서 수를 성공적으로 파싱 한 것이다. */

    printf("strtol() returned %ld\n", val);

    if (*endptr != '\0')        /* 반드시 오류는 아님... */
        printf("Further characters after number: \"%s\"\n", endptr);

    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[atof(3)]]</tt>, <tt>[[atoi(3)]]</tt>, <tt>[[atol(3)]]</tt>, <tt>[[strtod(3)]]</tt>, <tt>[[strtoimax(3)]]</tt>, <tt>[[strtoul(3)]]</tt>

----

2021-03-22
