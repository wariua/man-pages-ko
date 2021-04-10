## NAME

sysinfo - 시스템 정보 반환

## SYNOPSIS

```c
#include <sys/sysinfo.h>

int sysinfo(struct sysinfo *info);
```

## DESCRIPTION

`sysinfo()`는 메모리 및 스왑 사용, 그리고 평균 부하에 대한 몇 가지 통계를 반환한다.

리눅스 2.3.16까지에서 `sysinfo()`는 다음 구조체로 정보를 반환했다.

```c
struct sysinfo {
    long uptime;             /* 부팅 후 지난 초 */
    unsigned long loads[3];  /* 1분, 5분, 15분 평균 부하 */
    unsigned long totalram;  /* 사용 가능한 주 메모리 총크기 */
    unsigned long freeram;   /* 사용 가능한 메모리 크기 */
    unsigned long sharedram; /* 공유 메모리 양 */
    unsigned long bufferram; /* 버퍼에서 쓰는 메모리 */
    unsigned long totalswap; /* 스왑 공간 총크기 */
    unsigned long freeswap;  /* 아직 사용 가능한 스왑 공간 */
    unsigned short procs;    /* 현재 프로세스 개수 */
    char _f[22];             /* 구조체를 64비트로 채우기 */
};
```

위 구조체에서 메모리 및 스왑 필드의 크기는 바이트 단위이다.

리눅스 2.3.23(i386) 및 2.3.48(모든 아키텍처)부터는 다음 구조체이다.

```c
struct sysinfo {
    long uptime;             /* 부팅 후 지난 초 */
    unsigned long loads[3];  /* 1분, 5분, 15분 평균 부하 */
    unsigned long totalram;  /* 사용 가능한 주 메모리 총크기 */
    unsigned long freeram;   /* 사용 가능한 메모리 크기 */
    unsigned long sharedram; /* 공유 메모리 양 */
    unsigned long bufferram; /* 버퍼에서 쓰는 메모리 */
    unsigned long totalswap; /* 스왑 공간 총크기 */
    unsigned long freeswap;  /* 아직 사용 가능한 스왑 공간 */
    unsigned short procs;    /* 현재 프로세스 개수 */
    unsigned long totalhigh; /* 상위 메모리 총크기 */
    unsigned long freehigh;  /* 사용 가능한 상위 메모리 크기 */
    unsigned int mem_unit;   /* 메모리 단위 크기의 바이트 수 */
    char _f[20-2*sizeof(long)-sizeof(int)];
                             /* 구조체를 64비트로 채우기 */
};
```

위 구조체에서 메모리 및 스왑 필드의 크기는 `mem_unit` 바이트가 단위이다.

## RETURN VALUE

성공 시 `sysinfo()`는 0을 반환한다. 오류 시 -1을 반환하며 오류 원인을 나타내도록 `errno`를 설정한다.

## ERRORS

`EFAULT`
:   `info`가 유효한 주소가 아니다.

## VERSIONS

리눅스 0.98.pl6에서 `sysinfo()`가 처음 등장했다.

## CONFORMING TO

이 함수는 리눅스 전용이므로 이식성이 있어야 하는 프로그램에서는 사용하지 말아야 한다.

## NOTES

이 시스템 호출에서 제공하는 정보 모두를 `/proc/meminfo` 및 `/proc/loadavg`를 통해서도 얻을 수 있다.

## SEE ALSO

<tt>[[proc(5)]]</tt>

----

2017-09-15
