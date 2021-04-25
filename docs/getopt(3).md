## NAME

getopt, getopt_long, getopt_long_only, optarg, optind, opterr, optopt - 명령행 옵션 파싱

## SYNOPSIS

```c
#include <unistd.h>

int getopt(int argc, char *const argv[],
           const char *optstring);

extern char *optarg;
extern int optind, opterr, optopt;

#include <getopt.h>

int getopt_long(int argc, char *const argv[],
           const char *optstring,
           const struct option *longopts, int *longindex);

int getopt_long_only(int argc, char *const argv[],
           const char *optstring,
           const struct option *longopts, int *longindex);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`getopt()`:
:   `_POSIX_C_SOURCE >= 2 || _XOPEN_SOURCE`

`getopt_long()`, `getopt_long_only()`:
:   `_GNU_SOURCE`

## DESCRIPTION

`getopt()` 함수는 명령행 인자들을 파싱 한다. 인자 `argc`와 `argv`는 프로그램 호출 시 `main()` 함수로 전달된 인자 개수 및 배열이다. `argv`의 항목 중 '-'로 시작하는 (그러면서 "-"이나 "--"는 아닌) 게 옵션 항목이다. 그 항목의 (처음 '-'를 제외한) 문자들이 옵션 문자이다. `getopt()`를 반복해서 호출하면 옵션 항목 각각의 옵션 문자 각각을 차례로 반환한다.

변수 `optind`는 `argv`에서 다음으로 처리할 항목의 인덱스이다. 시스템에서 이 값을 1로 초기화 한다. 호출자가 이 값을 1로 재설정해서 동일한 `argv`를 다시 훑거나 새로운 인자 벡터를 훑을 수 있다.

`getopt()`에서 옵션 문자를 또 찾으면 그 문자를 반환하고, 외부 변수 `optind`와 정적 변수 `nextchar`를 갱신해서 다음 `getopt()` 호출에서 그 다음 옵션 문자 내지 `argv` 항목을 계속 훑을 수 있도록 한다.

옵션 문자가 더는 없으면 `getopt()`가 -1을 반환한다. 그때 `optind`는 옵션이 아닌 첫 번째 `argv` 항목의 `argv` 내 인덱스이다.

`optstring`은 유효한 옵션 문자들을 담은 문자열이다. 그 문자 뒤에 콜론이 있으면 옵션에 인자가 있어야 하며 `getopt()`에서는 같은 `argv` 항목 내의 다음 텍스트나 다음 `argv` 항목의 텍스트에 대한 포인터를 `optarg`에 집어넣는다. 콜론이 두 개면 그 옵션이 선택적 인자를 받는다는 뜻이다. 현재 `argv` 항목 내에 텍스트가 있으면 (즉 동일 단어에 옵션 이름과 함께 있으면, "-oarg"처럼) `optarg`로 반환되고 아니면 `optarg`가 0으로 설정된다. 이 방식은 GNU 확장이다. 그리고 `optstring`에 `W`가 있고 세미콜론이 따라오면 `-W foo`를 긴 옵션 `--foo`처럼 다룬다. (`-W` 옵션은 POSIX.2에서 구현 확장용으로 예약돼 있다.) 이 동작 방식은 GNU 확장이며 glibc 2 전의 라이브러리에서는 사용 불가능하다.

기본적으로 `getopt()`에서는 `argv`의 내용물을 서로 바꿔서 최종적으로 옵션 아닌 것들이 뒤로 가게 한다. 그와 다른 동작 방식도 두 가지 구현돼 있다. `optstring`의 첫 문자가 '+'이거나 환경 변수 `POSIXLY_CORRECT`가 설정돼 있으면 옵션 아닌 인자를 만나자마자 처리를 멈춘다. 또 `optstring`의 첫 문자가 '-'이면 옵션 아닌 `argv` 항목 각각을 문자 코드가 1인 옵션의 인자인 것처럼 처리한다. (옵션과 다른 `argv` 항목들을 아무 순서로나 받을 수 있고 그 둘에서의 순서에 관심 있는 프로그램에서 이 방식을 쓴다.) 훑는 방식이 뭐든 간에 특수 인자 "--"는 옵션 탐색을 강제로 끝낸다.

옵션 목록을 처리하는 동안 `getopt()`는 두 가지 오류를 탐지할 수 있다. (1) `optstring`에 지정하지 않은 옵션 문자가 있는 경우와 (2) 옵션 인자가 빠져 있는 경우(즉 명령행 마지막 옵션에 필수 인자가 없는 경우)이다. 그런 오류들을 다음과 같이 처리하고 보고한다.

* 기본적으로는 `getopt()`에서 오류 메시지를 표준 오류로 찍고, 문제의 옵션 문자를 `optopt`에 집어넣고, 함수 결과로 '?'를 반환한다.

* 호출자가 전역 변수 `opterr`를 0으로 설정했으면 `getopt()`에서 오류 메시지를 찍지 않는다. 호출자는 함수 반환 값이 '?'인지 검사해서 오류가 있었는지 알아낼 수 있다. (기본적으로 `opterr`는 0 아닌 값을 가지고 있다.)

* `optstring`의 (위에 설명한 선택적 '+' 내지 '-' 다음의) 첫 번째 문자가 콜론(':')이면 마찬가지로 `getopt()`가 오류 메시지를 찍지 않는다. 더불어 빠진 옵션 인자를 나타내는 데 '?' 대신 ':'를 반환한다. 그래서 호출자가 두 가지 오류를 구별할 수 있게 된다.

### `getopt_long()` 및 `getopt_long_only()`

`getopt_long()` 함수는 `getopt()`처럼 동작하되 대시 두 개로 시작하는 긴 옵션들도 받는다. (프로그램에서 긴 옵션만 받으려는 경우에는 `optstring`을 NULL아 아니라 빈 문자열("")로 지정해야 한다.) 짧게 줄인 옵션 문자가 유일하거나 이미 정의된 어떤 옵션과 일치하는 경우에는 긴 옵션 이름을 축약할 수도 있다. 긴 옵션에서는 `--arg=param`이나 `--arg param` 형태로 매개변수를 받을 수 있다.

`longopts`는 `<getopt.h>`에 다음처럼 선언된 `struct option`의 배열 첫 항목을 가리키는 포인터이다.

```c
struct option {
    const char *name;
    int         has_arg;
    int        *flag;
    int         val;
};
```

각 필드의 의미는 다음과 같다.

`name`
:   긴 옵션의 이름이다.

`has_arg`
:   `no_argument`(0)이면 옵션에 인자를 받지 않고, `required_argument`(1)이면 옵션에 인자가 필수이고, `optional_argument`(2)이면 옵션에서 선택적 인자를 받는다.

`flag`
:   긴 옵션에 대한 결과를 어떻게 반환하는지 지정한다. `flag`가 NULL이면 `getopt_long()`이 `val`을 반환한다. (예를 들어 호출 프로그램에서는 `val`을 대응하는 짧은 옵션 문자로 설정할 수 있다.) 아니면 `getopt_long()`이 0을 반환하며, 옵션을 찾았으면 `flag`가 가리키는 변수를 `val`로 설정하고 찾지 못했으면 그대로 둔다.

`val`
:   반환되는 값, 또는 `flag`가 가리키는 변수에 넣을 값.

배열 마지막 항목이 0으로 채워져 있어야 한다.

`longindex`가 NULL이 아니면 가리키는 변수를 `longopts`에서의 긴 옵션 인덱스로 설정한다.

`getopt_long_only()`는 `getopt_long()`과 비슷하되 "--"뿐 아니라 '-'로도 긴 옵션을 나타낼 수 있다. '-'로 ("--" 아님) 시작하는 옵션과 일치하는 긴 옵션이 없고 짧은 옵션은 있는 경우에는 짧은 옵션으로 파싱 한다.

## RETURN VALUE

옵션을 성공적으로 찾은 경우 `getopt()`가 옵션 문자를 반환한다. 명령행 옵션들을 모두 파싱 했으면 `getopt()`가 -1을 반환한다. `optstring`에 없는 옵션 문자를 만났으면 `getopt()`가 '?'을 반환한다. 인자가 빠져 있는 옵션을 만났으면 `optstring`의 첫 문자에 따라 반환 값이 정해진다. 즉 첫 문자가 ':'이면 ':'을 반환하고 아니면 '?'를 반환한다.

`getopt_long()`과 `getopt_long_only()`도 짧은 옵션을 인식했을 때는 옵션 문자를 반환한다. 긴 옵션인 경우에는 `flag`가 NULL이면 `val`을 반환하고 아니면 0을 반환한다. 오류 및 -1 반환은 `getopt()`에서와 같으며, 추가로 여러 가지로 해석할 수 있는 경우나 불필요한 매개변수가 있는 경우에 '?'를 반환한다.

## ENVIRONMENT

`POSIXLY_CORRECT`
:   설정된 경우 옵션 아닌 인자를 만나자마자 옵션 처리를 멈춘다.

`_<PID>_GNU_nonoption_argv_flags_`
:   `bash(1)` 2.0에서 이 변수를 사용했는데 어떤 인자가 와일드카드 확장의 결과이므로 옵션으로 다뤄선 안 된다는 걸 glibc에게 알려 주는 용도였다. `bash(1)` 버전 2.01에서 이 동작이 제거되었지만 glibc에는 지원이 남아 있다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `getopt()`, `getopt_long()`,<br>`getopt_long_only()` | 스레드 안전성 | MT-Unsafe race:getopt env |

## CONFORMING TO

`getopt()`:
:   환경 변수 `POSIXLY_CORRECT`가 설정돼 있다면 POSIX.1-2001, POSIX.1-2008, POSIX.2. 아니라면 이 함수들에서 `argv`의 항목들을 교환하므로 사실 `const`가 아니다. 그렇지만 다른 시스템과의 호환성을 위해 원형에 `const`를 쓴다.

    `optstring`에 '+' 및 '-'를 쓰는 것은 GNU 확장이다.

    일부 구식 구현들에서는 `getopt()`가 `<stdin.h>`에 선언돼 있었다. SUSv1에서는 선언이 `<unistd.h>`와 `<stdio.h>` 어느 쪽에서 나오든 허용했다. POSIX.1-1996에서 이 용도에 `<stdio.h>`를 쓰는 것을 LEGACY로 표시했다. POSIX.1-2001에서는 `<stdio.h>`에 선언이 나와야 한다고 요구하지 않는다.

`getopt_long()` 및 `getopt_long_only()`:
:   이 함수들은 GNU 확장이다.

## NOTES

여러 인자 벡터를 훑거나 같은 벡터를 여러 번 다시 훑는 프로그램에서 `optstring` 선두의 '+'와 '-' 같은 GNU 확장을 쓰고 싶거나 탐색 사이에 `POSIXLY_CORRECT`의 값을 바꾸고 싶은 경우에는 `optind`를 (전통적 값인 1 대신) 0으로 재설정해서 `getopt()`를 재초기화 해야 한다. (0으로 재설정하면 내부 초기화 루틴이 호출돼서 `POSIXLY_CORRECT`를 재확인하고 `optstring`의 GNU 확장을 확인한다.)

## EXAMPLES

### `getopt()`

다음의 간단한 예시 프로그램에서는 `getopt()`를 사용해 두 가지 프로그램 옵션을 처리한다. 연계 값이 없는 `-n` 옵션, 그리고 연계된 값이 필요한 `-t val` 옵션이다.

```c
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>

