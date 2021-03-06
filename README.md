# Core-wrappers

[![npm status](https://img.shields.io/npm/v/core-wrappers.svg)](https://www.npmjs.org/package/core-wrappers)
[![build status](https://api.travis-ci.org/akira-cn/core-wrappers.svg?branch=master)](https://travis-ci.org/akira-cn/core-wrappers) 
[![dependency status](https://david-dm.org/akira-cn/core-wrappers.svg)](https://david-dm.org/akira-cn/core-wrappers) 
[![coverage status](https://img.shields.io/coveralls/akira-cn/core-wrappers.svg)](https://coveralls.io/github/akira-cn/core-wrappers)

Core-wrappers is a small library exporting basic wrapper functions compatible with [ES7 Decorators](https://github.com/wycats/javascript-decorators). 

## Use it in nodeJS

A version compiled to ES5 in CJS format is published to npm as core-wrappers.

```bash
npm install core-wrappers
```

## Use it on browser

**core-wrappers CDN**

```html
<script src="https://s5.ssl.qhimg.com/!eda20dea/core-wrappers.min.js"></script>
```

You can use it with any AMD loader or **standalone**

```js
function test(){
	console.log(1);
}

test = CoreWrappers.once(test);

test();
test(); //WARN: This function should not be called more than 1 times.
```

## The wrapper and the decorator

A wrapper is a function that returns another function.

Wrapper:

```js
function myWrapperWithoutArguments(fn){
	return function(){
		//do sth...
		return fn.apply(this, arguments);
	}
}

function myWrapperWithArguments(...args){
	var fn = args[args.length - 1];
	return function(){
		//do sth...
		return fn.apply(this, arguments);
	}
}
```

A decorator is a function like [ES7 Decorators](https://github.com/wycats/javascript-decorators)

```js
function myDecoratorWithoutArguments(target, key, descriptor){
	//do sth...
	return descriptor;
}

function myDecoratorWithArguments(...args){
	return function(target, key, descriptor){
		//do sth...
		return descriptor;
	}
}
```

This library can transform a wrapper to a decorator.

Wrapper in ES5:

```js
var w = require('core-wrappers');

function Foo(){

}

Foo.prototype.bar = w.debounce(100, function(){
	//do sth.
});
```

Decorator in ES7:

```js
const w = require('core-wrappers');
const debounce = w.getDecorator('debounce');

class Foo{
	@debounce(100)
	bar(){
		//do sth.
	}
}
```

## API Doc

##### For both Wrappers and Decorates

* [@allow](#allow)
* [@bind](#bind)
* [@debounce](#debounce)
* [@defer](#defer)
* [@deprecate](#deprecate)
* [@enumerable](#enumerable)
* [@methodize](#methodize)
* [@multicast](#multicast)
* [@observable](#observable)
* [@once](#once)
* [@promisify](#promisify)
* [@readonly](#readonly)
* [@repeat](#repeat)
* [@spread](#spread)
* [@supressWarnings](#supressWarnings)
* [@throttle](#throttle)

##### For Decorates only

* [@decorator](#decorator)
* [@getDecorator](#getDecorator)

-----

### @allow

**allow(times, fn)**

Creates a version of the function that can only be called servaral times. Repeated calls exteeded by max times will have no effect, returning the value of undefined. Useful for initialization functions, instead of having to set a boolean flag and then check it later.

Wrapper:

```js
var allow = require('core-wrappers').allow;

var initialize = allow(1, createApplication);
initialize();
initialize();
// Application is only created once.
```

Decorator:

```js
const w = require('core-wrappers');
const allow = w.getDecorator('allow');

let times = 0;

class Foo{
	constructor(){
		initialize();
	}
	@allow(1)
	initialize(){
	  return times++;
	}
}

let f = new Foo(), f2 = new Foo();
expect(times).to.equal(1);
```

### @bind

**bind(...args, fn)**

Forces invocations of this function to always have `this` refer to the class instance, even if the function is passed around or would otherwise lose its `this` context. e.g. `var fn = context.method;` Popular with React components.

Wrapper:

```js
var bind = require('core-wrappers').bind;

function Person(name){
	this.name = name;
}
Person.prototype.getName = function(){
	return this.name;
}

var akira = new Person('akira');
akira.getName = bind(akira, akira.getName);
var getName = akira.getName;
console.log(getName()); //akira
```

Decorator:

```js
const w = require('core-wrappers');

const bind = w.getDecorator('bind');

class Person {
  constructor(name){
  	this.name =name;
  }
  @bind
  getName() {
  	return this.name;
  }
}

const akira = new Person('akira');
akira.getName = bind(akira, akira.getName);
let getName = akira.getName;
console.log(getName()); //akira
```

### @debounce

**debounce(wait, immediate, fn)**

Creates and returns a new debounced version of the passed function which will postpone its execution until after wait milliseconds have elapsed since the last time it was invoked. Useful for implementing behavior that should only happen after the input has stopped arriving. For example: rendering a preview of a Markdown comment, recalculating a layout after the window has stopped being resized, and so on.

At the end of the wait interval, the function will be called with the arguments that were passed most recently to the debounced function.

Pass true for the immediate argument to cause debounce to trigger the function on the leading instead of the trailing edge of the wait interval. Useful in circumstances like preventing accidental double-clicks on a "submit" button from firing a second time.

Wrapper:

```js
var w = require('core-wrappers');

var i = 0;
function inc(){
  i++;
}
var inc1 = w.debounce(100, inc);
inc1();
inc1();
inc1();
expect(i).to.equal(0);
setTimeout(function(){
  console.log(i); //2
  inc1();
  inc1();
  inc1();
}, 150);
setTimeout(function(){
  console.log(i); //3
  done();
}, 550);   

var inc2 = w.debounce(100, true, inc);
inc2();
inc2();
inc2();
console.log(i); //1   
```

Decorator:

```js
const w = require('core-wrappers');
const debounce = w.getDecorator('debounce');

class Foo{
  constructor(i){
    this.i = i;
  }
  @debounce(100, true)
  inc(){
    this.i++;
  }
}

let foo = new Foo(0);
foo.inc();
foo.inc();
foo.inc();
console.log(foo.i); //1

setTimeout(function(){
  foo.inc();
  console.log(foo.i); //2
}, 150);
```

### @decorator

**decorator(wrapper, ...args)**

The default decorator. Immediately applies the provided wrapper and arguments to the method.

```js
const w = require('core-wrappers');
const decorator = w.getDecorator();

class Foo{
  @decorator(w.allow, 2)
  bar(){
    return 1;
  }
}

let f = new Foo();
expect(f.bar()).to.equal(1);
expect(f.bar()).to.equal(1);
expect(f.bar()).to.equal(undefined);
```

### @defer

**defer(promisify, fn)**

Defers invoking the func until the current call stack has cleared. Any additional arguments are provided to func when it’s invoked.

If promisify is true, it returns a promise otherwise it returns the timer id.

Wrapper:

```js
var w = require('core-wrappers');

var i = 0;
function inc(){
  return i++;
}
inc = w.defer(inc);
inc();
expect(i).to.equal(0);
process.nextTick(function(ret){
  console.log(ret); //0
  console.log(i); //1
});
```

Decorator:

```js
const w = require('core-wrappers');
const defer = w.getDecorator('defer');

class Foo{
  constructor(i){
    this.i = i;
  }

  @defer(true)
  inc(){
    return this.i++;
  }
}

let foo = new Foo(0);
foo.inc().then(function(ret){
  console.log(ret); //0
  console.log(foo.i); //1
});
```

### @delay

**delay(wait, promisify, fn)**

Invokes func after wait milliseconds. 

If promisify is true, it returns a promise otherwise it returns the timer id.

### @deprecate

**deprecate(message, url, fn)**

Calls console.warn() with a deprecation message. Provide a custom message to override the default one. You can also provide a url, for further reading.

```js
const w = require('core-wrappers');

const deprecate = w.getDecorator('deprecate');

const suppressWarnings = w.getDecorator('suppressWarnings');

class Foo{
  @deprecate
  bar(){

  }
  @deprecate('We stopped bar2', 'http://knowyourmeme.com/memes/facepalm')
  bar2(){

  }
  @suppressWarnings
  bar3(){
    this.bar();
  }
}
var foo = new Foo();
foo.bar();
foo.bar2();
foo.bar3();
```

### @enumerable

**enumerable(target, key, isEnumerable, fn)**

Marks a property as being enumerable. Note that class methods are always nonenumerable, so this is only useful for instance properties.

Wrapper:

```js
var w = require('core-wrappers');

var obj = {};

w.enumerable(obj, 'add', false, function(x, y){
  return x + y;
});

expect(obj.add(1, 2)).to.equal(3);

expect(Object.keys(obj).indexOf('add')).to.equal(-1); 
```

Decorator:

```js
const w = require('core-wrappers');

const enumerable = w.getDecorator('enumerable');

class Foo{
  //Note: class methods are nonenumerable 
  //so this is only useful for instance properties.
  @enumerable(false)
  bar = 1
}

var foo = new Foo();
expect(foo.bar).to.equal(1);
expect(Object.keys(foo).indexOf('bar')).to.equal(-1);
```

### @methodize

**methodize([...keys,] fn)**

Invoke this wrapper without keys will add `this` context to the first argument of the function.

```js
const w = require('core-wrappers');

const methodize = w.getDecorator('methodize');

class Foo{
  x = 1;
  @methodize
  bar(self, y){
    return self.x + y;
  }
}

var foo = new Foo();
expect(foo.bar(2)).to.equal(3);
```

Invoke this wrapper with keys will add `this[key1], thie[key2]...` to the arguments of the function.

```js
const w = require('core-wrappers');

const methodize = w.getDecorator('methodize');

class Foo{
  x = 1;
  y = 2;
  @methodize('x','y')
  bar(x, y, z){
    return x + y + z;
  }
}

var foo = new Foo();
expect(foo.bar(3)).to.equal(6);
``` 

### @multicast

**multicast(fn)**

Allow the first argument of the function be an array and invokes function with the elements of the array one by one.

Wrapper:

```js
var w = require('core-wrappers');

function setColor(el, color){
    return el.style.color = color;
}

var setColorMany = w.multicast(setColor);

var els = document.querySelectorAll("ul>li:nth-child(2n+1)");
console.log(els);
setColorMany(Array.from(els), "red");
```

Decorator:

```js
const w = require('core-wrappers');
let multicast = w.getDecorator('multicast');
let spread = w.getDecorator('spread');

class Collection {
  constructor(){
    this.items = [];
  }

  @spread
  @multicast
  append(item){
    this.items.push(item);
    return this.items.slice(-1)[0];
  }
}

var c = new Collection();
c.append(1,2,3);
expect(c.items[1]).to.equal(2); //2
```

### @multiset

**multiset(fn)**

Allow setter's first argument to be a json object to set many properties at once.

```js
const w = require('core-wrappers');
let multiset = w.getDecorator('multiset');

class Store{
  constructor(){
    this.map = {};
  }
  @multiset
  setItem(key, value){
    this.map[key] = value;
  }
}

let store = new Store();
store.setItem('a', 1);
store.setItem({b:2, c:3});

expect(store.a).to.equal(1);
expect(store.b).to.equal(2);
expect(store.c).to.equal(3);
```

### @readonly

**readonly(obj, prop, fn)**

Marks a property or method readonly so that it cannot be reset or deleted.

Wrapper:

```js
var obj = {};

w.readonly(obj, 'add', function(x, y){
  return x + y;
});

expect(obj.add(1, 2)).to.equal(3);

expect(function(){
  obj.add = 11;
}).to.throw(Error);

expect(function(){
  delete obj.add;
}).to.throw(Error);
```

Decorator:

```js
const w = require('core-wrappers');

const readonly = w.getDecorator('readonly');

class Foo{
  @readonly
  bar(){
    return 1;
  }
}

var foo = new Foo();
expect(foo.bar()).to.equal(1);
expect(()=>foo.bar=11).to.throw(Error);
delete foo.bar;
expect(foo.bar()).to.equal(1);
```

### @observable

**observable(fn)**

Let function to be observable so we can trigger `before` and `after` events when the function is invoked. 

```js
var w = require('core-wrappers');

function add(x, y){
  return x + y;
}
expect(add(1,2)).to.equal(3);

add = w.observable(add);

add.before = function(args){
  args[1] += 4;
}

expect(add(1,2)).to.equal(7);
```

### @once

**once(fn)**

This wrapper is a shortcut of [allow(1, fn)](#allow).

### @promisify

**promisify(fn)**

Transform an asynchronous function with a callback(err, ...args) parameter to a function returns a promise.

Wrapper:

```js
var w = require('core-wrappers');

var fs = require('fs');

var readFile = w.promisify(fs.readFile);

readFile('/path/to/file').then(function(data){

});
```

Decorator:

```js
const w = require('core-wrappers');
let promisify = w.getDecorator('promisify');

class Site{
  constructor(url){
    this.url = url;
  }
  @promisify
  getData(callback){
    let request = require('request');
    request(this.url, callback);
  }
}

var site = new Site('https://registry.npmjs.org/core-wrappers');
site.getData().then(function(res){
  var data = JSON.parse(res[1]);
  expect(data.name).to.equal('core-wrappers');
  done();
});
```

### @repeat

**repeat(times, wait, fn)**

Call function many times.

### @spread

**spread(fn)**

See [multicast](#multicast)

### @suppressWarnings

**suppressWarnings(fn)**

See [deprecate](#deprecate)

### @throttle

**throttle(wait, options, fn)**

Creates and returns a new, throttled version of the passed function, that, when invoked repeatedly, will only actually call the original function at most once per every wait milliseconds. Useful for rate-limiting events that occur faster than you can keep up with.

By default, throttle will execute the function as soon as you call it for the first time, and, if you call it again any number of times during the wait period, as soon as that period is over. If you'd like to disable the leading-edge call, pass {leading: false}, and if you'd like to disable the execution on the trailing-edge, pass 
{trailing: false}.

```js
var w = require('core-wrappers');

var i = 0;
function inc(){
  i++;
}
inc = w.throttle(500, {leading:false}, inc);
inc();
expect(i).to.equal(0);
setTimeout(function(){
  console.log(i); //1
}, 550);
```

### getDecorator

**getDecorator(nameOrWrapper)**

Transform a wrapper function to a decorator function. If you pass a string as parameter, it will find a build-in wrapper to transform. 

A **wrapper** is a function that returns other function. A **decorator** is a function like [ES7 Decorators](https://github.com/wycats/javascript-decorators).

## LICENSE

[MIT](LICENSE)
