# Performant & Safe ArrayBuffer validations

## [Status](https://tc39.github.io/process-document/)

**Stage**: -1

**Authors/Champions**: (in alphabetical order)

- Yagiz Nizipli (node.js / OpenJS Foundation, [@yagiznizipli](https://twitter.com/yagiznizipli)
- Jordan Harband ([@ljharb](https://twitter.com/ljharb))

## The problems

### Detaching array buffers

Efficiently and safely validating and reusing array buffers requires `detaching` and `transfering` of the underlying backing store.

Current JavaScript features requires the use of `.slice()` in order to copy the underlying data of an `ArrayBuffer` to a new variable, which eventually doubles the memory usage (referencing the variable and the copied variable). The requirement of copying the arrayBuffer through a `slice` operation creates performance issues.

### Check if array buffer is detached

Currently there isn't any performant way of detecting whether an ArrayBuffer is detached or not. Following JavaScript implementation is the only possible implementation in the JavaScript userland.

```js
const assert = require('node:assert')

function isBufferDetached(buffer) {
  if (buffer.byteLength === 0) {
    try {
      new Uint8Array(buffer);
    } catch (error) {
      assert(error.name === 'TypeError');
      return true;
    }
  }
  return false
}
```

## FAQ

- What does `detach()` do?

Detaches this ArrayBuffer and all its views (typed arrays). Detaching sets the byte length of the buffer and all typed arrays to zero, preventing JavaScript from ever accessing underlying backing store. ArrayBuffer should have been externalized and must be detachable.

- What is the current usage for `detach()` in Node?

Node.js uses it's own implementation of `detachArrayBuffer` in [`webstreams`](https://github.com/nodejs/node/blob/main/lib/internal/webstreams/util.js#L134) and readable streams.

- What are the alternatives for `detach()`?

 Referencing [`ArrayBuffer.prototype.transfer()`](https://github.com/domenic/proposal-arraybuffer-transfer/tree/d4e00037420b87d0b5662c82b74d56b4ba1562ad#detaching-and-transferring) proposal:

> Some web platform APIs, notably the various postMessage() methods and the BYOB reader mode for ReadableStream, have a solution for this dillema. When you pass an ArrayBuffer (or wrapper around one, such as a typed array) to one of these APIs, they take ownership of the data block encapsulated in the ArrayBuffer.

- Does any of the engines support a similar functionality?

v8 & Webkit already supports similar functionality.

## References

- `detach()`
    - [v8 API](https://v8docs.nodesource.com/node-18.2/d5/d6e/classv8_1_1_array_buffer.html#abb7a2b60240651d16e17d02eb6f636cf)
    - [Webkit Implementation](https://github.com/WebKit/WebKit/blob/6545977030f491dd87b3ae9fd666f6b949ae8a74/Source/JavaScriptCore/runtime/ArrayBuffer.h#L307)
    - [Node.js alternative Public API](https://github.com/nodejs/node/pull/45512)
- `isDetached`
    - [Node.js Internal Implementation](https://github.com/nodejs/node/pull/45568)
    - [v8 implementation](https://github.com/v8/v8/commit/9df5ef70ff18977b157028fc55ced5af4bcee535)
    - [Webkit implementation](https://github.com/WebKit/WebKit/blob/6545977030f491dd87b3ae9fd666f6b949ae8a74/Source/JavaScriptCore/runtime/ArrayBuffer.h#L308)
    - [Node.js alternative Public API](https://github.com/nodejs/node/pull/45512)
    - [v8 optimization issue on Node.js](https://github.com/nodejs/node/blob/main/lib/querystring.js#L472)

## Possible Solutions

### `ArrayBuffer.prototype.detach()`

This is a proposal to add a new method, `detach()`, to JavaScript's `ArrayBuffer` class. 

### `ArrayBuffer.prototype.isDetached`

This is a proposal to add a new instance variable, `isDetached`, to JavaScript's `ArrayBuffer` class.

