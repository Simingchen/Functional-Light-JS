# 章节 6: 值的不变性质

在[第5章](ch5.md)中，我们讨论了减少副作用的重要性:应用程序的状态可能意外改变并导致意外(bug)的方式。使用这种地雷的地方越少，我们对代码就越有信心，代码的可读性也就越好。我们这一章的主题也是做同样的努力去减少副作用。

如果编程风格的幂等性是关于定义一个值更改操作，以便它只能影响状态一次，那么现在我们将注意力转向将发生更改的次数从1减少到0的目标。

现在让我们来探讨值的不变性，即在程序中我们只使用不可更改的值。

## 原始类型的不变性

js原始类型(`number`, `string`, `boolean`, `null`, and `undefined`)的值已经是不可变的;你无法改变他们:

```js
// 无效，也没有任何意义
2 = 2.5;
```

然而，js确实有一个特殊的行为，看起来它允许改变这样的原始类型值：“boxing”（boxing所谓的装箱，是指将基本数据类型转换为对应的引用类型的操作。而装箱又分为隐式装箱和显式装箱）。当您访问某些基本类型值（特别是`number`, `string`和`boolean`）的属性时，js会自动将该值包装在其对象对应项（分别为`number`, `string`和`boolean`）中。

考虑:

```js
var x = 2;

x.length = 4;

x;              // 2
x.length;       // undefined
```

数字通常没有`length`属性可用。`x.length = 4`设置试图添加一个新属性，但它会自动失败（或根据您的观点被忽略/丢弃）;`x` 继续保持简单的基本数字`2`。

但js允许`x.length = 4`语句运行看起来很麻烦，如果不是因为其他原因，读者可能会感到困惑。好消息是，如果使用strict模式（`"use strict";`），这样的语句将抛出错误。

如果您尝试改变这样一个值的显式装箱对象表示，会怎么样？

```js
var x = new Number( 2 );

// 运行正常
x.length = 4;
```

这个代码段中的`x`保存了对对象的引用，因此可以添加和更改自定义属性而不存在任何问题。

像`number`这样的简单原语的不可变性可能看起来相当明显。但是`string`值呢?JS开发人员有一个非常普遍的误解，认为字符串就像数组，因此可以更改。JS语法甚至暗示他们是“数组一样”与`[ ]访问操作符。然而，字符串也是不可变的:

```js
var s = "hello";

s[1];               // "e"

s[1] = "E";
s.length = 10;

s;                  // "hello"
```

尽管能够像访问数组一样访问`s[1]`，但JS字符串并不是真正的数组。设置 `s[1] = "E"`和`s.length = 10`和之前做过`x.length = 4`一样，都自动失败了。在严格模式下，这些赋值将失败，因为`1`属性和`length`属性在这个基本`string`值上都是只读的。

有趣的是，即使是被框起来的`String`对象值也会(基本上)不可变，因为如果您更改现有属性，它会在严格模式下抛出错误:

```js
"use strict";

var s = new String( "hello" );

s[1] = "E";         // error
s.length = 10;      // error

s[42] = "?";        // OK

s;                  // "hello"
```

## 此值到彼值

我们将在本章中进一步解释这个概念，但是首先要有一个清晰的理解:值的不变性并不意味着我们不能在程序的过程中改变值。一个没有改变状态的程序不是一个很有趣的程序!这也不意味着我们的变量不能有不同的值。这些都是关于价值不变性的普遍误解。

值不变性意味着*当*我们需要更改程序中的状态时，我们必须创建并跟踪一个新值，而不是更改一个现有值。

例如:

```js
function addValue(arr) {
    var newArr = [ ...arr, 4 ];
    return newArr;
}

