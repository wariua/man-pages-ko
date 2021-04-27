## NAME

getsubopt - 문자열에서 하위 옵션 인자 파싱

## SYNOPSIS

```c
#include <stdlib.h>

int getsubopt(char **restrict optionp, char *const *restrict tokens,
              char **restrict valuep);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`getsubopt()`:
:   `_XOPEN_SOURCE >= 500`<br>
    `    || /* glibc 2.12부터: */ _POSIX_C_SOURCE >= 200809L`

## DESCRIPTION

`getsubopt()`는 `optionp`로 받은 쉼표 구분 하위 옵션 목록을 파싱 한다. (보통 <tt>[[getopt(3)]]</tt>를 써서 명령행을 파싱 할 때 그런 하위 옵션 목록이 생긴다. 가령 <tt>[[mount(8)]]</tt>의 `-o` 옵션 참고.) 각 하위 옵션에 값이 있을 수도 있으며 그 경우 등호를 써서 하위 옵션 이름과 구별한다. 다음은 `optionp`로 전달될 수 있을 종류의 문자열 예시이다.

```text
ro,name=xyz
```

`tokens` 인자는 토큰에 대한 포인터들로 된 NULL 종료 배열의 포인터이다. `getsubopt()`가 `optionp`에서 그 토큰들을 찾게 된다. 토큰은 적어도 한 문자를 담은 서로 구별되는 널 종료 문자열이어야 하며 등호나 쉼표가 들어 있지 않아야 한다.

각 `getsubopt()` 호출은 다음 미처리 하위 옵션에 대한 정보를 `optionp`으로 반환한다. 서브 옵션의 첫 번째 등호는 (있다면) 그 하위 옵션의 이름과 값 사이 구분자로 해석한다. 값은 다음 쉽표까지 또는 (마지막 하위 옵션인 경우) 문자열 끝까지 이어진다. 하위 옵션의 이름이 `tokens`에 있는 어떤 이름과 일치하고 값 문자열이 있는 경우에 `getsubopt()`는 그 문자열의 주소를 `*valuep`에 설정한다. 그리고 `optionp`의 첫 번째 쉼표를 널 바이트로 덮어 써서 `*valuep`가 정확하게 그 하위 옵션의 "값 문자열"이 된다.

하위 옵션을 인식했지만 값 문자열은 찾지 못한 경우에는 `*valuep`를 NULL로 설정한다.

`getsubopt()` 반환 시 `optionp`는 다음 하위 옵션을 가리키거나, 마지막 하위 옵션을 방금 처리한 경우 문자열 끝에 있는 널 바이트('\0')를 가리킨다.

## RETURN VALUE

`optionp`의 첫 번째 하위 옵션을 인식한 경우 `getsubopt()`는 `tokens`에서 일치하는 하위 옵션 항목의 인덱스를 반환한다. 아니라면 -1을 반환하며 `*valuep`는 `name[=value]` 문자열 전체이다.

`*optionp`가 바뀌기 때문에 `getsubopt()` 호출 전의 첫 번째 하위 옵션은 `getsubopt()` 후의 첫 번째 하위 옵션과 (반드시) 같지 않다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `getsubopt()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## NOTES

문자열 `*optionp`에서 찾은 쉼표를 `getsubopt()`에서 덮어 쓰기 때문에 그 문자열이 쓰기 가능해야 한다. 즉 문자열 상수일 수 없다.

## EXAMPLES

다음 프로그램은 "-o" 옵션 다음의 하위 옵션을 받는다.

```c
#define _XOPEN_SOURCE 500
#include <stdlib.h>
#include <assert.h>
#include <stdio.h>

int
main(int argc, char **argv)
{
    enum {
        RO_OPT = 0,
        RW_OPT,
        NAME_OPT
    };
    char *const token[] = {
        [RO_OPT]   = "ro",
        [RW_OPT]   = "rw",
        [NAME_OPT] = "name",
        NULL
    };
    char *subopts;
    char *value;
    int opt;

    int readonly = 0;
    int readwrite = 0;
    char *name = NULL;
    int errfnd = 0;

    while ((opt = getopt(argc, argv, "o:")) != -1) {
        switch (opt) {
        case 'o':
            subopts = optarg;
            while (*subopts != '\0' && !errfnd) {

            switch (getsubopt(&subopts, token, &value)) {
            case RO_OPT:
                readonly = 1;
                break;

            case RW_OPT:
                readwrite = 1;
                break;

            case NAME_OPT:
                if (value == NULL) {
                    fprintf(stderr, "Missing value for "
                            "suboption '%s'\n", token[NAME_OPT]);
                    errfnd = 1;
                    continue;
                }

                name = value;
                break;

            default:
                fprintf(stderr, "No match found "
                        "for token: /%s/\n", value);
                errfnd = 1;
                break;
            }
        }
        if (readwrite && readonly) {
            fprintf(stderr, "Only one of '%s' and '%s' can be "
                    "specified\n", token[RO_OPT], token[RW_OPT]);
            errfnd = 1;
        }
        break;

        default:
            errfnd = 1;
        }
    }

    if (errfnd || argc == 1) {
        fprintf(stderr, "\nUsage: %s -o <suboptstring>\n", argv[0]);
        fprintf(stderr, "suboptions are 'ro', 'rw', "
                "and 'name=<value>'\n");
        exit(EXIT_FAILURE);
    }

    /* 프로그램 나머지... */

    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[getopt(3)]]</tt>

----

2021-03-22
