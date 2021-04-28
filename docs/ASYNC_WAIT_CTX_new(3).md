## NAME

ASYNC_WAIT_CTX_new, ASYNC_WAIT_CTX_free, ASYNC_WAIT_CTX_set_wait_fd, ASYNC_WAIT_CTX_get_fd, ASYNC_WAIT_CTX_get_all_fds, ASYNC_WAIT_CTX_get_changed_fds, ASYNC_WAIT_CTX_clear_fd - 비동기 작업 완료 대기를 관리하는 함수들

## SYNOPSIS

```c
#include <openssl/async.h>

ASYNC_WAIT_CTX *ASYNC_WAIT_CTX_new(void);
void ASYNC_WAIT_CTX_free(ASYNC_WAIT_CTX *ctx);
int ASYNC_WAIT_CTX_set_wait_fd(ASYNC_WAIT_CTX *ctx, const void *key,
                               OSSL_ASYNC_FD fd,
                               void *custom_data,
                               void (*cleanup)(ASYNC_WAIT_CTX *, const void *,
                                               OSSL_ASYNC_FD, void *));
int ASYNC_WAIT_CTX_get_fd(ASYNC_WAIT_CTX *ctx, const void *key,
                          OSSL_ASYNC_FD *fd, void **custom_data);
int ASYNC_WAIT_CTX_get_all_fds(ASYNC_WAIT_CTX *ctx, OSSL_ASYNC_FD *fd,
                               size_t *numfds);
int ASYNC_WAIT_CTX_get_changed_fds(ASYNC_WAIT_CTX *ctx, OSSL_ASYNC_FD *addfd,
                                   size_t *numaddfds, OSSL_ASYNC_FD *delfd,
                                   size_t *numdelfds);
int ASYNC_WAIT_CTX_clear_fd(ASYNC_WAIT_CTX *ctx, const void *key);
```

## DESCRIPTION

OpenSSL에서 비동기 동작을 구현하는 방식에 대한 개요는 <tt>[[ASYNC_start_job(3)]]</tt>을 보라. ASYNC_WAIT_CTX 객체는 비동기 "세션", 즉 관련 crypto 연산들의 집합을 나타낸다. 예를 들어 SSL 용어로는 SSL 연결(connection)과 일대일로 대응하게 된다.

응용 코드에서 `ASYNC_WAIT_CTX_new()`를 이용해 ASYNC_WAIT_CTX를 생성한 후에 `ASYNC_start_job()`을 호출해야 한다. (<tt>[[ASYNC_start_job(3)]]</tt> 참고.) 작업 시작 때 그 작업과 ASYNC_WAIT_CTX가 연계된다. ASYNC_WAIT_CTX는 한번에 ASYNC_JOB 하나에만 사용해야 하며, ASYNC_JOB이 완료된 후에는 다른 ASYNC_JOB에 재사용할 수 있다. 세션이 완료됐을 때 (가령 SSL 연결이 닫혔을 때) 응용 코드에서 `ASYNC_WAIT_CTX_free()`로 정리를 한다.

ASYNC_WAIT_CTX에 "대기" 파일 디스크립터들을 연계할 수 있다. `ASYNC_WAIT_CTX_get_all_fds()`를 호출하면서 `ctx` 매개변수에 ASYNC_WAIT_CTX에 대한 포인터를 주면 그 작업에 연계된 대기 파일 디스크립터들을 `*fd`로 반환한다. 반환하는 파일 디스크립터들의 수를 `*numfds`에 저장한다. 모든 파일 디스크립터들을 받기에 충분한 메모리를 `*fd`에 할당하는 것은 호출자의 책임이다. `fd`를 NULL 값으로 해서 `ASYNC_WAIT_CTX_get_all_fds()`를 호출하면 파일 디스크립터를 반환하지 않지만 `*numfds`는 채운다. 따라서 보통은 응용 코드에서 이 함수를 두 번 호출하게 된다. 한 번은 fd 개수를 얻기 위해서이고 충분한 메모리를 할당하고서 다시 호출한다. 비동기 엔진을 한 개만 쓰고 있다면 이 호출이 항상 fd 한 개만 반환할 것이다. 비동기 엔진을 여러 개 쓰고 있다면 여러 개를 반환할 수 있다.

`ASYNC_WAIT_CTX_get_changed_fds()` 함수를 이용해 `ASYNC_start_job()`이 `ASYNC_PAUSE`를 반환했던 마지막 호출 시점 이후 (`ASYNC_PAUSE` 결과를 받은 적이 없다면 ASYNC_WAIT_CTX를 생성한 이후) 변경된 fd가 있는지 탐지할 수 있다. 추가되고 삭제된 fd 개수 각각을 `numaddfds` 및 `numdelfds` 매개변수에 채운다. 그리고 추가되고 삭제된 fd들의 목록을 각각 `*addfd` 및 `*delfd`에 채운다. `ASYNC_WAIT_CTX_get_all_fds()`에서처럼 두 매개변수 각각이 NULL일 수 있으며, NULL이 아닌 경우 충분한 메모리를 할당하는 것은 호출자의 책임이다.

