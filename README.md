# ES6 for Humans

<br>

### Table of Contents

* [`let`, `const` と ブロックスコープ](#1-let-const-and-block-scoping)
* [アロー関数](#2-arrow-functions)
* [関数のデフォルトパラメーター](#3-default-function-parameters)
* [スプレッド演算子　／レスト演算子](#4-spread--rest-operator)
* [オブジェクトリテラルの拡張](#5-object-literal-extensions)
* [８進数と２進数リテラル](#6-octal-and-binary-literals)
* [配列とオブジェクトの分解](#7-array-and-object-destructuring)
* [super in Objects](#8-super-in-objects)
* [Template Literal and Delimiters](#9-template-literal-and-delimiters)
* [for...of vs for...in](#10-forof-vs-forin)
* [Map and WeakMap](#11-map-and-weakmap)
* [Set and WeakSet](#12-set-and-weakset)
* [Classes in ES6](#13-classes-in-es6)
* [Symbol](#14-symbol)
* [Iterators](#15-iterators)
* [Generators](#16-generators)
* [Promises](#17-promises)

<br>

### 他言語

* [中国語 (Thanks to barretlee)](http://www.barretlee.com/blog/2016/07/09/a-kickstarter-guide-to-writing-es6/)
* [ポルトガル語 (Thanks to alexmoreno)](https://github.com/alexmoreno/ES6-para-humanos)
* [ロシア語 (Thanks to etnolover)](https://github.com/etnolover/ES6-for-humans-translation)
* [韓国語 (Thanks to scarfunk)](https://github.com/metagrover/ES6-for-humans/tree/korean-version)
* [フランス語 (Thanks to tnga)](https://github.com/metagrover/ES6-for-humans/tree/french-version)
* [スペイン語 (Thanks to carletex)](https://github.com/metagrover/ES6-for-humans/tree/spanish-version)

<br>

### 1. let, const と ブロックスコープ

宣言文 `let` はブロックスコープと呼ばれるブロックを作ることが出来ます。
ES6では、関数スコープで使用していた`var` の代わりに、`let`の使用が推奨されています。

```javascript
var a = 2;
{
    let a = 3;
    console.log(a); // 3
    let a = 5; // TypeError: Identifier 'a' has already been declared
}
console.log(a); // 2
```

ブロックスコープにおけるもう一つの宣言文は `const` です。`const` は定数を生成します。
ES6の`const`は、値への参照を示す事になります。言い換えれば、この値は凍結されるわけではなく、割り当てているだけなのです。例を見て下さい。

```javascript
{
    const B = 5;
    B = 10; // TypeError: Assignment to constant variable

    const ARR = [5, 6];
    ARR.push(7);
    console.log(ARR); // [5,6,7]
    ARR = 10; // TypeError: Assignment to constant variable
    ARR[0] = 3; // 値は変更可能
    console.log(ARR); // [3,6,7]
}
```

頭の片隅に覚えていてほしいこと:
* `let`、`const`のホイスティングは今までの変数、関数のホイスティングから様変わりしました。`let`も`const`もどちらも、巻き上げるが、その宣言の前にアクセスすることは出来ません。これは、[Temporal Dead Zone(英語)](http://jsrocks.org/2015/01/temporal-dead-zone-tdz-demystified/)のためです。
* `let`と`const`のスコープは最も近い閉じたブロックになります。
* `const`を使用する際は、大文字で記述して下さい(一般的な慣習でもあります)
* `const`は宣言すると同時に定義しなければならなりません。

<br>

### 2. アロー関数

アロー関数はES6で関数を書く際の短縮表記のことです。アロー関数は`=>`に続く関数本体と、`(...)`で表される引数の一覧で定義します。

```javascript
// クラシカルな関数式
let addition = function(a, b) {
    return a + b;
};

// アロー関数で実装
let addition = (a, b) => a + b;
```

上記の例に加えて、アロー関数では`return`文を書く必要がありません。関数本体を簡潔に実装するためです。

これが通常のブロックで関数を記述した例です。

```javascript
let arr = ['apple', 'banana', 'orange'];

let breakfast = arr.map(fruit => {
    return fruit + 's';
});

console.log(breakfast); // ['apples', 'bananas', 'oranges']
```

**ちょっと待って！　もう一つ...**

アロー関数はコードそのものを短くするわけではありません。`this`を束縛する行為と密接に関係しています。

アロー関数の動作は、`this`の動きとともに通常の関数とは異なります。JavaScriptにおける其々の関数は`this`の文脈を定義できます。しかし、アロー関数が捉える`this`は、閉じた文脈となります。次のコードを見てください。

```javascript
function Person() {
    // Person()コンストラクターが定義する`this`はインスタンスそのものだ
    this.age = 0;

    setInterval(function growUp() {
        // strict mode　ではない時、 grouUp() 関数は `this` を 
        // globalオブジェクトとして定義する。Person()コンストラクターが定義した`this`
        // とは異なる
        this.age++;
    }, 1000);
}
var p = new Person();
```

ES3,ES5において、以上の事案に対してはthisを変数に割り当てることで対応してきました。

```javascript
function Person() {
    var self = this;
    self.age = 0;

    setInterval(function growUp() {
        // コールバックが参照する`self`変数は想定しているオブジェクトを指し示す
        self.age++;
    }, 1000);
}
```

上記から、アロー関数は`this`の値を最も近い閉じた文脈を捉えることができるため、ネストしたアロー関数に対しても、以下のコードのように想定通りの動きをすることになります。

```javascript
function Person() {
    this.age = 0;

    setInterval(() => {
        setTimeout(() => {
            this.age++; //　`this` は適切にpersonオブジェクトを参照する
        }, 1000);
    }, 1000);
}

var p = new Person();
```
[アロー関数内の`Lexical this`についてよく知りたければ参照](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions#Lexical_this)

<br>

### 3. 関数のデフォルトパラメーター

ES6では関数定義時にでデフォルトパラメーターを設定することができます。実例をみてください。

```javascript
let getFinalPrice = (price, tax = 0.7) => price + price * tax;
getFinalPrice(500); // 850
```

<br>

### 4. スプレッド演算子　／レスト演算子

`...` 演算子はスプレッド演算子、またはレスト演算子をとして動きます。使用の仕方によって動きが異なります。

イテラブルな何かと使用すると、`...`はスプレッド演算子として働く。

```javascript
function foo(x, y, z) {
    console.log(x, y, z);
}

let arr = [1, 2, 3];
foo(...arr); // 1 2 3
```

`...`のもう一つのよく知られた使い方は値を集めて配列にすることだ。これはレスト演算子として参照される。

```javascript
function foo(...args) {
    console.log(args);
}
foo(1, 2, 3, 4, 5); // [1, 2, 3, 4, 5]
```

<br>

### 5. オブジェクトリテラルの拡張

ES6ではオブジェクトリテラルの宣言の際に、プロパティの初期化と関数メソッドの定義を短縮して記述することができます。これはオブジェクトリテラルの定義からプロパティのキーを計算することができるためです。

```javascript
function getCar(make, model, value) {
    return {
        // プロパティの値を短縮して記述することで、
        // キーと変数名が一致したプロパティになります
        make,  // make: make   と同じ
        model, // model: model と同じ
        value, // value: value と同じ

        // computed values now work with
        // object literals
        // 記述された値はオブジェクトリテラルとして機能します。
        ['make' + make]: true,

        // メソッド定義の短縮形では`function`キーワードとコロンを省略することができます
        depreciate() {
            this.value -= 2500;
        }
    };
}

let car = getCar('Kia', 'Sorento', 40000);
console.log(car);
// {
//     make: 'Kia',
//     model:'Sorento',
//     value: 40000,
//     makeKia: true,
//     depreciate: function()
// }
```

<br>

### 6. ８進数と２進数リテラル

ES6では新たに８進数と２進数リテラルをサポートしました。
`0o`か`0O`で始まる number は８進数に変換されます。
以下のコードを見てください。

```javascript
let oValue = 0o10;
console.log(oValue); // 8

let bValue = 0b10; // 0b or 0B for binary
console.log(bValue); // 2
```

<br>

### 7. 配列とオブジェクトの分解

分解によってオブジェクトと配列を扱う際に一時的な変数の使用を避けることができます。

```javascript
function foo() {
    return [1, 2, 3];
}
let arr = foo(); // [1,2,3]

let [a, b, c] = foo();
console.log(a, b, c); // 1 2 3

function bar() {
    return {
        x: 4,
        y: 5,
        z: 6
    };
}
let { x: a, y: b, z: c } = bar();
console.log(a, b, c); // 4 5 6
```

<br>

### 8. オブジェクトにおける`super`の使用

ES6では`super`関数をプロトタイプと一緒に使用することを可能になりました。

```javascript
var parent = {
    foo() {
        console.log("Hello from the Parent");
    }
}

var child = {
    foo() {
        super.foo();
        console.log("Hello from the Child");
    }
}

Object.setPrototypeOf(child, parent);
child.foo(); // Hello from the Parent
             // Hello from the Child
```

<br>

### 9. テンプレートリテラルとデリミタ

ES6では文字列への代入をかんたんにできます。代入したと同時に自動的に評価されます。

* <code>\`${ ... }\`</code>は変数をレンダリングできる
* <code>\`</code> `\`バックスラッシュはデリミタとして使用する

```javascript
let user = 'Kevin';
console.log(`Hi ${user}!`); // Hi Kevin!
```

<br>

### 10. for...of vs for...in
* `for...of` 配列のようなイテラブルなオブジェクトをイテレート(順繰りに処理)する

```javascript
let nicknames = ['di', 'boo', 'punkeye'];
nicknames.size = 3;
for (let nickname of nicknames) {
    console.log(nickname);
}
// di
// boo
// punkeye
```

* `for...in` はオブジェクトの*数え上げる事ができる*プロパティをイテレートする

```javascript
let nicknames = ['di', 'boo', 'punkeye'];
nicknames.size = 3;
for (let nickname in nicknames) {
    console.log(nickname);
}
// 0
// 1
// 2
// size
```

<br>

### 11. Map and WeakMap

ES6 introduces new set of data structures called `Map` and `WeakMap`. Now, we actually use maps in JavaScript all the time. In fact every object can be considered as a `Map`.

An object is made of keys (always strings) and values, whereas in `Map`, any value (both objects and primitive values) may be used as either a key or a value. Have a look at this piece of code:

```javascript
var myMap = new Map();

var keyString = "a string",
    keyObj = {},
    keyFunc = function() {};

// setting the values
myMap.set(keyString, "value associated with 'a string'");
myMap.set(keyObj, "value associated with keyObj");
myMap.set(keyFunc, "value associated with keyFunc");

myMap.size; // 3

// getting the values
myMap.get(keyString);    // "value associated with 'a string'"
myMap.get(keyObj);       // "value associated with keyObj"
myMap.get(keyFunc);      // "value associated with keyFunc"
```

**WeakMap**

A `WeakMap` is a Map in which the keys are weakly referenced, that doesn’t prevent its keys from being garbage-collected. That means you don't have to worry about memory leaks.

Another thing to note here- in `WeakMap` as opposed to `Map` *every key must be an object*.

A `WeakMap` only has four methods `delete(key)`, `has(key)`, `get(key)` and `set(key, value)`.

```javascript
let w = new WeakMap();
w.set('a', 'b');
// Uncaught TypeError: Invalid value used as weak map key

var o1 = {},
    o2 = function(){},
    o3 = window;

w.set(o1, 37);
w.set(o2, "azerty");
w.set(o3, undefined);

w.get(o3); // undefined, because that is the set value

w.has(o1); // true
w.delete(o1);
w.has(o1); // false
```

<br>

### 12. Set and WeakSet

*Set* objects are collections of unique values. Duplicate values are ignored, as the collection must have all unique values. The values can be primitive types or object references.

```javascript
let mySet = new Set([1, 1, 2, 2, 3, 3]);
mySet.size; // 3
mySet.has(1); // true
mySet.add('strings');
mySet.add({ a: 1, b:2 });
```

You can iterate over a set by insertion order using either the `forEach` method or the `for...of` loop.

```javascript
mySet.forEach((item) => {
    console.log(item);
    // 1
    // 2
    // 3
    // 'strings'
    // Object { a: 1, b: 2 }
});

for (let value of mySet) {
    console.log(value);
    // 1
    // 2
    // 3
    // 'strings'
    // Object { a: 1, b: 2 }
}
```
Sets also have the `delete()` and `clear()` methods.

**WeakSet**

Similar to `WeakMap`, the `WeakSet` object lets you store weakly held *objects* in a collection. An object in the `WeakSet` occurs only once; it is unique in the WeakSet's collection.

```javascript
var ws = new WeakSet();
var obj = {};
var foo = {};

ws.add(window);
ws.add(obj);

ws.has(window); // true
ws.has(foo);    // false, foo has not been added to the set

ws.delete(window); // removes window from the set
ws.has(window);    // false, window has been removed
```

<br>

### 13. Classes in ES6

ES6 introduces new class syntax. One thing to note here is that ES6 class is not a new object-oriented inheritance model. They just serve as a syntactical sugar over JavaScript's existing prototype-based inheritance.

One way to look at a class in ES6 is just a new syntax to work with prototypes and contructor functions that we'd use in ES5.

Functions defined using the `static` keyword implement static/class functions on the class.

```javascript
class Task {
    constructor() {
        console.log("task instantiated!");
    }
    
    showId() {
        console.log(23);
    }
    
    static loadAll() {
        console.log("Loading all tasks..");
    }
}

console.log(typeof Task); // function
let task = new Task(); // "task instantiated!"
task.showId(); // 23
Task.loadAll(); // "Loading all tasks.."
```

**extends and super in classes**

Consider the following code:

```javascript
class Car {
    constructor() {
        console.log("Creating a new car");
    }
}

class Porsche extends Car {
    constructor() {
        super();
        console.log("Creating Porsche");
    }
}

let c = new Porsche();
// Creating a new car
// Creating Porsche
```

`extends` allow child class to inherit from parent class in ES6. It is important to note that the derived constructor must call `super()`.

Also, you can call parent class's method in child class's methods using `super.parentMethodName()`

[Read more about classes here](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Classes)

A few things to keep in mind:

* Class declarations are not hoisted. You first need to declare your class and then access it, otherwise ReferenceError will be thrown.
* There is no need to use `function` keyword when defining functions inside a class definition.

<br>

### 14. Symbol

A `Symbol` is a unique and immutable data type introduced in ES6. The purpose of a symbol is to generate a unique identifier but you can never get any access to that identifier.

Here’s how you create a symbol:

```javascript
var sym = Symbol("some optional description");
console.log(typeof sym); // symbol
```

Note that you cannot use `new` with `Symbol(…)`.

If a symbol is used as a property/key of an object, it’s stored in a special way that the property will not show up in a normal enumeration of the object’s properties.

```javascript
var o = {
    val: 10,
    [Symbol("random")]: "I'm a symbol",
};

console.log(Object.getOwnPropertyNames(o)); // val
```

To retrieve an object’s symbol properties, use `Object.getOwnPropertySymbols(o)`


<br>

### 15. Iterators

An iterator accesses the items from a collection one at a time, while keeping track of its current position within that sequence. It provides a `next()` method which returns the next item in the sequence. This method returns an object with two properties: done and value.

ES6 has `Symbol.iterator` which specifies the default iterator for an object. Whenever an object needs to be iterated (such as at the beginning of a for..of loop), its *@@iterator* method is called with no arguments, and the returned iterator is used to obtain the values to be iterated.

Let’s look at an array, which is an iterable, and the iterator it can produce to consume its values:

```javascript
var arr = [11,12,13];
var itr = arr[Symbol.iterator]();

itr.next(); // { value: 11, done: false }
itr.next(); // { value: 12, done: false }
itr.next(); // { value: 13, done: false }

itr.next(); // { value: undefined, done: true }
```

Note that you can write custom iterators by defining `obj[Symbol.iterator]()` with the object definition.

<br>

### 16. Generators

Generator functions are a new feature in ES6 that allow a function to generate many values over time by returning an object which can be iterated over to pull values from the function one value at a time.

A generator function returns an **iterable object** when it's called.
It is written using the new `*` syntax as well as the new `yield` keyword introduced in ES6.

```javascript
function *infiniteNumbers() {
    var n = 1;
    while (true) {
        yield n++;
    }
}

var numbers = infiniteNumbers(); // returns an iterable object

numbers.next(); // { value: 1, done: false }
numbers.next(); // { value: 2, done: false }
numbers.next(); // { value: 3, done: false }
```

Each time *yield* is called, the yielded value becomes the next value in the sequence.

Also, note that generators compute their yielded values on demand, which allows them to efficiently represent sequences that are expensive to compute, or even infinite sequences.

<br>

### 17. Promises

ES6 has native support for promises. A *promise* is an object that is waiting for an asynchronous operation to complete, and when that operation completes, the promise is either fulfilled(resolved) or rejected.

The standard way to create a Promise is by using the `new Promise()` constructor which accepts a handler that is given two functions as parameters. The first handler (typically named `resolve`) is a function to call with the future value when it's ready; and the second handler (typically named `reject`) is a function to call to reject the Promise if it can't resolve the future value.

```javascript
var p = new Promise(function(resolve, reject) {  
    if (/* condition */) {
        resolve(/* value */);  // fulfilled successfully
    } else {
        reject(/* reason */);  // error, rejected
    }
});
```

Every Promise has a method named `then` which takes a pair of callbacks. The first callback is called if the promise is resolved, while the second is called if the promise is rejected.

```javascript
p.then((val) => console.log("Promise Resolved", val),
       (err) => console.log("Promise Rejected", err));
```

Returning a value from `then` callbacks will pass the value to the next `then` callback.

```javascript
var hello = new Promise(function(resolve, reject) {  
    resolve("Hello");
});

hello.then((str) => `${str} World`)
     .then((str) => `${str}!`)
     .then((str) => console.log(str)) // Hello World!
```

When returning a promise, the resolved value of the promise will get passed to the next callback to effectively chain them together.
This is a simple technique to avoid "callback hell".

```javascript
var p = new Promise(function(resolve, reject) {  
    resolve(1);
});

var eventuallyAdd1 = (val) => {
    return new Promise(function(resolve, reject){
        resolve(val + 1);
    });
}

p.then(eventuallyAdd1)
 .then(eventuallyAdd1)
 .then((val) => console.log(val)) // 3
```
