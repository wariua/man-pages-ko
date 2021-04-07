## NAME

evp - 고수준 암호 함수

## SYNOPSIS

```c
#include <openssl/evp.h>
```

## DESCRIPTION

EVP 라이브러리는 암호 함수들에 대한 고수준 인터페이스를 제공한다.

`EVP_Seal*` 및 `EVP_Open*` 함수는 전자 "봉투(envelope)"를 구현하기 위한 공개키 암호화 및 복호화를 제공한다.

`EVP_DigestSign*` 및 `EVP_DigestVerify*` 함수는 전자 서명 및 메시지 인증 코드(MAC)를 구현한다. 이전의 `EVP_Sign*` 및 `EVP_Verify*` 함수도 참고하라.

대칭 암호는 `EVP_Encrypt*` 함수로 사용 가능하다. `EVP_Digest*` 함수는 메시지 다이제스트를 제공한다.

`EVP_PKEY*` 함수는 비대칭 알고리듬에 대한 고수준 인터페이스를 제공한다. 새 EVP_PKEY를 만드는 건 `EVP_PKEY_new(3)`를 보라. `EVP_PKEY_set1_RSA(3)` 페이지에서 기술하는 함수들을 이용해 EVP_PKEY를 특정 알고리듬의 개인키에 연계하거나 `EVP_PKEY_keygen(3)`으로 새 키를 생성할 수 있다. `EVP_PKEY_cmp(3)`으로 EVP_PKEY를 비교하거나 `EVP_PKEY_print_private(3)`으로 찍을 수 있다.

EVP_PKEY 함수들은 비대칭 알고리듬 연산 전체를 지원한다.

* 키 협상은 `EVP_PKEY_derive(3)`를 보라.

* 서명과 검증은 `EVP_PKEY_sign(3)`, `EVP_PKEY_verify(3)`, `EVP_PKEY_verify_recover(3)`를 보라. 단 이 함수들은 서명하는 데이터에 다이제스트를 수행하지 않는다. 따라서 보통은 `EVP_DigestSignInit(3)` 함수들을 쓰게 될 것이다.

* 암호화와 복호화는 `EVP_PKEY_encrypt(3)`와 `EVP_PKEY_decrypt(3)`를 보라. 단 이 함수들은 암호화와 복호화만 수행한다. 공개키 연산은 비싼 연산이므로 보통은 암호화되는 메시지를 `EVP_SealInit(3)` 및 `EVP_OpenInit(3)` 함수를 이용해 "전자 봉투"에 넣게 될 것이다.

`EVP_BytesToKey(3)` 함수는 좀 제한된 패스워드 기반 암호화를 지원한다. 매개변수들을 조심스럽게 선정하면 PKCS#5 PBKDF1 호환 구현이 가능하다. 하지만 새로 만드는 응용에서는 보통 이를 쓰지 않는 게 좋다. (대신 PKCS#5의 PBKDF2 등을 쓰는 게 좋다.)

`EVP_Encode*` 및 `EVP_Decode*` 함수는 베이스64 인코딩 및 디코딩을 구현한다.

대체 구현을 제공하는 ENGINE 모듈이 있으면 대칭 알고리듬(암호), 다이제스트, 비대칭 알고리듬(공개키 알고리듬) 모두를 대신할 수 있다. 암호나 다이제스트의 ENGINE 구현이 기본으로 등록되어 있으면 여러 EVP 함수들에서 자동으로 내장 소프트웨어 구현 대신 그 구현을 사용하게 된다. 더 자세한 내용은 `engine(3)` 맨 페이지를 확인하라.

여러 알고리듬들에 저수준의 알고리듬별 함수가 존재하기는 하지만 사용을 권하지 않는다. ENGINE과 함께 쓸 수 없으며 그 저수준 함수들로는 새 알고리듬의 ENGINE 버전에 접근할 수 없다. 또한 코드에 새 알고리듬을 도입하기 힘들게 만들고, 저수준에서는 일부 옵션들을 깔끔하게 지원하지 않으며, 어떤 연산들은 고수준 인터페이스를 쓸 때 더 효율적이다.

## SEE ALSO

`EVP_DigestInit(3)`, `EVP_EncryptInit(3)`, `EVP_OpenInit(3)`, `EVP_SealInit(3)`, `EVP_DigestSignInit(3)`, `EVP_SignInit(3)`, `EVP_VerifyInit(3)`, `EVP_EncodeInit(3)`, `EVP_PKEY_new(3)`, `EVP_PKEY_set1_RSA(3)`, `EVP_PKEY_keygen(3)`, `EVP_PKEY_print_private(3)`, `EVP_PKEY_decrypt(3)`, `EVP_PKEY_encrypt(3)`, `EVP_PKEY_sign(3)`, `EVP_PKEY_verify(3)`, `EVP_PKEY_verify_recover(3)`, `EVP_PKEY_derive(3)`, `EVP_BytesToKey(3)`, `ENGINE_by_id(3)`

## COPYRIGHT

Copyright 2000-2018 The OpenSSL Project Authors. All Rights Reserved.

Licensed under the OpenSSL license (the "License").  You may not use this file except in compliance with the License.  You can obtain a copy in the file LICENSE in the source distribution or at <https://www.openssl.org/source/license.html>.

----

2018-03-12
