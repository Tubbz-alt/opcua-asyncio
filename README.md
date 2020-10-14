OPC UA / IEC 62541 Client and Server for Python >= 3.7 and pypy3 .
http://freeopcua.github.io/, https://github.com/FreeOpcUa/opcua-asyncio

[![Build Status](https://travis-ci.org/FreeOpcUa/opcua-asyncio.svg?branch=master)](https://travis-ci.org/FreeOpcUa/opcua-asyncio)
[![Scrutinizer Code Quality](https://scrutinizer-ci.com/g/FreeOpcUa/opcua-asyncio/badges/quality-score.png?b=master)](https://scrutinizer-ci.com/g/FreeOpcUa/opcua-asyncio/?branch=master)

[![PyPI Package](https://badge.fury.io/py/asyncua.svg)](https://badge.fury.io/py/asyncua)

# opcua-asyncio

This repository is a fork of [python-opcua](https://github.com/FreeOpcUa/python-opcua) to rebase it completely on asyncio and drop support for Python < 3.7.
This library has also [sync wrapper](https://github.com/FreeOpcUa/opcua-asyncio/blob/master/asyncua/sync.py) over async API which may completely replace python-opcua in the future.

## Motivation

opcua-asyncio is an asyncio-based asynchronous OPC UA client based on python-opcua, removing hacks for support of python < 3.7.
Asynchronous programming allows for simpler code (e.g. less need for locks) and potentially performance gains.

---

OPC UA binary protocol implementation is quite complete and has been tested against many different OPC UA stacks. API offers both a low level interface to send and receive all UA defined structures and high level classes allowing to write a server or a client in a few lines. It is easy to mix high level objects and low level UA calls in one application.

Most low level code is autogenerated from xml specification, thus adding missing functionality to client or server is often trivial.

coverage.py reports a test coverage of over 95 % of code, most of non-tested code is autogenerated code that is not used yet.


# Installation

With pip

    pip install asyncua

# Usage

We assume that you already have some experience with Python, the asyncio module, the async / await syntax and the concept of asyncio Tasks.

## Client class

The `Client` class provides a high level API for connecting to APU UA servers, session management and access to basic
address space services.
The client can be used as a context manager. The client will automatically connect before the code inside the `with`
statement is executed. When your code leaves the `with` statement the client will disconnect.


```python
from asyncua import Client

async with Client(url='opc.tcp://localhost:4840/freeopcua/server/') as client:
    while True:
        # Do something with client
        node = client.get_node('i=85')
        value = await node.read_value()
```

Of course you can also call the `connect`, `disconnect` methods yourself if you do not want to use the context manager.

See the example folder and the code for more information on the client API.

## Node class

The `Node` class provides a high level API for management of nodes as well as data access services.

## Subscription class

The `Subscription` class provides a high level API for management of monitored items.

## Server class

The `Server` class provides a high level API for creation of OPC UA server instances.

# Documentation

The API remains mostly unchanged in regards to [python-opcua](https://github.com/FreeOpcUa/python-opcua).
Main difference is that many methods have been refactored to return coroutines.
Please have a look at [the examples](https://github.com/FreeOpcUa/opcua-asyncio/blob/master/examples) and/or the code.

The original documentation (for python-opcua) is available here [ReadTheDocs](http://python-opcua.readthedocs.org/en/latest/).

A simple GUI client is available: https://github.com/FreeOpcUa/opcua-client-gui

Browse the examples: https://github.com/FreeOpcUa/opcua-asyncio/tree/master/examples

A good starting point are the minimal examples.
Minimal client example: https://github.com/FreeOpcUa/opcua-asyncio/blob/master/examples/client-minimal.py
Minimal server example: https://github.com/FreeOpcUa/opcua-asyncio/blob/master/examples/server-minimal.py

A set of command line tools also available: https://github.com/FreeOpcUa/opcua-asyncio/tree/master/tools
* `uadiscover `(find_servers, get_endpoints and find_servers_on_network calls)
* `uals `(list children of a node)
* `uahistoryread`
* `uaread `(read attribute of a node)
* `uawrite `(write attribute of a node)
* `uacall `(call method of a node)
* `uasubscribe `(subscribe to a node and print datachange events)
* `uaclient `(connect to server and start python shell)
* `uaserver `(starts a demo OPC UA server)  
  `tools/uaserver --populate --certificate cert.pem --private_key pk.pem`

How to generate certificate: https://github.com/FreeOpcUa/opcua-asyncio/tree/master/examples/generate_certificate.sh

## Client support

What works:

* connection to server, opening channel, session
* browsing and reading attributes value
* getting nodes by path and nodeids
* creating subscriptions
* subscribing to items for data change
* subscribing to events
* adding nodes
* method call
* user and password
* history read
* login with certificate
* communication encryption
* removing nodes

Tested servers: freeopcua C++, freeopcua Python, prosys, kepware, beckhoff, winCC, B&R, …

Not implemented yet:

* localized text feature
* XML protocol
* UDP
* WebSocket
* maybe automatic reconnection...


## Server support

What works:

* creating channel and sessions
* read/set attributes and browse
* getting nodes by path and nodeids
* autogenerate address space from spec
* adding nodes to address space
* datachange events
* events
* methods
* basic user implementation (one existing user called admin, which can be disabled, all others are read only)
* encryption
* certificate handling
* removing nodes
* history support for data change and events
* more high level solution to create custom structures

Tested clients: freeopcua C++, freeopcua Python, uaexpert, prosys, quickopc

Not yet implemented:

* UDP
* WebSocket
* session restore
* alarms
* XML protocol
* views
* localized text features
* better security model with users and password


### Running a server on a Raspberry Pi

Setting up the standard address-space from XML is the most time-consuming step of the startup process which may lead to
long startup times on less powerful devices like a Raspberry Pi. By passing a path to a cache-file to the server constructor,
a shelve holding the address space will be created during the first startup. All following startups will make use of the
cache-file which leads to significantly better startup performance (~3.5 vs 125 seconds on a Raspberry Pi Model B).


# Development

Code follows PEP8 apart for line lengths which should be max 120 characters and OPC UA structures that keep camel case
from XML definition.

All protocol code is under opcua directory

- `asyncua/ua` contains all UA structures from specification, most are autogenerated
- `asyncua/common` contains high level objects and methods used both in server and client
- `asyncua/client` contains client specific code
- `asyncua/server` contains server specific code
- `asyncua/utils` contains some utilities function and classes
- `asyncua/tools` contains code for command lines tools
- `schemas` contains the XML and text files from specification and the python scripts used to autogenerate code
- `tests` contains tests
- `docs` contains files to auto generate documentation from doc strings
- `examples` contains many example files
- `examples/sync` contains many example files using sync API
- `tools` contains python scripts that can be used to run command line tools from repository without installing

## Running tests:

```
pytest -v -s
```

## Coverage

```
pytest -v -s --cov asyncua --cov-report=html
```
