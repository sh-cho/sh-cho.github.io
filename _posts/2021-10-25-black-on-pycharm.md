---
published: true
title: PyCharm black 설정 방법
---
파이참에서 파일 저장할 때 자동으로 black이 적용되도록 설정해보자


## environment
- OS: macOS 11.5.2
- PyCharm: 2021.2.1 community edition
- Python: 3.9.6
- black: 21.9b0


## pre-requisite
black이 설치되어 있어야 하고, 설치된 경로를 알아야 함


## Steps
```sh
$ pip install black
```
1. black 설치

```sh
# macOS, Linux, BSD
$ which black
/usr/local/bin/black     # possible location
/opt/homebrew/bin/black  # if using m1 mac + homebrew + pyenv

# Windows
$ where black
%LocalAppdata%\Programs\Python\Python36-32\Scripts\black.exe
```
2. black 설치 경로 확인

(PyCharm에서 감지되는 virtual environment를 쓰고 있으면 `$PyInterpreterDirectory$/black`을 사용하면 됨)

```
On macOS
PyCharm -> Preferences -> Tools -> External Tools

On Windows, Linux, BSD
File -> Settings -> Tools -> External Tools
```

3. External tools 메뉴로 들어간 뒤

![Screen Shot 2021-10-25 at 12.40.02 AM.png]({{site.baseurl}}/_posts/Screen Shot 2021-10-25 at 12.40.02 AM.png)

\+ 기호를 누르고

![Screen Shot 2021-10-25 at 12.41.41 AM.png]({{site.baseurl}}/_posts/Screen Shot 2021-10-25 at 12.41.41 AM.png)

tool을 위 사진처럼 설정하고 추가한다

- Name: Black
- Program: (2)에서 확인한 black 설치 경로
- Arguments: `"$FilePath$""`

이제 `Tools - External Tools - Black`을 이용해 현재 열려있는 파일에 black을 돌릴 수 있다. Keymap도 설정 가능하다 (문서 참조)


### (Option) 파일 저장할 때마다 black 돌리는 법

![Screen Shot 2021-10-25 at 1.18.32 AM.png]({{site.baseurl}}/_posts/Screen Shot 2021-10-25 at 1.18.32 AM.png)

1. 'File Watchers' 플러그인 설치
2. `Preferences - Tools - File Watchers`에서 + 버튼 눌러 새 watcher 추가
  - Name: Black
  - File type: Python
  - Scope: Project Files
  - Program: (2)에서 확인한 black 설치 경로
  - Arguments: `$FilePath$`
  - Output paths to refresh: `$FilePath$`
  - Working directory: `$ProjectFileDir$`
  - Advanced Options
    - "Auto-save edited files to trigger the watcher" 체크 해제
    - "Trigger the watcher on external changes" 체크 해제


## Reference
- [Editor integration (black readthedocs)](https://black.readthedocs.io/en/stable/integrations/editors.html)
