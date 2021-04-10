## NAME

SSL_CTX_set_mode, SSL_set_mode, SSL_CTX_get_mode, SSL_get_mode - SSL 엔진 모드 조작하기

## SYNOPSIS

```c
#include <openssl/ssl.h>

long SSL_CTX_set_mode(SSL_CTX *ctx, long mode);
long SSL_set_mode(SSL *ssl, long mode);

long SSL_CTX_get_mode(SSL_CTX *ctx);
long SSL_get_mode(SSL *ssl);
```

## DESCRIPTION

`SSL_CTX_set_mode()`는 `mode`의 비트마스크에 설정한 모드를 `ctx`에 추가한다. 이미 설정한 옵션들은 해제되지 않는다.

`SSL_set_mode()`는 `mode`의 비트마스크에 설정한 모드를 `ssl`에 추가한다. 이미 설정한 옵션들은 해제되지 않는다.

`SSL_CTX_get_mode()`는 `ctx`에 설정된 모드를 반환한다.

`SSL_get_mode()`는 `ssl`에 설정된 모드를 반환한다.

## NOTES

다음 모드 변경이 가능하다.

`SSL_MODE_ENABLE_PARTIAL_WRITE`
:   `SSL_write_ex(..., n, &r)`가 `0 < r < n`인 `r`을 반환하는 것을 (즉 한 바이트만 썼을 때에도 성공을 보고하는 것을) 허용한다. `SSL_write()`에도 비슷하게 동작한다. 설정돼 있지 않으면 (기본 동작) `SSL_write_ex()` 내지 `SSL_write()`가 덩어리 전체를 쓴 다음에만 성공을 보고한다. `SSL_write_ex()` 내지 `SSL_write()`가 성공으로 반환하고 나면 `r` 바이트만 써진 것이므로 다음 `SSL_write_ex()` 내지 `SSL_write()` 호출에서는 남아 있는 `n-r` 바이트만 보내야 한다. 즉 `write()`와 비슷하다.

`SSL_MODE_ACCEPT_MOVING_WRITE_BUFFER`
:   바뀐 버퍼 위치로 `SSL_write_ex()` 내지 `SSL_write()`를 재시도할 수 있게 만든다. (버퍼 내용물은 동일하게 유지돼야 한다.) 이 동작 방식이 기본이 아닌 건 논블로킹 `SSL_write()`가 논블로킹 `write()`처럼 동작한다는 오인을 피하기 위해서이다.

`SSL_MODE_AUTO_RETRY`
:   전송에서 블록 되는 경우에 응용에서 귀찮게 재시도할 필요가 없게 해 준다. 정상 동작 중 재협상이 일어나면 `SSL_read_ex(3)`, `SSL_read(3)`, `SSL_write_ex(3)`, `SSL_write(3)`는 `SSL_ERROR_WANT_READ`로 오류를 반환하며 재시도 필요를 나타낸다. 논블로킹 환경에서는 불완전 읽기/쓰기 동작을 응용에서 처리할 준비가 되어 있어야 한다. 하지만 블로킹 환경에서는 읽기/쓰기 동작이 오류 보고 없이 반환하는 걸 응용에서 다룰 준비가 항상 되어 있지가 않다. `SSL_MODE_AUTO_RETRY` 플래그는 핸드셰이크 및 성공 완료 후에만 읽기/쓰기 동작이 반환하게 한다.

`SSL_MODE_RELEASE_BUFFERS`
:   어떤 SSL에 대해 읽기 버퍼나 쓰기 버퍼가 더는 필요하지 않으면 버퍼에 사용 중인 메모리를 해제한다. 이 플래그를 쓰면 유휴 SSL 연결당 34k 정도를 절약할 수 있다. 이 플래그는 SSL v2 연결이나 DTLS 연결에는 효과가 없다.

`SSL_MODE_SEND_FALLBACK_SCSV`
:   ClientHello에 TLS_FALLBACK_SCSV를 보낸다. 프로토콜 버전을 내려서 재연결하는 응용에서만 설정하게 된다. 자세한 내용은 draft-ietf-tls-downgrade-scsv-00을 보라.

    응용에서 일반적인 핸드셰이크를 시도하는 경우에는 절대 이 플래크를 켜지 마라. 명확하게 후퇴 재시도를 하는 경우에만 draft-ietf-tls-downgrade-scsv-00의 지침에 따라서 사용해야 한다.

`SSL_MODE_ASYNC`
:   비동기 처리를 켠다. 이 모드가 설정돼 있으면 암호 연산 수행에 비동기 지원 엔진을 쓰는 경우 TLS I/O 동작이 `SSL_ERROR_WANT_ASYNC`로 재시도 필요를 나타낼 수 있다. `SSL_get_error(3)` 참고.

## RETURN VALUES

`SSL_CTX_set_mode()`와 `SSL_set_mode()`는 `mode` 추가 후의 새 모드 비트마스크를 반환한다.

`SSL_CTX_get_mode()`와 `SSL_get_mode()`는 현재 비트마스크를 반환한다.

## SEE ALSO

`ssl(7)`, `SSL_read_ex(3)`, `SSL_read(3)`, `SSL_write_ex(3)`, `SSL_write(3)`, `SSL_get_error(3)`

## HISTORY

OpenSSL 1.1.0에서 `SSL_MODE_ASYNC`가 처음 추가되었다.

## COPYRIGHT

Copyright 2001-2016 The OpenSSL Project Authors. All Rights Reserved.

Licensed under the OpenSSL license (the "License").  You may not use this file except in compliance with the License.  You can obtain a copy in the file LICENSE in the source distribution or at <https://www.openssl.org/source/license.html>.

----

2017-12-31
