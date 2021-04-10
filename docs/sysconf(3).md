## NAME

sysconf - 런타임에 구성 정보 얻기

## SYNOPSIS

```c
#include <unistd.h>

long sysconf(int name);
```

POSIX에서는 특정 옵션이 지원되는지, 또는 특정 구성 상수나 제한값이 무엇인지를 응용에서 컴파일 때나 실행 때에 검사할 수 있도록 한다.

컴파일 때에는 `<unistd.h>` 및/또는 `<limits.h>`를 포함시키고 특정 매크로 값을 검사하면 된다.

실행 때에는 여기 있는 `sysconf()` 함수를 사용해 숫자 값을 물어볼 수 있다. 그리고 <tt>[[fpathconf(3)]]</tt>와 <tt>[[pathconf(3)]]</tt>를 사용해 파일이 위치한 파일 시스템에 따라 다를 수 있는 숫자 값을 물어볼 수 있다. 또 <tt>[[confstr(3)]]</tt>을 사용해 문자열 값을 물어볼 수 있다.

이 함수들에서 얻는 값들은 시스템 구성 상수이다. 프로세스의 수명 동안 바뀌지 않는다.

옵션들에는 보통 `<unistd.h>`에 정의되어 있을 수 있는 상수 `_POSIX_FOO`가 있다. 이 상수가 정의되어 있지 않으면 런타임에 물어봐야 한다. -1으로 정의되어 있으면 그 옵션이 지원되지 않는 것이다. 0으로 정의되어 있으면 관련 함수와 헤더가 존재하는 것이지만 사용할 수 있는 지원이 어느 정도인지 런타임에 물어봐야 한다. -1이나 0 아닌 값으로 정의되어 있으면 그 옵션이 지원되는 것이다. 일반적으로 (200112L 같은) 그 값이 옵션을 기술하는 POSIX 리비전의 연도와 월을 나타낸다. glibc에서는 POSIX 리비전이 아직 발행되지 않은 동안은 1 값을 사용해 지원을 표시한다. `sysconf()` 인자는 `_SC_FOO`가 된다. 옵션들의 목록은 <tt>[[posixoptions(7)]]</tt>를 보라.

변수나 제한값들에는 보통 `<limits.h>`에 정의돼 있을 수 있는 상수 `_FOO`나 `<unistd.h>`에 정의돼 있을 수 있는 `_POSIX_FOO`가 있다. 그 제한이 지정돼 있지 않으면 상수가 정의되어 있지 않을 것이다. 상수가 정의되어 있으면 보장된 값을 주며 실제로는 더 큰 값을 지원할 수도 있다. 시스템에 따라 달라질 수 있는 그 값들을 응용에서 활용하고 싶으면 `sysconf()` 호출을 할 수 있다. `sysconf()` 인자는 `_SC_FOO`가 된다.

### POSIX.1 변수들

변수 이름, 값을 질의하는 데 쓰는 `sysconf()` 인자 이름, 그리고 짧은 설명이 온다.

먼저 POSIX.1 호환 값들이다.

`ARG_MAX` - `_SC_ARG_MAX`
:   <tt>[[exec(3)]]</tt> 계열 함수에 대한 인자들의 최대 길이. `_POSIX_ARG_MAX`(4096)보다 작지 않아야 한다.

`CHILD_MAX` - `_SC_CHILD_MAX`
:   사용자 ID당 동시 프로세스 최대 개수. `_POSIX_CHILD_MAX`(25)보다 작지 않아야 한다.

`HOST_NAME_MAX` - `_SC_HOST_NAME_MAX`
:   <tt>[[gethostname(2)]]</tt>이 반환하는 호스트명의 종료용 널 바이트 제외 최대 길이. `_POSIX_HOST_NAME_MAX`(255)보다 작지 않아야 한다.

`LOGIN_NAME_MAX` - `_SC_LOGIN_NAME_MAX`
:   로그인 이름의 종료용 널 바이트 포함 최대 길이. `_POSIX_LOGIN_NAME_MAX`(9)보다 작지 않아야 한다.

