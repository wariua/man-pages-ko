## NAME

stpcpy, strcasecmp, strcat, strchr, strcmp, strcoll, strcpy, strcspn, strdup, strfry, strlen, strncat, strncmp, strncpy, strncasecmp, strpbrk, strrchr, strsep, strspn, strstr, strtok, strxfrm, index, rindex - 문자열 연산

## SYNOPSIS

```c
#include <strings.h>

int strcasecmp(const char *s1, const char *s2);
    // 대소문자를 무시하고 문자열 s1과 s2를 비교한다.

int strncasecmp(const char *s1, const char *s2, size_t n);
    // 대소문자를 무시하고 문자열 s1과 s2의 처음 n 바이트를 비교한다.

char *index(const char *s, int c);
    // 문자열 s에서 문자 c의 첫 등장 위치에 대한 포인터를 반환한다.

char *rindex(const char *s, int c);
    // 문자열 s에서 문자 c의 마지막 등장 위치에 대한 포인터를 반환한다.

#include <string.h>

char *stpcpy(char *restrict dest, const char *restrict src);
    // src에서 dest로 문자열을 복사하며, dest에 있는 결과 문자열 끝에
    // 대한 포인터를 반환한다.

char *strcat(char *restrict dest, const char *restrict src);
    // 문자열 dest에 문자열 src를 덧붙이며, 포인터 dest를 반환한다.

char *strchr(const char *s, int c);
    // 문자열 s에서 문자 c의 첫 등장 위치에 대한 포인터를 반환한다.

int strcmp(const char *s1, const char *s2);
    // 문자열 s1과 s2를 비교한다.

char *strcpy(char *restrict dest, const char *restrict src);
    // 문자열 src를 dest로 복사하며, dest 시작점에 대한 포인터를
    // 반환한다.

size_t strcspn(const char *s, const char *reject);
    // 문자열 s에서 문자열 reject의 어느 바이트도 담고 있지 않은 첫
    // 부분의 길이를 계산한다.

char *strdup(const char *s);
    // malloc(3)으로 할당한 메모리에 문자열 s의 사본을 넣어 반환한다.

char *strfry(char *string);
    // string 내의 문자들을 무작위로 섞는다.

size_t strlen(const char *s);
    // 문자열 s의 길이를 반환한다.

char *strncat(char *restrict dest, const char *restrict src, size_t n);
    // 문자열 dest에 문자열 src를 최대 n 바이트까지 덧붙이며, 포인터
    // dest를 반환한다.

int strncmp(const char *s1, const char *s2, size_t n);
    // 문자열 s1과 s2를 최대 n 바이트까지 비교한다.

char *strncpy(char *restrict dest, const char *restrict src, size_t n);
    // 문자열 src를 dest로 최대 n 바이트까지 복사하며, dest 시작점에
    // 대한 포인터를 반환한다.

char *strpbrk(const char *s, const char *accept);
    // 문자열 s에서 문자열 accept의 바이트들 중 하나의 첫 등장 위치에
    // 대한 포인터를 반환한다.

char *strrchr(const char *s, int c);
    // 문자열 s에서 문자 c의 마지막 등장 위치에 대한 포인터를 반환한다.

char *strsep(char **restrict stringp, const char *restrict delim);
    // stringp에서 delim의 바이트들 중 하나로 구분된 첫 토큰을
    // 추출한다.

size_t strspn(const char *s, const char *accept);
    // 문자열 s에서 문자열 accept의 바이트들로만 이뤄진 첫 부분의
    // 길이를 계산한다.

char *strstr(const char *haystack, const char *needle);
    // 문자열 haystack에서 부분열 needle의 첫 등장 위치를 찾으며,
    // 발견한 부분열에 대한 포인터를 반환한다.

char *strtok(char *restrict s, const char *restrict delim);
    // 문자열 s에서 delim의 바이트들 중 하나로 구분된 토큰들을
    // 추출한다.

size_t strxfrm(char *restrict dest, const char *restrict src, size_t n);
    // src를 현재 로캘로 변형시켜서 처음 n 바이트를 dst로 복사한다.
```

## DESCRIPTION

문자열 함수들은 널 종료 문자열에 대해 동작을 수행한다. 각 함수에 대한 설명은 개별 맨 페이지를 보라.

## SEE ALSO

<tt>[[bstring(3)]]</tt>, <tt>[[index(3)]]</tt>, <tt>[[rindex(3)]]</tt>, <tt>[[stpcpy(3)]]</tt>, <tt>[[strcasecmp(3)]]</tt>, `strcat(3)`, <tt>[[strchr(3)]]</tt>, <tt>[[strcmp(3)]]</tt>, <tt>[[strcoll(3)]]</tt>, `strcpy(3)`, `strcspn(3)`, `strdup(3)`, <tt>[[strfry(3)]]</tt>, <tt>[[strlen(3)]]</tt>, <tt>[[strncasecmp(3)]]</tt>, `strncat(3)`, <tt>[[strncmp(3)]]</tt>, `strncpy(3)`, `strpbrk(3)`, <tt>[[strrchr(3)]]</tt>, <tt>[[strsep(3)]]</tt>, `strspn(3)`, <tt>[[strstr(3)]]</tt>, <tt>[[strtok(3)]]</tt>, <tt>[[strxfrm(3)]]</tt>

----

2021-03-22
