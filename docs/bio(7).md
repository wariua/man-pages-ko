## NAME

bio - 기본(Basic) I/O 추상화

## SYNOPSIS

```c
#include <openssl/bio.h>
```

## DESCRIPTION

BIO는 I/O를 추상화한 것으로 기반 I/O의 여러 세부 사항들을 응용에게 감춰 준다. 응용에서 I/O에 BIO를 사용하면 SSL 연결, 비암호화 네트워크 연결, 파일 I/O를 투명하게 다룰 수 있다.

BIO에는 두 종류가 있는데, 원천/싱크 BIO와 필터 BIO이다.

이름에서 알 수 있듯 원천/싱크 BIO는 데이터의 원천 및/또는 싱크(sink)이다. 예로는 소켓 BIO와 파일 BIO가 있다.

필터 BIO는 한 BIO에서 데이터를 받아서 다른 BIO나 응용으로 전달한다. 데이터가 변경되지 않을 수도 있고 (가령 메시지 다이제스트 BIO) 변환될 수도 있다 (가령 암호화 BIO). 수행하는 I/O 동작에 따라 필터 BIO의 효과가 달라질 수도 있다. 예를 들어 암호화 BIO는 쓰는 경우에는 데이터를 암호화하고 읽는 경우에는 데이터를 복호화할 것이다.

BIO를 연결해서 사슬로 만들 수 있다. (단일 BIO는 한 개짜리 사슬이다.) 사슬은 일반적으로 한 개의 원천/싱크 BIO와 한 개 이상의 필터 BIO로 이뤄진다. 첫 번째 BIO에서 읽거나 쓴 데이터가 사슬을 거쳐 끝(일반적으로 원천/싱크 BIO)까지 간다.

(메모리 BIO 같은) 어떤 BIO들은 `BIO_new()` 호출 직후에 사용이 가능하다. (파일 BIO 같은) 다른 BIO들은 어떤 추가 초기화가 필요하며, 그런 BIO를 생성하고 초기화하기 위한 보조 함수가 존재하는 경우가 많다.

BIO 사슬에 `BIO_free()`를 호출하면 BIO 한 개만 해제하므로 메모리 누수가 발생한다.

단일 BIO에 `BIO_free_all()`을 호출하는 것은 반환 값을 버린다는 점을 제외하면 `BIO_free()`를 호출하는 것과 효과가 같다.

보통 `type` 인자는 BIO_METHOD 포인터를 반환하는 함수로 제공한다. 그런 함수들에는 명명 관행이 있는데, 보통 원천/싱크 BIO는 `BIO_s_*()`이고 필터 BIO는 `BIO_f_*()`이다.

## EXAMPLE

메모리 BIO 만들기:

```c
BIO *mem = BIO_new(BIO_s_mem());
```

## SEE ALSO

`BIO_ctrl(3)`, `BIO_f_base64(3)`, `BIO_f_buffer(3)`, `BIO_f_cipher(3)`, `BIO_f_md(3)`, `BIO_f_null(3)`, `BIO_f_ssl(3)`, `BIO_find_type(3)`, `BIO_new(3)`, `BIO_new_bio_pair(3)`, `BIO_push(3)`, `BIO_read_ex(3)`, `BIO_s_accept(3)`, `BIO_s_bio(3)`, `BIO_s_connect(3)`, `BIO_s_fd(3)`, `BIO_s_file(3)`, `BIO_s_mem(3)`, `BIO_s_null(3)`, `BIO_s_socket(3)`, `BIO_set_callback(3)`, `BIO_should_retry(3)`

## COPYRIGHT

Copyright 2000-2019 The OpenSSL Project Authors. All Rights Reserved.

Licensed under the OpenSSL license (the "License").  You may not use this file except in compliance with the License.  You can obtain a copy in the file LICENSE in the source distribution or at <https://www.openssl.org/source/license.html>.

----

2021-03-25
