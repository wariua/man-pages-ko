## NAME

makedev, major, minor - 장치 번호 관리하기

## SYNOPSIS

```c
#include <sys/sysmacros.h>

dev_t makedev(unsigned int maj, unsigned int min);

unsigned int major(dev_t dev);
unsigned int minor(dev_t dev);
```

## DESCRIPTION

장치 ID는 두 부분으로 되어 있다. 주(major) ID는 장치의 유형을 나타내고 부(minor) ID는 그 유형 내에서 개별 장치 인스턴스를 나타낸다. `dev_t` 타입으로 장치 ID를 표현한다.

주 장치 ID와 부 장치 ID를 주면 `makedev()`가 이를 합쳐서 장치 ID를 만들어 함수 결과로 반환한다. 그 장치 ID를 예를 들어 <tt>[[mknod(2)]]</tt>에 줄 수 있다.

`major()`와 `minor()` 함수는 반대 작업을 수행한다. 즉, 장치 ID를 주면 각각 주 부분과 부 부분을 반환한다. 예를 들어 <tt>[[stat(2)]]</tt>이 반환한 구조체의 장치 ID를 분해하는 데 이 매크로들이 유용할 수 있다.

## ATTRIBUTES

이 절에서 사용하는 용어들에 대한 설명은 <tt>[[attributes(7)]]</tt>를 보라.

| 인터페이스 | 속성 | 값 |
| --- | --- | --- |
| `makedev()`, `major()`, `minor()` | 스레드 안전성 | MT-Safe |

## CONFORMING TO

`makedev()`, `major()`, `minor()` 함수는 POSIX.1에 명세돼 있지 않지만 다른 여러 시스템에 존재한다.

## NOTES

이 인터페이스들은 매크로로 정의돼 있다. glibc 2.3.3부터는 GNU 전용 함수 `gnu_dev_makedev()`, `gnu_dev_major()`, `gnu_dev_minor()`의 별칭이다. 내보이는 이름들이긴 하지만 전통적 이름이 더 이식성이 좋다.

BSD에선 `<sys/types.h>`를 통해 이 매크로들의 정의를 드러낸다. 버전에 따라선 glibc에서도 적당한 기능 검사 매크로가 정의돼 있으면 그 헤더 파일에서 매크로 정의를 드러낸다. 하지만 그 동작은 glibc 2.25에서 폐기 예정이 되었으며, glibc 2.28부터는 `<sys/types.h>`가 더이상 그 정의를 제공하지 않는다.

## SEE ALSO

<tt>[[mknod(2)]]</tt>, <tt>[[stat(2)]]</tt>

----

2021-03-22
