## NAME

basename, dirname - 경로명 요소 파싱 하기

## SYNOPSIS

```c
#include <libgen.h>

char *dirname(char *path);
char *basename(char *path);
```

## DESCRIPTION

주의: 두 가지 다른 `basename()` 함수가 있다. 아래 참고.

`dirname()` 및 `basename()` 함수는 널로 끝나는 경로명 문자열을 디렉터리 부분과 파일명 부분으로 나눈다. 보통 경우에 `dirname()`은 마지막 '/' 전까지의 문자열을 반환하고 `basename()`은 그 마지막 '/' 다음 부분을 반환한다. 끝에 붙은 '/' 문자는 경로명에 포함시키지 않는다.

`path` 안에 슬래시가 없으면 `dirname()`은 문자열 "."을 반환하고 `basename()`은 `path`의 사본을 반환한다. `path`가 문자열 "/"이면 `dirname()`과 `basename()` 모두 문자열 "/"를 반환한다. `path`가 널 포인터거나 빈 문자열에 대한 포인터면 `dirname()`과 `basename()` 모두 문자열 "."를 반환한다.

`dirname()`이 반환한 문자열, "/", `basename()`이 반환한 문자열을 이어 붙이면 완전한 경로명이 나온다.

`dirname()`과 `basename()`에서 `path`의 내용을 변경할 수도 있으므로 이 함수들을 호출할 때는 사본을 전달하는 게 바람직할 수도 있다.

이 함수들이 정적으로 할당한 메모리에 대한 포인터를 반환할 수 있고 이어지는 호출에서 그 메모리에 덮어 쓸 수도 있다. 또는 `path`의 일부분에 대한 포인터를 반환할 수도 있으므로 함수가 반환한 포인터가 더는 필요치 않게 될 때까지는 `path`를 변경하거나 해제하지 말아야 한다.

(SUSv2에서 가져온) 다음 예시 목록은 여러 경로에 대해 `dirname()`과 `basename()`이 반환하는 문자열을 보여 준다.

| 경로       | dirname | basename |
| ---------- | ------- | -------- |
| `/usr/lib` | `/usr`  | `lib`    |
| `/usr/`    | `/`     | `usr`    |
| `usr`      | `.`     | `usr`    |
| `/`        | `/`     | `/`      |
| `.`        | `.`     | `.`      |
| `..`       | `.`     | `..`     |

## RETURN VALUE

`dirname()`과 `basename()` 모두 널 종료 문자열에 대한 포인터를 반환한다. (이 포인터를 <tt>[[free(3)]]</tt>로 전달해선 안 된다.)

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `basename()`, `dirname()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## NOTES

두 가지 서로 다른 `basename()` 버전이 있다. 위에서 기술한 POSIX 버전, 그리고 다음과 같이 얻는 GNU 버전이다.

```c
#define _GNU_SOURCE         /* feature_test_macros(7) 참고 */
#include <string.h>
```

GNU 버전에서는 절대 인자를 변경하지 않으며 `path` 끝에 슬래시가 있을 때, 특히 `path`가 "/"일 때에도 빈 문자열을 반환한다. `dirname()`의 GNU 버전은 없다.

glibc에서는 `<libgen.h>`를 포함하면 POSIX 버전 `basename()`을 얻고 아니면 GNU 버전을 얻는다.

## BUGS

glibc 구현에서 이 함수들의 POSIX 버전이 `path` 인자를 변경하며, "/usr/" 같은 정적 문자열로 호출 시 세그먼테이션 폴트가 발생한다.

glibc 2.2.1 전에서 `dirname()`의 glibc 버전이 끝에 '/' 문자가 있는 경로명을 올바로 처리하지 못했으며 NULL 인자를 주면 세그먼테이션 폴트가 발생했다.

## EXAMPLES

다음 코드 조각은 `basename()` 및 `dirname()` 사용 방식을 보여 준다.

```c
char *dirc, *basec, *bname, *dname;
char *path = "/etc/passwd";

dirc = strdup(path);
basec = strdup(path);
dname = dirname(dirc);
bname = basename(basec);
printf("dirname=%s, basename=%s\n", dname, bname);
```

## SEE ALSO

`basename(1)`, `dirname(1)`

----

2021-03-22
