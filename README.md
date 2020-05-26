# signed_xmlrpc - Send signed XML RPC Requests

[![PyPI version](https://img.shields.io/pypi/v/ftrace.svg?logo=pypi&logoColor=FFE873)](https://pypi.org/project/ftrace/)
[![Supported Python versions](https://img.shields.io/pypi/pyversions/ftrace.svg?logo=python&logoColor=FFE873)](https://pypi.org/project/ftrace/)
[![PyPI downloads](https://img.shields.io/pypi/dm/ftrace.svg)](https://pypistats.org/packages/ftrace)
[![GitHub](https://img.shields.io/github/license/manfred-kaiser/python-ftrace.svg)](LICENSE)

`signed_xmlrpc` is a python library send signed xml rpc requests.

This library can be used in cyber defense exercises when communication with a compromised server
and using credentials like usernames and passwords is not possible, because an attacker can use those to compromise more services and servers.

> :warning: **do not use this library in proiduction environments!**

## Installation

`pip install signed_xmlrpc`

## Create Key Pair

At this time, the ecdsa library (https://pypi.org/project/ecdsa/) is used to handle signature verification.

```python

from ecdsa import SigningKey
sk = SigningKey.generate()

# private_key
print(base64.b64encode(sk.to_string()))

# public_key
print(base64.b64encode(sk.verifying_key.to_string()))

```


## Example Server

```python
import base64

from ecdsa import VerifyingKey
from signed_xmlrpc.server import SignedXMLRPCServer, SignedRequestHandler

public_key = b'dmTk8IGtxQBC4lPuk9tXUIJqbiz4G01qLEzmt5Fmh9AkpqOWwcSyyVeDczrhGWe7'

# if the signature is not required, the standard python xmlrpc library can be used as client
SignedRequestHandler.REQUIRE_SIGNATURE = True

SignedXMLRPCServer(
    VerifyingKey.from_string(base64.b64decode(public_key)),
    ('0.0.0.0', 8081)
).serve_forever()
```

## Example Client

```python
import base64
from ecdsa import SigningKey
from signed_xmlrpc.client import SigningTransport
import xmlrpc

private_key = b'BxbHQpNKpwKmYOs1RDSMg1vkIYsTTP3o'

server = xmlrpc.client.ServerProxy(
    'http://127.0.0.1:8081',
    transport=SigningTransport(
        private_key=SigningKey.from_string(
            base64.b64decode(private_key)
        )
    )
)
print(server.ping())
```