`NGROUPS_MAX` - `_SC_NGROUPS_MAX`
:   추가 그룹 ID의 최대 개수.

클럭 틱 - `_SC_CLK_TCK`
:   초당 클럭 틱 수. 대응하는 변수는 구식이 되었다. 당연히 `CLK_TCK`라는 이름이었다. (참고: `CLOCKS_PER_SEC` 매크로는 정보를 안 준다. 분명 1000000일 것이다.)

`OPEN_MAX` - `_SC_OPEN_MAX`
:   프로세스가 어느 시점에 열어 둘 수 있는 파일의 최대 개수. `_POSIX_OPEN_MAX`(20)보다 작지 않아야 한다.

`PAGESIZE` - `_SC_PAGESIZE`
:   페이지의 바이트 단위 크기. 1보다 작지 않아야 한다.

`PAGE_SIZE` - `_SC_PAGE_SIZE`
:   `PAGESIZE`/`_SC_PAGESIZE`과 같은 의미. (POSIX에는 `PAGESIZE`와 `PAGE_SIZE` 둘 다 정의돼 있다.)

`RE_DUP_MAX` - `_SC_RE_DUP_MAX`
:   <tt>[[regexec(3)]]</tt>와 <tt>[[regcomp(3)]]</tt>에서 허용하는 BRE 반복 횟수. `_POSIX2_RE_DUP_MAX`(255)보다 작지 않아야 한다.

`STREAM_MAX` - `_SC_STREAM_MAX`
:   프로세스가 어느 시점에 열어 둘 수 있는 스트림의 최대 개수. 정의되어 있는 경우 표준 C 매크로 `FOPEN_MAX`와 같은 값이다. `_POSIX_STREAM_MAX`(8)보다 작지 않아야 한다.

`SYMLOOP_MAX` - `_SC_SYMLOOP_MAX`
:   결정 과정이 `ELOOP`을 반환하기 전에 경로명에 심볼릭 링크가 나올 수 있는 최대 횟수. `_POSIX_SYMLOOP_MAX`(8)보다 작지 않아야 한다.

`TTY_NAME_MAX` - `_SC_TTY_NAME_MAX`
:   터미널 장치 이름의 종료용 널 바이트 포함 최대 길이. `_POSIX_TTY_NAME_MAX`(9)보다 작지 않아야 한다.

`TZNAME_MAX` - `_SC_TZNAME_MAX`
:   타임존 이름의 최대 바이트 수. `_POSIX_TZNAME_MAX`(6)보다 작지 않아야 한다.

`_POSIX_VERSION` - `_SC_VERSION`
:   POSIX.1 표준이 승인된 연도와 월을 `YYYYMML` 형식으로 나타낸다. 즉, `199009L` 값은 1990년 9월 리비전을 나타낸다.

### POSIX.2 변수들

다음은 유틸리티들에 제한값을 주는 POSIX.2 값들이다.

`BC_BASE_MAX` - `_SC_BC_BASE_MAX`
:   `bc(1)` 유틸리티가 받아들이는 `obase` 최댓값을 나타낸다.

`BC_DIM_MAX` - `_SC_BC_DIM_MAX`
:   `bc(1)`가 허용하는 배열 내 항목 최대 개수를 나타낸다.

`BC_SCALE_MAX` - `_SC_BC_SCALE_MAX`
:   `bc(1)`가 허용하는 `scale` 최댓값을 나타낸다.

`BC_STRING_MAX` - `_SC_BC_STRING_MAX`
:   `bc(1)`가 받아들이는 문자열 최대 길이를 나타낸다.

`COLL_WEIGHTS_MAX` - `_SC_COLL_WEIGHTS_MAX`
:   로캘 정의 파일에서 `LC_COLLATE` `order` 키워드 항목에 부여할 수 있는 가중치의 최대 개수를 나타낸다.

`EXPR_NEST_MAX` - `_SC_EXPR_NEST_MAX`
:   `expr(1)`에서 괄호 안에 식을 넣을 수 있는 최대 횟수이다.

