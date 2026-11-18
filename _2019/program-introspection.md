---
layout: lecture
title: "프로그램 내부 살펴보기"
presenter: Anish
date: 2019-01-29
order: 1
video:
  aspect: 62.5
  id: 74MhV-7hYzg
translation_status: draft
---

# 디버깅

printf 디버깅으로 충분하지 않을 때는 디버거를 사용하세요.

디버거는 프로그램 실행에 개입하여
다음과 같은 일을 할 수 있게 해 줍니다.

- 특정 줄에 도달하면 프로그램 실행을 중단하기
- 프로그램을 한 단계씩 실행하기
- 변수의 값 살펴보기
- 그 밖의 여러 고급 기능

## GDB/LLDB

[GDB](https://www.gnu.org/software/gdb/)와 [LLDB](https://lldb.llvm.org/).
C와 비슷한 여러 언어를 지원합니다.

[example.c](/2019/files/example.c)를 살펴봅시다. 디버그 플래그를 사용해 컴파일하세요.
`gcc -g -o example example.c`.

GDB를 여세요.

`gdb example`

몇 가지 명령어는 다음과 같습니다.

- `run`
- `b {name of function}` - 중단점 설정
- `b {file}:{line}` - 중단점 설정
- `c` - 계속 실행
- `step` / `next` / `finish` - 함수 안으로 진입 / 함수 안으로 들어가지 않고 한 단계 실행 / 함수 밖으로 나가기
- `p {variable}` - 변수의 값 출력
- `watch {expression}` - 표현식의 값이 바뀔 때 작동하는 감시점 설정
- `rwatch {expression}` - 값을 읽을 때 작동하는 감시점 설정
- `layout`

## PDB

[PDB](https://docs.python.org/3/library/pdb.html)는 Python 디버거입니다.

PDB로 진입하고 싶은 위치에 `import pdb; pdb.set_trace()`를 넣으세요. PDB는 기본적으로
디버거(GDB와 같은)와 Python 셸을 합친 것입니다.

## 웹 브라우저 개발자 도구

디버거의 또 다른 예로, 이번에는 그래픽 인터페이스가 있습니다.

# strace

프로그램이 하는 시스템 호출을 관찰합니다. `strace {program}`.

# 프로파일링

프로파일링의 종류에는 CPU, 메모리 등이 있습니다.

가장 간단한 프로파일러는 `time`입니다.

## Go

CPU 프로파일러를 사용해 테스트 코드를 실행합니다. `go test -cpuprofile=cpu.out`

프로파일을 분석합니다. `go tool pprof -web cpu.out`

메모리 프로파일러를 사용해 테스트 코드를 실행합니다. `go test -memprofile=mem.out`

프로파일을 분석합니다. `go tool pprof -web mem.out`

## Perf

기본 성능 통계입니다. `perf stat {command}`

프로파일러를 사용해 프로그램을 실행합니다. `perf record {command}`

프로파일을 분석합니다. `perf report`
