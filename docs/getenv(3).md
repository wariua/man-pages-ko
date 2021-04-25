## NAME

getenv, secure_getenv - 환경 변수 얻기

## SYNOPSIS

```c
#include <stdlib.h>

char *getenv(const char *name);
char *secure_getenv(const char *name);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`secure_getenv()`:
:   `_GNU_SOURCE`

## DESCRIPTION

`getenv()` 함수는 환경 목록에서 환경 변수 `name`을 탐색해서 대응하는 `value` 문자열 포인터를 반환한다.

GNU 한정인 `secure_getenv()` 함수는 `getenv()`와 비슷하되 "안전 실행"이 요구되는 경우에는 NULL을 반환한다. 호출 프로세스가 돌리는 프로그램이 적재될 때 다음 조건들 중 하나가 참이었으면 안전 실행이 요구된다.

* 프로세스의 실효 사용자 ID가 그 실제 사용자 ID와 일치하지 않았거나 프로세스의 실효 그룹 ID가 그 실제 그룹 ID와 일치하지 않았다. (보통 이는 set-user-ID 내지 set-group-ID 프로그램을 실행한 결과다.)

* 실행 파일에 실효 역능 비트가 설정돼 있었다.

* 프로세스의 허용 역능 집합이 비어 있지 않다.

어떤 리눅스 보안 모듈에 의해서 안전 실행이 요구될 수도 있다.

`secure_getenv()` 함수는 범용 라이브러리들에서 쓰기 위한 것으로, set-user-ID 내지 set-group-ID 프로그램에서 환경을 잘못 신뢰한 경우 발생할 수 있는 취약점들을 피하기 위한 것이다.

## RETURN VALUE

`getenv()` 함수는 환경 안의 값에 대한 포인터를 반환한다. 일치하는 변수가 없으면 NULL을 반환한다.

## VERSIONS

glibc 2.17에서 `secure_getenv()`가 처음 등장했다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `getenv()`, `secure_getenv()` | 스레드 안전성 | MT-Safe env |

## CONFORMING TO

`getenv()`: POSIX.1-2001, POSIX.1-2008, C89, C99, SVr4, 4.3BSD.

`secure_getenv()`는 GNU 확장이다.

## NOTES

환경 목록 안의 문자열들은 `name=value` 형태이다.

일반적인 구현 방식에서는 `getenv()`가 환경 목록 내의 문자열에 대한 포인터를 반환한다. 호출자가 그 문자열을 변경하지 않도록 주의해야 한다. 프로세스의 환경을 바꾸게 될 수 있기 때문이다.

`getenv()` 구현이 재진입 가능이 아닐 수도 있다. `getenv()`의 반환 값이 가리키는 문자열이 정적으로 할당돼 있을 수 있으며 그래서 이어지는 `getenv()`, <tt>[[putenv(3)]]</tt>, <tt>[[setenv(3)]]</tt>, <tt>[[unsetenv(3)]]</tt> 호출에 의해 바뀔 수 있다.

커널에서 사용자 공간으로 전달되는 보조 벡터에 담긴 `AT_SECURE` 플래그가 `secure_getenv()`의 "안전 실행" 모드를 제어한다.

## SEE ALSO

<tt>[[clearenv(3)]]</tt>, <tt>[[getauxval(3)]]</tt>, <tt>[[putenv(3)]]</tt>, <tt>[[setenv(3)]]</tt>, <tt>[[unsetenv(3)]]</tt>, <tt>[[capabilities(7)]]</tt>, <tt>[[environ(7)]]</tt>

----

2021-03-22
