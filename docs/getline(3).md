## NAME

getline, getdelim - 구분자 사용 문자열 입력

## SYNOPSIS

```c
#include <stdio.h>

ssize_t getline(char **lineptr, size_t *n, FILE *stream);

ssize_t getdelim(char **lineptr, size_t *n, int delim, FILE *stream);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`getline()`, `getdelim()`:
:   glibc 2.10부터:
    :   `_POSIX_C_SOURCE >= 200809L`

    glibc 2.10 전:
    :   `_GNU_SOURCE`

## DESCRIPTION

`getline()`은 `stream`으로부터 행 하나를 통째로 읽어서 그 텍스트를 담은 버퍼의 주소를 `*lineptr`에 저장한다. 그 버퍼는 널로 끝나며 개행 문자가 있으면 그대로 포함한다.

호출 전에 `*lineptr`이 NULL로 설정되어 있고 `*n`이 0으로 설정되어 있으면 행을 저장하기 위한 버퍼를 `getline()`에서 할당하게 된다. 사용자 프로그램에서 이 버퍼를 해제해야 하는데, `getline()`이 실패한 경우도 마찬가지이다.

그렇지 않고 `getline()` 호출 전에 `*lineptr`이 <tt>[[malloc(3)]]</tt>으로 할당한 `*n` 바이트 크기 버퍼에 대한 포인터를 담고 있을 수 있다. 그 버퍼가 행을 담기에 충분히 크지 않으면 `getline()`에서 <tt>[[realloc(3)]]</tt>으로 크기를 조정하고 `*lineptr`과 `*n`을 적절히 갱신한다.

어느 경우에든 호출 성공 시에 `*lineptr`과 `*n`이 각각 버퍼 주소와 할당 크기를 나타내도록 갱신된다.

`getdelim()`은 `getline()`처럼 동작하되 `delimiter` 인자로 개행 외의 행 구분자를 지정할 수 있다. `getline()`에서처럼 입력에 구분자 문자 없이 파일 끝에 도달했으면 구분자 문자가 버퍼에 추가되지 않는다.

## RETURN VALUE

성공 시 `getline()`과 `getdelim()`은 읽은 문자 수를 반환하는데, 구문 문자는 포함하고 종료용 널 바이트(`'\0'`)는 제외한다. 이 값을 이용하면 읽은 행에 널 바이트가 포함된 경우를 다룰 수 있다.

두 함수 모두 행을 읽는 데 실패하면 (파일 끝 상태 포함) -1을 반환한다. 오류 발생 시 오류 원인을 나타내도록 `errno`를 설정한다.

## ERRORS

`EINVAL`
:   잘못된 인자. (`n`이나 `lineptr`이 NULL이거나, `stream`이 유효하지 않음.)

`ENOMEM`
:   행 버퍼 할당 내지 재할당 실패.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `getline()`, `getdelim()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

`getline()`과 `getdelim()` 모두 원래는 GNU 확장이었다. POSIX.1-2008에서 표준화되었다.

## EXAMPLE

```c
#define _GNU_SOURCE
#include <stdio.h>
#include <stdlib.h>

int
main(int argc, char *argv[])
{
    FILE *stream;
    char *line = NULL;
    size_t len = 0;
    ssize_t nread;

    if (argc != 2) {
        fprintf(stderr, "Usage: %s <file>\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    stream = fopen(argv[1], "r");
    if (stream == NULL) {
        perror("fopen");
        exit(EXIT_FAILURE);
    }

    while ((nread = getline(&line, &len, stream)) != -1) {
        printf("Retrieved line of length %zu:\n", nread);
        fwrite(line, nread, 1, stdout);
    }

    free(line);
    fclose(stream);
    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

`read(2)`, <tt>[[fgets(3)]]</tt>, <tt>[[fopen(3)]]</tt>, `fread(3)`, <tt>[[scanf(3)]]</tt>

----

2019-03-06
