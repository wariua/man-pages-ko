## NAME

crypt, crypt_r - 패스워드 및 데이터 암호화

## SYNOPSIS

```c
#include <unistd.h>

char *crypt(const char *key, const char *salt);

#include <crypt.h>

char *crypt_r(const char *key, const char *salt,
              struct crypt_data *restrict data);
```

`-lcrypt`로 링크.

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

`crypt()`:
:   glibc 2.28부터:
    :   `_DEFAULT_SOURCE`

    glibc 2.27 및 이전:
    :   `_XOPEN_SOURCE`

`crypt_r()`:
:   `_GNU_SOURCE`

## DESCRIPTION

`crypt()`는 패스워드 암호화 함수이다. 데이터 암호화 표준(Data Encryption Standard) 알고리듬을 바탕으로 하드웨어 구현 키 탐색을 어렵게 하기 위한 변경 등을 가해서 사용한다.

`key`는 사용자가 입력한 패스워드이다.

`salt`는 `[a-zA-Z0-9./]` 집합에서 고른 두 문자로 된 문자열이다. 이 문자열을 사용해 4096가지 방식 중 하나로 알고리듬에 변화를 준다.

`key`의 처음 여덟 글자 각각에서 하위 7비트를 뽑아서 56비트 키를 얻는다. 이 56비트 키를 사용해 상수 열(일반적으로 모두 0으로 된 열)을 반복적으로 암호화한다. 반환 값이 암호화된 패스워드를 가리키는데, 출력 가능한 ASCII 문자 13개로 된 열이다. (그 중 처음 두 문자는 솔트 자체를 나타낸다.) 반환 값이 가리키는 것이 정적 데이터이므로 매 호출마다 그 내용물을 덮어 쓴다.

경고: 키 공간을 이루는 값들이 2\*\*56 개, 즉 7.2e16 개이다. 대규모 병렬 컴퓨터를 쓰면 이 키 공간 전체를 전수 검색하는 게 가능하다. 또 `crack(1)` 같은 소프트웨어는 이 키 공간에서 사람들이 패스워드에 많이 쓰는 부분을 검색한다. 따라서 패스워드를 선정할 때는 적어도 흔한 단어나 이름은 피해야 한다. 그리고 깰 수 있는 패스워드인지 선정 과정에서 검사해 주는 `passwd(1)` 프로그램을 쓰기를 권장한다.

DES 알고리듬 자체에 있는 몇 가지 특이성 때문에 `crypt()` 인터페이스를 패스워드 인증 외의 용도에 쓰는 건 좋지 않은 선택이다. 혹시 암호 관련 프로젝트에 `crypt()`를 사용할 계획이라면 생각을 바꿔야 한다. 암호화에 대한 괜찮은 책을 하나 읽어 보고 널리 쓰는 DES 라이브러리들 중 하나를 찾아 보라.

`crypt_r()`은 `crypt()`의 재진입 가능 버전이다. `data`가 가리키는 구조체를 사용해 결과 데이터와 내부용 정보를 저장한다. 호출자가 이 구조체를 할당하는 것 외에 추가해 해 줄 일은 처음 `crypt_r()`를 호출하기 전에 `data->initialized`를 0으로 설정하는 것이다.

## RETURN VALUE

성공 시 암호화된 패스워드를 반환한다. 오류 시 NULL을 반환한다.

## ERRORS

`EINVAL`
:   `salt`가 잘못된 형식이다.

`ENOSYS`
:   `crypt()` 함수가 구현돼 있지 않다. 미국 수출 규제 때문일 수 있다.

`EPERM`
:   `/proc/sys/crypto/fips_enabled`에 0 아닌 값이 있는데 DES 같은 약한 방식 암호화를 사용하려는 시도가 이뤄졌다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `crypt()` | 스레드 안전성 | MT-Unsafe race:crypt |
| `crypt_r()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

`crypt()`: POSIX.1-2001, POSIX.1-2008, SVr4, 4.3BSD.

`crypt_r()`은 GNU 확장이다.

## NOTES

### glibc에서 사용 가능 여부

`crypt()`, <tt>[[encrypt(3)]]</tt>, <tt>[[setkey(3)]]</tt> 함수는 POSIX.1-2008 XSI Options Group for Encryption의 일부이며 선택적이다. 이 인터페이스들이 사용 가능하지 않은 경우에는 심볼 상수 `_XOPEN_CRYPT`가 정의돼 있지 않거나 -1로 정의돼 있으며, 런타임에는 <tt>[[sysconf(3)]]</tt>로 사용 가능 여부를 확인할 수 있다. 배포판에서 glibc의 crypt를 `libxcrypt`로 전환한 경우가 여기 해당할 수 있다. 그런 배포판에서 응용을 다시 컴파일할 때 프로그래머는 `_XOPEN_CRYPT`가 사용 가능하지 않은 걸 감지하면 함수 원형을 위해 `<crypt.h>`를 포함시켜야 한다. 사용 가능하지 않은 경우 `libxcrypt`가 ABI 호환인 대체물이 돼 준다.

### glibc에서의 기능들

이 함수의 glibc 버전에서는 다른 암호화 알고리듬들을 추가로 지원한다.

`salt`가 `$id$`로 시작해서 그 뒤에 선택적으로 "$"로 끝나는 문자열이 오는 경우에는 결과가 다음 형태가 된다.

```text
$id$salt$encrypted
```

`id`는 DES 대신 쓰는 암호화 방식을 나타내며 이 값이 패스워드 문자열 나머지를 어떻게 해석해야 하는지 결정한다. 다음 `id` 값을 지원한다.

| ID | 방법 |
| --- | --- |
| 1 | MD5 |
| 2a | 블로피시 (glibc 메인라인에 없음. 일부 리눅스 배포판에서 추가) |
| 5 | SHA-256 (glibc 2.7부터) |
| 6 | SHA-512 (glibc 2.7부터) |

그래서 `$5$salt$encrypted`와 `$6$salt$encrypted`는 각각 SHA-256 및 SHA-512 기반 함수로 암호화한 패스워드를 담고 있다.

"`salt`"는 솔트에서 "`$id$`" 다음에 오는 최대 16 문자를 나타낸다. 그리고 패스워드 문자열의 "`encrypted`" 부분은 실제 계산한 패스워드이다. 이 문자열의 크기는 고정돼 있다.

| | |
| --- | --- |
| MD5 | 22 문자 |
| SHA-256 | 43 문자 |
| SHA-512 | 86 문자 |

"`salt`" 및 "`encrypted`"의 문자들은 `[a-zA-Z0-9./]` 집합에서 고른다. MD5 및 SHA 구현에서는 (DES에서처럼 처음 8바이트가 아니라) `key` 전체를 계산에 쓴다.

glibc 2.7부터 SHA-256 및 SHA-512 구현에서 해싱 라운드 수를 사용자가 지정할 수 있으며 기본은 5000번이다. 솔트의 "`$id$`" 다음에 "`rounds=xxx$`"가 오는 경우에는 (`xxx`는 정수) 결과가 다음 형태가 된다.

```text
$id$rounds=yyy$salt$encrypted
```

여기서 `yyy`는 실제 적용된 해싱 라운드 수이다. 실제 적용된 라운드 수는 `xxx`가 1000보다 작은 경우에는 1000이고, `xxx`가 999999999보다 큰 경우에는 999999999이고, 그 외에는 `xxx`와 같다.

## SEE ALSO

`login(1)`, `passwd(1)`, <tt>[[encrypt(3)]]</tt>, <tt>[[getpass(3)]]</tt>, `passwd(5)`

----

2021-03-22