addValue( [1,2,3] );    // [1,2,3,4]
```

注意，我们没有更改`arr`引用的数组，而是创建了一个新数组(`newArr`)，其中包含现有值加上新的`4` 值。

根据我们在[第5章](ch5.md)中讨论的关于副作用的原因/影响来分析`addValue(..)`。它是纯洁的吗?它是否具有参考透明性?给定相同的数组，它总是会产生相同的输出吗?它有没有副作用和副作用?答案为：是的

假设`[1,2,3]`数组表示来自以前一些操作的数据序列，并且我们存储在某个变量中。这是我们目前的状态。如果我们想计算应用程序的下一个状态，我们调用`addValue(..)`。但是我们希望下一个状态的计算是直接和明确的。因此，`addValue(..)`操作接受直接输入，返回直接输出，并避免了修改`arr`引用的原始数组的副作用。

这意味着我们可以计算`[1,2,3,4]`的新状态，并完全控制状态的转换。程序的任何其他部分都不能意外地将我们提前转换到那个状态，或者完全转换到另一个状态，比如`[1,2,3,5]`。通过对我们的值进行约束并将它们视为不可变的，我们大大减少了令人惊讶的表面区域，使我们的程序更易于阅读、推理和最终信任。

`arr`引用的数组实际上是可变的。我们只是选择不去改变它，所以我们实践了价值不变的精神。

我们也可以对对象使用复制而不是变异的策略。思考下:

```js
function updateLastLogin(user) {
    var newUserRecord = Object.assign( {}, user );
    newUserRecord.lastLogin = Date.now();
    return newUserRecord;
}

var user = {
    // ..
};

user = updateLastLogin( user );
```

### 非局部

非原始值由引用持有，当作为参数传递时，复制的是引用，而不是值本身。

如果在程序的一个部分中有一个对象或数组，并将其传递给在程序另一个部分中的函数，那么该函数现在可以通过这个引用副本影响值，并以可能意想不到的方式对其进行修改。

换句话说，如果作为参数传递，非原始值将变为非本地值。可能需要考虑整个程序来理解是否会更改这样的值。

思考:

```js
var arr = [1,2,3];

foo( arr );

console.log( arr[0] );
```

从表面上看，您期望`arr[0]`仍然是值`1`。但真的是这样吗?您不知道，因为`foo(..)` *可能*使用传递给它的引用副本更改了数组。

我们在前一章已经看到了一个避免这种意外的技巧:

```js
var arr = [1,2,3];

foo( [...arr] );         // 拷贝一个数组

console.log( arr[0] );      // 1
```

稍后，我们将看到另一种策略，用于保护不受来自下面的值意外突变的影响。

##  重赋值

你如何描述“常量”?在你进入下一段之前想一下这个问题。

<p align="center">
    * * * *
</p>

你们中的一些人可能会这么描述，“一个不能改变的值”，“一个不能改变的变量”，或者类似的东西。意思相近，但不完全正确。我们应该对常量使用的精确定义是:不能重新分配的变量。

这种吹毛求疵是非常重要的，因为它阐明了一个常量实际上与这个值无关，只是说，无论一个常量持有什么值，这个变量都不能被重新分配任何其他值。但它没有说明价值本身的性质。

思考:

```js
var x = 2;
```

如前所述，值`2`是一个不可更改(不可变)的原始值。如果我把代码改成:

```js
const x = 2;
```

`const`关键字的出现，通常被称为“常量声明”，实际上根本没有改变`2`的性质;它只是变得不可改变的，而且将永远不变。

这是真的，这后面的一行将报错失败:

```js
// 试着改变`x`看看!
x = 3;      // Error!
```

但是，我们没有改变任何关于值的东西。我们试图重新分配变量`x`。所涉及的值几乎是偶然的。

要证明`const`与值的本质无关，思考下:

```js
const x = [ 2 ];
```

数组是常量吗?*不* `x`是一个常量，因为它不能被重新分配。但是下面这句话完全可以:

```js
x[0] = 3;
```

为什么?因为数组仍然是完全可变的，即使`x`是一个常量。

围绕`const`和“常量”的混淆只处理赋值而不处理值语义，可以长篇大论了。似乎每一种语言中都有相当多的开发人员遇到了相同类型的混淆。实际上，Java反对使用const，并引入了一个新的关键字final，至少在一定程度上是为了将自己从“常量”语义的混乱中分离出来。

抛开混淆的影响，如果`const`与创建不可变值没有任何关系，那么它对于FPer有什么重要性呢?

### 意图

`const`的使用告诉代码的读者，*那个*变量不会被重新分配。作为意图的一个信号，`const`通常被高度赞扬为JavaScript的一个受欢迎的附加功能，并在代码可读性方面得到了普遍的改进。

在我看来，这主要是炒作;这些说法没有多少实质内容。我只看到用这种方式表示你的意图所带来的最轻微的好处。而当你把这个数字与几十年来围绕它的困惑(暗示着值的不变性)进行对比时，我不认为`const`有什么分量。

为了支持我的断言，让我们考虑作用域。`const`创建了一个块作用域的变量，这意味着变量只存在于一个本地化的块中:

```js
// 一些代码

