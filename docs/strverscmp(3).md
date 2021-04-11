## NAME

strverscmp - 두 버전 문자열 비교하기

## SYNOPSIS

```c
#define _GNU_SOURCE         /* feature_test_macros(7) 참고 */
#include <string.h>

int strverscmp(const char *s1, const char *s2);
```

## DESCRIPTION

`jan1`, `jan2`, ..., `jan9`, `jan10`, ... 이런 파일들이 있는데 `ls(1)` 하면 순서가 `jan1`, `jan10`, ..., `jan2`, ..., `jan9` 식으로 나오는 게 이상해 보일 때가 종종 있다. 이걸 바로잡기 위해 GNU에서 `ls(1)`에 `-v` 옵션을 도입했는데, 그 구현에서 <tt>[[versionsort(3)]]</tt>를 사용하며 다시 거기서 `strverscmp()`를 사용한다.

인즉 `strcmp(3)`가 사전적 순서만 찾는 반면 `strverscmp()`가 하는 일은 두 문자열을 비교하여 "올바른" 순서를 찾는 것이다. 이 함수는 로캘 범주 `LC_COLLATE`를 사용하지 않는다. 따라서 기본적으로 문자열이 ASCII일 것으로 예상되는 상황들을 위한 것이다.

이 함수가 하는 일은 이렇다. 두 문자열이 같으면 0을 반환한다. 그렇지 않으면 앞에서는 두 문자열이 같고 거기부터는 다른 위치를 찾는다. 그리고 그 위치를 포함하는 (거기서 시작하거나 끝날 수도 있는) 가장 긴 연속된 숫자 열을 찾는다. 양쪽 모두나 어느 한쪽에서 이 열이 비어 있으면 `strcmp(3)`가 반환할 결과를 반환한다 (바이트 값의 크기에 따라 정렬). 그렇지 않으면 두 숫자 열을 수로서 비교한다. 이때 선두에 0이 한 개 이상 있으면 그 앞에 소수점이 있는 것처럼 해석한다. (그래서 특히 선두에 0이 더 많은 숫자 열이 선두에 0이 더 적은 숫자 열 앞에 오도록 한다.) 그리하여 `000`, `00`, `01`, `010`, `09`, `0`, `1`, `9`, `10` 순서가 된다.

## RETURN VALUE

`strverscmp()` 함수는 `s1`이 `s2`보다 앞이거나, 같거나, 뒤인 경우에 각각 0보다 작거나, 같거나, 큰 정수를 반환한다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `strverscmp()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

이 함수는 GNU 확장이다.

## EXAMPLE

아래 프로그램을 이용해 `strverscmp()`의 동작 방식을 볼 수 있다. `strverscmp()`를 사용해 명령행 인자로 받은 두 문자열을 비교한다. 용례는 다음과 같다.

```text
$ ./a.out jan1 jan10
jan1 < jan10
```

### 프로그램 소스

```c
#define _GNU_SOURCE
#include <string.h>
#include <stdio.h>
#include <stdlib.h>

int
main(int argc, char *argv[])
{
    int res;

    if (argc != 3) {
        fprintf(stderr, "Usage: %s <string1> <string2>\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    res = strverscmp(argv[1], argv[2]);

    printf("%s %s %s\n", argv[1],
            (res < 0) ? "<" : (res == 0) ? "==" : ">", argv[2]);

    exit(EXIT_SUCCESS);
}
```

## SEE ALSO

`rename(1)`, `strcasecmp(3)`, `strcmp(3)`, <tt>[[strcoll(3)]]</tt>

----

2019-03-06
