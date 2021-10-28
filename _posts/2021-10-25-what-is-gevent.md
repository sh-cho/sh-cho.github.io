---
published: true
title: gevent란 무엇인가
tags:
  - gevent
  - python
  - greenlet
  - thread
  - concurrency
  - parallelism
---

{% include video id="GunMToxbE0E" provider="youtube" %}

> Kavya Joshi - A tale of concurrency through creativity in Python: a deep dive into how gevent works.

Pycon2016의 위 영상을 보며 요약한 내용.


## Introduction
- 비동기 I/O란 무엇인가? (What is asynchronous I/O?)
- gevent란 무엇인가? (What is gevent?)

한가지 예시를 들어보자

```python
def download_photos(user):
    conn = get_auth_connection(user)
    photos = get_photos(conn)
    save_photos(user, photos)
```
페이스북에서 어떤 유저의 사진을 모두 받아오는 서비스가 있다고 생각해보자. 코드는 심플하다

```python
def downloader():
    users = get_users()
    for user in users:
    	download_photos(user)  # network I/O
```
만약 위 서비스가 인기가 많아져 사용자가 많아지면 download 부분이 네트워크 병목이 된다.

1. multiprocessing
2. threading
3. event-driven programming (twisted, tornado, asyncore, ...)
4. green_thread

일반적 해결방법은 위와 같다. 하나씩 알아보자


### 1. multiprocessing
```python
import multiprocessing as mp

def downloader():
    pool = []
    for user in users:
    	p = mp.Process(download_photos, user)
        pool.append(p)
        p.start()
  	
    for p in pool:
    	p.join()
```
유저 한 명당 프로세스를 만든다.

프로세스 내에서 네트워크 I/O는 blocking 되어 있긴 하지만, 프로세스 사이에는 영향이 없다.

동시성(concurrency)과 병렬성(parallelism) 둘다 챙길 수 있다. 하지만 프로세스를 만드는 것은 오버헤드가 크다. -> scalable 하지 않음


### 2. threading
```python
import threading

def downloader():
    pool = []
    for user in users:
    	t = threading.Thread(download_photos, user)
        pool.append(t)
        t.start()
    
    for t in pool:
    	t.join()
```
쓰레드는 프로세스에 비해 상대적으로 경량이므로 여러개를 만드는 게 문제가 되지 않지만, 단점은 제대로 개발하기가 힘들다. (아마도 synchronization을 얘기하는 듯)

가장 큰 문제는 CPython의 경우 GIL 때문에 한번에 한 쓰레드만 동작한다는 것. 따라서 사실상 멀티쓰레딩이 의미가 없다.


### 3. event-driven programming
```python
import tiwsted

def download_photos():
    # modify this to add callbacks

def downloader():
    # something something loop.run()
```
concurrency + scalable 달성 가능

하지만 문제는 event loop 실행부터 콜백 등록까지 전부 처리해야 함 -> 어렵다


### 4. green threads
먼저 그린 쓰레드의 특징부터 알아보자

- user space: OS does not create or manage them
- cooperatively scheduled: OS does not schedule or preempty them (명시적 yield 필요)
- lightweight

요약하면 OS(커널)의 유저 영역에서 동작하는 쓰레드이다. Cooperative(협력적) 라는 점이 특징인데, OS가 컨트롤하지 않으니 선점도 불가, 컨트롤하려면 명시적 yield가 필요하다.

```python
import gevent
from gevent import monkey

monkey.patch_all()

def downloader():
    pool = []
    for user in users:
    	g = gevent.Greenlet(download_photos, user)
        g.start()
        pool.append(g)
    gevent.joinall(pool)
```
코드를 보면 (2) threading과 비슷하다. Greenlet은 어떻게 동작하며, gevent는 어떻게 이를 이용해 경량 쓰레드로 동작하는지는 천천히 알아보자.


## The Building Blocks
gevent는 greenlet + libev로 구성돼있는데, 어떻게 구성되어 있는지 간단히 확인해보자.

