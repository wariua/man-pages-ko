## NAME

getumask - 파일 생성 마스크 얻기

## SYNOPSIS

```c
#define _GNU_SOURCE             /* feature_test_macros(7) 참고 */
#include <sys/types.h>
#include <sys/stat.h>

mode_t getumask(void);
```

## DESCRIPTION

이 함수는 현재의 파일 생성 마스크를 반환한다. 스레드 안전이라고 (즉 <tt>[[umask(2)]]</tt> 라이브러리 호출과 락을 공유) 문서화 돼 있다는 걸 빼면 다음과 동등하다.

```c
mode_t getumask(void)
{
    mode_t mask = umask( 0 );
    umask(mask);
    return mask;
}
```

## CONFORMING TO

이 함수는 베이퍼웨어 GNU 확장이다.

## NOTES

glibc 매뉴얼에 이 함수가 문서화 돼 있기는 하지만 glibc 버전 2.24 현재 리눅스에서 구현돼 있지 않다. (프로세스 umask를 알아내기 위한 스레드에 안전한 방법은 <tt>[[umask(2)]]</tt>를 보라.)

## SEE ALSO

<tt>[[umask(2)]]</tt>

----

2017-09-15