`LINE_MAX` - `_SC_LINE_MAX`
:   표준 입력이나 파일에서 유틸리티가 읽는 입력 행의 최대 길이. 끝의 개행을 위한 공간을 포함한다.

`RE_DUP_MAX` - `_SC_RE_DUP_MAX`
:   구간 표기 `\{m,n\}` 사용 시 정규 표현식의 최대 반복 횟수.

`POSIX2_VERSION` - `_SC_2_VERSION`
:   POSIX.2 표준의 버전을 `YYYYMML` 형식으로 나타낸다.

`POSIX2_C_DEV` - `_SC_2_C_DEV`
:   POSIX.2 C 언어 개발 설비들을 지원하는지 여부를 나타낸다.

`POSIX2_FORT_DEV` - `_SC_2_FORT_DEV`
:   POSIX.2 포트란 개발 유틸리티들을 지원하는지 여부를 나타낸다.

`POSIX2_FORT_RUN` - `_SC_2_FORT_RUN`
:   POSIX.2 포트란 런타임 유틸리티들을 지원하는지 여부를 나타낸다.

`_POSIX2_LOCALEDEF` - `_SC_2_LOCALEDEF`
:   `localedef(1)`를 통한 POSIX.2 로캘 생성을 지원하는지 여부를 나타낸다.

`POSIX2_SW_DEV` - `_SC_2_SW_DEV`
:   POSIX.2 소프트웨어 개발 유틸리티 옵션을 지원하는지 여부를 나타낸다.

다음 값들도 존재하지만 표준이 아닐 수 있다.

\- `_SC_PHYS_PAGES`
:   물리적 메모리의 페이지 수. 이 값과 `_SC_PAGESIZE` 값을 곱하면 넘칠 수도 있음에 유의하라.

\- `_SC_AVPHYS_PAGES`
:   물리적 메모리의 현재 사용 가능한 페이지 수.

\- `_SC_NPROCESSORS_CONF`
:   구성된 프로세서 개수. <tt>[[get_nprocs_conf(3)]]</tt>도 참고.

\- `_SC_NPROCESSORS_ONLN`
:   현재 온라인인 (사용 가능한) 프로세서 개수. <tt>[[get_nprocs_conf(3)]]</tt>도 참고.

## RETURN VALUE

`sysconf()`의 반환 값은 다음 중 하나이다.

* 오류 시 -1을 반환하며 오류 원인을 나타내도록 `errno`를 설정한다. (예를 들어 `EINVAL`로 `name`이 유효하지 않음을 나타낸다.)

* `name`이 최대 내지 최소 제한에 해당하며 그 제한값이 불확정이면 -1을 반환하며 `errno`는 바꾸지 않는다. (불확정 제한을 오류와 구별하려면 호출 전에 `errno`를 0으로 설정하고서 -1이 반환되었을 때 `errno`가 0인지 확인하면 된다.)

* `name`이 옵션에 해당하면 그 옵션을 지원하는 경우 양수 값을 반환하고 그 옵션을 지원하지 않는 경우 -1을 반환한다.

* 그 외의 경우에 옵션 내지 제한의 현재 값을 반환한다. 이 값은 응용을 컴파일 할 때 `<unistd.h>`나 `<limits.h>`에서 응용에게 기술한 대응 값보다 더 제약적이지 않을 것이다.

## ERRORS

`EINVAL`
:   `name`이 유효하지 않다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `sysconf()` | 스레드 안전성 | MT-Safe env |

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008.

## BUGS

`ARG_MAX`를 사용하기가 어렵다. <tt>[[exec(3)]]</tt>를 위한 인자 공간 중 얼마만큼을 사용자의 환경 변수들이 소모하는지 명세되어 있지 않기 때문이다.

반환되는 일부 값들이 아주 클 수도 있다. 즉, 메모리 할당에 쓰기에 적합하지 않다.

## SEE ALSO

`bc(1)`, `expr(1)`, `getconf(1)`, `locale(1)`, <tt>[[confstr(3)]]</tt>, <tt>[[fpathconf(3)]]</tt>, <tt>[[pathconf(3)]]</tt>, <tt>[[posixoptions(7)]]</tt>

----

2019-05-09
