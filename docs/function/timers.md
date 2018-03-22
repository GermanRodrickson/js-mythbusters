# Timers

**Timers** are functions to be called at some future period of time.

This process is highly related with how Node.js event loop works, where the function execution is divided in a serie of steps.

```  
	Node.js Event Loop steps  

   ┌───────────────────────┐
┌─>│        timers         │
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
│  │     I/O callbacks     │
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
│  │     idle, prepare     │
│  └──────────┬────────────┘      ┌───────────────┐
│  ┌──────────┴────────────┐      │   incoming:   │
│  │         poll          │<─────┤  connections, │
│  └──────────┬────────────┘      │   data, etc.  │
│  ┌──────────┴────────────┐      └───────────────┘
│  │        check          │
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
└──┤    close callbacks    │
   └───────────────────────┘
```  
  
Depending *where* you want execute your code and your JavaScript engine (Node.js or browser) you can find different ways to do that.

## As soon as possible

If you want to delay your code to be executed it asynchronously as soon as possible in the next event loop, use **[process.nextTick](https://nodejs.org/api/process.html#process_process_nexttick_callback_args)**:

```js
process.nextTick(() => {
 console.log('enqueue to be executed asap')
})
```

It's *slightly* faster than [**setImmediate**](), another timer function available at Node.js.

The difference between both methods are:

- `process.nextTick` delay the function **before** I/O callbacks cycle.
- `setImmediate` will be execute your code **after** I/O callbacks cycle.

Here a simple benchmark you can execute to see timing difference: 

```js  
'use strict'

var bench = require('fastbench')

var run = bench([
  function benchSetImmediate (done) {
    setImmediate(done)
  },
  function benchNextTick (done) {
    process.nextTick(done)
  }
], 1024 * 1024)

// run them two times
run(run)
```

## In a moment in the future

For the rest of case, use [**setTimeout**]():

```js
setTimeout(() => {
	console.log('Delaying the execution at least 1seg.')
}, 1000)
```

These case includes the scenarios where `process.nextTick` or `setImmediate` are not available.

In this case, the *as soon as possible* possible way is `setTimeout(fn, 0)`, where **0** is the minimum quantity of time to wait after execute **fn**:

```
setTimeout(() => {
	console.log('delaying execution asap!')
}, 0)
```

Keep in mind this is not exactly equivalent than `process.nextTick` or `setImmediate`, just is the best effort to have similar effect because event loop lifecycle manipulation is limited.

## What happens with `Promise`

When you use `Promise` API, automagically the code involved is inernally wrapped using a Timer:

```js
const getRandom () => Promise.resolve(Math.random())

getRandom()
  .then((randomNumber) => {
    // This execution is delayed in the time, 
    // even the code envolved is synchronously.
    console.log(`My random number is ${randomNumber}`)
})
```

Each piece of code envolved with a Promise will be executed asynchronously.