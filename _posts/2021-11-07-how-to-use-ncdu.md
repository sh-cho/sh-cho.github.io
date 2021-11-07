---
published: true
title: ncdu 사용법
tags:
  - ncurses
  - ncdu
  - mac
  - du
---
> Ncdu is a disk usage analyzer with an ncurses interface. It is designed to find space hogs on a remote server where you don't have an entire graphical setup available, but it is a useful tool even on regular desktop systems. Ncdu aims to be fast, simple and easy to use, and should be able to run in any minimal POSIX-like environment with ncurses installed.

[ncdu(NCurses Disk Usage)](https://dev.yorhel.nl/ncdu)는 ncurses 인터페이스를 이용해 구현된 디스크 사용량 분석 프로그램이다. `du`의 ncurses 버전이다.


## Installation
```sh
brew install ncdu
```
적절한 패키지매니저를 사용해 설치한다.


## How to use
```sh
ncdu [options] dir
```
터미널에서 `ncdu` 커맨드를 입력하면 실행된다. `man ncdu`로 더 자세한 옵션을 확인할 수 있다.

ncdu 창에서 `?`를 입력하면 도움말 창이 나오는데 단축키와 포맷 내용을 확인할 수 있다.

### 1. Keys
- up, k: 커서 위로 이동
- down, j: 커서 아래로 이동
- right/enter: 선택한 디렉토리 열기
- left, <, h: 부모 디렉토리 열기
- n: 이름순 정렬
- s: 크기순 정렬
- C: 파일 개수 순 정렬
- M: mtime(수정 시간)순 정렬 (`-e` flag 필요)
- d: 선택한 파일 또는 디렉토리 제거
- t: 정렬시 폴더 따로 정렬 토글
- g: 퍼센티지와 그래프 출력 토글
- a: apprent size/disk usage 토글
- c: child item 개수 출력 토글
- m: latest mtime 출력 토글 (`-e` flag 필요)
- e: hidden/excluded 파일 출력 토글
- i: 선택중인 파일/폴더 정보 표시
- r: 현재 디렉토리 재 계산
- b: 현재 디렉토리에서 쉘 키기
- q: ncdu 종료

### 2. Format
```
X [size] [graph] [file or directory]
```
각 아이템이 위 내용으로 구성되는데, X는 아래와 같은 경우에만 표시된다

- `!`: 해당 디렉토리를 읽는 동안 오류 발생
- `.`: 서브디렉토리를 읽는 동안 오류 발생
- `<`: 파일이나 디렉토리가 통계에서 제외됨
- `e`: 빈 디렉토리
- `>`: 디렉토리가 다른 파일시스템에 있음
- `@`: 파일도 아니고 디렉토리도 아님 (ex. symlink, socket, ...)
- `^`: 리눅스 psuedo-filesystem
- `H`: hard link
- `F`: firm link

### example
![ncdu example]({{site.baseurl}}/assets/images/posts/ncdu-example.png)

홈 디렉토리에서 ncdu를 사용한 모습

Library 폴더의 용량이 커서 확인해보니 homebrew cache가 용량이 엄청 불어나 있었다.
