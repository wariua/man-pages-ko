## NAME

envz_add, envz_entry, envz_get, envz_merge, envz_remove, envz_strip - 환경 문자열 지원

## SYNOPSIS

```c
#include <envz.h>

error_t envz_add(char **restrict envz, size_t *restrict envz_len,
               const char *restrict name, const char *restrict value);

char *envz_entry(const char *restrict envz, size_t envz_len,
               const char *restrict name);

char *envz_get(const char *restrict envz, size_t envz_len,
               const char *restrict name);

error_t envz_merge(char **restrict envz, size_t *restrict envz_len,
               const char *restrict envz2, size_t envz2_len,
               int override);

void envz_remove(char **restrict envz, size_t *restrict envz_len,
               const char *restrict name);

void envz_strip(char **restrict envz, size_t *restrict envz_len);
```

## DESCRIPTION

이 함수들은 glibc 전용이다.

argz 벡터는 문자 버퍼 포인터에 길이가 함께 있는 것이다. (<tt>[[argz_add(3)]]</tt> 참고.) envz 벡터는 argz 벡터의 특수한 형태로, 문자열들이 "이름=값" 형태로 되어 있다. 첫 번째 '=' 뒤에 있는 것을 모두 값이라고 본다. '='가 없으면 값이 NULL인 것으로 한다. (반면 '='로 끝나는 경우의 값은 빈 문자열 ""이다.)

이 함수들은 envz 벡터를 다루기 위한 것이다.

`envz_add()`는 envz 벡터 (`*envz`, `*envz_len`)에 문자열 "`name`=`value`"(`value`가 NULL이 아닌 경우)나 "`name`"(`value`가 NULL인 경우)을 추가하고 `*envz`와 `*envz_len`을 갱신한다. `name`이 같은 항목이 존재하면 제거하고 추가한다.

`envz_entry()`는 envz 벡터 (`envz`, `envz_len`)에서 `name`을 탐색하며, 찾으면 그 항목을 반환하고 없으면 NULL을 반환한다.

`envz_get()`은 envz 벡터 (`envz`, `envz_len`)에서 `name`을 탐색하며, 찾으면 그 값을 반환하고 없으면 NULL을 반환한다. ('=' 없는 `name` 항목이 있으면 값이 NULL일 수 있다는 점에 유의하라.)

`envz_merge()`는 `envz2`의 각 항목을 `envz_add()`를 쓴 것처럼 `*envz`에 추가한다. `override`가 참이면 `envz2`에 있는 값이 `*envz`에 있는 같은 이름의 값을 대체하고, 아니면 그러지 않는다.

`envz_remove()`는 (`*envz`, `*envz_len`)에 `name` 항목이 있으면 제거한다.

`envz_strip()`은 값이 NULL인 항목을 모두 제거한다.

## RETURN VALUE

메모리 할당을 하는 envz 함수들은 모두 반환 타입이 `error_t`(정수 타입)이며, 성공 시 0을 반환하고 할당 오류 발생 시 `ENOMEM`을 반환한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `envz_add()`, `envz_entry()`, `envz_get()`,<br>`envz_merge()`, `envz_remove()`, `envz_strip()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

이 함수들은 GNU 확장이다. 조심해서 써야 한다.

## EXAMPLES

```c
#include <stdio.h>
#include <stdlib.h>
#include <envz.h>

int
main(int argc, char *argv[], char *envp[])
{
    int e_len = 0;
    char *str;

    for (int i = 0; envp[i] != NULL; i++)
        e_len += strlen(envp[i]) + 1;

    str = envz_entry(*envp, e_len, "HOME");
    printf("%s\n", str);
    str = envz_get(*envp, e_len, "HOME");
    printf("%s\n", str);
    exit(EXIT_SUCCESS);
}
```


## SEE ALSO

<tt>[[argz_add(3)]]</tt>

----

2021-03-22
