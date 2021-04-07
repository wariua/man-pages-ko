## NAME

euidaccess, eaccess - 파일에 대한 실효 사용자의 권한 확인하기

## SYNOPSIS

```c
#define _GNU_SOURCE             /* feature_test_macros(7) 참고 */
#include <unistd.h>

int euidaccess(const char *pathname, int mode);
int eaccess(const char *pathname, int mode);
```

## DESCRIPTION

`euidaccess()`는 <tt>[[access(2)]]</tt>처럼 인자 `pathname`이 나타내는 파일의 권한과 존재 여부를 확인한다. 하지만 <tt>[[access(2)]]</tt>에서 프로세스의 실제 사용자 및 그룹 식별자들로 검사를 수행하는 반면 `euidaccess()`에서는 실효 식별자들을 쓴다.

`mode`는 `R_OK`, `W_OK`, `X_OK`, `F_OK`를 한 개 이상 조합한 마스크이며 <tt>[[access(2)]]</tt>에서와 의미가 같다.

`eaccess()`는 `euidaccess()`와 이름만 다른 함수이며 일부 다른 시스템들과의 호환성을 위한 것이다.

## RETURN VALUE

성공 시 (모든 요청 권한이 허가됨) 0을 반환한다. 오류 시 (권한을 묻는 `mode`의 비트 중 최소 하나가 거부됨, 또는 어떤 다른 오류 발생) -1을 반환하며 `errno`를 적절히 설정한다.

## ERRORS

<tt>[[access(2)]]</tt>에서와 같음.

## VERSIONS

glibc 버전 2.4에서 `eaccess()` 함수가 추가되었다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `euidaccess()`, `eaccess()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

이 함수들은 비표준이다. 일부 다른 시스템에 `eaccess()` 함수가 있다.

## NOTES

*경고*: 이 함수를 사용해 파일에 대한 프로세스의 권한을 확인한 다음 그 정보에 따라 어떤 동작을 수행하는 방식은 경쟁 조건으로 이어진다. 두 단계 사이에서 파일 권한이 바뀔 수도 있기 때문이다. 일반적으로 그냥 원하는 동작을 시도하고서 권한 오류가 발생하면 그걸 처리하는 방식이 더 안전하다.

이 함수는 항상 심볼릭 링크를 역참조한다. 심볼릭 링크에 대한 권한을 확인하려면 `AT_EACCESS` 및 `AT_SYMLINK_NOFOLLOW` 플래그와 함께 <tt>[[faccessat(2)]]</tt>을 사용하면 된다.

## SEE ALSO

<tt>[[access(2)]]</tt>, <tt>[[chmod(2)]]</tt>, <tt>[[chown(2)]]</tt>, <tt>[[faccessat(2)]]</tt>, <tt>[[open(2)]]</tt>, <tt>[[setgid(2)]]</tt>, <tt>[[setuid(2)]]</tt>, <tt>[[stat(2)]]</tt>, <tt>[[credentials(7)]]</tt>, <tt>[[path_resolution(7)]]</tt>

----

2017-09-15