int
main(int argc, char *argv[])
{
    int flags, opt;
    int nsecs, tfnd;

    nsecs = 0;
    tfnd = 0;
    flags = 0;
    while ((opt = getopt(argc, argv, "nt:")) != -1) {
        switch (opt) {
        case 'n':
            flags = 1;
            break;
        case 't':
            nsecs = atoi(optarg);
            tfnd = 1;
            break;
        default: /* '?' */
            fprintf(stderr, "Usage: %s [-t nsecs] [-n] name\n",
                    argv[0]);
            exit(EXIT_FAILURE);
        }
    }

    printf("flags=%d; tfnd=%d; nsecs=%d; optind=%d\n",
            flags, tfnd, nsecs, optind);

    if (optind >= argc) {
        fprintf(stderr, "Expected argument after options\n");
        exit(EXIT_FAILURE);
    }

    printf("name argument = %s\n", argv[optind]);

    /* 나머지 코드 생략 */

    exit(EXIT_SUCCESS);
}
```

### `getopt_long()`

다음 예시 프로그램에서는 `getopt_long()`의 여러 기능을 사용하는 걸 보여 준다.

```c
#include <stdio.h>     /* for printf */
#include <stdlib.h>    /* for exit  */
#include <getopt.h>

int main(int argc, char **argv) {
    int c;
    int digit_optind = 0;

    while (1) {
        int this_option_optind = optind ? optind : 1;
        int option_index = 0;
        static struct option long_options[] = {
            {"add",     required_argument, 0,  0 },
            {"append",  no_argument,       0,  0 },
            {"delete",  required_argument, 0,  0 },
            {"verbose", no_argument,       0,  0 },
            {"create",  required_argument, 0, 'c'},
            {"file",    required_argument, 0,  0 },
            {0,         0,                 0,  0 }
        };

        c = getopt_long(argc, argv, "abc:d:012",
                 long_options, &option_index);
        if (c == -1)
            break;

        switch (c) {
        case 0:
            printf("option %s", long_options[option_index].name);
            if (optarg)
                printf(" with arg %s", optarg);
            printf("\n");
            break;

        case '0':
        case '1':
        case '2':
            if (digit_optind != 0 && digit_optind != this_option_optind)
              printf("digits occur in two different argv-elements.\n");
            digit_optind = this_option_optind;
            printf("option %c\n", c);
            break;

        case 'a':
            printf("option a\n");
            break;

        case 'b':
            printf("option b\n");
            break;

        case 'c':
            printf("option c with value '%s'\n", optarg);
            break;

        case 'd':
            printf("option d with value '%s'\n", optarg);
            break;

        case '?':
            break;

        default:
            printf("?? getopt returned character code 0%o ??\n", c);
        }
    }

    if (optind < argc) {
        printf("non-option ARGV-elements: ");
        while (optind < argc)
            printf("%s ", argv[optind++]);
        printf("\n");
    }

    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

`getopt(1)`, <tt>[[getsubopt(3)]]</tt>

----

2021-03-22
