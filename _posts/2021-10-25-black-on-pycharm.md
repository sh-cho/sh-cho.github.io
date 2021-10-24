---
published: false
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

3. External tool 



### (Option) run Black on every file save



## Reference
- [Editor integration (black readthedocs)](https://black.readthedocs.io/en/stable/integrations/editors.html)
