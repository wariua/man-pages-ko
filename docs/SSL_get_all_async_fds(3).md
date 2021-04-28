## NAME

SSL_waiting_for_async, SSL_get_all_async_fds, SSL_get_changed_async_fds - 비동기 동작 관리하기

## SYNOPSIS

```c
#include <openssl/async.h>
#include <openssl/ssl.h>

int SSL_waiting_for_async(SSL *s);
int SSL_get_all_async_fds(SSL *s, OSSL_ASYNC_FD *fd, size_t *numfds);
int SSL_get_changed_async_fds(SSL *s, OSSL_ASYNC_FD *addfd, size_t *numaddfds,
                              OSSL_ASYNC_FD *delfd, size_t *numdelfds);
```

## DESCRIPTION

`SSL_waiting_for_async()`는 SSL 연결이 현재 비동기 동작이 완료되기를 기다리고 있는지 알아낸다. (<tt>[[SSL_CTX_set_mode(3)]]</tt>의 `SSL_MODE_ASYNC` 모드 참고.)

`SSL_get_all_async_fds()`는 파일 디스크립터들의 목록을 반환하는데, 이를 `select()`나 `poll()` 호출에 써서 현재 비동기 동작이 완료되었는지 여부를 알아낼 수 있다. 동작이 완료되면 파일 디스크립터에 데이터가 "읽기 준비"인 것으로 나타나게 된다. (그 파일 디스크립터에서 실제로 데이터를 읽지는 말아야 한다.) SSL 객체에서 현재 비동기 작업 완료를 기다리고 있는 경우에만 (즉 `SSL_ERROR_WANT_ASYNC`를 받았을 때만 - `SSL_get_error(3)` 참고) 이 함수를 호출해야 한다. 보통 그 목록에는 파일 디스크립터가 한 개만 있을 것이다. 하지만 비동기 지원 엔진을 여러 개 사용 중이라면 여러 개가 있을 수도 있다. 반환되는 파일 디스크립터 수가 `*numfds`에 저장되고 파일 디스크립터들이 `*fds`에 저장된다. `fds` 매개변수는 NULL일 수도 있고, 그 경우 어떤 파일 디스크립터도 반환되지 않지만 `*numfds`는 채워진다. `*fds`에 충분한 메모리를 할당하는 것은 호출자의 책임이므로 보통은 이 함수를 두 번 (한 번은 `fds` 매개변수를 NULL로, 또 한 번은 아니게) 호출한다.

`SSL_get_changed_async_fds()`는 마지막으로 `SSL_ERROR_WANT_ASYNC`를 받은 이후 (`SSL_ERROR_WANT_ASYNC`를 받은 적이 없다면 SSL 객체를 생성한 이후) 목록에 추가되었거나 목록에서 삭제된 비동기 파일 디스크립터들의 목록을 반환한다. `SSL_get_all_async_fds()`처럼 `*addfd`와 `*delfd`에 충분한 메모리를 할당하는 것은 호출자의 책임이며, NULL일 수도 있다. 추가된 fd 개수와 삭제된 fd 개수가 각각 `*numaddfds`와 `*numdelfds`에 저장된다.

## RETURN VALUES

`SSL_waiting_for_async()`는 현재 SSL 동작이 비동기 동작 완료를 기다리고 있으면 1을 반환하고 아니면 0을 반환한다.

`SSL_get_all_async_fds()`와 `SSL_get_changed_async_fds()`는 성공 시 1을 반환하고 오류 시 0을 반환한다.

## NOTES

윈도우 플랫폼에서는 `openssl/async.h` 헤더에 필요한 몇 가지 타입들이 보통 `windows.h`를 포함시켜야 사용 가능해진다. 그런데 가장 먼저 포함시키는 헤더들 중 하나인 `windows.h`를 언제 포함시킬지를 응용 개발자가 통제할 수 있어야 하는 경우가 많다. 따라서 `async.h`에 앞서 `windows.h`를 포함시키는 것을 응용 개발자의 책임으로 규정한다.

## SEE ALSO

`SSL_get_error(3)`, <tt>[[SSL_CTX_set_mode(3)]]</tt>

## HISTORY

OpenSSL 1.1.0에서 `SSL_waiting_for_async()`, `SSL_get_all_async_fds()`, `SSL_get_changed_async_fds()`가 처음 추가되었다.

## COPYRIGHT

Copyright 2016-2020 The OpenSSL Project Authors. All Rights Reserved.

Licensed under the OpenSSL license (the "License").  You may not use this file except in compliance with the License.  You can obtain a copy in the file LICENSE in the source distribution or at <https://www.openssl.org/source/license.html>.

----

2021-03-25
