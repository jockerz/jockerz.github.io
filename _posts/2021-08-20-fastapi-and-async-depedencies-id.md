---
title: Demo FastAPI dengan aioredis
description: "FastAPI menggunakan Async Dependensi (aioredis)"
date: 2021-08-20 12:34:56
en_url: "/fastapi-and-async-depedencies"
categories: [python, fastapi]
---


<div class="note info">
  <h5>Catatan</h5>
  <p>Link ke repository code keseluruhan ada di bawah</p>
</div>

Ide utama dari postingan ini adalah membuat objek `async redis` pada `event` `startup` supaya kita bisa menggunakan sintaks `async / await`.
Dengan pendekatan seperti pada postingan ini kita pastikan data kita di redis tidak terganggu dengan kode `test`.

Sekarang kita menulis aplikasi web API dengan [FastAPI][fastapi] dengan depedensi [`aioredis`][aioredis] untuk terhubung ke [Redis][redis] dan library [fakeredis][fakeredis] untuk __testing__.


## Persiapan

Kita siapkan __environment__ kode kita di satu direktori.

```shell
mkdir fastapi-demo-aioredis
cd fastapi-demo-aioredis
virtualenv -p python3 venv
source venv/bin/activate
pip install aioredis fastapi uvicorn
```

Struktur direktory ini (`fastapi-demo-aioredis`)  kurang lebih seperti berikut.

```
app
├── app.py
├── deps.py
├── __init__.py
└── routes.py
tests
├── conftest.py
├── __init__.py
└── test_routes.py
main.py
```


## Koding

`app/app.py`
```python
import aioredis
from fastapi import Depends, FastAPI
from fastapi.requests import Request
from fastapi.responses import JSONResponse

from .routes import router

REDIS_URL = 'redis://localhost'
REDIS_DB = 0
REDIS_PASS = 'RedisPassword'


def create_app(redis=None):
    """
    redis: Redis instance / coroutine
    """

    app = FastAPI()

    app.include_router(router)

    @app.on_event('startup')
    async def startup():
        nonlocal redis

        if redis is None:
            # if redis is None, create the instance on start up
            redis = await aioredis.from_url(
                REDIS_URL, db=REDIS_DB, password=REDIS_PASS
            )
        assert await redis.ping()

    @app.middleware('http')
    async def http_middleware(request: Request, call_next):
        nonlocal redis

        # Initial response when exception raised on 
        #  `call_next` function
        err = {'error': True, 'message': "Internal server error"},
        response = JSONResponse(err, status_code=500)
        
        try:
            request.state.redis = redis
            response = await call_next(request)
        finally:
            return response

    return app

```

`app/deps.py`
```python
from fastapi.requests import Request


def get_redis(request: Request):
    return request.state.redis

```

Sekarang mari kita buat `router`-nya.

`app/routes.py`
```python
from fastapi import APIRouter, Depends

from app.deps import get_redis

router = APIRouter()


@router.post('/post')
async def save_key_value(
    key: str, value: str, redis=Depends(get_redis)
):
    await redis.set(key, value)
    return {'success': True, 'data': {key: value}}


@router.get('/get')
async def get_value(key: str, redis=Depends(get_redis)):
    value = await redis.get(key)
    return {
        'success': True if value else False,
        'data': {key: value}
    }

```

Lalu jalankan.

```shell
uvicorn main:app --lifespan on  # --port 8001
```

Jika sudah berjalan normar, kita bisa akses `UI Swagger`-nya di [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)


## Test

### Persiapan

Install modul-modul yang diperlukan untuk test.
```shell
pip install pytest pytest-asyncio fakeredis httpx pytest
```


### Test Fixtures

Karena kita mempersiapankan kode fungsi `create_app` yang menerima argument instansi `redis`, kita bisa gunakan modul `aioredis` dari `fakeredis` untuk testing.

Sekarang kita siapkan fixture untuk test kita. 

Docs: [pytest fixture][pytest]

`tests/conftest.py`
```python
import pytest
from fakeredis import aioredis
from httpx import AsyncClient

from app.app import create_app


@pytest.fixture
async def redis():
    print(dir(aioredis))
    return aioredis.FakeRedis(encoding='utf-8')


@pytest.fixture
def app(redis):
    app = create_app(redis=redis)
    return app


@pytest.fixture
def http_client(app):
    return AsyncClient(app=app, base_url='http://test')

```

Kemudian kita tulis kode test.

`tests/test_routes.py`
```python
import pytest


@pytest.mark.asyncio
class TestRoutes:
    async def test_post_success(self, http_client):
        async with http_client as http:
            resp = await http.post('/post', params={
                'key': 'test_key', 'value': 'test_value'
            })

        assert resp.status_code == 200
        assert resp.json()['success'] == True
        assert resp.json()['data'] \
               == {'test_key': 'test_value'}

    async def test_get_success(self, http_client, redis):
        await redis.set('test_get_key', 'test_get_value')

        async with http_client as http:
            resp = await http.get('/get', params={
                'key': 'test_get_key'
            })
        
        assert resp.status_code == 200
        assert resp.json()['success'] == True
        assert resp.json()['data'] \
               == {'test_get_key': 'test_get_value'}

    async def test_get_not_found(self, http_client):
        async with http_client as http:
            resp = await http.get('/get', params={
                'key': 'test_not_avail'
            })
        
        assert resp.status_code == 200
        assert resp.json()['success'] == False

```

Jalankan test.

```shell
pytest
```

Contoh keluaran `command`-nya.
```
============= test session starts =============
platform linux -- Python 3.7.3, pytest-6.2.4, py-1.10.0, pluggy-0.13.1
rootdir: /home/jockerz/fastapi-aioredis
plugins: asyncio-0.15.1, anyio-3.3.0
collected 3 items

tests/test_routes.py ...             [100%]

============= 3 passed in 0.06s =============
```

Link repository: [github.com/jockerz/fastapi-aioredis][repo]


[aioredis]: https://aioredis.readthedocs.io/en/latest/
[fakeredis]: https://github.com/jamesls/fakeredis/
[fastapi]: https://fastapi.tiangolo.com
[pytest]: https://docs.pytest.org/en/latest/how-to/fixtures.html
[redis]: https://redis.io/
[repo]: https://github.com/jockerz/fastapi-demo-aioredis
