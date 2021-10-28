---
published: false
title: PintOS 입문
tags:
  - OS
  - C
  - computer science
  - kernel
---
## Pintos에 대해

> Welcome to Pintos. Pintos is a simple operating system framework for the 80x86 architecture. It supports kernel threads, loading and running user programs, and a file system,
but it implements all of these in a very simple way. In the Pintos projects, you and your
project team will strengthen its support in all three of these areas. You will also add a
virtual memory implementation.
>
> -- Introduction

일단 설명에 앞서 자료부터 보자
- https://web.stanford.edu/class/cs140/projects/pintos/pintos.pdf

Pintos(핀토스)는 2004년 스탠포드 대학에서 만든 교육용 OS이다. 리눅스 커널은 매우 방대해서 초심자가 이걸로 운영체제 공부를 하는 것은 매우 어려운데, 커밋만 백만개가 넘었으며 소스코드 라인도 백만줄이 넘고, 80% 이상의 코드가 하드웨어 지원을 위한 디바이스 드라이버 코드로 사실 OS 기초를 공부하는 데는 적합하지 않다.

따라서 OS 공부용으로 상대적으로 간단한 기능만 있는 Pintos가 개발됐다.

x86 아키텍처만 지원하고, 따라서 Bochs, QEMU 등 x86 에뮬레이터를 사용해야 한다.

Introduction에서 말하고 있듯이 pintos는
- kernel threads
- loading and running user program
- file system

을 지원하며, 이를 매우 간단한 방식으로 구현하게 될 것이다. 핀토스 구현을 완료하면 이 세 영역에 강점을 가질 수 있을 것이라는 응원의 말도 써 있다.


## Pintos 설치

### 1. 환경 설정

(m1 mac + homebrew 기준)

```sh
$ brew install qemu
$ sudo ln -s /opt/homebrew/bin/qemu-system-i386 /usr/local/bin/qemu
```
qemu 설치 후 symlink를 만들어서 local bin에 넣는다

```
$ qemu --version
QEMU emulator version 6.1.0
Copyright (c) 2003-2021 Fabrice Bellard and the QEMU Project developers
```
이런식으로 나오면 완료

### 2. 다운로드

- https://web.stanford.edu/class/cs140/projects/pintos/pintos.tar.gz

다운로드 후 `tar xvf pintos.tar.gz`로 압축 해제


## Troubleshooting (on Mac)
```
# src/threads/Make.vars
SIMULATOR == --qemu
```
bochs로 되어 있는데 qemu로 수정


```sh
$ make
...
cp ../Makefile.build build/Makefile
cd build && /Library/Developer/CommandLineTools/usr/bin/make all
../../Make.config:37: *** Compiler (i386-elf-gcc) not found.  Did you set $PATH properly?  Please refer to the Getting Started section in the documentation for details. ***
/bin/sh: i386-elf-ld: command not found
...
```
make 했을 때 i386-elf-gcc가 없다고 나오면


## References
- https://web.stanford.edu/class/cs140/projects/
- https://oslab.kaist.ac.kr/pintosslides/
- https://www.youtube.com/watch?v=3p2lUXH2fxU
- https://github.com/maojie/pintos_mac/pull/2/files/312098388750de0b2bfa372a772065940dc70fa2
- https://bowbowbow.tistory.com/9
