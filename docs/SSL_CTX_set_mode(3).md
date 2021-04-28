## NAME

SSL_CTX_set_mode, SSL_CTX_clear_mode, SSL_set_mode, SSL_clear_mode, SSL_CTX_get_mode, SSL_get_mode - SSL 엔진 모드 조작하기

## SYNOPSIS

```c
#include <openssl/ssl.h>

long SSL_CTX_set_mode(SSL_CTX *ctx, long mode);
long SSL_CTX_clear_mode(SSL_CTX *ctx, long mode);
long SSL_set_mode(SSL *ssl, long mode);
long SSL_clear_mode(SSL *ssl, long mode);

long SSL_CTX_get_mode(SSL_CTX *ctx);
long SSL_get_mode(SSL *ssl);
```

## DESCRIPTION

`SSL_CTX_set_mode()`는 `mode`의 비트마스크에 설정한 모드를 `ctx`에 추가한다. 이미 설정한 옵션들은 해제되지 않는다. `SSL_CTX_clear_mode()`는 `mode`의 비트마스크에 설정한 모드를 `ctx`에서 제거한다.

`SSL_set_mode()`는 `mode`의 비트마스크에 설정한 모드를 `ssl`에 추가한다. 이미 설정한 옵션들은 해제되지 않는다. `SSL_clear_mode()`는 `mode`의 비트마스크에 설정한 모드를 `ssl`에서 제거한다.

`SSL_CTX_get_mode()`는 `ctx`에 설정된 모드를 반환한다.

`SSL_get_mode()`는 `ssl`에 설정된 모드를 반환한다.

## NOTES

다음 모드 변경이 가능하다.

`SSL_MODE_ENABLE_PARTIAL_WRITE`
:   `SSL_write_ex(..., n, &r)`가 `0 < r < n`인 `r`을 반환하는 것을 (즉 한 바이트만 썼을 때에도 성공을 보고하는 것을) 허용한다. `SSL_write()`에도 비슷하게 동작한다. 설정돼 있지 않으면 (기본 동작) `SSL_write_ex()` 내지 `SSL_write()`가 덩어리 전체를 쓴 다음에만 성공을 보고한다. `SSL_write_ex()` 내지 `SSL_write()`가 성공으로 반환하고 나면 `r` 바이트만 써진 것이므로 다음 `SSL_write_ex()` 내지 `SSL_write()` 호출에서는 남아 있는 `n-r` 바이트만 보내야 한다. 즉 `write()`와 비슷하다.

`SSL_MODE_ACCEPT_MOVING_WRITE_BUFFER`
:   바뀐 버퍼 위치로 `SSL_write_ex()` 내지 `SSL_write()`를 재시도할 수 있게 만든다. (버퍼 내용물은 동일하게 유지돼야 한다.) 이 동작 방식이 기본이 아닌 건 논블로킹 `SSL_write()`가 논블로킹 `write()`처럼 동작한다는 오인을 피하기 위해서이다.

`SSL_MODE_AUTO_RETRY`
:   정상 동작 중에도 응용에서 눈치채지 못하는 비응용 데이터 레코드를 주고받아야 할 수도 있다. `SSL_read_ex(3)` 및 `SSL_read(3)`에서 비응용 데이터 레코드를 처리하면 실패를 반환하고 `SSL_ERROR_WANT_READ`로 재시도 필요를 나타낼 수 있다. `SSL_MODE_AUTO_RETRY` 플래그는 그런 비응용 데이터 레코드를 처리한 경우에 반환하지 말고 다음 레코드 처리를 시도하게 만든다.

    논블로킹 환경에서는 읽기/쓰기 동작의 불완전한 처리를 응용에서 다룰 준비가 되어 있을 것이다. 논블로킹 `BIO`에 `SSL_MODE_AUTO_RETRY`를 설정하면 더이상 데이터가 없거나 응용 데이터 레코드를 처리할 때까지 비응용 데이터 레코드들을 처리하게 된다.

    블로킹 환경에서는 재시도 요청 같은 중간 보고를 반환하는 함수를 다룰 준비가 항상 되어 있지는 않은데, `SSL_MODE_AUTO_RETRY` 플래그를 설정하면 응용 데이터 레코드를 성공적으로 처리한 후나 실패 시에만 함수가 반환하게 된다.

    블로킹 `BIO`를 `select()`나 `poll()` 같은 것과 조합해 쓰는 경우에는 `SSL_MODE_AUTO_RETRY`를 끄는 게 유용할 수 있다. 안 그러면 비응용 레코드만 오고 응용 데이터는 안 온 경우에 `SSL_read()` 내지 `SSL_read_ex()` 호출에서 실행이 멈출 수도 있다.