### Greenlet

```python
g = gevent.Greenlet(download_photos, user)
```
앞서 green thread를 만들 때 위와 같이 썼다. 저 Greenlet은 어디서 오는건지 확인해보자

```python
# src/gevent/greenlet.py
from greenlet import greenlet

...

class Greenlet(greenlet):
    """
    A light-weight cooperatively-scheduled execution unit.
    """
    ...
```
보면 Greenlet이 greenlet 모듈의 greenlet을 상속받아서 만들어진다. 뭔가 벌써 헷갈리지만 일단은 그냥 '그렇구나' 정도로 넘어가자.

> Greenlet: "A light-weight cooperatively-scheduled execution unit."

Greenlet의 주석을 보면 이게 어떤 역할을 하는건지 명확한데, 경량이면서 cooperatively(협력적으로) 스케줄되는 실행 단위라고 써 있다.

```python
from greenlet import greenlet

gr1 = greenlet(print_red)
gr2 = greenlet(print_blue)
gr1.switch()

def print_red():
    print("red")
    gr2.switch()
    print("red done")

def print_blue():
    print("blue")
    gr1.switch()
    print("blue done")

# Expected outputs
#
# red
# blue
# red done
```
위처럼 gr1, gr2라는 greenlet을 만들고 `gr1.switch()`를 이용해 1번 greenlet을 실행시켰다.

.switch()는 현재 greenlet을 일시정지하고, 다음 greenlet으로 실행을 넘긴다. 다시 switch로 이전 greenlet이 실행을 넘겨받으면, 함수 즉 서브루틴(subroutine)처럼 맨 처음부터 시작하는 것이 아닌, 일시정지 한 부분부터 실행된다.

이 동작.. 어디서 많이 본 것 같다

-> greenlet == 코루틴(Coroutine) 임을 알 수 있다

원리는 greenlet이 C extension 모듈인데, 내부적으로 C 콜스택 포인터를 조작하는 stack slicing(스택 슬라이싱)을 써서 코루틴별 start 위치로 돌아가 resume할 수 있다고 한다. 이 부분의 구현은 어셈블리로 되어있다 한다. (소스를 보진 않았다)

자 이제 greenlet은 c 확장모듈이며, Gevent 안에 greenlet c 확장을 사용할 수 있게 해주는 껍데기가 들어있다는 것을 알 수 있다.


### libev
```python
g = gevent.Greenlet(download_photos, user)
g.start()
```
greenlet은 뭐하는건지 대충 알 것 같다. 그 다음줄의 start()는 뭘 하는걸까?

```python
# src/gevent/greenlet.py
def start(self):
    """Schedule the greenlet to run in this loop iteration"""
    if self._start_event is None:
        _call_spawn_callbacks(self)
        hub = get_my_hub(self)
        self._start_event = hub.loop.run_callback(self.switch)
```
gevent 소스를 다시 보자. `greenlet.start()`의 맨 끝 줄을 보면 어쩌구 저쩌구 해서 `hub.loop.run_callback`을 실행시키는 것을 알 수 있다.

여기서 loop는 이벤트 루프(event loop)이다. gevent는 이벤트 루프를 위해 libev를 사용하는데, 역시 C로 작성된 모듈이다.

libev는 `event_handler` 콜백을 등록할 수 있는 API를 제공한다. 또한 이벤트를 감시(watch)할 수 있는 기능을 제공한다. 즉, I/O Multiplexing인데, select, poll, epoll, kqueue, ... 등에서 사용 가능한 방식을 쓴다고 함

> "Hey Loop, **Wait** for a write on this socket and call parse_recv() when that happens."

예를 들면 이벤트루프한테 어떤 소켓이 특정 이벤트가 발생할 때까지 기다리라고 하자. 이걸 구현하려면

```python
fd = make_nonblocking(socket_fd)
loop.io_watch(fd, write, callback_fn)
loop.run()

# in loop..
while True:
    # block for I/O
    # call *pending* io_watchers
```
이런식으로 된다. non-blocking 소켓을 만들고, 이벤트 루프에 `io_watch()` 메소드로 콜백을 등록한다.

