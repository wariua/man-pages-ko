## NAME

clearerr, feof, ferror - 스트림 상태 확인 및 재설정

## SYNOPSIS

```c
#include <stdio.h>

void clearerr(FILE *stream);
int feof(FILE *stream);
int ferror(FILE *stream);
```

## DESCRIPTION

`clearerr()` 함수는 `stream`이 가리키는 스트림에서 파일 끝 표시와 오류 표시를 지운다.

`feof()` 함수는 `stream`이 가리키는 스트림의 파일 끝 표시를 검사해서 설정돼 있으면 0 아닌 값을 반환한다. 그 파일 끝 표시는 `clearerr()` 함수로만 지울 수 있다.

`ferror()` 함수는 `stream`이 가리키는 스트림의 오류 표시를 검사해서 설정돼 있으면 0 아닌 값을 반환한다. 그 오류 표시는 `clearerr()` 함수로만 재설정할 수 있다.

대응하는 논블로킹 버전은 <tt>[[unlocked_stdio(3)]]</tt> 참고.

## RETURN VALUE

`feof()` 함수는 `stream`에 파일 끝 표시가 설정돼 있으면 0 아닌 값을 반환한다. 아니면 0을 반환한다.

`ferror()` 함수는 `stream`에 오류 표시가 설정돼 있으면 0 아닌 값을 반환한다. 아니면 0을 반환한다.

## ERRORS

이 함수들은 실패하지 않으며 `errno`를 설정하지 않는다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `clearerr()`, `feof()`, `ferror()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

`clearerr()`, `feof()`, `ferror()` 함수는 C89, C99, POSIX.1-2001, POSIX.1-2008을 준수한다.

## NOTES

POSIX.1-2008에서는 `stream`이 유효한 경우에 이 함수들이 `errno` 값을 바꾸지 않는다고 명세하고 있다.

## SEE ALSO

<tt>[[open(2)]]</tt>, <tt>[[fdopen(3)]]</tt>, <tt>[[fileno(3)]]</tt>, <tt>[[stdio(3)]]</tt>, <tt>[[unlocked_stdio(3)]]</tt>

----

2021-03-22
