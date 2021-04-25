## NAME

fpathconf, pathconf - 파일에 대한 구성 값 얻기

## SYNOPSIS

```c
#include <unistd.h>

long fpathconf(int fd, int name);
long pathconf(const char *path, int name);
```

## DESCRIPTION

`fpathconf()`는 열린 파일 디스크립터 `fd`에 대해 구성 옵션 `name`의 값을 얻는다.

`pathconf()`는 파일명 `path`에 대해 구성 옵션 `name`의 값을 얻는다.

`<unistd.h>`에 정의된 대응하는 매크로들은 최솟값이다. 다를 수도 있는 값을 응용에서 이용하고 싶으면 더 후한 결과를 내놓을 수도 있는 `fpathconf()`나 `pathconf()` 호출을 할 수 있다.

`name`을 다음 상수들 중 하나로 설정하면 다음 구성 옵션들을 반환한다.

`_PC_LINK_MAX`
:   파일에 대한 링크의 최대 개수. `fd` 내지 `path`가 디렉터리를 가리키는 경우에는 그 디렉터리 전체에 값이 적용되는 것이다. 대응하는 매크로는 `_POSIX_LINK_MAX`다.

`_PC_MAX_CANON`
:   형식 있는 입력 행의 최대 길이이며 `fd` 내지 `path`가 터미널을 가리켜야 한다. 대응하는 매크로는 `_POSIX_MAX_CANON`이다.

`_PC_MAX_INPUT`
:   입력 행의 최대 길이이며 `fd` 내지 `path`가 터미널을 가리켜야 한다. 대응하는 매크로는 `_POSIX_MAX_INPUT`이다.

`_PC_NAME_MAX`
:   디렉터리 `path` 내지 `fd` 내에 프로세스가 생성할 수 있는 파일명의 최대 길이. 대응하는 매크로는 `_POSIX_NAME_MAX`이다.

`_PC_PATH_MAX`
:   `path` 내지 `fd`가 현재 작업 디렉터리일 때 상대 경로명의 최대 길이. 대응하는 매크로는 `_POSIX_PATH_MAX`이다.

`_PC_PIPE_BUF`
:   FIFO나 파이프에 원자적으로 기록할 수 있는 최대 바이트 수. `fpathconf()`의 경우 `fd`가 파이프나 FIFO를 가리켜야 한다. `pathconf()`의 경우 `path`가 FIFO나 디렉터리를 가리켜야 하며, 후자의 경우 반환 값은 그 디렉터리에 생성되는 FIFO에 해당하는 것이다. 대응하는 매크로는 `_POSIX_PIPE_BUF`이다.

`_PC_CHOWN_RESTRICTED`
:   <tt>[[chown(2)]]</tt> 및 <tt>[[fchown(2)]]</tt>을 사용해 파일의 사용자 ID를 바꾸는 것이 적절한 특권을 가진 프로세스로 제한되어 있고, 파일의 그룹 ID를 프로세스의 실효 그룹 ID나 추가 그룹 ID들 중 하나가 아닌 값으로 바꾸는 것이 적절한 특권을 가진 프로세스로 제한되어 있으면 양수 값을 반환한다. POSIX.1에 따르면 이 변수는 항상 -1이 아닌 값으로 정의되어 있어야 한다. 대응하는 매크로는 `_POSIX_CHOWN_RESTRICTED`이다.

    `fd` 내지 `path`가 디렉터리를 가리키는 경우에는 그 디렉터리 내의 모든 파일들에 값이 적용되는 것이다.

`_PC_NO_TRUNC`
:   `_POSIX_NAME_MAX`보다 긴 파일명에 접근하려 하면 오류가 발생하는 경우 0 아닌 값을 반환한다. 대응하는 매크로는 `_POSIX_NO_TRUNC`이다.

`_PC_VDISABLE`
:   특수 문자 처리를 끌 수 있으면 0 아닌 값을 반환하며 `fd` 내지 `path`가 터미널을 가리켜야 한다.

## RETURN VALUE

이 함수들의 반환 값은 다음 중 하나이다.

* 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다. (예를 들어 `EINVAL`로 `name`이 유효하지 않음을 나타낸다.)

* `name`이 최대 내지 최소 제한에 해당하며 그 제한값이 불확정이면 -1을 반환하며 `errno`는 바꾸지 않는다. (불확정 제한을 오류와 구별하려면 호출 전에 `errno`를 0으로 설정하고서 -1이 반환되었을 때 `errno`가 0인지 확인하면 된다.)

* `name`이 옵션에 해당하면 그 옵션을 지원하는 경우 양수 값을 반환하고 그 옵션을 지원하지 않는 경우 -1을 반환한다.

* 그 외의 경우에 옵션 내지 제한의 현재 값을 반환한다. 이 값은 응용을 컴파일 할 때 `<unistd.h>`나 `<limits.h>`에서 응용에게 기술한 대응 값보다 더 제약적이지 않을 것이다.

## ERRORS

`EACCES`
:   (`pathconf()`) `path`의 경로 선두부 내의 한 디렉터리에 대해 탐색 권한이 거부되었다.

`EBADF`
:   (`fpathconf()`) `fd`가 유효한 파일 디스크립터가 아니다.

`EINVAL`
:   `name`이 유효하지 않다.

`EINVAL`
:   구현에서 `name`과 지정 파일의 연계를 지원하지 않는다.

`ELOOP`
:   (`pathconf()`) `path`를 해석하는 동안 너무 많은 심볼릭 링크를 만났다.

`ENAMETOOLONG`
:   (`pathconf()`) `path`가 너무 길다.

`ENOENT`
:   (`pathconf()`) `path`의 어느 요소가 존재하지 않거나 `path`가 빈 문자열이다.

`ENOTDIR`
:   (`pathconf()`) `path`에 디렉터리로 쓰인 어느 요소가 실제로는 디렉터리가 아니다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `fpathconf()`, `pathconf()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## NOTES

`_PC_NAME_MAX`와 같은 `name`에 대해 반환된 값보다 긴 이름의 파일이 해당 디렉터리에 존재할 수도 있다.

반환되는 일부 값들이 아주 클 수도 있다. 즉, 메모리 할당에 쓰기에 적합하지 않다.

## SEE ALSO

`getconf(1)`, <tt>[[open(2)]]</tt>, <tt>[[statfs(2)]]</tt>, <tt>[[confstr(3)]]</tt>, <tt>[[sysconf(3)]]</tt>

----

2021-03-22
