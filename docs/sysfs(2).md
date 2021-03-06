## NAME

sysfs - 파일 시스템 타입 정보 얻기

## SYNOPSIS

```c
int sysfs(int option, const char *fsname);
int sysfs(int option, unsigned int fs_index, char *buf);
int sysfs(int option);
```

## DESCRIPTION

**참고**: 보통 `/sys`에 마운트 되는 `sysfs` 파일 시스템에 대한 정보를 찾으려는 거라면 <tt>[[sysfs(5)]]</tt>를 보라.

(구식이 된) `sysfs()` 시스템 호출은 현재 커널 내의 파일 시스템 타입에 대한 정보를 반환한다. `sysfs()` 호출의 구체적 형태와 반환 정보는 적용되는 `option` 값에 따라 달라진다.

1
:   파일 시스템 식별 문자열 `fsname`을 파일 시스템 타입 인덱스로 바꾼다.

2
:   파일 시스템 타입 인덱스 `fs_index`를 널 종료 파일 시스템 식별 문자열로 바꾼다. 그 문자열이 `buf`가 가리키는 버퍼에 기록된다. `buf`에 그 문자열을 담을 충분한 공간이 있도록 해야 한다.

3
:   현재 커널 내의 파일 시스템 타입 총수를 반환한다.

파일 시스템 타입 인덱스는 0번부터 시작한다.

## RETURN VALUE

성공 시 `sysfs()`는 옵션 `1`에 대해선 파일 시스템 인덱스를, 옵션 `2`에 대해선 0을, 옵션 `3`에 대해선 현재 구성된 파일 시스템 수를 반환한다. 오류 시 -1을 반환하며 오류를 나타내도록 `errno`를 설정한다.

## ERRORS

`EFAULT`
:   `fsname`이나 `buf`가 접근 가능한 주소 공간 밖에 있다.

`EINVAL`
:   `fsname`이 유효한 파일 시스템 유형 식별자가 아니다. `fs_index`가 범위를 벗어난다. `option`이 유효하지 않다.

## CONFORMING TO

SVr4.

## NOTES

시스템 V에서 유래한 이 시스템 호출은 구식이 되었다. 쓰지 말아야 한다. `/proc`이 있는 시스템에서는 `/proc`를 통해 같은 정보를 얻을 수 있으므로 그 인터페이스를 쓰면 된다.

## BUGS

libc 내지 glibc 지원이 없다. `buf` 크기가 얼마여야 하는지 알아낼 방법이 없다.

## SEE ALSO

<tt>[[proc(5)]]</tt>, <tt>[[sysfs(5)]]</tt>

----

2021-03-22
