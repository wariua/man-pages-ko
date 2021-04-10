## NAME

tmpfile - 임시 파일 만들기

## SYNOPSIS

```c
#include <stdio.h>

FILE *tmpfile(void);
```

## DESCRIPTION

`tmpfile()` 함수는 유일한 임시 파일을 이진 읽기/쓰기 (w+b) 모드로 연다. 파일이 닫히거나 프로그램이 끝날 때 파일이 자동으로 삭제된다.

## RETURN VALUE

`tmpfile()` 함수는 스트림 디스크립터를 반환한다. 유일한 파일명을 생성할 수 없거나 그 유일한 파일을 열 수 없으면 NULL을 반환한다. 그 경우 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EACCES`
:   파일의 경로 선두부의 디렉터리에 대해 탐색 권한이 거부되었다.

`EEXIST`
:   유일한 파일명을 만들어 낼 수 없음.

`EINTR`
:   시그널에 의해 호출이 중단되었다. <tt>[[signal(7)]]</tt> 참고.

`EMFILE`
:   열린 파일 디스크립터 개수에 대한 프로세스별 제한에 도달했다.

`ENFILE`
:   열린 파일 총개수에 대한 시스템 전역 제한에 도달했다.

`ENOSPC`
:   디렉터리에 새 파일명을 추가할 공간이 없다.

`EROFS`
:   읽기 전용 파일 시스템.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `tmpfile()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, C89, C99, SVr4, 4.3BSD, SUSv2.

## NOTES

POSIX.1-2001 명세: 스트림을 열 수 없는 경우 `stdout`으로 오류 메시지를 쓸 수도 있다.

표준에서는 `tmpfile()`에서 사용할 디렉터리를 명세하고 있지 않다. glibc에서는 `<stdio.h>`에 정의된 경로 선두부 `P_tmpdir`로 시도하고, 실패 시 `/tmp` 디렉터리를 쓴다.

## SEE ALSO

<tt>[[exit(3)]]</tt>, <tt>[[mkstemp(3)]]</tt>, <tt>[[mktemp(3)]]</tt>, <tt>[[tempnam(3)]]</tt>, <tt>[[tmpnam(3)]]</tt>

----

2016-03-15
