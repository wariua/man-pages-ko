## NAME

stime - 시간 설정하기

## SYNOPSIS

```c
#include <time.h>

int stime(const time_t *t);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

<dl>
<dt><code>stime()</code>:</dt>
<dd>
 <dl>
 <dt>glibc 2.19부터:</dt>
 <dd><code>_DEFAULT_SOURCE</code></dd>
 <dt>glibc 2.19 및 이전:</dt>
 <dd><code>_SVID_SOURCE</code></dd>
 </dl>
</dd>
</dl>

## DESCRIPTION

`stime()`은 시스템의 시간 및 날짜 개념을 설정한다. `t`가 가리키는 시간은 에포크 1970-01-01 00:00:00 +0000 (UTC) 이후의 초로 측정한다. 수퍼유저만 `stime()`을 실행할 수 있다.

## RETURN VALUE

성공 시 0을 반환한다. 오류 시 -1을 반환하며 `errno`를 적절히 설정한다.

## ERRORS

<dl>
<dt><code>EFAULT</code></dt>
<dd>사용자 공간의 정보를 얻어 오는 과정에 오류.</dd>
<dt><code>EPERM</code></dt>
<dd>호출 프로세스에게 충분한 특권이 없다. 리눅스에서는 <code>CAP_SYS_TIME</code> 특권이 필요하다.</dd>
</dl>

## CONFORMING TO

SVr4.

## SEE ALSO

`date(1)`, <tt>[[settimeofday(2)]]</tt>, <tt>[[capabilities(7)]]</tt>

----

2016-03-15
