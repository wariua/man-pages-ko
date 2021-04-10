## NAME

realpath - 정규화 된 절대 경로명 반환

## SYNOPSIS

```c
#include <limits.h>
#include <stdlib.h>

char *realpath(const char *path, char *resolved_path);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`realpath()`:
:   `_XOPEN_SOURCE >= 500`<br>
    `    || /* glibc 2.19부터: */ _DEFAULT_SOURCE`<br>
    `    || /* glibc 버전 <= 2.19: */ _BSD_SOURCE`

## DESCRIPTION

`realpath()`는 널 종료 문자열 `path`에서 심볼릭 링크를 모두 확장하고 `/./`, `/../`, 중복 '`/`' 문자를 풀어서 정규화 된 절대 경로명을 만들어 낸다. 결과로 나온 경로명을 `resolved_path`가 가리키는 버퍼에 최대 `PATH_MAX` 바이트까지의 널 종료 문자열로 저장한다. 결과 경로에는 심볼릭 링크나 `/./`, `/../` 요소가 없게 된다.

`resolved_path`를 NULL로 지정하면 `realpath()`에서 <tt>[[malloc(3)]]</tt>을 사용해 풀린 경로명을 담을 최대 `PATH_MAX` 바이트 버퍼를 할당하고 그 버퍼에 대한 포인터를 반환한다. 호출자가 <tt>[[free(3)]]</tt>를 써서 그 버퍼를 해제해야 한다.

## RETURN VALUE

오류가 없으면 `realpath()`는 `resolved_path`에 대한 포인터를 반환한다.

아니면 NULL을 반환하며 그때 배열 `resolved_path`의 내용은 규정돼 있지 않다. 그리고 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EACCES`
:   경로 선두부의 어느 요소에 대해 읽기 내지 탐색 권한이 거부되었다.

`EINVAL`
:   `path`가 NULL이다. (glibc 버전 2.3 전에서는 `resolved_path`가 NULL인 경우에도 이 오류를 반환한다.)

`EIO`
:   파일 시스템에서 읽기를 하는 동안 I/O 오류가 발생했다.

`ELOOP`
:   경로명을 변환하는 동안 너무 많은 심볼릭 링크를 만났다.

`ENAMETOOLONG`
:   경로명의 어느 요소가 `NAME_MAX` 글자를 넘었다. 또는 전체 경로명이 `PATH_MAX` 글자를 넘었다.

`ENOENT`
:   해당 파일이 존재하지 않는다.

`ENOMEM`
:   메모리 부족.

`ENOTDIR`
:   경로 선두부의 한 요소가 디렉터리가 아니다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `realpath()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

4.4BSD, POSIX-1.2001.

POSIX.1-2001에서는 `resolved_path`가 NULL일 때의 동작을 구현에서 규정한다고 한다. POSIX.1-2008에서는 이 페이지에서 기술하는 동작 방식을 명세한다.

## NOTES

4.4BSD와 솔라리스에서 경로명 길이 제한은 (`<sys/param.h>`에 있는) `MAXPATHLEN`이다. SUSv2에서는 `<limits.h>`에 있거나 <tt>[[pathconf(3)]]</tt> 함수로 얻을 수 있는 `PATH_MAX` 및 `NAME_MAX`를 규정하고 있다. 일반적으로 소스가 다음과 같이 될 것이다.

```c
#ifdef PATH_MAX
  path_max = PATH_MAX;
#else
  path_max = pathconf(path, _PC_PATH_MAX);
  if (path_max <= 0)
    path_max = 4096;
#endif
```

(하지만 BUGS 절 참고.)

### GNU 확장

호출이 `EACCES`나 `ENOENT`로 실패하고 `resolved_path`가 NULL이 아니면 읽기 가능하지 않거나 존재하지 않은 `path`의 선두부를 `resolved_path`로 반환한다.

## BUGS

이 함수의 POSIX.1-2001 표준 버전에는 설계상 결함이 있다. 출력 버퍼 `resolved_path`에 적절한 크기를 결정하는 것이 불가능하기 때문이다. POSIX.1-2001에 따르면 `PATH_MAX` 크기의 버퍼로 충분하지만 `PATH_MAX`가 꼭 상수로 정의되어 있어야 하는 것이 아니고 <tt>[[pathconf(3)]]</tt>로 얻어야 할 수도 있다. 그런데 <tt>[[pathconf(3)]]</tt>에 묻는 것이 실제로 도움이 되지 않는 것이, 한편으로는 POSIX에서 <tt>[[pathconf(3)]]</tt>의 결과가 아주 커서 메모리 할당에 부적합할 수도 있다고 경고하고 있으며 또한 <tt>[[pathconf(3)]]</tt>가 `PATH_MAX`에 제한이 없음을 뜻하는 -1을 반환할 수도 있다. POSIX.1-2001에 표준화되어 있지 않지만 POSIX.1-2008에는 표준화되어 있는 `resolved_path == NULL` 기능을 통해 이 설계 문제를 피할 수 있다.

## SEE ALSO

`realpath(1)`, <tt>[[readlink(2)]]</tt>, <tt>[[canonicalize_file_name(3)]]</tt>, <tt>[[getcwd(3)]]</tt>, <tt>[[pathconf(3)]]</tt>, <tt>[[sysconf(3)]]</tt>

----

2017-09-15