{
    const x = 2;

    // 几行代码
}

// 一些代码
```

通常，块被认为是最好的设计只有几行。如果您的代码块超过10行，大多数开发人员会建议您重构。所以`const x = 2`最多只适用于后面的9行代码。

程序的任何其他部分都不能影响`x`的赋值。

我的主张是，程序的可读性基本上与这个相同:

```js
// 一些代码

{
    let x = 2;

    // 几行代码
}

// 一些代码
```

如果您查看`let x = 2;`后面的几行代码，您将能够很容易地看出`x`实际上“没有”重新分配。对我来说，这是一个**更强的信号**——实际上不是重新分配它!——而不是使用一些容易混淆的 `const`声明来表示“不会重新分配它”。

此外，让我们考虑一下这段代码可能第一眼就传达给读者的信息:

```js
const magicNums = [1,2,3,4];
```

难道您的代码的读者(错误地)认为您的意图是永远不修改数组，这至少是可能的(可能的)吗?对我来说，这似乎是一个合理的推论。想象一下他们的疑惑，如果稍后您实际上允许`magicNums`引用的数组值发生突变。这会让他们感到惊讶吗?

更糟的是，如果您故意以某种方式修改`magicNums`，结果却不为读者所知，该怎么办?随后，在代码中，他们看到了`magicNums`的用法，并假设(同样是错误的)它仍然是`[1,2,3,4]`，因为他们将您的意图理解为“不会更改这个”。

我认为您应该使用`var`或`let`来声明变量，以保存要进行更改的值。我认为这实际上比使用`const`更清楚地表达了你的意图。

但`const`的麻烦还不止于此。还记得我们在这一章的开头说过，要将值视为不可变的，就意味着当我们的状态需要更改时，我们必须创建一个新值，而不是对它进行修改吗?一旦你创建了这个新数组，你打算怎么处理它?如果使用`const`声明对它的引用，则不能重新分配它。


```js
const magicNums = [1,2,3,4];

// 然后:
magicNums = magicNums.concat( 42 );  // 噢, 不能重新分配
```

那么，下一步怎么做？

In this light, I see `const` as actually making our efforts to adhere to FP harder, not easier. My conclusion: `const` is not all that useful. It creates unnecessary confusion and restricts us in inconvenient ways. I only use `const` for simple constants like:

```js
const PI = 3.141592;
```

The value `3.141592` is already immutable, and I'm clearly signaling, "this `PI` will always be used as stand-in placeholder for this literal value." To me, that's what `const` is good for. And to be frank, I don't use many of those kinds of declarations in my typical coding.

I've written and seen a lot of JavaScript, and I just think it's an imagined problem that very many of our bugs come from accidental reassignment.

One of the reasons FPers so highly favor `const` and avoid reassignment is because of equational reasoning. Though this topic is more related to other languages than JS and goes beyond what we'll get into here, it is a valid point. However, I prefer the pragmatic view over the more academic one.

For example, I've found measured use of variable reassignment can be useful in simplifying the description of intermediate states of computation. When a value goes through multiple type coercions or other transformations, I don't generally want to come up with new variable names for each representation:

```js
var a = "420";

// later

a = Number( a );

// later

a = [ a ];
```

If after changing from `"420"` to `420`, the original `"420"` value is no longer needed, then I think it's more readable to reassign `a` rather than come up with a new variable name like `aNum`.

The thing we really should worry more about is not whether our variables get reassigned, but **whether our values get mutated**. Why? Because values are portable; lexical assignments are not. You can pass an array to a function, and it can be changed without you realizing it. But a reassignment will never be unexpectedly caused by some other part of your program.

### It's Freezing in Here

There's a cheap and simple way to turn a mutable object/array/function into an "immutable value" (of sorts):

```js
var x = Object.freeze( [2] );
```

The `Object.freeze(..)` utility goes through all the properties/indices of an object/array and marks them as read-only, so they cannot be reassigned. It's sorta like declaring properties with a `const`, actually! `Object.freeze(..)` also marks the properties as non-reconfigurable, and it marks the object/array itself as non-extensible (no new properties can be added). In effect, it makes the top level of the object immutable.

Top level only, though. Be careful!

```js
var x = Object.freeze( [ 2, 3, [4, 5] ] );

// not allowed:
x[0] = 42;

