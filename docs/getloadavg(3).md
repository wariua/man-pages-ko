## NAME

getloadavg - 시스템 부하 평균치 얻기

## SYNOPSIS

```c
#include <stdlib.h>

int getloadavg(double loadavg[], int nelem);
```

glibc 기능 확인 매크로 요건 (<tt>[[feature_test_macros(7)]]</tt> 참고):

<dl>
<dt><code>getloadavg()</code>:</dt>
<dd>
 <dl>
 <dt>glibc 2.19부터:</dt>
 <dd><code>_DEFAULT_SOURCE</code></dd>
 <dt>glibc 2.19까지:</dt>
 <dd><code>_BSD_SOURCE</code></dd>
 </dl>
</dd>
</dl>

## DESCRIPTION

`getloadavg()` 함수는 다양한 시간에 대해 평균한 시스템 동작 큐의 프로세스 개수를 반환한다. 샘플을 `nelem` 개까지 얻어 와서 `loadavg[]`의 항목들에 연속으로 할당한다. 시스템에서 샘플을 최대 3개로 제약하는데, 각각 최근 1분, 5분, 15분에 대한 평균을 나타낸다.

## RETURN VALUE

부하 평균을 얻을 수 없는 경우 -1을 반환한다. 그 외의 경우 실제로 얻어 온 샘플의 수를 반환한다.

## VERSIONS

glibc 버전 2.2부터 이 함수가 사용 가능하다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `getloadavg()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

POSIX.1에 없음. BSD 및 솔라리스에 있음.

## SEE ALSO

`uptime(1)`, <tt>[[proc(5)]]</tt>

----

2016-03-15
