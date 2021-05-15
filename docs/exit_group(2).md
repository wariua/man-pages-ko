## NAME

exit_group - 프로세스 내의 모든 스레드 끝내기

## SYNOPSIS

```c
#include <linux/unistd.h>

void exit_group(int status);
```

## DESCRIPTION

이 시스템 호출은 호출 스레드뿐 아니라 호출 프로세스의 스레드 그룹의 모든 스레드들을 종료시킨다는 점을 제외하면 <tt>[[_exit(2)]]</tt>과 동등하다.

## RETURN VALUE

이 시스템 호출은 반환하지 않는다.

## VERSIONS

리눅스 2.5.35부터 이 호출이 존재한다.

## CONFORMING TO

이 호출은 리눅스 전용이다.

## NOTES

glibc 2.3부터 <tt>[[_exit(2)]]</tt> 래퍼 함수 호출 시 이 시스템 호출을 부른다.

## SEE ALSO

<tt>[[_exit(2)]]</tt>

----

2008-11-27
