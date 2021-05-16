## NAME

des_crypt, ecb_crypt, cbc_crypt, des_setparity, DES_FAILED - 빠른 DES 암호화

## SYNOPSIS

```c
#include <rpc/des_crypt.h>

int ecb_crypt(char *key, char *data, unsigned int datalen,
              unsigned int mode);
int cbc_crypt(char *key, char *data, unsigned int datalen,
              unsigned int mode, char *ivec);

void des_setparity(char *key);

int DES_FAILED(int status);
```

## DESCRIPTION

`ecb_crypt()`와 `cbc_crypt()`는 NBS DES(데이터 암호화 표준)을 구현한다. 이 루틴들은 <tt>[[crypt(3)]]</tt>보다 더 빠르고 범용적이다. 또한 DES 하드웨어가 있으면 그걸 활용할 수 있다. `ecb_crypt()`는 ECB(전자 코드북) 모드로 암호화를 하는데, 각 데이터 블록을 독립적으로 암호화하는 방식이다. `cbc_crypt()`는 CBC(암호 블록 체인) 모드로 암호화를 하는데, 연속한 블록들을 연결하는 방식이다. CBC 모드는 블록 삽입, 삭제, 바꿔치기를 방어할 수 있다. 또한 평문 내의 규칙성이 암호문에 등장하지 않게 된다.

이 루틴들을 사용하는 방법은 이렇다. 첫 번째 인자 `key`는 패리티 있는 8바이트 암호화 키이다. 키의 패리티는 DES에서는 각 바이트의 최하위 비트이며 `des_setparity()`로 설정한다. 두 번째 인자 `data`는 암호화나 복호화를 할 데이터를 담는다. 세 번째 인자 `datalen`은 `data`의 바이트 단위 길이인데 8의 배수여야 한다. 네 번째 인자 `mode`는 몇 가지를 OR 해서 만든다. 암호화 방향으로 `DES_ENCRYPT` 아니면 `DES_DECRYPT`를 OR 한다. 그리고 소프트웨어 대 하드웨어 암호화로 `DES_HW` 아니면 `DES_SW`를 OR 한다. `DES_HW`를 지정했는데 하드웨어가 없으면 소프트웨어에서 암호화를 수행하고서 루틴이 `DESERR_NOHWDEVICE`를 반환한다. `cbc_crypt()`에서 `ivec` 인자는 체인 처리를 위한 8바이트 초기화 벡터이다. 반환 시에 다음 초기화 벡터로 갱신돼 있다.

## RETURN VALUE

`DESERR_NONE`
:   오류 없음.

`DESERR_NOHWDEVICE`
:   암호화가 성공하긴 했지만 요청한 하드웨어가 아니라 소프트웨어에서 이뤄졌다.

`DESERR_HWERROR`
:   하드웨어나 드라이버에서 오류가 발생했다.

`DESERR_BADPARAM`
:   루틴 인자가 잘못되었다.

결과 상태 `stat`이 있을 때 처음 두 가지 상태에 대해서만 매크로 `DES_FAILED(stat)`이 거짓이다.

## VERSIONS

glibc 버전 2.1에서 이 함수들이 추가되었다.

더이상 안전하지 않다고 보는 DES 블록 암호를 이용하기 때문에 `ecb_crypt()`, `cbc_crypt()`, `crypt_r()`, `des_setparity()`가 glibc 2.28에서 제거되었다. 응용들은 `libgcrypt` 같은 현대적인 암호 라이브러리로 전환하는 게 좋다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `ecb_crypt()`, `cbc_crypt()`, `des_setparity()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

4.3BSD. POSIX.1에는 없음.

## SEE ALSO

`des(1)`, <tt>[[crypt(3)]]</tt>, <tt>[[xcrypt(3)]]</tt>

----

2021-03-22
