# Stackpress Library

[![NPM Package](https://img.shields.io/npm/v/@stackpress/lib.svg?style=flat)](https://www.npmjs.com/package/@stackpress/lib)
[![Tests Status](https://img.shields.io/github/actions/workflow/status/stackpress/lib/test.yml)](https://github.com/stackpress/lib/actions)
[![Coverage Status](https://coveralls.io/repos/github/stackpress/lib/badge.svg?branch=main)](https://coveralls.io/github/stackpress/lib?branch=main)
[![Commits](https://img.shields.io/github/last-commit/stackpress/lib)](https://github.com/stackpress/lib/commits/main/)
[![License](https://img.shields.io/badge/license-Apache%202.0-blue.svg?style=flat)](https://github.com/stackpress/lib/blob/main/LICENSE)

Shared library used across Stackpress projects. Standardize type 
definitions across different projects and potentially save time for 
developers who would otherwise need to define these common types 
themselves.

## Install

```bash
$ npm i @stackpress/lib
```

## Usage

 - [Data](#data)
   - [Nest](#nest)
   - [ReadonlyMap](#rmap)
   - [ReadonlyNest](#rnest)
   - [ReadonlySet](#rset)
 - [Event](#event)
   - [EventEmitter](#emitter)
   - [EventRouter](#router)
   - [EventTerminal](#terminal)
 - [Queue](#queue)
   - [ItemQueue](#item)
   - [TaskQueue](#task)
 - [File System](#system)
 - [Exception](#exception)
 - [Reflection](#reflection)
 - [Status](#status)

<a name="data"></a>

## Data

The data library helps with common data management scenarios.

  - [Nest](#nest)
  - [ReadonlyMap](#rmap)
  - [ReadonlyNest](#rnest)
  - [ReadonlySet](#rset)

<a name="nest"></a>

### Nest

A nest is a data object that contain other nested data objects or arrays. 

```js
import { Nest, UnknownNest } from '@stackpress/lib';

const nest = new Nest<UnknownNest>({
  foo: { bar: 'zoo' }
});

registry.set('foo', 'zoo', ['foo', 'bar', 'zoo'])

nest.has('foo') //--> true
nest.has('foo', 'bar') //--> true
nest.has('bar', 'foo') //--> false
nest.get('foo', 'zoo') //--> ['foo', 'bar', 'zoo']
nest.get('foo', 'zoo', 1) //--> 'bar'
nest.get<string>('foo', 'zoo', 1) // as string

registry.remove('foo', 'bar')

nest.has('foo', 'bar') //--> false
nest.has('foo', 'zoo') //--> true
```

<a name="rmap"></a>

### Readonly Map

A type of `Map` that is readonly.

```js
import { ReadonlyMap } from  '@stackpress/lib';

const map = new ReadonlyMap([['foo', 'bar'], ['bar', 'zoo']]);

map.get('foo') //--> bar
map.has('bar') //--> zoo

map.entries() //--> [['foo', 'bar'], ['bar', 'zoo']]

map.forEach(console.log);
```

<a name="rnest"></a>

### Readonly Nest

A type of `Nest` that is readonly.

```js
import { ReadonlyNest, UnknownNest } from '@stackpress/lib';

const nest = new Nest<UnknownNest>({
  foo: { bar: 'zoo' }
});

nest.has('foo') //--> true
nest.has('foo', 'bar') //--> true
nest.has('bar', 'foo') //--> false
nest.get('foo', 'zoo') //--> ['foo', 'bar', 'zoo']
nest.get('foo', 'zoo', 1) //--> 'bar'
nest.get<string>('foo', 'zoo', 1) // as string
```

<a name="rset"></a>

### Readonly Set

A type of `Set` that is readonly.

```js
import { ReadonlySet } from  '@stackpress/lib';

const set = new ReadonlySet(['foo', 'bar', 'zoo']);

set.size //--> 3
set.has('bar') //--> true
set.values() //--> ['foo', 'bar', 'zoo']

set.forEach(console.log);
```

<a name="event"></a>

## Event

The event library is a set of tools for event driven projects.

  - [EventEmitter](#emitter)
  - [EventRouter](#router)
  - [EventTerminal](#terminal)

<a name="emitter"></a>

### EventEmitter

A class that implements the observer pattern for handling events. 
The interface follows `node:events`, but you can set priority levels per 
event action. EventEmitter works on server and browser.

```js
import { EventEmitter } from '@stackpress/lib';

type EventMap = Record<string, [ number ]> & {
  'trigger something': [ number ]
};

const emitter = new EventEmitter<EventMap>()

emitter.on('trigger something', async x => {
  console.log('something triggered', x + 1)
})

emitter.on(/trigger (something)/, async x => {
  console.log('(something) triggered', x + 2)
}, 2)

await emitter.trigger('trigger something', 1)
```

<a name="router"></a>

### EventRouter

An `expressjs` like router that uses `EventEmitter` in the back-end.
Like `EventEmitter`, you can set priority levels per route and works 
also in the browser.

```js
import { EventRouter } from '@stackpress/lib';

type RouteMap = Record<string, [ Map, Map ]>;

const router = new EventRouter<RouteMap>()

router.get('/foo/bar', async (req, res) => {
  res.set('foo', 'bar')
})

router.all('/foo/bar', async (req, res) => {
  res.set('bar', req.get('bar'))
}, 2)

//router.connect('/foo/bar', async (req, res) => {})
//router.delete('/foo/bar', async (req, res) => {})
//router.head('/foo/bar', async (req, res) => {})
//router.options('/foo/bar', async (req, res) => {})
//router.patch('/foo/bar', async (req, res) => {})
//router.post('/foo/bar', async (req, res) => {})
//router.put('/foo/bar', async (req, res) => {})
//router.trace('/foo/bar', async (req, res) => {})
//router.route('METHOD', '/foo/bar', async (req, res) => {})

await router.emit('GET /foo/bar', new Map([['foo', 'zoo']]), new Map())
```

<a name="terminal"></a>

### EventTerminal

A basic event driven cli toolset that uses events in the background. 
This enables using events on the command line.

```js
import { EventTerminal } from '@stackpress/lib';

const cli = new EventTerminal(process.argv)

cli.on('foo', async params => {
  const foo = cli.expect([ 'foo', 'f' ], 'bar')
  EventTerminal.info(foo) //--> bar
  console.log(params) //--> { foo: 'bar' }
})

await cli.run()
```

<a name="queue"></a>

## Queue

The queue library is used to order items like callbacks and return 
the correct sequence.

  - [ItemQueue](#item)
  - [TaskQueue](#task)

<a name="item"></a>

### ItemQueue

An item queue orders and consumes items sequencially (FIFO by default).

```js
import { ItemQueue } from '@stackpress/lib';

const queue = new ItemQueue<string>()

queue.push('a')

queue.shift('b')

queue.add('c', 10)

queue.consume() //--> c
queue.consume() //--> b
queue.consume() //--> a
```

<a name="task"></a>

### TaskQueue

A task queue orders and runs callbacks sequencially.

```js
import { TaskQueue } from '@stackpress/lib';

const queue = new TaskQueue()

queue.push(async x => {
  console.log(x + 1)
})

queue.shift(async x => {
  await Helper.sleep(2000)
  console.log(x + 2)
})

queue.add(async x => {
  console.log(x + 3)
}, 10)

await queue.run(1)
```

<a name="system"></a>

## FileSystem

The file system library is an interface to interact with various file 
systems *(ie. node:fs, virtual fs, browser fs, webpack fs, etc..)* .
It just requires the minimum functions compared to `node:fs`. 

```js
interface FileSystem {
  existsSync(path: string): boolean;
  readFileSync(path: string, encoding: BufferEncoding): string;
  realpathSync(string: string): string;
  lstatSync(path: string): FileStat;
  writeFileSync(path: string, data: string): void;
  mkdirSync(path: string, options?: FileRecursiveOption): void
  createReadStream(path: string): FileStream;
  unlinkSync(path: string): void;
};
```

A file loader is a set of common tools for locating, loading, importing 
files through out your project and `node_modules`.

```js
import { NodeFS, FileLoader } from '@stackpress/lib';

const loader = new FileLoader(new NodeFS());

loader.modules() //--> ./node_modules/
loader.relative('/path/from/source.file', '/path/to/destination.file') //--> '../destination'
loader.resolve('@/project/index') //--> [cwd]/project/index.js
loader.absolute('../project/index') //--> [cwd]/project/index.js
```

<a name="exception"></a>

## Exception

Exceptions are used to give more information of an error that has occured.

```js
import { Exception } from '@stackpress/lib';

const exception = new Exception('Invalid Parameters: %s', 2)
  .withErrors({
    name: 'required',
    pass: 'missing number'
  })
  .withCode(500)
  .withPosition(100, 200)

exception.toResponse() //--> 
/*{
  code: 500,
  status: 'Server Error',
  error: 'Invalid Parameters 2',
  start: 100,
  end: 200
}*/

exception.trace() //--> [{ method, file, line, char}, ...]

throw Exception.for('Unknown Error')
```

<a name="reflection"></a>

## Reflection

Uses `CallSite` to produce proper stack tracing and information about 
functions being executed.

```js
import { Reflection } from '@stackpress/lib';

const reflection = Reflection.stack(); //--> Reflection

reflection[0].column //--> 3
reflection[0].file //--> /path/to/file
reflection[0].func //--> Function
reflection[0].funcName //--> 'main'
reflection[0].line //--> 3
reflection[0].method //--> <none>
reflection[0].self //--> undefined
reflection[0].toObject()
```