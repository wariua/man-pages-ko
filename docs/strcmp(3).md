## NAME

strcmp, strncmp - 두 문자열 비교하기

## SYNOPSIS

```c
#include <string.h>

int strcmp(const char *s1, const char *s2);
int strncmp(const char *s1, const char *s2, size_t n);
```

## DESCRIPTION

`strcmp()` 함수는 두 문자열 `s1`과 `s2`를 비교한다. 로캘을 고려하지 않는다. (로캘에 근거한 비교는 <tt>[[strcoll(3)]]</tt>을 보라.) 부호 없는 문자로 비교를 한다.

`strcmp()`는 비교 결과를 나타내는 정수를 반환한다.

* `s1`과 `s2`가 같으면 0
* `s1`이 `s2`보다 작으면 음수 값
* `s1`이 `s2`보다 크면 양수 값

`strncmp()`는 이와 비슷하되, `s1`과 `s2`의 처음 (최대) `n` 바이트만 비교한다.

## RETURN VALUE

`strcmp()`와 `strncmp()` 함수는 `s1`이 (또는 그 처음 `n` 바이트가) `s2`보다 작거나, 일치하거나, 큰 경우에 0보다 작거나, 같거나, 큰 정수를 반환한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `strcmp()`, `strncmp()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, C89, C99, SVr4, 4.3BSD.

## NOTES

POSIX.1에서는 다음처럼만 명세하고 있다.

> 0 아닌 반환 값의 부호는 비교 대상 문자열들에서 첫 번째로 다른 (`unsigned char` 타입으로 해석한) 바이트 쌍의 값 차이의 부호에 의해 정해진다.

glibc에서는 다른 여러 구현체들과 마찬가지로 마지막으로 비교한 `s1`의 바이트에서 마지막으로 비교한 `s2`의 바이트를 뺀 산술 결과 값이 반환 값이다. (두 문자가 같은 경우에는 차이가 0이다.)

## EXAMPLES

아래 프로그램을 사용해 (인자가 2개일 때) `strcmp()`와 (인자가 3개일 때) `strncmp()`의 동작을 볼 수 있다. 다음은 `strcmp()`를 쓰는 예시다.

```text
$ ./string_comp ABC ABC
<str1> and <str2> are equal
$ ./string_comp ABC AB      # 'C'는 ASCII 67, 'C' - ' ' = 67
<str1> is greater than <str2> (67)
$ ./string_comp ABA ABZ     # 'A'는 ASCII 65, 'Z'는 ASCII 90
<str1> is less than <str2> (-25)
$ ./string_comp ABJ ABC
<str1> is greater than <str2> (7)
$ ./string_comp $'\201' A   # 0201 - 0101 = 0100 (십진수 64)
<str1> is greater than <str2> (64)
```

위 예에서 8비트 ASCII 코드를 담은 문자열을 만들어 내기 위해 `bash(1)` 전용 문법을 쓴다. 결과를 보면 문자열 비교에 부호 없는 문자를 쓰는 걸 알 수 있다.

다음은 `strncmp()`를 쓰는 예시다.

```text
$ ./string_comp ABC AB 3
<str1> is greater than <str2> (67)
$ ./string_comp ABC AB 2
<str1> and <str2> are equal in the first 2 bytes
```

### 프로그램 소스

```c
/* string_comp.c

   Licensed under GNU General Public License v2 or later.
*/
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int
main(int argc, char *argv[])
{
    int res;

    if (argc < 3) {
        fprintf(stderr, "Usage: %s <str1> <str2> [<len>]\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    if (argc == 3)
        res = strcmp(argv[1], argv[2]);
    else
        res = strncmp(argv[1], argv[2], atoi(argv[3]));

    if (res == 0) {
        printf("<str1> and <str2> are equal");
        if (argc > 3)
            printf(" in the first %d bytes\n", atoi(argv[3]));
        printf("\n");
    } else if (res < 0) {
        printf("<str1> is less than <str2> (%d)\n", res);
    } else {
        printf("<str1> is greater than <str2> (%d)\n", res);
    }

    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[bcmp(3)]]</tt>, <tt>[[memcmp(3)]]</tt>, <tt>[[strcasecmp(3)]]</tt>, <tt>[[strcoll(3)]]</tt>, <tt>[[string(3)]]</tt>, <tt>[[strncasecmp(3)]]</tt>, <tt>[[strverscmp(3)]]</tt>, `wcscmp(3)`, `wcsncmp(3)`, `ascii(7)`

----

2021-03-22
