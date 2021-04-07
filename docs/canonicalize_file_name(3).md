## NAME

canonicalize_file_name - 정규화 된 절대 경로명 반환

## SYNOPSIS

```c
#define _GNU_SOURCE         /* feature_test_macros(7) 참고 */
#include <stdlib.h>

char *canonicalize_file_name(const char *path);
```

## DESCRIPTION

`canonicalize_file_name()` 함수는 `path`에 대응하는 정규화 된 절대 경로명을 담은 널 종료 문자열을 반환한다. 반환 문자열에서는 심볼릭 링크와 `.` 및 `..` 경로명 요소가 풀려 있다. 연달아 있는 슬래시(`/`) 문자들은 슬래시 한 개로 바뀐다.

반환 문자열은 `canonicalize_file_name()`에서 동적으로 할당한 것이므로 더는 필요치 않을 때 호출자가 <tt>[[free(3)]]</tt>로 해제해야 한다.

`canonicalize_file_name(path)` 호출은 다음 호출과 동등하다.

```c
realpath(path, NULL);
```

## RETURN VALUE

성공 시 `canonicalize_file_name()`은 널 종료 문자열을 반환한다. 오류 시 (가령 경로명 부분이 읽기 가능하지 않거나 존재하지 않으면) `canonicalize_file_name()`이 NULL을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

<tt>[[realpath(3)]]</tt> 참고.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `canonicalize_file_name()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

이 함수는 GNU 확장이다.

## SEE ALSO

<tt>[[readlink(2)]]</tt>, <tt>[[realpath(3)]]</tt>

----

2017-09-15