이벤트 루프 내부는 I/O 대기 -> 대기중인 io_watcher 호출 이렇게 반복될 것이다.

```python
# in loop (libev)
while True:
    # call *all* pre_block_watchers
    # block for I/O
    # call *pending* io_watchers
    # call *all* post_block_watchers
```
libev는 루프 내부에 두 단계를 더해서 이벤트 루프를 커스터마이즈 할 수 있는 여지를 제공한다. `pre_block_watchers`와 `post_block_watchers`를 호출하는 두 단계가 추가된 것을 확인할 수 있다.

pre_block_watchers은 다른 event mechanism을 이벤트 루프에 적용할 수 있는 hook을 제공한다

> "Hey Loop, **if there are coroutines ready to run, run them first** and then / wait for a write ... blah blah"

이벤트 메커니즘엔 여러가지가 있겠지만 코루틴을 이런식으로 이벤트 루프랑 연결해 쓸 수 있다.


## Putting It Together
```python
import gevent
from gevent import monkey

monkey.patch_all()

def downloader():
    pool = []
    for user in users:
    	g = gevent.Greenlet(download_photos, user)
        g.start()
        pool.append(g)
    gevent.joinall(pool)
```
다시 코드를 보자. 코드에는 `greenlet.switch()`나 `loop.run()`과 같은 내용이 보이지 않는데 어떻게 동작하는 걸까?

이제 gevent가 어떻게 greenlet과 libev를 한데 묶어서 쓰는지 알아보자.

```python
g = gevent.Greenlet(download_photos, user)

# in src
class Greenlet(greenlet):
    def __init__(self, run=None, ...):
        greenlet.__init__(self, None, get_hub())

# in get_hub()
g.parent = Hub
```
Greenlet 생성자를 보면 `get_hub()`가 있는데, 이 부분이 바로 parent를 허브 greenlet으로 설정하는 부분이다.

```python
class Hub(greenlet):
    def __init__(self):
        self.loop = ...   # event loop!
```
Hub greenlet는 이벤트 루프를 담고 있고 동작을 담당한다. 쓰레드마다 1개의 이벤트 루프 또는 Hub greenlet이 있는 것이다.

```python
g.start()
...
self.parent.loop.run_callback(self.switch)
```
`g.start()`가 호출되면 greenlet의 switch(start or resume) 함수를 이벤트 루프에 등록하게 되고, `pre_block_watchers`로 등록한다.

```python
for user in users:
    g = gevent.Greenlet(download_photos, user)
    g.start()
    pool.append(g)
gevent.joinall(pool)
```
이제 이벤트루프와 그린쓰레드의 조합은 알 것 같다. 실제로 gevent가 async I/O를 어떻게 가능하게 할까?

```python
from gevent import monkey
monkey.patch_all()
```
바로 몽키패칭(monkey patching)인데, 기존 socket을 gevent.socket으로 바꿔친다.

```
create: fd = make_nonblocking(socket_fd)
send  : loop.io_watch(fd, write, callback_fn)
        loop.run()
```
소켓을 만들 때 논블락킹 소켓을 만들고, send 등 소켓 함수를 쓸 때 이벤트 루프를 사용하도록 만든다.


## Wrap-up / Q&A

- 병렬성 x (no parallelism)
- 협력적이지 않은(non-cooperative) 코드가 프로세스 전체를 블락킹 할 수 있다
  - ex1) C Extension -> 순수 파이썬 라이브러리 사용을 고려
  - ex2) compute-bound greenlets -> gevent.sleep(0) 또는 greenlet blocking detection
- 몽키패칭은 import 순서 등 고려할 점이 많고 디버깅이 어렵다

gevent는 위와 같은 한계가 있다.

이제 gevent와 greenlet, libev, Hub, monkeypatching 등 내부 구성요소에 대해 어느정도 익숙해졌으리라 믿는다.
