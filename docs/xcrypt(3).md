## NAME

xencrypt, xdecrypt, passwd2des - RFS 패스워드 암호화

## SYNOPSIS

```c
#include <rpc/des_crypt.h>

void passwd2des(char *passwd, char *key);

int xencrypt(char *secret, char *passwd);

int xdecrypt(char *secret, char *passwd);
```

## DESCRIPTION

**경고**: 새 코드에서는 이 함수들을 사용하지 말아야 한다. 어떤 식의 충분한 암호학적 보안성도 보장하지 못한다.

`passwd2des()` 함수는 임의 길이의 문자열 `passwd`를 받아서 길이 8인 문자 배열 `key`를 채운다. 배열 `key`는 DES 키에 쓰기에 적합하다. 즉 각 바이트의 0번 비트에 홀수 패리티가 설정돼 있다. 여기 설명하는 다른 두 함수는 모두 이 함수를 이용해 인자 `passwd`를 DES 키로 바꾼다.

`xencrypt()` 함수는 16진수 ASCII 문자열 `secret`을 받아서 (그 문자열의 길이는 16의 배수여야 한다.) `passwd2des()`를 써서 `passwd`로부터 얻은 DES 키를 이용해 암호화한 다음 같은 길이의 16진수 문자열인 결과를 `secret`으로 내놓는다.

`xdecrypt()` 함수는 반대 동작을 수행한다.

## RETURN VALUE

`xencrypt()` 및 `xdecrypt()` 함수는 성공 시 1을 반환하고 오류 시 0을 반환한다.

## VERSIONS

glibc 2.1 및 이후에서 이 함수들이 사용 가능하다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `passwd2des`, `xencrypt()`, `xdecrypt()` | 스레드 안전성 | MT-Safe |

## BUGS

위에 언급된 헤더 파일에 원형이 빠져 있다.

## SEE ALSO

<tt>[[cbc_crypt(3)]]</tt>

----

2019-03-06
