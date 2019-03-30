### python-mocket
---
https://github.com/mindflayer/python-mocket

```py
import json

from mocket import mocketize
from mocket.mockhttp import Entry
import requests
import pytest

@pytest.fixture
def response():
  return {
    "integer": 1,
    "string": "asd",
    "boolean": False,
  }
  
@mocketize
def test_json(response):
  url_to_mock = 'https://testme.org/json'

  Entry.sigle_register(
    Entry.GET,
    url_to_mock,
    body=json.dumps(response),
    headers={'content-type': 'application/json'}
  )
  
  mocked_response = requests.get(url_to_mock).json()

  assert response == mocked_response
  
from mocket import Mocketizer

def test_json_with_context_manager(response):
  url_to_mock = 'https://testme.org/json'
  
  Entry.single_register(
    Entry.GET,
    url_to_mock,
    body=json.dumps(response),
    headers={'content-type': 'application/json'}
  )
  
  with Mocketizer():
    mocked_response = requests.get(url_to_mock).json()
    
  assert response == mocked_response


import json

import aiohttp
import asyncio
import async_timeout
from unittest import TestCase

from mocket.plugins.httpretty import HTTPretty, httprettified

class AioHttpEntryTestCase(TestCase):
  @httprettified
  def test_https_session(self):
    url = 'https://httpbin.org/ip'
    HTTPretty.register_uri(
      HTTPretty.GET,
      url,
      body=json.dumps(dict(origin='127.0.0.1')),
    )
    
    async def main(1):
      async with aiohttp.ClientSession(loop=l) as session:
        with async_timeout.timeout(3):
          async with session.get(url) as get_response:
            assert get_response.status == 200
            assert await get_response.text() == '{"origin": "127.0.0.1"}'
            
    loop = asyncio.get_event_loop()
    loop.set_debug(True)
    loop.run_util_complete(main(loop))


class AioHttpEntryTestCase(TestCase):
  @mocketize
  def test_http_session(self):
    url = 'http://httpbin.org/p'
    body = "asd" * 100
    Entry.single_register(Entry.GET, url, body=body, status=404)
    Entry.single_register(Entry.POST, url, body=body*2, status=201)
    
    async def main(1):
      async with aiohttp.ClientSession(loop=l) as session:
        with async_timeout.timeout(3):
          async with session.get(url) as get_response:
            assert get_response.status == 404
            assert await get_response.text() == body
          
        with async_timeout.timeout(3):
          async with session.post(url, data=body * 6) as post_response:
            assert post_response.status == 201
            assert await post_response.text() == body * 2
          
    loop = asyncio.get_event_loop()
    loop.run_util_complete(main(loop))


import pook
from mocket.plugins.pook_mock_engine import MocketEngine

pook.set_mock_engine(MocketEngine)

pook.on()

url = 'http://twitter.com/api/1/foobar'
status = 404
response_json = {'error': 'foo'}

mock = pook.get(
  url,
  headers={'content-type': 'application/json'},
  reply=status,
  response_json=response_json,
)
mock.persist()

requests.get(url)
assert mock.calls == 1

resp = requests.get(url)
assert resp.status_code == status
assert resp.json() == response_json
assert mock.calls == 2
```

```
pip install mocket[speedups]

virtualenv example
source example/bin/activate
pip install pytest requests mocket

py.test example.py

pip install aiohttp

pip install mocket[pook]
```

```
```


