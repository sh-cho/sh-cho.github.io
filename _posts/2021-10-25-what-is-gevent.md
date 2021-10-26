---
published: false
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
쓰레드는 프로세스에 비해 경량이므로 여러개를 만드는 게 문제가 되지 않지만, 단점은 제대로 개발하기가 힘들다.

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

요약하면 OS(커널)의 유저 영역에서 동작하는 쓰레드이다. Cooperative 하다는 점이 특징인데, OS가 컨트롤하지 않으니 선점도 불가, 컨트롤하려면 명시적 yield가 필요하다.

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
(~08:00)


## Putting It Together


## Wrap-up / Q&A


## Reference


