<img alt="erre.js" src="https://cdn.rawgit.com/GianlucaGuarini/erre/master/erre-logo.svg" width="50%"/>

[![Build Status][travis-image]][travis-url]

[![NPM version][npm-version-image]][npm-url]
[![NPM downloads][npm-downloads-image]][npm-url]
[![MIT License][license-image]][license-url]

## Description

Erre is a modern, performant and tiny (~0.5kb minified) streams script using generators that runs on modern browsers and node.
It can be used to manage any kind of sync and async event series and it's inspired to bigger libraries like:
  - [baconjs](https://baconjs.github.io/)
  - [RxJS](http://reactivex.io/rxjs/)


## Usage

You can use it to create and manipulate simple event streams

```js
import erre from 'erre'

const stream = erre(
  string => string.toUpperCase(),
  string => [...string].reverse().join('')
)

stream.onValue(console.log) // EVOL, ETAH

stream.push('love')
stream.push('hate')
```

It supports async and sync event chains thanks to [ruit](https://github.com/GianlucaGuarini/ruit)

```js
const userNamesStream = erre(
  async user => await patchUsers(user), // async function returning a users collection
  users => users.map(user => user.name)
)

userNamesStream.onValue(console.log) // ['John'...]

userNamesStream.push({
  name: 'John',
  role: 'Doctor',
  age: 24
})
```

## API

### erre(...functions)
##### @returns [`stream`](#stream)

Create an `erre` stream object.
The initial `functions` list is optional and it represents the chain of async or sync events needed to generate the final stream output received via [`onValue`](#streamonvaluecallback) callbacks

### stream

#### stream.push(value)
##### @returns `stream`

Push a new value into the stream that will be asynchronously modified and returned as argument to [`stream.onValue`](#streamonvaluecallback) method

<details>
 <summary>Example</summary>

```js
const stream = erre()
stream.onValue(console.log) // 1
stream.push(1)
```

</details>

#### stream.onValue(callback)
##### @returns `stream`

Add a callback that will be called receiving the output of the stream asynchronously


<details>
 <summary>Example</summary>

```js
const stream = erre(val => val + 1)

stream.onValue(console.log) // 2
stream.onValue(val => console.log(val * 2)) // 4

stream.push(1)
```

</details>

#### stream.onError(callback)
##### @returns `stream`

Add a callback that will be called in case of errors or promise rejections during the output generation

<details>
 <summary>Example</summary>

```js
const stream = erre(val => {
  throw 'error'
})

stream.onValue(console.log) // never called!!
stream.onError(console.log) // 'error'

stream.push(1)
```

</details>

#### stream.onEnd(callback)
##### @returns `stream`

Add a callback that will be called when the stream will be ended

<details>
 <summary>Example</summary>

```js
const stream = erre()

stream.onEnd(() => console.log('ended!')) // ended!

stream.end()
```

</details>

#### stream.connect(function)
##### @returns `stream`

Enhance the stream adding a new operation to the functions chain to generate its output

<details>
 <summary>Example</summary>

```js
const stream = erre(val => val + 1)

stream.onValue(console.log) // 2, 4
stream.push(1)

// enhance the stream
stream.connect(val => val * 2)
stream.push(1)
```

</details>

#### stream.end()
##### @returns `stream`

End the stream

<details>
 <summary>Example</summary>

```js
const stream = erre(val => val + 1)

stream.onValue(console.log) // 2
stream.push(1)

// end the stream
stream.end()

// no more events
stream.push(1)
stream.push(1)
stream.push(1)
```

</details>

#### stream.end()
##### @returns new `stream` object

Create a new stream object inheriting the function chain from its parent

<details>
 <summary>Example</summary>

```js
const stream = erre(val => val + 1)

stream.onValue(console.log) // 2, 3
stream.push(1)

const fork = stream.fork()
fork.onValue(console.log)
fork.connect(val => val * 10) // 20, 60

// 2 independent streams
fork.push(1)
stream.push(2)
fork.push(5)
```

</details>

### erre.cancel()

Static function that if returned by any of the stream functions chain can be used to filter or stop the computation

<details>
 <summary>Example</summary>

```js
const stream = erre(val => {
  if (typeof val !== 'number') return erre.cancel()
  return val + 1
})

stream.onValue(console.log) // 2, 3
stream.push(1)
stream.push('foo') // filtered
stream.push('1') // filtered
stream.push(2)
```

</details>

# TODO List

- [ ] erre.fromDOM - to stream DOM nodes events
- [ ] erre.merge - to merge multiple stream results into one

[travis-image]:https://img.shields.io/travis/GianlucaGuarini/erre.svg?style=flat-square
[travis-url]:https://travis-ci.org/GianlucaGuarini/erre

[license-image]:http://img.shields.io/badge/license-MIT-000000.svg?style=flat-square
[license-url]:LICENSE

[npm-version-image]:http://img.shields.io/npm/v/erre.svg?style=flat-square
[npm-downloads-image]:http://img.shields.io/npm/dm/erre.svg?style=flat-square
[npm-url]:https://npmjs.org/package/erre