`SSL_MODE_RELEASE_BUFFERS`
:   어떤 SSL에 대해 읽기 버퍼나 쓰기 버퍼가 더는 필요하지 않으면 버퍼에 사용 중인 메모리를 해제한다. 이 플래그를 쓰면 유휴 SSL 연결당 34k 정도를 절약할 수 있다. 이 플래그는 SSL v2 연결이나 DTLS 연결에는 효과가 없다.

`SSL_MODE_SEND_FALLBACK_SCSV`
:   ClientHello에 TLS_FALLBACK_SCSV를 보낸다. 프로토콜 버전을 내려서 재연결하는 응용에서만 설정하게 된다. 자세한 내용은 draft-ietf-tls-downgrade-scsv-00을 보라.

    응용에서 일반적인 핸드셰이크를 시도하는 경우에는 절대 이 모드를 켜지 마라. 명확하게 후퇴 재시도를 하는 경우에만 draft-ietf-tls-downgrade-scsv-00의 지침에 따라서 사용해야 한다.

`SSL_MODE_ASYNC`
:   비동기 처리를 켠다. 이 모드가 설정돼 있으면 암호 연산 수행에 비동기 지원 엔진을 쓰는 경우 TLS I/O 동작이 `SSL_ERROR_WANT_ASYNC`로 재시도 필요를 나타낼 수 있다. `SSL_get_error(3)` 참고.

`SSL_MODE_DTLS_SCTP_LABEL_LENGTH_BUG`
:   OpenSSL 이전 버전들에는 종단간 공유 비밀값 계산에 쓰는 레이블 길이를 계산하는 데 버그가 있었다. 마지막의 0이 레이블 길이에 포함되는 버그였다. 이 옵션을 설정하면 그 동작 방식을 켜서 그런 잘못된 구현체와 연동이 가능하게 한다. 이 옵션을 설정하면 올바른 구현체와의 연동에 문제가 생긴다는 점에 유의하라. SCTP 상의 DTLS에만 적용된다.

기본적으로 모든 모드가 꺼져 있다. 단, SSL_MODE_AUTO_RETRY 모드는 1.1.1부터 기본적으로 켜져 있다.

## RETURN VALUES

`SSL_CTX_set_mode()`와 `SSL_set_mode()`는 `mode` 추가 후의 새 모드 비트마스크를 반환한다.

`SSL_CTX_get_mode()`와 `SSL_get_mode()`는 현재 비트마스크를 반환한다.

## SEE ALSO

`ssl(7)`, `SSL_read_ex(3)`, `SSL_read(3)`, `SSL_write_ex(3)`, `SSL_write(3)`, `SSL_get_error(3)`

## HISTORY

OpenSSL 1.1.0에서 `SSL_MODE_ASYNC`가 처음 추가되었다.

## COPYRIGHT

Copyright 2001-2020 The OpenSSL Project Authors. All Rights Reserved.

Licensed under the OpenSSL license (the "License").  You may not use this file except in compliance with the License.  You can obtain a copy in the file LICENSE in the source distribution or at <https://www.openssl.org/source/license.html>.

----

2021-03-25
