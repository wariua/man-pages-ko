## NAME

uname - 현재 커널에 대한 이름과 정보 얻기

## SYNOPSIS

```c
#include <sys/utsname.h>

int uname(struct utsname *buf);
```

## DESCRIPTION

`uname()`은 시스템 정보를 `buf`가 가리키는 구조체로 반환한다. `utsname` 구조체는 `<sys/utsname.h>`에 정의되어 있다.

```c
struct utsname {
    char sysname[];    /* 운영 체제 이름 (가령 "Linux") */
    char nodename[];   /* "구현에서 규정하는 어떤 망"
                          내에서의 이름 */
    char release[];    /* 운영 체제 릴리스 (가령 "2.6.28") */
    char version[];    /* 운영 체제 버전 */
    char machine[];    /* 하드웨어 식별자 */
#ifdef _GNU_SOURCE
    char domainname[]; /* NIS 또는 YP 도메인 이름 */
#endif
};
```

`struct utsname` 내 배열들의 길이는 명세되어 있지 않다 (NOTES 참고). 그 필드들은 널 바이트(`'\0'`)로 끝난다.

## RETURN VALUE

성공 시 0을 반환한다. 오류 시 -1을 반환하며 `errno`를 적절히 설정한다.

## ERRORS

<dl>
<dt><code>EFAULT</code></dt>
<dd><code>buf</code>가 유효하지 않다.</dd>
</dl>

## CONFORMING TO

POSIX.1-2001, POSIX.1-2008, SVr4. 4.3BSD에는 `uname()` 호출이 없다.

`domainname` 멤버(NIS 또는 YP 도메인 이름)는 GNU 확장이다.

## NOTES

이 함수는 시스템 호출이고 아마 운영 체제는 자기 이름과 릴리스, 버전을 알고 있을 것이다. 또 자기가 어떤 하드웨어 위에서 도는지도 안다. 그래서 구조체의 네 필드에는 의미가 있다. 반면 `nodename` 필드에는 의미가 없다. 어떤 규정돼 있지 않은 망에서 현 머신의 이름인데, 보통 머신들은 여러 망 내에 있고 여러 이름을 가지고 있다. 게다가 커널이 그런 내용을 알 방법도 없으므로 여기에 뭐라고 답할지를 커널에게 알려 주어야 한다. `domainname` 필드에서도 마찬가지이다.

이를 위해 리눅스에서는 시스템 호출 `sethostname(2)`과 `setdomainname(2)`을 사용한다. 참고로 어느 표준에서도 `sethostname(2)`으로 설정한 호스트명이 `uname()`이 반환하는 구조체의 `nodename` 필드와 같은 문자열이라고 말하지 않는다. (실제로 어떤 시스템에서는 256바이트 호스트명과 8바이트 노드명이 가능하다.) 하지만 리눅스에서는 그렇다. `setdomainname(2)`과 `domainname` 필드도 마찬가지이다.

구조체의 필드 길이는 다양하다. 어떤 운영 체제나 라이브러리에서는 9, 33, 65, 257을 하드코딩 해서 사용한다. 다른 시스템에서는 `SYS_NMLN`이나 `_SYS_NMLN`, `UTSLEN`, `_UTSNAME_LENGTH`를 사용한다. 이 상수들을 이용하는 건 분명 좋지 않은 생각이다. 그냥 `sizeof(...)`를 사용하라. 인터넷 호스트명을 담을 수 있도록 257을 선택하는 경우가 많다.

`/proc/sys/kernel/{ostype,hostname,osrelease,version,domainname}`을 통해서도 utsname 정보 일부에 접근할 수 있다.

### C 라이브러리/커널 차이

시간이 흐르며 `utsname` 구조체가 커지면서 세 가지 `uname()` 버전이 생겼다. `sys_olduname()` (슬롯 `__NR_oldolduname`), `sys_uname()` (슬롯 `__NR_olduname`), 그리고 `sys_newuname()` (슬롯 `__NR_uname`)이다. 첫 번째에서는 모든 필드를 길이 9로 사용했고, 두 번째에서는 65를 사용했다. 세 번째에서도 65를 사용하지만 `domainname` 필드를 추가한다. glibc의 `uname()` 래퍼 함수에서 이런 세부 사항을 응용에게 감춰 주고 커널이 제공하는 가장 최신 버전의 시스템 호출을 부른다.

## SEE ALSO

`uname(1)`, <tt>[[getdomainname(2)]]</tt>, <tt>[[gethostname(2)]]</tt>, <tt>[[namespaces(7)]]</tt>

----

2019-03-06