// oops, still allowed:
x[2][0] = 42;
```

`Object.freeze(..)` provides shallow, naive immutability. You'll have to walk the entire object/array structure manually and apply `Object.freeze(..)` to each sub-object/array if you want a deeply immutable value.

But contrasted with `const` which can confuse you into thinking you're getting an immutable value when you aren't, `Object.freeze(..)` *actually* gives you an immutable value.

Recall the protection example from earlier:

```js
var arr = Object.freeze( [1,2,3] );

foo( arr );

console.log( arr[0] );          // 1
```

Now `arr[0]` is quite reliably `1`.

This is so important because it makes reasoning about our code much easier when we know we can trust that a value doesn't change when passed somewhere that we do not see or control.

## Performance

Whenever we start creating new values (arrays, objects, etc.) instead of mutating existing ones, the obvious next question is: what does that mean for performance?

If we have to reallocate a new array each time we need to add to it, that's not only churning CPU time and consuming extra memory; the old values (if no longer referenced) are also being garbage collected. That's even more CPU burn.

Is that an acceptable trade-off? It depends. No discussion or optimization of code performance should happen **without context.**

If you have a single state change that happens once (or even a couple of times) in the whole life of the program, throwing away an old array/object for a new one is almost certainly not a concern. The churn we're talking about will be so small -- probably mere microseconds at most -- as to have no practical effect on the performance of your application. Compared to the minutes or hours you will save not having to track down and fix a bug related to unexpected value mutation, there's not even a contest here.

Then again, if such an operation is going to occur frequently, or specifically happen in a *critical path* of your application, then performance -- consider both performance and memory! -- is a totally valid concern.

Think about a specialized data structure that's like an array, but that you want to be able to make changes to and have each change behave implicitly as if the result was a new array. How could you accomplish this without actually creating a new array each time? Such a special array data structure could store the original value and then track each change made as a delta from the previous version.

Internally, it might be like a linked-list tree of object references where each node in the tree represents a mutation of the original value. Actually, this is conceptually similar to how **Git** version control works.

<p align="center">
    <img src="images/fig18.png" width="33%">
</p>

In this conceptual illustration, an original array `[3,6,1,0]` first has the mutation of value `4` assigned to position `0` (resulting in `[4,6,1,0]`), then `1` is assigned to position `3` (now `[4,6,1,1]`), finally `2` is assigned to position `4` (result: `[4,6,1,1,2]`). The key idea is that at each mutation, only the change from the previous version is recorded, not a duplication of the entire original data structure. This approach is much more efficient in both memory and CPU performance, in general.

Imagine using this hypothetical specialized array data structure like this:

```js
var state = specialArray( 4, 6, 1, 1 );

var newState = state.set( 4, 2 );

state === newState;                 // false

state.get( 2 );                     // 1
state.get( 4 );                     // undefined

newState.get( 2 );                  // 1
newState.get( 4 );                  // 2

newState.slice( 2, 5 );             // [1,1,2]
```

The `specialArray(..)` data structure would internally keep track of each mutation operation (like `set(..)`) as a *diff*, so it won't have to reallocate memory for the original values (`4`, `6`, `1`, and `1`) just to add the `2` value to the end of the list. But importantly, `state` and `newState` point at different versions (or views) of the array value, so **the value immutability semantic is preserved.**

Inventing your own performance-optimized data structures is an interesting challenge. But pragmatically, you should probably use a library that already does this well. One great option is [Immutable.js](http://facebook.github.io/immutable-js), which provides a variety of data structures, including `List` (like array) and `Map` (like object).

Consider the previous `specialArray` example but using `Immutable.List`:

```js
var state = Immutable.List.of( 4, 6, 1, 1 );

var newState = state.set( 4, 2 );

state === newState;                 // false

state.get( 2 );                     // 1
state.get( 4 );                     // undefined

newState.get( 2 );                  // 1
newState.get( 4 );                  // 2

