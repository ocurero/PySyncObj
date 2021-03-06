# PySyncObj

[ ![Build Status] [travis-image] ] [travis] [ ![Coverage Status] [coverage-image] ] [coverage] [ ![Release] [release-image] ] [releases] [ ![License] [license-image] ] [license] [ ![gitter] [gitter-image] ] [gitter] [ ![docs] [docs-image] ] [docs]

[travis-image]: https://travis-ci.org/bakwc/PySyncObj.svg?branch=master
[travis]: https://travis-ci.org/bakwc/PySyncObj

[coverage-image]: https://coveralls.io/repos/github/bakwc/PySyncObj/badge.svg?branch=master
[coverage]: https://coveralls.io/github/bakwc/PySyncObj?branch=master

[release-image]: https://img.shields.io/badge/release-0.2.3-blue.svg?style=flat
[releases]: https://github.com/bakwc/PySyncObj/releases

[license-image]: https://img.shields.io/badge/license-MIT-blue.svg?style=flat
[license]: LICENSE.txt

[gitter-image]: https://badges.gitter.im/bakwc/PySyncObj.svg
[gitter]: https://gitter.im/bakwc/PySyncObj?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge

[docs-image]: https://readthedocs.org/projects/pysyncobj/badge/?version=latest
[docs]: http://pysyncobj.readthedocs.io/en/latest/

PySyncObj is a python library for building fault-tolerant distributed systems. It provides the ability to replicate your application data between multiple servers.

- It use [raft protocol](http://raft.github.io/) for leader election and log replication.
- It supports log compaction. It use fork for copy-on-write while serializing data on disk.
- It supports [dynamic membership changes ](https://github.com/bakwc/PySyncObj/wiki/Dynamic-membership-change) (cluster reconfiguration).
- It supports in-memory and on-disk serialization. You can use in-memory mode for small data and on-disk for big one.
- It has encryption - you can set password and use it in external network.
- It supports python2 and python3 on linux, macos and windows. No dependencies required (only optional one, eg. cryptography).
- Configurable event loop - it can works in separate thread with it's own event loop - or you can call onTick function inside your own one.
- Convenient interface - you can easily transform arbitrary class into a replicated one (see example below).

## Content
 * [Install](#install)
 * [Basic Usage](#usage)
 * [API Documentation](http://pysyncobj.readthedocs.io)
 * [Performance](#performance)

## Install
PySyncObj itself:
```bash
pip install pysyncobj
```
Cryptography for encryption (optional):
```bash
pip install cryptography
```

## Usage
Consider you have a class that implements counter:
```python
class MyCounter(object):
	def __init__(self):
		self.__counter = 0

	def incCounter(self):
		self.__counter += 1

	def getCounter(self):
		return self.__counter
```
So, to transform your class into a replicated one:
 - Inherit it from SyncObj
 - Initialize SyncObj with a self address and a list of partner addresses. Eg. if you have `serverA`, `serverB` and `serverC` and want to use 4321 port, you should use self address `serverA:4321` with partners `[serverB:4321, serverC:4321]` for your application, running at `serverA`; self address `serverB:4321` with partners `[serverA:4321, serverC:4321]` for your application at `serverB`; self address `serverC:4321` with partners `[serverA:4321, serverB:4321]` for app at `serverC`.
 - Mark all your methods that modifies your class fields with `@replicated` decorator.
So your final class will looks like:
```python
class MyCounter(SyncObj):
	def __init__(self):
		super(MyCounter, self).__init__('serverA:4321', ['serverB:4321', 'serverC:4321'])
		self.__counter = 0

	@replicated
	def incCounter(self):
		self.__counter += 1

	def getCounter(self):
		return self.__counter
```
And thats all! Now you can call `incCounter` on `serverA`, and check counter value on `serverB` - they will be synchronized. You can look at [examples](https://github.com/bakwc/PySyncObj/tree/master/examples) and [test_syncobj.py](https://github.com/bakwc/PySyncObj/blob/master/test_syncobj.py) for more use-cases or read [API documentation](http://pysyncobj.readthedocs.io).

## Performance
![15K rps on 3 nodes; 14K rps on 7 nodes;](http://pastexen.com/i/Ge3lnrM1OY.png "RPS vs Cluster Size")
![22K rps on 10 byte requests; 5K rps on 20Kb requests;](http://pastexen.com/i/0RIsrKxJsV.png "RPS vs Request Size")
