# 章节7: 闭包和对象

## 达成共识

首先，确保当我们提到闭包和对象时，我们都达成了共识。 我们探讨JavaScript如何处理这两种机制的上下文，特别是普通的函数闭包（参见[第2章“保持作用域”](ch2.md/#keeping-scope)）和普通对象（键值对集合） ）。

普通函数闭包例子:

```js
function outer() {
    var one = 1;
    var two = 2;

    return function inner(){
        return one + two;
    };
}

var three = outer();

three();            // 3
```

普通对象例子:

```js
var obj = {
    one: 1,
    two: 2
};

function three(outer) {
    return outer.one + outer.two;
}

three( obj );       // 3
```

当谈论到“闭包”时，许多人会想到许多额外的部分，例如异步回调，甚至包含封装和信息隐藏的模块模式。 类似地，“对象”带来了类，“this”，原型以及大量其他方法和模式。

随着我们学习深入，我们将探讨这些重要的额外的部分，但是现在，我们先记住这里所说的“闭包”和“对象”的最简单的解释，这样可以使我们保持清晰。

## 相似

闭包和对象的关系可能并不明显，我们首先探讨它们的相似之处。

基于这个讨论，我先简单地确定两件事：

没有闭包的编程语言可以用对象模拟闭包。
没有对象的编程语言可以用闭包来模拟对象。

换句话说，我们可以将闭包和对象视为事物的两种不同表示。

### 状态

思考以下代码：

```js
function outer() {
    var one = 1;
    var two = 2;

    return function inner(){
        return one + two;
    };
}

var obj = {
    one: 1,
    two: 2
};
```

`inner()`函数和对象`obj`作用域都包含两个状态元素：`one`的值为`1`，`two`的值为`2`。在语法和机制上，这些状态的表示是不同的。但从概念上讲，它们非常相似。

事实上，将对象表示为闭包或将闭包表示为对象是相当简单的。来吧，尝试一下：

```js
var point = {
    x: 10,
    y: 12,
    z: 14
};
```

你想到什么了吗?

```js
function outer() {
    var x = 10;
    var y = 12;
    var z = 14;

    return function inner(){
        return [x,y,z];
    }
};

var point = outer();
```

**注意:** 每次`inner()` 函数调用并返回一个新的数组(简称, 对象!) 。原因是JS没提供在不封装多个值的情况下“返回”多个值的功能。这在技术上并不违反对象即闭包这个问题，因为它只是暴露/传递值的实现细节；状态跟踪本身仍然是基于对象的。使用ES6+数组析构，我们可以声明性地忽略临时中间数组：`var [x,y,z] = point()`。从开发人员的角度来看，这些值是单独存储的，并通过闭包(而非对象)跟踪。

如果有嵌套对象怎么办？

```js
var person = {
    name: "Kyle Simpson",
    address: {
        street: "123 Easy St",
        city: "JS'ville",
        state: "ES"
    }
};
```

同样的，可以使用嵌套闭包实现：

```js
function outer() {
    var name = "Kyle Simpson";
    return middle();

    // ********************

    function middle() {
        var street = "123 Easy St";
        var city = "JS'ville";
        var state = "ES";

        return function inner(){
            return [name,street,city,state];
        };
    }
}

var person = outer();
```

让我们试试从闭包转为对象：

```js
function point(x1,y1) {
    return function distFromPoint(x2,y2){
        return Math.sqrt(
            Math.pow( x2 - x1, 2 ) +
            Math.pow( y2 - y1, 2 )
        );
    };
}

var pointDistance = point( 1, 1 );

pointDistance( 4, 5 );      // 5
```

`distFromPoint(..)`封装了`x1`和`y1`, 但我们可以显式地传入值作为对象：

```js
function pointDistance(point,x2,y2) {
    return Math.sqrt(
        Math.pow( x2 - point.x1, 2 ) +
        Math.pow( y2 - point.y1, 2 )
    );
};

pointDistance(
    { x1: 1, y1: 1 },
    4,  // x2
    5   // y2
);
// 5
```

显式的传入`point`对象替换隐式的闭包状态。

### 行为也如此!

对象和闭包不仅表示表示状态集合的方法，而且还包含函数/方法。将数据与其行为捆绑在一起有一个奇特的名称：封装。

试想一下:

```js
function person(name,age) {
    return happyBirthday(){
        age++;
        console.log(
            `Happy ${age}th Birthday, ${name}!`
        );
    }
}

var birthdayBoy = person( "Kyle", 36 );

birthdayBoy();          // Happy 37th Birthday, Kyle!
```

内部函数`happyBirthday()` 通过闭包引入`name` 和 `age` 使其功能与状态保持一致。（注：保持了变量引用和不销毁）

我们可以通过将`this`绑定到对象来实现相同的能力：

```js
var birthdayBoy = {
    name: "Kyle",
    age: 36,
    happyBirthday() {
        this.age++;
        console.log(
            `Happy ${this.age}th Birthday, ${this.name}!`
        );
    }
};

birthdayBoy.happyBirthday();
// Happy 37th Birthday, Kyle!
```

我们仍然用`happyBirthday()`函数来表示状态数据的封装，但是使用对象而不是闭包。 而且我们不必将对象显式传递给函数（与前面的示例一样）； JavaScript的`this`绑定很容易创建一个隐式绑定。

另一方面分析此关系：闭包将单个函数与一组状态相关联，而对象可以保持相同状态，而持有相同状态的对象可以有任意数量的函数对该状态进行操作。

事实上，你甚至可以使用单个闭包作为接口来暴露多个方法。 思考以下使用两种方法的传统对象：

```js
var person = {
    firstName: "Kyle",
    lastName: "Simpson",
    first() {
        return this.firstName;
    },
    last() {
        return this.lastName;
    }
}

person.first() + " " + person.last();
// Kyle Simpson
```

只使用闭包，可以这么做：

```js
function createPerson(firstName,lastName) {
    return API;

    // ********************

    function API(methodName) {
        switch (methodName) {
            case "first":
                return first();
                break;
            case "last":
                return last();
                break;
        };
    }

    function first() {
        return firstName;
    }

    function last() {
        return lastName;
    }
}

var person = createPerson( "Kyle", "Simpson" );

person( "first" ) + " " + person( "last" );
// Kyle Simpson
```

这些程序在人机工程学上看起来有点不同，事实上只是相同程序行为的不同实现而已。

### (不)可变数据

许多人最初会认为闭包和对象在可变性方面表现不同，闭包能保护免受外部改变（影响），而对象不会。但事实证明，两种形式都有相同的可变行为。

正如我们在 [第6章](ch6.md)所讨论的，需要关心的，**值**的可变性，这是值本身的一个特征，无论它在何处或如何赋值：

```js
function outer() {
    var x = 1;
    var y = [2,3];

    return function inner(){
        return [ x, y[0], y[1] ];
    };
}

var xyPublic = {
    x: 1,
    y: [2,3]
};
```

`outer()`的内部变量`x`（基础类型）值是不可变的 - 记住，像`2`这样的基本类型都是不可变的。 但是，数组`y`（引用类型）引用的值是可变的。 `xyPublic`里的`x`和`y`属性也是这个道理。

我们可以通过指出`y`本身就是一个数组来强调对象和闭包对可变性没有影响，因此我们需要进一步分解这个例子：

```js
function outer() {
    var x = 1;
    return middle();

    // ********************

    function middle() {
        var y0 = 2;
        var y1 = 3;

        return function inner(){
            return [ x, y0, y1 ];
        };
    }
}

var xyPublic = {
    x: 1,
    y: {
        0: 2,
        1: 3
    }
};
```

如果您将其视为“turtles (aka, objects) all the way down”（注：不太理解作者用意），那么在最低级别上，所有状态数据都是基础类型，并且所有基础类型都是不可变值。

无论使用嵌套对象表示此状态，还是使用嵌套闭包表示此状态，值都是不可变的。

### 同构

“同构”这个术语在JavaScript中经常出现，通常用于指可以在服务器和浏览器中使用/共享的代码。不久前，我写了一篇博客文章，对“同构”这个词的用法进行了抨击，实际上它有一个明确而重要的含义，但却被用错地方。

以下列举这篇文章的部分节选：

> 同构是什么意思呢？ 我们可以用数学术语，社会学或生物学来解释，同构概念是两个结构相似但不相同的东西。

> 在所有这些用法中，同构和相等的区别在这里：如果两个值在所有方面完全相同，则它们是相等的，但如果它们以不同的方式表示但仍具有1对1的双向性，则它们是同构的 映射关系。

> 换句话说，如果你可以从A映射（转换）到B然后用逆映射返回到A，那么A和B可以称为是同构的。

回顾一下[第2章](ch2.md/#brief-math-review)，我们讨论了函数的数学定义，即输入和输出之间的映射，这在学术上被称为态射。 同构是一种双射（又称双向）态射的特殊情况，它不仅要求映射必须能够在任一方向上进行，而且要求行为在任何一种形式中都是相同的。

但是，我们不要考虑数学术语，而是将同构与代码联系起来。 再次引用我的博文：

> 如果真有这种东西，同构JS会是什么？它可能是一组JS代码被转换成另一组JS代码，而且（重要的是）如果有需要，可以将后者转换回前者。

正如我们在前面的 “闭包作为对象” 和 “对象作为闭包” 示例中所说的那样，它们的表达可以任意替换。因此，它们彼此是同构的。

简单地说，闭包和对象是状态(及其相关功能)的同构表示。

下次当你听到有人说 “X与Y是同构” 时，他们的意思是“X和Y可以从任何一个方向转换到另一个方向，而不会丢失特性。”

### 深入内部结构

因此，从编写代码的角度来看可以将对象看作闭包的同构表示。但我们也观察到，闭包系统实际上可以用对象实现——而且很可能是这样的!

考虑以下代码：JS如何追踪`x`变量，以便` inner()`函数在`outer()`函数运行之后仍然保持它的引用?

```js
function outer() {
    var x = 1;

    return function inner(){
        return x;
    };
}
```

我们可以想象，`outer()`的作用域(定义的所有变量的集合)被实现为一个带有属性的对象。所以，从概念上讲，在内存的某个地方，类似有这样的东西:

```js
scopeOfOuter = {
    x: 1
};
```

然后对于`inner()`函数，当创建时，它得到一个(空的)作用域对象叫做`scopeOfInner`，它通过它的`[[Prototype]]`链接到`scopeOfOuter`对象，大概是这样的:

```js
scopeOfInner = {};
Object.setPrototypeOf( scopeOfInner, scopeOfOuter );
```

然后，在`inner()`内部，当它引用词法变量`x`时，它实际上更像是:

```js
return scopeOfInner.x;
```

`scopeOfInner`没有`x`属性，但它是[[Prototype]]` -链接到`scopeOfOuter`，它确实有`x`属性。通过原型委托访问`scopeOfOuter.x`将返回“1”值。

通过这种方式，我们可以看到为什么`outer()` 的作用域(通过闭包)在完成之后仍然保留:因为`scopeOfInner`对象与`scopeOfOuter`对象相链接，从而保持该对象及其属性的活动和状态。

这些都是概念性的。我并不是说JS引擎使用对象和原型。但它完全有可能以类似的方式工作。

实际上，许多语言都通过对象实现闭包。其他语言使用闭包实现对象。但我们会让读者发挥他们的想象力来理解它是如何运作的。

## 两个观点

闭包和对象是等价的?不完全是。我打赌它们比您在开始本章之前所认为的更相似，但是它们仍然有重要的区别。

这些差异不应被视为缺点或反对使用的理由;这是错误的观点。它们应该被看作是使其中一个或另一个更适合(和可读!)用于给定任务的特性和优势。

### 结构可变性

从概念上讲，闭包的结构不是可变的。

换句话说，您永远不能向闭包添加或删除状态。闭包是声明变量的一个特性(在作者/编译时固定)，并且不敏感于任何运行时条件——当然，假设您使用严格模式和/或避免使用`eval(..)`之类的欺骗自己不会出错的方式!

注: JS引擎可以在技术上剔除一个闭包，以剔除其范围内不再使用的任何变量，但这是一个高级优化，对开发人员来说是透明的。无论引擎实际上是否执行这些优化，我认为对于开发人员来说，假设闭包是针对范围而不是针对变量的是最安全的。如果你不想让它作用域影响，就不要对他修改!

然而，对象在默认情况下是相当可变的。您可以自由地从对象中添加或删除(`delete`)属性/索引，只要该对象没有被冻结(`Object.freeze(..)`)。

根据程序中的运行时条件，可以跟踪更多(或更少)的状态，这可能是代码的一个优势。

例如，让我们想象一下在游戏中跟踪按键事件。几乎可以肯定的是，您将考虑使用数组来完成以下操作:

```js
function trackEvent(evt,keypresses = []) {
    return [ ...keypresses, evt ];
}

var keypresses = trackEvent( newEvent1 );

keypresses = trackEvent( newEvent2, keypresses );
```

注:你注意到为什么我没有直接将`push(..)`推到`keypresses`中了吗?因为在FP中，我们通常希望将数组视为不可变的数据结构，可以重新创建并添加到其中，但不能直接更改。我们用副作用的影响来换取明确的重新分配(稍后会详细介绍)。

虽然我们没有改变数组的结构，但如果我们愿意，我们可以。稍后会详细介绍。

但是数组并不是跟踪不断增长的`evt`对象“列表”的唯一方法。我们可以使用闭包:

```js
function trackEvent(evt,keypresses = () => []) {
    return function newKeypresses() {
        return [ ...keypresses(), evt ];
    };
}

var keypresses = trackEvent( newEvent1 );

keypresses = trackEvent( newEvent2, keypresses );
```

Do you spot what's happening here?

Each time we add a new event to the "list", we create a new closure wrapped around the existing `keypresses()` function (closure), which captures the current `evt`. When we call the `keypresses()` function, it will successively call all the nested functions, building up an intermediate array of all the individually closed-over `evt` objects. Again, closure is the mechanism that's tracking all the state; the array you see is only an implementation detail of needing a way to return multiple values from a function.

So which one is better suited for our task? No surprise here, the array approach is probably a lot more appropriate. The structural immutability of a closure means our only option is to wrap more closure around it. Objects are by default extensible, so we can just grow the array as needed.

By the way, even though I'm presenting this structural (im)mutability as a clear difference between closure and object, the way we're using the object as an immutable value is actually more similar than not.

Creating a new array for each addition to the array is treating the array as structurally immutable, which is conceptually symmetrical to closure being structurally immutable by its very design.

### Privacy

Probably one of the first differences you think of when analyzing closure vs. object is that closure offers "privacy" of state through nested lexical scoping, whereas objects expose everything as public properties. Such privacy has a fancy name: information hiding.

Consider lexical closure hiding:

```js
function outer() {
    var x = 1;

    return function inner(){
        return x;
    };
}

var xHidden = outer();

xHidden();          // 1
```

Now the same state in public:

```js
var xPublic = {
    x: 1
};

xPublic.x;          // 1
```

There are some obvious differences around general software engineering principles -- consider abstraction, the module pattern with public and private APIs, etc. -- but let's try to constrain our discussion to the perspective of FP; this is, after all, a book about functional programming!

#### Visibility

It may seem that the ability to hide information is a desired characteristic of state tracking, but I believe the FPer might argue the opposite.

One of the advantages of managing state as public properties on an object is that it's easier to enumerate (and iterate!) all the data in your state. Imagine you wanted to process each keypress event (from the earlier example) to save it to a database, using a utility like:

```js
function recordKeypress(keypressEvt) {
    // database utility
    DB.store( "keypress-events", keypressEvt );
}
```

If you already have an array -- just an object with public numerically named properties -- this is very straightforward using a built-in JS array utility `forEach(..)`:

```js
keypresses.forEach( recordKeypress );
```

But if the list of keypresses is hidden inside closure, you'll have to expose a utility on the public API of the closure with privileged access to the hidden data.

For example, we can give our closure-`keypresses` example its own `forEach`, like built-in arrays have:

```js
function trackEvent(
    evt,
    keypresses = {
        list() { return []; },
        forEach() {}
    }
) {
    return {
        list() {
            return [ ...keypresses.list(), evt ];
        },
        forEach(fn) {
            keypresses.forEach( fn );
            fn( evt );
        }
    };
}

// ..

keypresses.list();      // [ evt, evt, .. ]

keypresses.forEach( recordKeypress );
```

The visibility of an object's state data makes using it more straightforward, whereas closure obscures the state making us work harder to process it.

#### Change Control

If the lexical variable `x` is hidden inside a closure, the only code that has the freedom to reassign it is also inside that closure; it's impossible to modify `x` from the outside.

As we saw in [Chapter 6](ch6.md), that fact alone improves the readability of code by reducing the surface area that the reader must consider to predict the behavior of any given variable.

The local proximity of lexical reassignment is a big reason why I don't find `const` as a feature that helpful. Scopes (and thus closures) should in general be pretty small, and that means there will only be a few lines of code that can affect reassignment. In `outer()` above, we can quickly inspect to see that no line of code reassigns `x`, so for all intents and purposes it's acting as a constant.

This kind of guarantee is a powerful contributor to our confidence in the purity of a function, for example.

On the other hand, `xPublic.x` is a public property, and any part of the program that gets a reference to `xPublic` has the ability, by default, to reassign `xPublic.x` to some other value. That's a lot more lines of code to consider!

That's why in [Chapter 6, we looked at `Object.freeze(..)`](ch6.md/#its-freezing-in-here) as a quick-n-dirty means of making all of an object's properties read-only (`writable: false`), so that they can't be reassigned unpredictably.

Unfortunately, `Object.freeze(..)` is both all-or-nothing and irreversible.

With closure, you have some code with the privilege to change, and the rest of the program is restricted. When you freeze an object, no part of the code will be able to reassign. Moreover, once an object is frozen, it can't be thawed out, so the properties will remain read-only for the duration of the program.

In places where I want to allow reassignment but restrict its surface area, closures are a more convenient and flexible form than objects. In places where I want no reassignment, a frozen object is a lot more convenient than repeating `const` declarations all over my function.

Many FPers take a hard-line stance on reassignment: it shouldn't be used. They will tend to use `const` to make all closure variables read-only, and they'll use `Object.freeze(..)` or full immutable data structures to prevent property reassignment. Moreover, they'll try to reduce the amount of explicitly declared/tracked variables and properties wherever possible, preferring value transfer -- function chains, `return` value passed as argument, etc. -- instead of intermediate value storage.

This book is about "Functional-Light" programming in JavaScript, and this is one of those cases where I diverge from the core FP crowd.

I think variable reassignment can be quite useful, and when used appropriately, quite readable in its explicitness. It's certainly been my experience that debugging is a lot easier when you can insert a `debugger` or breakpoint, or track a watch expression.

### Cloning State

As we learned in [Chapter 6](ch6.md), one of the best ways we prevent side effects from eroding the predictability of our code is to make sure we treat all state values as immutable, regardless of whether they are actually immutable (frozen) or not.

If you're not using a purpose-built library to provide sophisticated immutable data structures, the simplest approach will suffice: duplicate your objects/arrays each time before making a change.

Arrays are easy to clone shallowly -- just use `...` array spread:

```js
var a = [ 1, 2, 3 ];

var b = [...a];
b.push( 4 );

a;          // [1,2,3]
b;          // [1,2,3,4]
```

Objects can be shallow-cloned relatively easily too:

```js
var o = {
    x: 1,
    y: 2
};

// in ES2018+, using object spread:
var p = { ...o };
p.y = 3;

// in ES6/ES2015+:
var p = Object.assign( {}, o );
p.y = 3;
```

If the values in an object/array are themselves non-primitives (objects/arrays), to get deep cloning you'll have to walk each layer manually to clone each nested object. Otherwise, you'll have copies of shared references to those sub-objects, and that's likely to create havoc in your program logic.

Did you notice that this cloning is possible only because all these state values are visible and can thus be easily copied? What about a set of state wrapped up in a closure; how would you clone that state?

That's much more tedious. Essentially, you'd have to do something similar to our custom `forEach` API method earlier: provide a function inside each layer of the closure with the privilege to extract/copy the hidden values, creating new equivalent closures along the way.

Even though that's theoretically possible -- another exercise for the reader! -- it's far less practical to implement than you're likely to justify for any real program.

Objects have a clear advantage when it comes to representing state that we need to be able to clone.

### Performance

One reason objects may be favored over closures, from an implementation perspective, is that in JavaScript objects are often lighter-weight in terms of memory and even computation.

But be careful with that as a general assertion: there are plenty of things you can do with objects that will erase any performance gains you may get from ignoring closure and moving to object-based state tracking.

Let's consider a scenario with both implementations. First, the closure-style implementation:

```js
function StudentRecord(name,major,gpa) {
    return function printStudent(){
        return `${name}, Major: ${major}, GPA: ${gpa.toFixed(1)}`;
    };
}

var student = StudentRecord( "Kyle Simpson", "CS", 4 );

// later

student();
// Kyle Simpson, Major: CS, GPA: 4.0
```

The inner function `printStudent()` closes over three variables: `name`, `major`, and `gpa`. It maintains this state wherever we transfer a reference to that function -- we call it `student()` in this example.

Now for the object (and `this`) approach:

```js
function StudentRecord(){
    return `${this.name}, Major: ${this.major}, \
GPA: ${this.gpa.toFixed(1)}`;
}

var student = StudentRecord.bind( {
    name: "Kyle Simpson",
    major: "CS",
    gpa: 4
} );

// later

student();
// Kyle Simpson, Major: CS, GPA: 4.0
```

The `student()` function -- technically referred to as a "bound function" -- has a hard-bound `this` reference to the object literal we passed in, such that any later call to `student()` will use that object for its `this`, and thus be able to access its encapsulated state.

Both implementations have the same outcome: a function with preserved state. But what about the performance; what differences will there be?

**Note:** Accurately and actionably judging performance of a snippet of JS code is a very dodgy affair. We won't get into all the details here, but I urge you to read *You Don't Know JS: Async & Performance*, specifically Chapter 6, "Benchmarking & Tuning", for more details.

If you were writing a library that created a pairing of state with its function -- either the call to `StudentRecord(..)` in the first snippet or the call to `StudentRecord.bind(..)` in the second snippet -- you're likely to care most about how those two perform. Inspecting the code, we can see that the former has to create a new function expression each time. The second one uses `bind(..)`, which is not as obvious in its implications.

One way to think about what `bind(..)` does under the covers is that it creates a closure over a function, like this:

```js
function bind(orinFn,thisObj) {
    return function boundFn(...args) {
        return origFn.apply( thisObj, args );
    };
}

var student = bind( StudentRecord, { name: "Kyle.." } );
```

In this way, it looks like both implementations of our scenario create a closure, so the performance is likely to be about the same.

However, the built-in `bind(..)` utility doesn't really have to create a closure to accomplish the task. It simply creates a function and manually sets its internal `this` to the specified object. That's potentially a more efficient operation than if we did the closure ourselves.

The kind of performance savings we're talking about here is miniscule on an individual operation. But if your library's critical path is doing this hundreds or thousands of times or more, that savings can add up quickly. Many libraries -- Bluebird being one such example -- have ended up optimizing by removing closures and going with objects, in exactly this means.

Outside of the library use-case, the pairing of the state with its function usually only happens relatively few times in the critical path of an application. By contrast, typically the usage of the function+state -- calling `student()` in either snippet -- is more common.

If that's the case for some given situation in your code, you should probably care more about the performance of the latter versus the former.

Bound functions have historically had pretty lousy performance in general, but have recently been much more highly optimized by JS engines. If you benchmarked these variations a couple of years ago, it's entirely possible you'd get different results repeating the same test with the latest engines.

A bound function is now likely to perform at least as good if not better as the equivalent closed-over function. So that's another tick in favor of objects over closures.

I just want to reiterate: these performance observations are not absolutes, and the determination of what's best for a given scenario is very complex. Do not just casually apply what you've heard from others or even what you've seen on some other earlier project. Carefully examine whether objects or closures are appropriately efficient for the task.

## Summary

The truth of this chapter cannot be written out. One must read this chapter to find its truth.

----

Coining some Zen wisdom here was my attempt at being clever. But you deserve a proper summary of this chapter's message.

Objects and closures are isomorphic to each other, which means that they can be used somewhat interchangeably to represent state and behavior in your program.

Representation as a closure has certain benefits, like granular change control and automatic privacy. Representation as an object has other benefits, like easier cloning of state.

The critically thinking FPer should be able to conceive any segment of state and behavior in the program with either representation, and pick the representation that's most appropriate for the task at hand.
