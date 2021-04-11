## NAME

strtok, strtok_r - 문자열에서 토큰 뽑아내기

## SYNOPSIS

```c
#include <string.h>

char *strtok(char *str, const char *delim);

char *strtok_r(char *str, const char *delim, char **saveptr);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`strtok_r()`:
:   `_POSIX_C_SOURCE`<br>
    `    || /* glibc 버전 <= 2.19: */ _BSD_SOURCE || _SVID_SOURCE`

## DESCRIPTION

`strtok()` 함수는 문자열을 0개 이상의 비어 있지 않은 토큰들의 열로 나눈다. `strtok()`에 대한 첫 번째 호출에서 분해할 문자열을 `str`로 지정해야 한다. 같은 문자열을 분해하는 후속 호출 각각에서는 `str`이 NULL이어야 한다.

`delim` 인자는 분해 문자열에서 토큰들을 구분하는 바이트들의 집합을 나타낸다. 같은 문자열을 분해하는 연이은 호출에서 호출자가 `delim`에 다른 문자열들을 지정할 수도 있다.

`strtok()` 호출 각각은 다음 토큰을 담은 널 종료 문자열에 대한 포인터를 반환한다. 이 문자열에는 구분자 바이트가 포함되어 있지 않다. 토큰이 더 없으면 `strtok()`이 NULL을 반환한다.

같은 문자열에 대해 동작하는 연이은 `strtok()` 호출에서는 포인터를 하나 유지하여 다음 토큰 탐색을 시작할 지점을 알아낸다. 첫 번째 `strtok()` 호출에서는 이 포인터를 문자열 첫 번째 바이트를 가리키도록 설정한다. 다음 토큰의 시작점을 알아내기 위해 `str` 내에서 다음에 나오는 구분자 아닌 바이트를 탐색한다. 그런 바이트를 찾으면 다음 토큰의 시작점으로 삼는다. 그런 바이트를 찾지 못하면 더는 토큰이 없는 것이고 `strtok()`이 NULL을 반환한다. (그래서 비어 있거나 구분자들만 담고 있는 문자열에 대해선 첫 번째 호출에서 `strtok()`이 NULL을 반환하게 된다.)

다음 번 구분자 바이트를 찾거나 종료용 널 바이트(`'\0'`)를 만날 때까지 탐색하여 각 토큰의 끝을 찾는다. 구분자 바이트를 찾으면 널 바이트로 덮어 써서 현재 토큰의 끝을 만든다. 그리고 `strtok()`에서 그 다음 바이트에 대한 포인터를 저장한다. 다음 토큰을 검색할 때 그 포인터를 시작점으로 사용하게 된다. 이 경우에 `strtok()`은 발견한 토큰 시작점에 대한 포인터를 반환한다.

위 설명 내용에 따라서 분해 문자열 내의 연속하는 두 개 이상의 구분자 바이트 열은 한 개의 구분자로 본다. 그리고 문자열 시작이나 끝에 있는 구분자 바이트들은 무시된다. 이를 달리 말하자면, `strtok()`이 반환하는 토큰은 항상 비어 있지 않은 문자열이다. 그래서 예를 들어 문자열 "`aaa::bbb,`"가 있을 때 구분자 문자열을 "`;,`"로 지정해서 `strtok()`을 연이어 호출하면 문자열 "`aaa`"와 "`bbb`", 그리고 널 포인터를 반환할 것이다.

`strtok_r()` 함수는 `strtok()`의 재진입 가능 버전이다. `saveptr` 인자는 `char *` 변수에 대한 포인터이다. 같은 문자열을 분해하는 연이은 호출들에서 문맥을 유지하기 위해 `strtok_r()` 내부에서 그 변수를 사용한다.

첫 번째 `strtok_r()` 호출에서 `str`은 분해할 문자열을 가리켜야 하며 `saveptr`의 값은 무시된다. 후속 호출에서 `str`은 NULL이어야 하며 `saveptr`은 이전 호출 이후 바뀌지 않았어야 한다.

`saveptr` 인자를 다르게 지정한 `strtok_r()` 호출들을 이용해 여러 문자열을 동시에 분해할 수 있다.

## RETURN VALUE

`strtok()` 및 `strtok_r()` 함수는 다음 토큰에 대한 포인터를 반환한다. 토큰이 더 없으면 NULL을 반환한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `strtok()` | 스레드 안전성 | MT-Unsafe race:strtok |
| `strtok_r()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

`strtok()`
:   POSIX.1-2001, POSIX.1-2008, C89, C99, SVr4, 4.3BSD.

`strtok_r()`
:   POSIX.1-2001, POSIX.1-2008.

## BUGS

이 함수들을 사용할 때는 조심해야 한다. 꼭 사용하겠다면 다음에 유념해야 한다.

* 이 함수들은 첫 번째 인자를 변경한다.

* 이 함수들은 상수 문자열에 사용할 수 없다. 

* 구분자 바이트의 원래 값을 알 수 없게 된다.

* `strtok()` 함수는 분해하는 동안 정적 버퍼를 사용하며, 따라서 스레드 안전하지 않다. 이게 문제가 된다면 `strtok_r()`을 사용하라.

## EXAMPLE

아래 프로그램에서는 `strtok_r()`을 쓰는 중첩 루프를 이용해 문자열을 두 단계의 토큰들로 나눈다. 첫 번째 명령행 인자는 분해할 문자열을 나타낸다. 두 번째 인자는 문자열을 "큰" 토큰으로 분리하는 데 쓸 구분자 바이트(들)을 나타낸다. 세 번째 인자는 "큰" 토큰을 하위 토큰으로 분리하는 데 쓸 구분자 바이트(들)을 나타낸다.

이 프로그램이 내놓는 출력의 예는 다음과 같다.

```text
$ ./a.out 'a/bbb///cc;xxx:yyy:' ':;' '/'
1: a/bbb///cc
         --> a
         --> bbb
         --> cc
2: xxx
         --> xxx
3: yyy
         --> yyy
```

### 프로그램 소스

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int
main(int argc, char *argv[])
{
    char *str1, *str2, *token, *subtoken;
    char *saveptr1, *saveptr2;
    int j;

    if (argc != 4) {
        fprintf(stderr, "Usage: %s string delim subdelim\n",
                argv[0]);
        exit(EXIT_FAILURE);
    }

    for (j = 1, str1 = argv[1]; ; j++, str1 = NULL) {
        token = strtok_r(str1, argv[2], &saveptr1);
        if (token == NULL)
            break;
        printf("%d: %s\n", j, token);

        for (str2 = token; ; str2 = NULL) {
            subtoken = strtok_r(str2, argv[3], &saveptr2);
            if (subtoken == NULL)
                break;
            printf(" --> %s\n", subtoken);
        }
    }

    exit(EXIT_SUCCESS);
}
```

`strtok()`을 사용하는 또 다른 예시 프로그램을 <tt>[[getaddrinfo_a(3)]]</tt>에서 찾을 수 있다.

## SEE ALSO

`index(3)`, <tt>[[memchr(3)]]</tt>, `rindex(3)`, `strchr(3)`, <tt>[[string(3)]]</tt>, `strpbrk(3)`, <tt>[[strsep(3)]]</tt>, `strspn(3)`, `strstr(3)`, `wcstok(3)`

----

2019-03-06
