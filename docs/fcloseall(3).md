## NAME

fcloseall - 열린 스트림 모두 닫기

## SYNOPSIS

```c
#define _GNU_SOURCE         /* feature_test_macros(7) 참고 */
#include <stdio.h>

int fcloseall(void);
```

## DESCRIPTION

`fcloseall()` 함수는 호출 프로세스의 열린 스트림들을 모두 닫는다. 각 스트림의 버퍼에 있는 출력을 (<tt>[[fflush(3)]]</tt>처럼) 기록한 다음 스트림을 닫는다. 버퍼에 있는 입력은 버린다.

표준 스트림 `stdin`, `stdout`, `stderr`도 닫는다.

## RETURN VALUE

모든 파일이 성공적으로 닫혔으면 0을 반환한다. 오류 시 `EOF`를 반환한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `fcloseall()` | 스레드 안전성 | MT-Unsafe race:streams |

`fcloseall()` 함수에서 스트림 락을 잡지 않으므로 스레드 안전이 아니다.

## CONFORMING TO

이 함수는 GNU 확장이다.

## SEE ALSO

<tt>[[close(2)]]</tt>, <tt>[[fclose(3)]]</tt>, <tt>[[fflush(3)]]</tt>, <tt>[[fopen(3)]]</tt>, <tt>[[setbuf(3)]]</tt>

----

2021-03-22
