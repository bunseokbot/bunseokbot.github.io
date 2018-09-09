---
title: Celery 에서 worker별로 task를 부여하는 방법
description: Celery Routing을 이용한 worker별 효율적인 task 분배 방법
categories:
 - celery
tags: celery, Python, Django
---

## 0. 도입하게 된 계기
최근 개인적으로 준비하고 있는 프로젝트가 하나 있는데, 그 프로젝트에서 돌고 돌다가 결국 Celery를 도입하게 되었다. (나중에 다른 글을 통해서 공개할 수 있으면 공개하도록 하겠다.)

지금 진행하고 있는 프로젝트에는 task가 2개가 있고 이를 concurrency를 이용해서 10개의 worker를 돌리고 있지만 이 과정에서 문제가 발생하였다.

A task 으로의 요청이 어느 순간 2만건 가까이 몰려온 적이 있었는데, A task가 2만건이 Queue에 한꺼번에 쌓여 있게 되면서 B task에 요청에 들어와도 기존에 쌓여있던 A task때문에 제대로 처리 못하는 상황이 발생했다.



## 1. Celery Routing

일단 Routing 이라는 이름에서 볼 수 있듯이 Celery task를 routing 해주는 것이 Celery Routing 이다. (?)

이걸 통해서 A task는 A worker로 돌리고, B task는 B worker로 돌릴 수 있게 해주면서 A task에 일이 몰려도 B는 그냥 느긋하게 쾌적한 결과를 받아낼 수 있다는 장점이 있다.

(돈 더 주면 쾌적하게 사용할 수 있는 premium_task 같은 걸 만들면 잘 되지 않을까..)



## 2. 적용 방법

일단 이렇게 되어 있다고 가정하도록 해보자.

tasks.py

```python
from celery import Celery

import time

app = Celery('tasks', broker='redis://localhost:6379', backend='redis://localhost:6379')

@app.task
def slow_task(x):
	time.sleep(x)
	return x


@app.task
def quick_task(x):
	return x
```



run_tasks.py

```python
from tasks import slow_task, quick_task

for i in range(10, 100):
	slow_task.delay(i)

quick_task.delay(100)
```



일단 이렇게 짠 뒤에 아래와 같이 실행하면 worker가 정상적으로 돌아간다.

```bash
$ celery -A tasks worker --loglevel=info
```



worker가 돌아 가는걸 확인한 뒤에 내가 원하는 task를 돌리도록 하는 `run_tasks.py` 파일을 실행해 보자

```bash
$ python run_tasks.py
.. (task_ids) ...
```



여기서 이렇게 하면 위에서 명시한 slow_task가 89개가 모두 return 될 때 까지 quick_task는 실행조차 못한다.

근데 문제는 앞에 80개 정도면 적당한 시간에 실행 되겠지만 만약에 앞에 2만개가 쌓여 있다면..?

실제로 이것 때문에 계속 문제가 발생했고, slow_task에서 task는 이미 만료되서 quick_task의 내용만 받으면 되는 상황이였는데 이걸 받지 못해 실제로 엄청난 delay가 발생하면서 제대로 된 정보를 제공하지 못하는 엄청난 문제가 발생했었다.



이를 해결하기 위해서는 아래와 같이 코드를 고치면 된다.

tasks.py

```python
from celery import Celery
from kombu import Queue

import time

app = Celery('tasks', broker='redis://localhost:6379', backend='redis://localhost:6379')
app.conf.task_default_queue = 'default'
app.conf.task_queues = (
	Queue('slow_tasks', routing_key='slow.#'),
	Queue('quick_tasks', routing_key='quick.#'),
)


@app.task
def slow_task(x):
	time.sleep(x)
	return x


@app.task
def quick_task(x):
	return x
```



run_tasks.py

```python
from tasks import slow_task, quick_task

for i in range(10, 100):
	slow_task.apply_async(args=[i], queue='slow_tasks')

quick_task.apply_async(args=[100], queue='quick_tasks')
```



이렇게 하고 아래와 같이 worker를 각각 돌려주면 된다.

```bash
$ celery -A tasks worker -Q slow_tasks --loglevel=info
$ celery -A tasks worker -Q quick_tasks --loglevel=info
```

실제로는 foreground여서 이렇게 실행하지는 못하지만 설명을 위해 이렇게 명시 하였습니다.



이렇게 실행하고 `run_tasks.py` 파일을 실행하게 되면 우리가 원하는 대로 quick_task와 slow_task 가 분리되서 돌아가기에 좀 더 효율적으로 분리해서 실행시킬 수 있다.



## 3. 참고 링크

[Celery 공식 사이트](http://docs.celeryproject.org/en/master/userguide/routing.html)



