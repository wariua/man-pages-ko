## NAME

encrypt, setkey, encrypt_r, setkey_r - 64비트 메시지 암호화하기

## SYNOPSIS

```c
#define _XOPEN_SOURCE       /* feature_test_macros(7) 참고 */
#include <unistd.h>

void encrypt(char block[64], int edflag);

#define _XOPEN_SOURCE       /* feature_test_macros(7) 참고 */
#include <stdlib.h>

void setkey(const char *key);

#define _GNU_SOURCE         /* feature_test_macros(7) 참고 */
#include <crypt.h>

void setkey_r(const char *key, struct crypt_data *data);
void encrypt_r(char *block, int edflag, struct crypt_data *data);
```

모두 `-lcrypt`로 링크 필요.

## DESCRIPTION

이 함수들은 64비트 메시지를 암호화 및 복호화한다. `setkey()` 함수는 `encrypt()`에서 쓰는 키를 설정한다. 여기 쓰이는 `key` 인자는 64바이트짜리 배열이며 각 바이트가 숫자 값 1 또는 0이다. n=8\*i-1인 key[n]은 무시하며, 따라서 실제 키 길이는 56비트다.

`encrypt()` 함수는 전달받은 버퍼를 변경하는데, `edflag`가 0이면 암호화, 1이면 복호화다. `key` 인자처럼 `block` 역시도 암호화하는 실제 값의 비트 벡터 표현이다. 같은 벡터로 결과가 반환된다.

이 두 함수는 재진입 가능하지 않다. 즉 키 데이터를 정적 저장 공간에 둔다. `setkey_r()` 및 `encrypt_r()` 함수는 재진입 가능 버전이다. 다음 구조체에 키 데이터를 담는다.

```c
struct crypt_data {
    char     keysched[16 * 8];
    char     sb0[32768];
    char     sb1[32768];
    char     sb2[32768];
    char     sb3[32768];
    char     crypt_3_buf[14];
    char     current_salt[2];
    long int current_saltbits;
    int      direction;
    int      initialized;
};
```

`setkey_r()`을 호출하기 전에 `data->initialized`를 0으로 설정해야 한다.

## RETURN VALUE

이 함수들은 아무 값도 반환하지 않는다.

## ERRORS

위 함수들을 호출하기 전에 `errno`를 0으로 설정해야 한다. 성공 시에는 바뀌지 않는다.

<dl>
<dt><code>ENOSYS</code></dt>
<dd>함수가 제공되지 않는다. (예를 들어 과거의 미국 수출 규제 때문에.)</dd>
</dl>

## VERSIONS

더 이상 안전하지 않다고 보는 DES 블록 암호를 이용하기 때문에 `crypt()`, `crypt_r()`, `setkey()`, `setkey_r()`이 glibc 2.28에서 제거되었다. 응용들은 `libgcrypt` 같은 현대적인 암호 라이브러리로 전환하는 게 좋다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `encrypt()`, `setkey()` | 스레드 안전성 | MT-Unsafe race:crypt |
| `encrypt_r()`, `setkey_r()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

`encrypt()`, `setkey()`: POSIX.1-2001, POSIX.1-2008, SUS, SVr4.

함수 `encrypt_r()`과 `setkey_r()`은 GNU 확장이다.

## NOTES

### glibc에서 사용 가능 여부

<tt>[[crypt(3)]]</tt> 참고.

### glibc에서의 기능들

glibc 2.2에서 이 함수들은 DES 알고리듬을 사용한다.

## EXAMPLE

```c
#define _XOPEN_SOURCE
#include <stdio.h> #include <stdlib.h>
#include <unistd.h>
#include <crypt.h>

int
main(void)
{
    char key[64];
    char orig[9] = "eggplant";
    char buf[64];
    char txt[9];
    int i, j;

    for (i = 0; i < 64; i++) {
        key[i] = rand() & 1;
    }

    for (i = 0; i < 8; i++) {
        for (j = 0; j < 8; j++) {
            buf[i * 8 + j] = orig[i] >> j & 1;
        }
        setkey(key);
    }
    printf("Before encrypting: %s\n", orig);

    encrypt(buf, 0);
    for (i = 0; i < 8; i++) {
        for (j = 0, txt[i] = '\0'; j < 8; j++) {
            txt[i] |= buf[i * 8 + j] << j;
        }
        txt[8] = '\0';
    }
    printf("After encrypting:  %s\n", txt);

    encrypt(buf, 1);
    for (i = 0; i < 8; i++) {
        for (j = 0, txt[i] = '\0'; j < 8; j++) {
            txt[i] |= buf[i * 8 + j] << j;
        }
        txt[8] = '\0';
    }
    printf("After decrypting:  %s\n", txt);
    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

<tt>[[cbc_crypt(3)]]</tt>, <tt>[[crypt(3)]]</tt>, <tt>[[ecb_crypt(3)]]</tt>

----

2018-04-30
