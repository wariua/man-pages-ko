## NAME

argz_add, argz_add_sep, argz_append, argz_count, argz_create, argz_create_sep, argz_delete, argz_extract, argz_insert, argz_next, argz_replace, argz_stringify - argz 목록을 다루는 함수들

## SYNOPSIS

```c
#include <argz.h>

error_t argz_add(char **restrict argz, size_t *restrict argz_len,
                const char *restrict str);

error_t argz_add_sep(char **restrict argz, size_t *restrict argz_len,
                const char *restrict str, int delim);

error_t argz_append(char **restrict argz, size_t *restrict argz_len,
                const char *restrict buf, size_t buf_len);

size_t argz_count(const char *argz, size_t argz_len);

error_t argz_create(char *const argv[], char **restrict argz,
                size_t *restrict argz_len);

error_t argz_create_sep(const char *restrict str, int sep,
                char **restrict argz, size_t *restrict argz_len);

void argz_delete(char **restrict argz, size_t *restrict argz_len,
                char *restrict entry);

void argz_extract(const char *restrict argz, size_t argz_len,
                char **restrict argv);

error_t argz_insert(char **restrict argz, size_t *restrict argz_len,
                char *restrict before, const char *restrict entry);

char *argz_next(const char *restrict argz, size_t argz_len,
                const char *restrict entry);

error_t argz_replace(char **restrict argz, size_t *restrict argz_len,
                const char *restrict str, const char *restrict with,
                unsigned int *restrict replace_count);

void argz_stringify(char *argz, size_t len, int sep);
```

## DESCRIPTION

이 함수들은 glibc 전용이다.

argz 벡터는 문자 버퍼에 대한 포인터에 길이가 함께 있는 것이다. 문자 버퍼는 문자열의 배열로 해석하게 되어 있으며 널 바이트(`'\0'`)로 그 문자열들을 구분한다. 길이가 0이 아닌 경우 버퍼의 마지막 바이트가 널 바이트여야 한다.

이 함수들은 argz 벡터를 다루기 위한 것이다. (NULL,0) 쌍 역시 argz 벡터이며, 역으로 길이 0인 argz 벡터는 포인터가 널이어야 한다. 비어 있지 않은 argz 벡터를 할당하는 것은 <tt>[[malloc(3)]]</tt>으로 이뤄지며, 따라서 <tt>[[free(3)]]</tt>를 이용해 다시 없앨 수 있다.

`argz_add()`는 배열 `*argz` 끝에 문자열 `str`을 추가하고 `*argz`와 `*argz_len`을 갱신한다.

`argz_add_sep()`도 비슷하되 문자열 `str`을 구분자 `delim`으로 나눠지는 하위 문자열들로 쪼갠다. 예를 들어 유닉스 검색 경로에 구분자 ':'로 이 함수를 쓸 수 있을 것이다.

`argz_append()`는 argz 벡터 (`*argz`, `*argz_len`) 뒤에 (`buf`, `buf_len`)을 덧붙이고 `*argz`와 `*argz_len`을 갱신한다. (그래서 `*argz_len`이 `buf_len`만큼 커지게 된다.)

`argz_count()`는 (`argz`, `argz_len`) 내의 문자열 개수, 즉 널 바이트(`'\0'`) 개수를 센다.

`argz_create()`는 `(char *) 0`으로 끝나는 유닉스 방식 인자 벡터 `argv`를 argz 벡터 (`*argz`, `*argz_len`)으로 변환한다.

`argz_create_sep()`은 널 종료 문자열 `str`을 구분자 `sep`이 있는 모든 곳에서 나눠서 argz 벡터 (`*argz`, `*argz_len`)으로 변환한다.

`argz_delete()`는 argz 벡터 (`*argz`, `*argz_len`)에서 `entry`가 가리키는 하위 문자열을 제거하고 `*argz`와 `*argz_len`을 갱신한다.

`argz_extract()`는 `argz_create()`의 반대이다. argz 벡터 (`argz`, `argz_len`)을 받아서 `argv`에서 시작하는 배열을 하위 문자열 포인터들과 마지막의 NULL로 채워서 유닉스 방식 argv 벡터를 만든다. 배열 `argv`에는 포인터 `argz_count(argz, argz_len) + 1` 개만큼의 공간이 있어야 한다.

`argz_insert()`는 `argz_delete()`의 반대이다. argz 벡터 (`*argz`, `*argz_len`)의 `before` 위치에 인자 `entry`를 끼워 넣고 `*argz`와 `*argz_len`을 갱신한다. `before`가 NULL이면 끝에 `entry`를 집어넣는다.

`argz_next()`는 argz 벡터를 순회하기 위한 함수이다. `entry`가 NULL이면 첫 번째 항목을 반환하고, 아니면 다음 항목을 반환한다. 다음 항목이 없으면 NULL을 반환한다.

`argz_replace()`는 각 `str`을 `with`로 교체하며 필요한 대로 argz를 재할당한다. `replace_count`가 NULL이 아니면 교체 횟수만큼 `*replace_count`를 올린다.

`argz_stringify()`는 `argz_create_sep()`의 반대이다. 마지막을 뺀 모든 널 바이트(`'\0'`)를 `sep`으로 교체해서 argz 벡터를 보통 문자열로 변환한다.

## RETURN VALUE

메모리 할당을 하는 argz 함수들은 모두 반환 타입이 `error_t`(정수 타입)이며, 성공 시 0을 반환하고 할당 오류 발생 시 `ENOMEM`을 반환한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `argz_add()`, `argz_add_sep()`, `argz_append()`,<br>`argz_count()`, `argz_create()`, `argz_create_sep()`,<br>`argz_delete()`, `argz_extract()`, `argz_insert()`,<br>`argz_next()`, `argz_replace()`, `argz_stringify()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

이 함수들은 GNU 확장이다.

## BUGS

종료용 널 바이트가 없는 argz 벡터가 세그먼테이션 폴트로 이어질 수 있다.

## SEE ALSO

<tt>[[envz_add(3)]]</tt>

----

2021-03-22
