# ctx-core/be

A general purpose, modular, &amp; scalable context library for contextual state management. Keeps state encapsulated in the be_ memo functions. Useful for:

* UI state
* reactive state management
* complex application state graphs
* build sequencing & devops
* web animation api orchestration

| imports          | size  |
|------------------|:-----:|
| ctx_             | 33 B  |
| ns_ctx_          | 85 B  |
| be_              | 101 B  |
| be_ ctx_         | 131 B |
| be_ ns_ctx_      | 191 B |
| be_ ctx_ ns_ctx_ | 191 B |

The source of this library is in the [ctx-core](https://github.com/ctx-core/ctx-core) package.

```ts
import { be_ } from 'ctx-core/be'
``` 

## install

```shell
npm i @ctx-core/be
# or
npm i ctx-core 
```

## basic usage

The be function memoizes the return value of the callback on the `ctx`. The be function also memoizes on `another_ctx`.

[//]: @formatter:off
```ts
import { be_, ctx_ } from 'ctx-core/be'
const ctx = ctx_()
const i_am_memo_ = be_(()=>{
	console.info('I am a memo')
  return 123
})
i_am_memo_(ctx) // prints 'I am a memo' & returns 123
i_am_memo_(ctx) // returns 123
const another_ctx = ctx_()
i_am_memo_(another_ctx) // prints 'I am a memo' & returns 123
i_am_memo_(another_ctx) // returns 123
```
[//]: @formatter:on

## usage with a reaactive library such as rmemo

The be function can hold reactive state. Dependency injecting `ctx` allows sharing the same reactive state across in-process libraries. Calling state in a particular order is not necessary. Dependencies load before dependents. ctx-core/be works with any reactive state management library.

[//]: @formatter:off
```ts
import { be_, ctx_, memo_ } from 'ctx-core/rmemo' // rmemo also exports be_ & ctx_
const ctx = ctx_()
const i_am_rmemo$_ = be_(()=>{
	console.info('I m rmemo')
	return memo_(()=>1)
})
i_am_rmemo$_(ctx)       // prints 'I am a rmemo' & returns a memo
i_am_rmemo$_(ctx)       // returns a memo
i_am_rmemo$_(ctx)()     // returns 1
i_am_rmemo$_(ctx).set(2) 
i_am_rmemo$_(ctx)()     // returns 2
```
[//]: @formatter:on

## calling other be functions with helpers

Be functions can call each other. Defining state enforces the correct dependency load order. Loading any piece of state in the graph loads enforces the correct load order. This is useful for reactive state dependencies. This covers dev ops. Each step only runs one time in the correct order. In this example, rmemo handles async installs.

[//]: @formatter:off
```ts
import { be_, be_memo_pair_, ctx_, memo_, nullish_none_,rmemo__wait } from 'ctx-core/rmemo' // rmemo also exports be_ & ctx_
const [pkg_a$_, pkg_a_] = be_memo_pair_((ctx, $)=>{
    pkg_a__install()
      .then(()=>$.val = true)
      .catch(err=>console.error(err)) // remember to handle promise errors
    return false // initial synchronous return value
  })
const [pkg_b$_, pkg_b_] = be_memo_pair_((ctx, $)=>{
	if (pkg_a_(ctx)) {
		pkg_b__install()
			.then(()=>$.val = true)
			.catch(err=>console.error(err))
	}
	return false
})
const [pkg_c$_, pkg_c_] = be_memo_pair_((ctx, $)=>{
	if (pkg_b_(ctx)) {
		pkg_c__install__async(ctx)
      .then(()=>$.val = true)
      .catch(err=>console.error(err))
  }
	return false
})
const [pkg_d$_, pkg_d_] = be_memo_pair_((ctx, $)=>{
	if (pkg_a_(ctx) && pkg_c_(ctx)) {
		pkg_c__install(ctx)
      .then(()=>$.val = true)
      .catch(err=>console.error(err))
  }
	return false
})
const ctx = ctx_()
await Promise.all([
	// wait for packages to return a truthy value with a 20 second timeout. Even if the packages called out of order, the installations will occur in the correct order exactly 1 time.
	rmemo__wait(pkg_d$_(ctx), $=>$, 20_000),
	rmemo__wait(pkg_c$_(ctx), $=>$, 20_000),
	rmemo__wait(pkg_b$_(ctx), $=>$, 20_000),
	rmemo__wait(pkg_a$_(ctx), $=>$, 20_000),
])
```
[//]: @formatter:on