newState.toArray().slice( 2, 5 );   // [1,1,2]
```

A powerful library like Immutable.js employs sophisticated performance optimizations. Handling all the details and corner-cases manually without such a library would be quite difficult.

When changes to a value are few or infrequent and performance is less of a concern, I'd recommend the lighter-weight solution, sticking with built-in `Object.freeze(..)` as discussed earlier.

## Treatment

What if we receive a value to our function and we're not sure if it's mutable or immutable? Is it ever OK to just go ahead and try to mutate it? **No.** As we asserted at the beginning of this chapter, we should treat all received values as immutable -- to avoid side effects and remain pure -- regardless of whether they are or not.

Recall this example from earlier:

```js
function updateLastLogin(user) {
    var newUserRecord = Object.assign( {}, user );
    newUserRecord.lastLogin = Date.now();
    return newUserRecord;
}
```

This implementation treats `user` as a value that should not be mutated; whether it *is* immutable or not is irrelevant to reading this part of the code. Contrast that with this implementation:

```js
function updateLastLogin(user) {
    user.lastLogin = Date.now();
    return user;
}
```

That version is a lot easier to write, and even performs better. But not only does this approach make `updateLastLogin(..)` impure, it also mutates a value in a way that makes both the reading of this code, as well as the places it's used, more complicated.

**We should treat `user` as immutable**, always, because at this point of reading the code we do not know where the value comes from, or what potential issues we may cause if we mutate it.

Nice examples of this approach can be seen in various built-in methods of the JS array, such as `concat(..)` and `slice(..)`:

```js
var arr = [1,2,3,4,5];

var arr2 = arr.concat( 6 );

arr;                    // [1,2,3,4,5]
arr2;                   // [1,2,3,4,5,6]

var arr3 = arr2.slice( 1 );

arr2;                   // [1,2,3,4,5,6]
arr3;                   // [2,3,4,5,6]
```

Other array prototype methods that treat the value instance as immutable and return a new array instead of mutating: `map(..)` and `filter(..)`. The `reduce(..)`/`reduceRight(..)` utilities also avoid mutating the instance, though they don't by default return a new array.

Unfortunately, for historical reasons, quite a few other array methods are impure mutators of their instance: `splice(..)`, `pop(..)`, `push(..)`, `shift(..)`, `unshift(..)`, `reverse(..)`, `sort(..)`, and `fill(..)`.

It should not be seen as *forbidden* to use these kinds of utilities, as some claim. For reasons such as performance optimization, sometimes you will want to use them. But you should never use such a method on an array value that is not already local to the function you're working in, to avoid creating a side effect on some other remote part of the code.

<a name="hiddenmutation"></a>

Recall one of the implementations of [`compose(..)` from Chapter 4](ch4.md/#user-content-generalcompose):

```js
function compose(...fns) {
    return function composed(result){
        // copy the array of functions
        var list = [...fns];

        while (list.length > 0) {
            // take the last function off the end of the list
            // and execute it
            result = list.pop()( result );
        }

        return result;
    };
}
```

The `...fns` gather parameter is making a new local array from the passed-in arguments, so it's not an array that we could create an outside side effect on. It would be reasonable then to assume that it's safe for us to mutate it locally. But the subtle gotcha here is that the inner `composed(..)` which closes over `fns` is not "local" in this sense.

Consider this different version which doesn't make a copy:

```js
function compose(...fns) {
    return function composed(result){
        while (fns.length > 0) {
            // take the last function off the end of the list
            // and execute it
            result = fns.pop()( result );
        }

        return result;
    };
}

var f = compose( x => x / 3, x => x + 1, x => x * 2 );

f( 4 );     // 3

f( 4 );     // 4 <-- uh oh!
```

The second usage of `f(..)` here wasn't correct, since we mutated that `fns` during the first call, which affected any subsequent uses. Depending on the circumstances, making a copy of an array like `list = [...fns]` may or may not be necessary. But I think it's safest to assume you need it -- even if only for readability sake! -- unless you can prove you don't, rather than the other way around.

Be disciplined and always treat *received values* as immutable, whether they are or not. That effort will improve the readability and trustability of your code.

## Summary

Value immutability is not about unchanging values. It's about creating and tracking new values as the state of the program changes, rather than mutating existing values. This approach leads to more confidence in reading the code, because we limit the places where our state can change in ways we don't readily see or expect.

`const` declarations (constants) are commonly mistaken for their ability to signal intent and enforce immutability. In reality, `const` has basically nothing to do with value immutability, and its usage will likely create more confusion than it solves. Instead, `Object.freeze(..)` provides a nice built-in way of setting shallow value immutability on an array or object. In many cases, this will be sufficient.

For performance-sensitive parts of the program, or in cases where changes happen frequently, creating a new array or object (especially if it contains lots of data) is undesirable, for both processing and memory concerns. In these cases, using immutable data structures from a library like **Immutable.js** is probably the best idea.

The importance of value immutability on code readability is less in the inability to change a value, and more in the discipline to treat a value as immutable.
