---
title: Simple Service Timer with Twisted
description: "Write simple timer service with Twisted"
date: 2023-09-18 12:34:56
en: 1
id_url: "/twisted-servis-timer"
categories: [python, twisted]
---

A timer service can be very useful to help us write a script that do a task every some time.
[Twisted][twisted_main] will help us to do this.

This time, we will write a script that do 2 tasks with each timer.

## Envrironment Preparation

TO start out code, we prepare the Python environment.

```sh
# Make environment on `venv` directory using Python 3
$ virtualenv -ppython3 venv

# Activate the environment
source venv/bin/activate

# Install twisted
pip install twisted[tls]
# Twisted HTTP Client
pip install treq
```

## A Simple Twisted Service

`timer-service.py`

```python
from twisted.application import service
from twisted.logger import Logger

# Main service
application = service.Application('My Timer App')
log = Logger()
```

We will run this script using [`twistd`](https://docs.twisted.org/en/stable/core/howto/basics.html#twistd) command. See [_this link_: Twisted Application][tx_app] for more information.


To run this script use command below
```sh
# run twistd --help to show usage of this command
twistd -ny timer-service.py
```

## Task 1: Show Current Time Task

Now we write a function that show current time.
This function going to be called in every 5 seconds.

`timer-service.py`
```python
# ... added after the code before
def get_current_time():
    log.info("Current time: {current_time}", current_time=time.asctime())


# Call every 5 seconds
timer_service_1 = TimerService(step=5, callable=get_current_time)
timer_service_1.setServiceParent(application)

```

## Task 2: Get My IP Address

This task will fetch our IP from `https://httpbin.org/ip` using [treq][treq].

```python
# ... tambahkan setelah kode sebelummnya
class GetMyIPService:

    @inlineCallbacks
    def show_my_ip(self, response):
        response = yield response.json()
        log.info("My IP: {response}", response=response['ip'])

    def call(self):
        d = treq.get('https://httpbin.org/ip')
        d.addCallback(self.show_my_ip)
        return d


get_ip_srv = GetMyIPService()
# Call every 15 seconds
timer_service_2 = TimerService(step=15, callable=get_ip_srv.call)
timer_service_2.setServiceParent(application)
```

## Complete Code

```python
import time

import treq
from twisted.application import service
from twisted.application.internet import TimerService
from twisted.internet.defer import inlineCallbacks
from twisted.logger import Logger

# Main service
application = service.Application('My Timer App')
# Logger
log = Logger()


def get_current_time():
    log.info("Current time: {current_time}", current_time=time.asctime())


# Call every 5 seconds
timer_service_1 = TimerService(step=5, callable=get_current_time)
timer_service_1.setServiceParent(application)


class GetMyIPService:

    @inlineCallbacks
    def show_my_ip(self, response):
        response = yield response.json()
        log.info("My IP: {response}", response=response['ip'])

    def call(self):
        d = treq.get('https://httpbin.org/ip')
        d.addCallback(self.show_my_ip)
        return d


get_ip_srv = GetMyIPService()
# Call every 15 seconds
timer_service_2 = TimerService(step=15, callable=get_ip_srv.call)
timer_service_2.setServiceParent(application)
```

## Run the Service

Run with command
```sh
# run twistd --help to show usage of this command
twistd -ny timer-service.py
```

Log message example
```
2023-09-19T00:51:51+0700 [-] Loading timer-service.py...
2023-09-19T00:51:51+0700 [-] Loaded.
2023-09-19T00:51:51+0700 [twisted.scripts._twistd_unix.UnixAppLogger#info] twistd 23.8.0 (/home/jockerz/jockerz.github.io/codes/venv/bin/python 3.11.2) starting up.
2023-09-19T00:51:51+0700 [twisted.scripts._twistd_unix.UnixAppLogger#info] reactor class: twisted.internet.epollreactor.EPollReactor.
2023-09-19T00:51:51+0700 [<unknown>#info] Current time: Tue Sep 19 00:51:51 2023
2023-09-19T00:51:51+0700 [twisted.web.client._HTTP11ClientFactory#info] Starting factory _HTTP11ClientFactory(<function HTTPConnectionPool._newConnection.<locals>.quiescentCallback at 0x7fbeef0d2d40>, <twisted.internet.endpoints._WrapperEndpoint object at 0x7fbeef0c7390>)
2023-09-19T00:51:52+0700 [<unknown>#info] My IP: 182.253.127.230
2023-09-19T00:51:56+0700 [<unknown>#info] Current time: Tue Sep 19 00:51:56 2023
2023-09-19T00:52:01+0700 [<unknown>#info] Current time: Tue Sep 19 00:52:01 2023
2023-09-19T00:52:06+0700 [<unknown>#info] Current time: Tue Sep 19 00:52:06 2023
2023-09-19T00:52:06+0700 [<unknown>#info] My IP: 182.253.127.230
2023-09-19T00:52:11+0700 [<unknown>#info] Current time: Tue Sep 19 00:52:11 2023
```


## Referensi

- [Twisted][twisted_main]
- [Dokumentasi twisted][twisted_docs]
- [Twisted Application][tx_app]
- [treq][treq]


[twisted_main]: https://twisted.org/
[twisted_docs]: https://docs.twisted.org/en/stable/
[tx_app]: https://docs.twisted.org/en/stable/core/howto/basics.html
[treq]: https://treq.readthedocs.