비동기 지원 코드(가령 엔진)를 구현할 때는 자주 바뀌는 fd들이 "엎치락뒤치락" 하는 걸 줄이도록 ASYNC_WAIT_CTX의 수명 동안 안정적인 fd를 반환하는 게 좋다. 하지만 응용에게는 이에 대한 어떤 보장도 제공하지 않는다.

응용에서 select나 poll 같은 시스템 함수 호출을 이용해 파일 디스크립터가 "읽기" 준비가 되기를 기다릴 수 있다. ("읽기" 준비가 되는 것은 작업을 재개해야 한다는 표시이다.) 제공받는 파일 디스크립터가 없다면 응용에서 주기적으로 작업을 재시작하는 방식으로 작업이 실행을 이어갈 준비가 되어 있는지 확인해 봐야 할 것이다.

비동기 지원 코드(가령 엔진)에서 <tt>[[ASYNC_get_wait_ctx(3)]]</tt>를 통해 작업의 현재 ASYNC_WAIT_CTX를 얻을 수 있고 `ASYNC_WAIT_CTX_set_wait_fd()` 호출로 대기에 사용할 파일 디스크립터를 제공할 수 있다. 보통은 엔진에서 `ASYNC_pause_job()`을 호출하기 직전에 그렇게 하며 말단 사용자가 하지는 않는다. `ASYNC_WAIT_CTX_get_fd()`를 이용해 기존의 연계 파일 디스크립터를 얻을 수 있으며 `ASYNC_WAIT_CTX_clear_fd()`를 이용해 연계를 없앨 수 있다. 두 함수 모두 `key` 값을 필요로 하는데, 이는 그 비동기 지원 코드에 고유한 값이다. 유일한 어떤 값도 가능하지만 엔진에 대한 `ENGINE *`가 좋은 후보가 될 것이다. `custom_data` 매개변수에는 어떤 값도 가능하며 이후 `ASYNC_WAIT_CTX_get_fd()` 호출에서 그 값을 반환한다. `ASYNC_WAIT_CTX_set_wait_fd()` 함수에서는 "정리" 루틴에 대한 포인터도 기대한다. NULL일 수도 있지만 제공 시에는 그 ASYNC_WAIT_CTX가 해제될 때 자동으로 호출되므로 엔진에서 fd나 기타 자원을 닫을 기회가 된다. 참고: `ASYNC_WAIT_CTX_clear_fd()` 호출을 통해 fd를 직접 없애는 경우에는 "정리" 루틴이 호출되지 않는다.

일반적인 사용례로는 비동기 지원 엔진이 있을 수 있다. 일단 사용자 코드에서 암호 연산을 개시한다. 엔진이 그 연산을 비동기적으로 개시하고서 `ASYNC_WAIT_CTX_set_wait_fd()`에 이어 `ASYNC_pause_job()`을 호출해서 사용자 코드로 제어를 돌려준다. 그러면 사용자 코드에서는 다른 작업을 수행하거나 그 대기 파일 디스크립터에 "select" 내지 기타 유사 함수를 호출해서 작업이 준비 상태가 되기를 기다릴 수 있다. 그리고 엔진에서는 대기 파일 디스크립터를 "읽기 가능"으로 만들어서 작업을 재개해야 한다는 신호를 줄 수 있다. 실행이 재개되면 엔진에서는 대기 파일 디스크립터에서 깨우는 신호를 없애야 한다.

## RETURN VALUES

`ASYNC_WAIT_CTX_new()`는 새로 할당한 ASYNC_WAIT_CTX에 대한 포인터를 반환하며 오류 시 NULL을 반환한다.

`ASYNC_WAIT_CTX_set_wait_fd()`, `ASYNC_WAIT_CTX_get_fd()`, `ASYNC_WAIT_CTX_get_all_fds()`, `ASYNC_WAIT_CTX_get_changed_fds()`, `ASYNC_WAIT_CTX_clear_fd()`는 모두 성공 시 1을 반환하고 오류 시 0을 반환한다.

## NOTES

윈도우 플랫폼에서는 `openssl/async.h` 헤더에 필요한 몇 가지 타입들이 보통 `windows.h`를 포함시켜야 사용 가능해진다. 그런데 가장 먼저 포함시키는 헤더들 중 하나인 `windows.h`를 언제 포함시킬지를 응용 개발자가 통제할 수 있어야 하는 경우가 많다. 따라서 `async.h`에 앞서 `windows.h`를 포함시키는 것을 응용 개발자의 책임으로 규정한다.

## SEE ALSO

`crypto(7)`, <tt>[[ASYNC_start_job(3)]]</tt>

## HISTORY

OpenSSL 1.1.0에서 `ASYNC_WAIT_CTX_new()`, `ASYNC_WAIT_CTX_free()`, `ASYNC_WAIT_CTX_set_wait_fd()`, `ASYNC_WAIT_CTX_get_fd()`, `ASYNC_WAIT_CTX_get_all_fds()`, `ASYNC_WAIT_CTX_get_changed_fds()`, `ASYNC_WAIT_CTX_clear_fd()`가 처음 추가되었다.

## COPYRIGHT

Copyright 2016-2020 The OpenSSL Project Authors. All Rights Reserved.

Licensed under the OpenSSL license (the "License").  You may not use this file except in compliance with the License.  You can obtain a copy in the file LICENSE in the source distribution or at <https://www.openssl.org/source/license.html>.

----

2021-03-25
