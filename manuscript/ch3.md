# 章节3：管理函数的输入

[第2章](ch2.md)探讨了JS函数的核心性质，并为函数以及“FP函数”奠定了基础。但是，为了充分利用函数编程的力量，我们还需要模式和实践来操纵函数来改变和调整它们的交互——使它们按我们的意愿走。

具体来说，我们这一章将集中在函数的参数输入上。当您将所有不同形状的函数放在一起时，您将很快面临输入的数量/顺序/类型的不兼容性，以及需要在不同的时间指定某些输入

事实上，出于可读性的风格考虑，有时您需要以完全隐藏其输入的方式定义函数!

这些技术对于使函数真正的*function*是绝对必要的。

## 多对一

假设您将一个函数传递给一个实用程序，实用程序将向该函数发送多个参数。但是您可能只希望函数接收单个参数。

我们可以设计一个简单的助手来包装一个函数调用，以确保只传递一个参数。由于这有效地强制将函数视为一元函数，我们将其命名为:

<a name="unary"></a>

```js
function unary(fn) {
    return function onlyOneArg(arg){
        return fn( arg );
    };
}
```
对于这样的代码，许多函数编程使用者倾向于使用更短的' => '箭头函数语法(参见[第2章，“没有function定义的函数”](ch2.md/#functions-without-function))，例如:

```js
var unary =
    fn =>
        arg =>
            fn( arg );
```

**注意:**毫无疑问，这更简洁，甚至稀疏。但我个人觉得，无论它在数学符号的对称性上获得了什么，它在整体可读性上损失得更多，因为所有函数都是匿名的，而且模糊了范围边界，使得解密闭包变得更加神秘。

使用“unary(..)”的一个常见例子是使用“map(..)”实用程序(参见[章节 9，“map函数”](ch9.md/#map))和“parseInt(..)”。map(..)为列表中的每一项调用一个mapper函数，每次调用mapper函数时，都会传入三个参数:“value”、“idx”、“arr”。

这通常没什么大不了的，除非您试图使用某个东西作为映射函数，如果它传递了太多参数，那么它的行为就会不正确。
想一想:

```js
["1","2","3"].map( parseInt );
// [1,NaN,NaN]
```
对于“parseInt(str,radix)”，很明显，当“map(..)”在第二个参数位置传递“index”时，它被“parseInt(..)”解释为进制“基数”，这是我们不想要的。

' unary(..) '创建一个函数，该函数将忽略传递给它的除第一个参数外的所有参数，这意味着传入的' index '不会被' parseInt(..) '接收，并被误认为'基数':

<a name="mapunary"></a>

```js
["1","2","3"].map( unary( parseInt ) );
// [1,2,3]
```

### 一对一

说到只有一个参数的函数，函数式编程工具库中的另一个公共基础实用程序是一个函数，它接受一个参数，只返回未触及的值：

```js
function identity(v) {
    return v;
}

// 或者使用ES6箭头函数
var identity =
    v =>
        v;
```

这个实用程序看起来非常简单，几乎没有什么用处。但是，即使是简单的函数，在函数式编程的世界中也是有用的。就像他们在表演中说的:没有小角色，只有小演员。

例如，假设您想使用正则表达式拆分一个字符串，但是结果数组中可能有一些空值。为了抛弃这些，我们可以使用JS的'filter(..) '数组操作(参见[章节 9, "filter函数"](ch9.md/#filter))和' identity(..) '作为参数回调:

```js
var words = "   Now is the time for all...  ".split( /\s|\b/ );
words;
// ["","Now","is","the","time","for","all","...",""]

words.filter( identity );
// ["Now","is","the","time","for","all","..."]
```
因为' identity(..) '只返回传递给它的值，JS强制每个值要么为' true '要么为' false '，这决定了是否保留或排除最后数组中的每个值。

提示：在前面的例子中，可以用作谓词的另一个一元函数是JS内置的“Boolean(..)”函数，它显式地强制一个值为“true”或“false”。

使用' identity(..) '的另一个例子是作为默认函数代替转换:

```js
function output(msg,formatFn = identity) {
    msg = formatFn( msg );
    console.log( msg );
}

function upper(txt) {
    return txt.toUpperCase();
}

output( "Hello World", upper );     // HELLO WORLD
output( "Hello World" );            // Hello World
```
您还可以看到' identity(..) '用作' map(..) '调用的默认转换函数，或作为函数列表的' reduce(..) '中的初始值;这两个实用程序都将在[第9章](ch9.md)中讨论。

### 不可更改的

某些API不允许将值直接传递给方法，但要求传递函数，即使该函数实际上只是返回值。JS上的“then(..)”方法就是这样一个API：

```js
// 不起作用:
p1.then( foo ).then( p2 ).then( bar );

// 替换成:
p1.then( foo ).then( function(){ return p2; } ).then( bar );
```

许多人声称ES6`=>`箭头函数是最好的“解决方案”：

```js
p1.then( foo ).then( () => p2 ).then( bar );
```

但是有一个fp实用程序更适合此任务：
```js
function constant(v) {
    return function value(){
        return v;
    };
}

// or the ES6 => form
var constant =
    v =>
        () =>
            v;
```

有了这个整洁的小fp工具，我们可以正确地解决“then（…）”的烦恼：

```js
p1.then( foo ).then( constant( p2 ) ).then( bar );
```

**警告：**虽然`() =>p2`箭头函数版本比`constant(p2)`简短，但我建议你不要这么使用。arrow函数从自身外部返回一个值，从fp的角度来看，这有点糟糕。我们稍后将在本书中讨论这些行为的陷阱（见[第5章](ch5.md)）。

## 使参数适应参数

我们可以使用多种模式和技巧来调整函数的签名，以匹配我们希望为其提供的参数类型。

回想一下[第2章中的这个函数签名](ch2.md/user content funcparamdestr)，它强调使用数组参数解构：

```js
function foo( [x,y,...args] = [] ) {
```

如果传入数组，但希望将其内容视为单个参数，则此模式很方便。`因此，foo(..)`在技术上是一元的——当它被执行时，只有一个参数（数组）将被传递给它。但是在函数内部，您可以分别处理不同的输入（`x`，`y`，等等）。

但是，有时您无法将函数的声明更改为使用数组参数析构化。例如，假设这些函数：

```js
function foo(x,y) {
    console.log( x + y );
}

function bar(fn) {
    fn( [ 3, 9 ] );
}

bar( foo );         // 失败
```
你看出`bar(foo)`失败的原因了吗？

数组`[3,9]`作为单个值发送到'fn(..)`，但'foo(..)`需要分别为'x'和'y'。如果我们能把“foo(..)”的声明改为“function foo([x，y])”，我们就可以了。或者，如果我们可以更改“bar(…)”的行为，使调用成为“fn(…[3,9])”，则值“3”和“9”将分别传入。

有时，当您有两个函数以这种方式不兼容时，您将无法更改它们的声明/定义。那么，你怎么能同时使用它们呢？

我们可以定义一个中间函数来调整一个函数，这样它就会展开一个单独的接收数组作为它的参数:

<a name="spreadargs"></a>

```js
function spreadArgs(fn) {
    return function spreadFn(argsArr){
        return fn( ...argsArr );
    };
}

// 或者使用ES6箭头函数
var spreadArgs =
    fn =>
        argsArr =>
            fn( ...argsArr );
```

**注意:**我将这个中间函数称为“spreadArgs(..)”，但是在像Ramda这样的库中，它通常被称为“apply(..)”。

现在我们可以使用“spreadArgs(..)”来调整“foo(..)”作为“bar(..)”的正确输入:

```js
bar( spreadArgs( foo ) );           // 12
```

现在还不清楚为什么会出现这种情况，但你会经常看到。实际上，“spreadArgs(..)”允许我们定义一些函数，这些函数通过数组“返回”多个值，但仍然将这些值作为另一个函数的输入单独处理。

当我们讨论一个“spreadArgs(..)”工具时，让我们也定义一个工具来处理相反的操作:

```js
function gatherArgs(fn) {
    return function gatheredFn(...argsArr){
        return fn( argsArr );
    };
}

// 或者使用ES6箭头函数
var gatherArgs =
    fn =>
        (...argsArr) =>
            fn( argsArr );
```

**注意:**在Ramda中，这个实用程序被称为“unapply(..)”，因为它与“apply(..)”相反。我认为用“扩展”/“收集”的术语更能描述这个。

我们可以使用这个实用程序将单个参数收集到一个数组中，这可能是因为我们希望将一个具有数组参数析构的函数调整为另一个单独传递参数的实用程序。我们将[在第9章更全面地讨论' reduce(..) '](ch9.md/#reduce);简而言之，它用两个单独的参数反复调用它的减速函数，我们现在可以将它们*集合*在一起:

```js
function combineFirstTwo([ v1, v2 ]) {
    return v1 + v2;
}

[1,2,3,4,5].reduce( gatherArgs( combineFirstTwo ) );
// 15
```

## 现在与稍后的调用

如果一个函数接受多个参数，您可能希望预先指定其中一些参数，其余的参数留待以后指定。

思考下面这个函数:

```js
function ajax(url,data,callback) {
    // ..
}
```

假设您想设置几个API调用，其中url在前面是已知的，但是处理响应的数据和回调要到稍后才会知道。

当然，您可以推迟执行' ajax(..) '调用，直到所有的位都被知道，并在那个时候引用URL的某个全局常量。但另一种方法是创建一个函数引用，它已经预先设置了“url”参数。

我们要做的是创建一个新函数，它仍然调用' ajax(..) '，并手动将第一个参数设置API URL，同时等待稍后接受其他两个参数:

```js
function getPerson(data,cb) {
    ajax( "http://some.api/person", data, cb );
}

function getOrder(data,cb) {
    ajax( "http://some.api/order", data, cb );
}
```

手动指定这些函数的封装当然是可能的，但它可能会变得相当乏味，特别是如果还会有不同的参数预置变化，如:

```js
function getCurrentUser(cb) {
    getPerson( { user: CURRENT_USER_ID }, cb );
}
```

函数编程人员非常习惯的一种实践是寻找重复执行相同类型的操作的模式，并尝试将这些操作转换为通用的可重用实用程序。事实上，我相信这已经是你们许多读者的本能，所以这并不是函数式编程所独有的。但毫无疑问，这对函数式编程很重要。

要构思这样一个用于参数预置的实用程序，让我们从概念上研究一下发生了什么，而不只是查看这里显示的手动实现。

一种明确的方法是“getOrder(data,cb)”函数是“ajax(url,data,cb)”函数的“部分应用程序”。这个术语来自于这样一个概念:参数被“应用”到函数调用站点的参数上。正如您所看到的，我们只在前面应用了一些参数——具体地说，就是“url”参数的参数——其余的参数将在稍后应用。

更正式地说，部分应用严格地减少了函数的特性;记住，这是期望参数输入的个数。对于“getOrder(..)”函数，我们将原来的“ajax(..)”函数的特性从3减少到2。

定义一个`partial(..)`工具函数：

```js
function partial(fn,...presetArgs) {
    return function partiallyApplied(...laterArgs){
        return fn( ...presetArgs, ...laterArgs );
    };
}

// 或使用ES6箭头函数
var partial =
    (fn,...presetArgs) =>
        (...laterArgs) =>
            fn( ...presetArgs, ...laterArgs );
```

**提示:**不要只看这个片段的表面价值。暂停几分钟，消化一下这个实用程序到底发生了什么。确保你真的明白了。

' partial(..) '函数接受一个' fn '函数，我们对该函数进行了部分应用。然后，传入的任何后续参数都被收集到“presetArgs”数组中，并保存到后面。

创建一个新的内部函数(称为' partiallyApplied(..) '，只是为了清晰起见)并返回;内部函数自身的参数被收集到一个名为“laterArgs”的数组中。

注意到内部函数中对' fn '和' presetArgs '的引用了吗?这是怎么回事?在' partial(..) '运行完成后，内部函数如何能够继续访问' fn '和' presetArgs ' ?如果您回答的是**闭包**，那么你是正确的!内部函数“partiallyApplied(..)”对“fn”和“presetArgs”变量都关闭，无论函数运行在何处，以后都可以继续访问它们，。这就是为什么理解闭包是至关重要的!

当“partiallyapplied(..)”函数在程序中的其他地方执行时，它使用关闭的“fn”来执行原始函数，首先提供任何“presetargs”部分应用程序参数，然后再提供其他“laterargs”参数。

如果有什么让人困惑的话，停下来再读一遍。相信我，你会很高兴我们能更深入地了解。

现在，让我们使用' partial(..) '函数来实现前面那些部分应用的函数:

```js
var getPerson = partial( ajax, "http://some.api/person" );

var getOrder = partial( ajax, "http://some.api/order" );
```

花点时间考虑一下“getPerson(..)”的形状/内部结构。它看起来是这样的:

```js
var getPerson = function partiallyApplied(...laterArgs) {
    return ajax( "http://some.api/person", ...laterArgs );
};
```

getOrder(..)也是如此。但是' getCurrentUser(..) '呢?

```js
// 版本 1
var getCurrentUser = partial(
    ajax,
    "http://some.api/person",
    { user: CURRENT_USER_ID }
);

// 版本 2
var getCurrentUser = partial( getPerson, { user: CURRENT_USER_ID } );
```

我们可以使用直接指定的“url”和“data”参数定义“getCurrentUser(..)”(版本1)，或者将“getCurrentUser(..)”定义为“getPerson(..)”部分应用程序的部分应用程序，只指定附加的“data”参数(版本2)。

版本2更便于表达，因为它重用了已经定义的内容。因此，我认为它更符合函数式编程的概念。

为了确保我们理解这两个版本在背后是如何工作的，它们看起来分别有点像:

```js
// 版本 1
var getCurrentUser = function partiallyApplied(...laterArgs) {
    return ajax(
        "http://some.api/person",
        { user: CURRENT_USER_ID },
        ...laterArgs
    );
};

// 版本 2
var getCurrentUser = function outerPartiallyApplied(...outerLaterArgs){
    var getPerson = function innerPartiallyApplied(...innerLaterArgs){
        return ajax( "http://some.api/person", ...innerLaterArgs );
    };

    return getPerson( { user: CURRENT_USER_ID }, ...outerLaterArgs );
}
```

同样，停止并重新阅读这些代码片段，以确保您理解其中的内容。

**注意:** 版本 2包含一个额外的函数包装层。这可能听起来很奇怪，也没有必要，但这只是函数编程中你想要真正熟悉的东西之一。在阅读文本的过程中，我们将把许多层函数相互包装起来。记住，这是*函数*编程!

让我们看一下部分应用程序的另一个有用的例子。考虑一个' add(..) '函数，它接受两个参数并将它们相加:

```js
function add(x,y) {
    return x + y;
}
```

现在假设我们想要一个数字列表，并在每个列表中添加一个特定的数字。我们将使用内置在JS数组中的 `map(..)`(see [Chapter 9, "Map"](ch9.md/#map)) ：

```js
[1,2,3,4,5].map( function adder(val){
    return add( 3, val );
} );
// [4,5,6,7,8]
```

我们不能直接将' add(..) '传递给' map(..) '的原因是' add(..) '的签名与' map(..) '的映射函数不匹配。这就是部分应用程序可以帮助我们的地方:我们可以修改`add(..)`的签名，使之匹配:

```js
[1,2,3,4,5].map( partial( add, 3 ) );
// [4,5,6,7,8]
```

`partial(add,3)`调用生成一个新的一元函数，该函数只需要一个参数。

`map(..)`将循环遍历数组(`[1,2,3,4,5]`)，并分别为每个值重复调用这个一元函数一次。因此,有效地调用`add(3,1)`, `add(3,2)`, `add(3,3)`, `add(3,4)`, and `add(3,5)`，产生结果的数组是'[4,5,6,7,8]'

### `bind(..)函数`

javascript函数都有一个名为`bind(..)`的内置函数。它有两个功能：预设`this`上下文和应用参数。

我认为把这两种功能混为一谈是非常错误的。有时您需要绑定`this`上下文，而不是应用参数。其他时候，您可能希望作为应用参数，根本不关心`this`绑定。我是从来没有同时需要这两者的。

后一种方案比较尴尬（不设置`this`上下文的应用函数），因为必须为 `this`绑定参数（第一个）传递一个可忽略的占位符，通常为“null”。

想一想:

```js
var getPerson = ajax.bind( null, "http://some.api/person" );
```

那个`null` 让人心烦。JS为部分应用程序提供了一个内置的实用程序，这还是比较方便的。然而，大多数函数编程程序员更喜欢在他们选择的函数编程库中使用专用的`partial(..)`实用程序。

### 反转参数

回想一下，我们的Ajax函数的签名是:`ajax( url, data, cb )`。如果我们想要部分应用 `cb`，但要等到稍后指定`data`和`url`，又该怎么办?我们可以创建一个实用程序来包装一个函数，以逆转其参数顺序:

```js
function reverseArgs(fn) {
    return function argsReversed(...args){
        return fn( ...args.reverse() );
    };
}

// 运用箭头函数
var reverseArgs =
    fn =>
        (...args) =>
            fn( ...args.reverse() );
```

现在我们可以颠倒`ajax(..)`参数的顺序，这样我们就可以从右边而不是左边部分地应用。为了恢复预期的顺序，我们将反转后面部分应用的函数:

```js
var cache = {};

var cacheResult = reverseArgs(
    partial( reverseArgs( ajax ), function onResult(obj){
        cache[obj.id] = obj;
    } )
);

// later:
cacheResult( "http://some.api/person", { user: CURRENT_USER_ID } );
```

Instead of manually using `reverseArgs(..)` (twice!) for this purpose, we can define a `partialRight(..)` which partially applies the rightmost arguments. Under the covers, it can use the same double-reverse trick:

<a name="partialright"></a>

```js
function partialRight(fn,...presetArgs) {
    return reverseArgs(
        partial( reverseArgs( fn ), ...presetArgs.reverse() )
    );
}

var cacheResult = partialRight( ajax, function onResult(obj){
    cache[obj.id] = obj;
});

// later:
cacheResult( "http://some.api/person", { user: CURRENT_USER_ID } );
```

Another more straightforward (and certainly more performant) implementation of `partialRight(..)` that doesn't use the double-reverse trick:

```js
function partialRight(fn,...presetArgs) {
    return function partiallyApplied(...laterArgs){
        return fn( ...laterArgs, ...presetArgs );
    };
}

// or the ES6 => arrow form
var partialRight =
    (fn,...presetArgs) =>
        (...laterArgs) =>
            fn( ...laterArgs, ...presetArgs );
```

None of these implementations of `partialRight(..)` guarantee that a specific parameter will receive a specific partially applied value; it only ensures that the partially applied value(s) appear as the rightmost (aka, last) argument(s) passed to the original function.

For example:

```js
function foo(x,y,z,...rest) {
    console.log( x, y, z, rest );
}

var f = partialRight( foo, "z:last" );

f( 1, 2 );          // 1 2 "z:last" []

f( 1 );             // 1 "z:last" undefined []

f( 1, 2, 3 );       // 1 2 3 ["z:last"]

f( 1, 2, 3, 4 );    // 1 2 3 [4,"z:last"]
```

The value `"z:last"` is only applied to the `z` parameter in the case where `f(..)` is called with exactly two arguments (matching `x` and `y` parameters). In all other cases, the `"z:last"` will just be the rightmost argument, however many arguments precede it.

## One at a Time

Let's examine a technique similar to partial application, where a function that expects multiple arguments is broken down into successive chained functions that each take a single argument (arity: 1) and return another function to accept the next argument.

This technique is called currying.

To first illustrate, let's imagine we had a curried version of `ajax(..)` already created. This is how we'd use it:

```js
curriedAjax( "http://some.api/person" )
    ( { user: CURRENT_USER_ID } )
        ( function foundUser(user){ /* .. */ } );
```

The three sets of `(..)`s denote three chained function calls. But perhaps splitting out each of the three calls helps see what's going on better:

```js
var personFetcher = curriedAjax( "http://some.api/person" );

var getCurrentUser = personFetcher( { user: CURRENT_USER_ID } );

getCurrentUser( function foundUser(user){ /* .. */ } );
```

Instead of taking all the arguments at once (like `ajax(..)`), or some of the arguments up front and the rest later (via `partial(..)`), this `curriedAjax(..)` function receives one argument at a time, each in a separate function call.

Currying is similar to partial application in that each successive curried call partially applies another argument to the original function, until all arguments have been passed.

The main difference is that `curriedAjax(..)` will return a function (we call it `personFetcher(..)`) that expects **only the next argument** `data`, not one that (like the earlier `getPerson(..)`) can receive all the rest of the arguments.

If an original function expected five arguments, the curried form of that function would take just the first argument, and return a function to accept the second. That one would take just the second argument, and return a function to accept the third. And so on.

So currying unwinds a single higher-arity function into a series of chained unary functions.

How might we define a utility to do this currying? Consider:

<a name="curry"></a>

```js
function curry(fn,arity = fn.length) {
    return (function nextCurried(prevArgs){
        return function curried(nextArg){
            var args = [ ...prevArgs, nextArg ];

            if (args.length >= arity) {
                return fn( ...args );
            }
            else {
                return nextCurried( args );
            }
        };
    })( [] );
}

// or the ES6 => arrow form
var curry =
    (fn,arity = fn.length,nextCurried) =>
        (nextCurried = prevArgs =>
            nextArg => {
                var args = [ ...prevArgs, nextArg ];

                if (args.length >= arity) {
                    return fn( ...args );
                }
                else {
                    return nextCurried( args );
                }
            }
        )( [] );
```

The approach here is to start a collection of arguments in `prevArgs` as an empty `[]` array, and add each received `nextArg` to that, calling the concatenation `args`. While `args.length` is less than `arity` (the number of declared/expected parameters of the original `fn(..)` function), make and return another `curried(..)` function to collect the next `nextArg` argument, passing the running `args` collection along as its `prevArgs`. Once we have enough `args`, execute the original `fn(..)` function with them.

By default, this implementation relies on being able to inspect the `length` property of the to-be-curried function to know how many iterations of currying we'll need before we've collected all its expected arguments.

**Note:** If you use this implementation of `curry(..)` with a function that doesn't have an accurate `length` property, you'll need to pass the `arity` (the second parameter of `curry(..)`) to ensure `curry(..)` works correctly. `length` will be inaccurate if the function's parameter signature includes default parameter values, parameter destructuring, or is variadic with `...args` (see [Chapter 2](ch2.md)).

Here's how we would use `curry(..)` for our earlier `ajax(..)` example:

```js
var curriedAjax = curry( ajax );

var personFetcher = curriedAjax( "http://some.api/person" );

var getCurrentUser = personFetcher( { user: CURRENT_USER_ID } );

getCurrentUser( function foundUser(user){ /* .. */ } );
```

Each call partially applies one more argument to the original `ajax(..)` call, until all three have been provided and `ajax(..)` is actually invoked.

Remember our example from the discussion of partial application about adding `3` to each value in a list of numbers? As currying is similar to partial application, we could do that task with currying in almost the same way:

```js
[1,2,3,4,5].map( curry( add )( 3 ) );
// [4,5,6,7,8]
```

The difference between the two? `partial(add,3)` vs `curry(add)(3)`.

Why might you choose `curry(..)` over `partial(..)`? It might be helpful in the case where you know ahead of time that `add(..)` is the function to be adapted, but the value `3` isn't known yet:

```js
var adder = curry( add );

// later
[1,2,3,4,5].map( adder( 3 ) );
// [4,5,6,7,8]
```

Let's look at another numbers example, this time adding a list of them together:

```js
function sum(...nums) {
    var total = 0;
    for (let num of nums) {
        total += num;
    }
    return total;
}

sum( 1, 2, 3, 4, 5 );                       // 15

// now with currying:
// (5 to indicate how many we should wait for)
var curriedSum = curry( sum, 5 );

curriedSum( 1 )( 2 )( 3 )( 4 )( 5 );        // 15
```

The advantage of currying here is that each call to pass in an argument produces another function that's more specialized, and we can capture and use *that* new function later in the program. Partial application specifies all the partially applied arguments up front, producing a function that's waiting for all the rest of the arguments **on the next call**.

If you wanted to use partial application to specify one parameter (or several!) at a time, you'd have to keep calling `partial(..)` again on each successive partially applied function. By contrast, curried functions do this automatically, making working with individual arguments one-at-a-time more ergonomic.

Both currying and partial application use closure to remember the arguments over time until all have been received, and then the original function can be invoked.

### Visualizing Curried Functions

Let's examine more closely the `curriedSum(..)` from the previous section. Recall its usage: `curriedSum(1)(2)(3)(4)(5)`; five subsequent (chained) function calls.

What if we manually defined a `curriedSum(..)` instead of using `curry(..)`? How would that look?

```js
function curriedSum(v1) {
    return function(v2){
        return function(v3){
            return function(v4){
                return function(v5){
                    return sum( v1, v2, v3, v4, v5 );
                };
            };
        };
    };
}
```

Definitely uglier, no question. But this is an important way to visualize what's going on with a curried function. Each nested function call is returning another function that's going to accept the next argument, and that continues until we've specified all the expected arguments.

When trying to decipher curried functions, I've found it helps me tremendously if I can unwrap them mentally as a series of nested functions.

In fact, to reinforce that point, let's consider the same code but written with ES6 arrow functions:

```js
curriedSum =
    v1 =>
        v2 =>
            v3 =>
                v4 =>
                    v5 =>
                        sum( v1, v2, v3, v4, v5 );
```

And now, all on one line:

```js
curriedSum = v1 => v2 => v3 => v4 => v5 => sum( v1, v2, v3, v4, v5 );
```

Depending on your perspective, that form of visualizing the curried function may be more or less helpful to you. For me, it's a fair bit more obscured.

But the reason I show it that way is that it happens to look almost identical to the mathematical notation (and Haskell syntax) for a curried function! That's one reason why those who like mathematical notation (and/or Haskell) like the ES6 arrow function form.

### Why Currying and Partial Application?

With either style -- currying (such as `sum(1)(2)(3)`) or partial application (such as `partial(sum,1,2)(3)`) -- the call-site unquestionably looks stranger than a more common one like `sum(1,2,3)`. So **why would we ever go this direction** when adopting FP? There are multiple layers to answering that question.

The first and most obvious reason is that both currying and partial application allow you to separate in time/space (throughout your codebase) when and where separate arguments are specified, whereas traditional function calls require all the arguments to be present at the same time. If you have a place in your code where you'll know some of the arguments and another place where the other arguments are determined, currying or partial application are very useful.

Another layer to this answer, specifically for currying, is that composition of functions is much easier when there's only one argument. So a function that ultimately needs three arguments, if curried, becomes a function that needs just one, three times over. That kind of unary function will be a lot easier to work with when we start composing them. We'll tackle this topic later in [Chapter 4](ch4.md).

But the most important layer is specialization of generalized functions, and how such abstraction improves readability of code.

Consider our running `ajax(..)` example:

```js
ajax(
    "http://some.api/person",
    { user: CURRENT_USER_ID },
    function foundUser(user){ /* .. */ }
);
```

The call-site includes all the information necessary to pass to the most generalized version of the utility (`ajax(..)`). The potential readability downside is that it may be the case that the URL and the data are not relevant information at this point in the program, but yet that information is cluttering up the call-site nonetheless.

Now consider:

```js
var getCurrentUser = partial(
    ajax,
    "http://some.api/person",
    { user: CURRENT_USER_ID }
);

// later

getCurrentUser( function foundUser(user){ /* .. */ } );
```

In this version, we define a `getCurrentUser(..)` function ahead of time that already has known information like URL and data preset. The call-site for `getCurrentUser(..)` then isn't cluttered by information that **at that point of the code** isn't relevant.

Moreover, the semantic name for the function `getCurrentUser(..)` more accurately depicts what is happening than just `ajax(..)` with a URL and data would.

That's what abstraction is all about: separating two sets of details -- in this case, the *how* of getting a current user and the *what* we do with that user -- and inserting a semantic boundary between them, which eases the reasoning of each part independently.

Whether you use currying or partial application, creating specialized functions from generalized ones is a powerful technique for semantic abstraction and improved readability.

### Currying More Than One Argument?

The definition and implementation I've given of currying thus far is, I believe, as true to the spirit as we can likely get in JavaScript.

Specifically, if we look briefly at how currying works in Haskell, we can observe that multiple arguments always go in to a function one at a time, one per curried call -- other than tuples (analogous to arrays for our purposes) that transport multiple values in a single argument.

For example, in Haskell:

```haskell
foo 1 2 3
```

This calls the `foo` function, and has the result of passing in three values `1`, `2`, and `3`. But functions are automatically curried in Haskell, which means each value goes in as a separate curried-call. The JS equivalent of that would look like `foo(1)(2)(3)`, which is the same style as the `curry(..)` I presented earlier.

**Note:** In Haskell, `foo (1,2,3)` is not passing in those three values at once as three separate arguments, but a tuple (kinda like a JS array) as a single argument. To work, `foo` would need to be altered to handle a tuple in that argument position. As far as I can tell, there's no way in Haskell to pass all three arguments separately with just one function call; each argument gets its own curried-call. Of course, the presence of multiple calls is transparent to the Haskell developer, but it's a lot more syntactically obvious to the JS developer.

For these reasons, I think the `curry(..)` that I demonstrated earlier is a faithful adaptation, or what I might call "strict currying". However, it's important to note that there's a looser definition used in most popular JavaScript FP libraries.

Specifically, JS currying utilities typically allow you to specify multiple arguments for each curried-call. Revisiting our `sum(..)` example from before, this would look like:

```js
var curriedSum = looseCurry( sum, 5 );

curriedSum( 1 )( 2, 3 )( 4, 5 );            // 15
```

We see a slight syntax savings of fewer `( )`, and an implied performance benefit of now having three function calls instead of five. But other than that, using `looseCurry(..)` is identical in end result to the narrower `curry(..)` definition from earlier. I would guess the convenience/performance factor is probably why frameworks allow multiple arguments. This seems mostly like a matter of taste.

We can adapt our previous currying implementation to this common looser definition:

<a name="loosecurry"></a>

```js
function looseCurry(fn,arity = fn.length) {
    return (function nextCurried(prevArgs){
        return function curried(...nextArgs){
            var args = [ ...prevArgs, ...nextArgs ];

            if (args.length >= arity) {
                return fn( ...args );
            }
            else {
                return nextCurried( args );
            }
        };
    })( [] );
}
```

Now each curried-call accepts one or more arguments (as `nextArgs`). We'll leave it as an exercise for the interested reader to define the ES6 `=>` version of `looseCurry(..)` similar to how we did it for `curry(..)` earlier.

### No Curry for Me, Please

It may also be the case that you have a curried function that you'd like to essentially un-curry -- basically, to turn a function like `f(1)(2)(3)` back into a function like `g(1,2,3)`.

The standard utility for this is (un)shockingly typically called `uncurry(..)`. Here's a simple naive implementation:

```js
function uncurry(fn) {
    return function uncurried(...args){
        var ret = fn;

        for (let arg of args) {
            ret = ret( arg );
        }

        return ret;
    };
}

// or the ES6 => arrow form
var uncurry =
    fn =>
        (...args) => {
            var ret = fn;

            for (let arg of args) {
                ret = ret( arg );
            }

            return ret;
        };
```

**Warning:** Don't just assume that `uncurry(curry(f))` has the same behavior as `f`. In some libraries the uncurrying would result in a function like the original, but not all of them; certainly our example here does not. The uncurried function acts (mostly) the same as the original function if you pass as many arguments to it as the original function expected. However, if you pass fewer arguments, you still get back a partially curried function waiting for more arguments; this quirk is illustrated in the following snippet:

```js
function sum(...nums) {
    var sum = 0;
    for (let num of nums) {
        sum += num;
    }
    return sum;
}

var curriedSum = curry( sum, 5 );
var uncurriedSum = uncurry( curriedSum );

curriedSum( 1 )( 2 )( 3 )( 4 )( 5 );        // 15

uncurriedSum( 1, 2, 3, 4, 5 );              // 15
uncurriedSum( 1, 2, 3 )( 4 )( 5 );          // 15
```

Probably the more common case of using `uncurry(..)` is not with a manually curried function as just shown, but with a function that comes out curried as a result of some other set of operations. We'll illustrate that scenario later in this chapter in the ["No Points" discussion](#no-points).

## Order Matters

In Chapter 2, we explored the [named arguments pattern](ch2.md/#named-arguments). One primary advantage of named arguments is not needing to juggle argument ordering, thereby improving readability.

We've looked at the advantages of using currying/partial application to provide individual arguments to a function separately. But the downside is that these techniques are traditionally based on positional arguments; argument ordering is thus an inevitable headache.

Utilities like `reverseArgs(..)` (and others) are necessary to juggle arguments to get them into the right order. Sometimes we get lucky and define a function with parameters in the order that we later want to curry them, but other times that order is incompatible and we have to jump through hoops to reorder.

The frustration is not merely that we need to use some utility to juggle the properties, but the fact that the usage of the utility clutters up our code a bit with extra noise. These kinds of things are like little paper cuts; one here or there isn't a showstopper, but the pain can certainly add up.

Can we improve currying/partial application to free it from these ordering concerns? Let's apply the tricks from named arguments style and invent some helper utilities for this adaptation:

```js
function partialProps(fn,presetArgsObj) {
    return function partiallyApplied(laterArgsObj){
        return fn( Object.assign( {}, presetArgsObj, laterArgsObj ) );
    };
}

function curryProps(fn,arity = 1) {
    return (function nextCurried(prevArgsObj){
        return function curried(nextArgObj = {}){
            var [key] = Object.keys( nextArgObj );
            var allArgsObj = Object.assign(
                {}, prevArgsObj, { [key]: nextArgObj[key] }
            );

            if (Object.keys( allArgsObj ).length >= arity) {
                return fn( allArgsObj );
            }
            else {
                return nextCurried( allArgsObj );
            }
        };
    })( {} );
}
```

**Tip:** We don't even need a `partialPropsRight(..)` because we don't need to care about what order properties are being mapped; the name mappings make that ordering concern moot!

Here's how to use those helpers:

```js
function foo({ x, y, z } = {}) {
    console.log( `x:${x} y:${y} z:${z}` );
}

var f1 = curryProps( foo, 3 );
var f2 = partialProps( foo, { y: 2 } );

f1( {y: 2} )( {x: 1} )( {z: 3} );
// x:1 y:2 z:3

f2( { z: 3, x: 1 } );
// x:1 y:2 z:3
```

Even with currying or partial application, order doesn't matter anymore! We can now specify which arguments we want in whatever sequence makes sense. No more `reverseArgs(..)` or other nuisances. Cool!

**Tip:** If this style of function arguments seems useful or interesting to you, check out coverage of my [FPO library in Appendix C](apC.md/#bonus-fpo).

### Spreading Properties

Unfortunately, we can only take advantage of currying with named arguments if we have control over the signature of `foo(..)` and define it to destructure its first parameter. What if we wanted to use this technique with a function that had its parameters individually listed (no parameter destructuring!), and we couldn't change that function signature? For example:

```js
function bar(x,y,z) {
    console.log( `x:${x} y:${y} z:${z}` );
}
```

Just like the `spreadArgs(..)` utility earlier, we can define a `spreadArgProps(..)` helper that takes the `key: value` pairs out of an object argument and "spreads" the values out as individual arguments.

There are some quirks to be aware of, though. With `spreadArgs(..)`, we were dealing with arrays, where ordering is well defined and obvious. However, with objects, property order is less clear and not necessarily reliable. Depending on how an object is created and properties set, we cannot be absolutely certain what enumeration order properties would come out.

Such a utility needs a way to let you define what order the function in question expects its arguments (e.g., property enumeration order). We can pass an array like `["x","y","z"]` to tell the utility to pull the properties off the object argument in exactly that order.

That's decent, but it's also unfortunate that it then *obligates* us to add that property-name array even for the simplest of functions. Is there any kind of trick we could use to detect what order the parameters are listed for a function, in at least the common simple cases? Fortunately, yes!

JavaScript functions have a `.toString()` method that gives a string representation of the function's code, including the function declaration signature. Dusting off our regular expression parsing skills, we can parse the string representation of the function, and pull out the individually named parameters. The code looks a bit gnarly, but it's good enough to get the job done:

```js
function spreadArgProps(
    fn,
    propOrder =
        fn.toString()
        .replace( /^(?:(?:function.*\(([^]*?)\))|(?:([^\(\)]+?)
            \s*=>)|(?:\(([^]*?)\)\s*=>))[^]+$/, "$1$2$3" )
        .split( /\s*,\s*/ )
        .map( v => v.replace( /[=\s].*$/, "" ) )
) {
    return function spreadFn(argsObj){
        return fn( ...propOrder.map( k => argsObj[k] ) );
    };
}
```

**Note:** This utility's parameter parsing logic is far from bullet-proof; we're using regular expressions to parse code, which is already a faulty premise! But our only goal here is to handle the common cases, which this does reasonably well. We only need a sensible default detection of parameter order for functions with simple parameters (including those with default parameter values). We don't, for example, need to be able to parse out a complex destructured parameter, because we wouldn't likely be using this utility with such a function, anyway. So, this logic gets the job done 80% of the time; it lets us override the `propOrder` array for any other more complex function signature that wouldn't otherwise be correctly parsed. That's the kind of pragmatic balance this book seeks to find wherever possible.

Let's illustrate using our `spreadArgProps(..)` utility:

```js
function bar(x,y,z) {
    console.log( `x:${x} y:${y} z:${z}` );
}

var f3 = curryProps( spreadArgProps( bar ), 3 );
var f4 = partialProps( spreadArgProps( bar ), { y: 2 } );

f3( {y: 2} )( {x: 1} )( {z: 3} );
// x:1 y:2 z:3

f4( { z: 3, x: 1 } );
// x:1 y:2 z:3
```

While order is no longer a concern, usage of functions defined in this style requires you to know what each argument's exact name is. You can't just remember, "oh, the function goes in as the first argument" anymore. Instead, you have to remember, "the function parameter is called 'fn'." Conventions can create consistency of naming that lessens this burden, but it's still something to be aware of.

Weigh these trade-offs carefully.

## No Points

A popular style of coding in the FP world aims to reduce some of the visual clutter by removing unnecessary parameter-argument mapping. This style is formally called tacit programming, or more commonly: point-free style. The term "point" here is referring to a function's parameter input.

**Warning:** Stop for a moment. Let's make sure we're careful not to take this discussion as an unbounded suggestion that you go overboard trying to be point-free in your FP code at all costs. This should be a technique for improving readability, when used in moderation. But as with most things in software development, you can definitely abuse it. If your code gets harder to understand because of the hoops you have to jump through to be point-free, stop. You won't win a blue ribbon just because you found some clever but esoteric way to remove another "point" from your code.

Let's start with a simple example:

```js
function double(x) {
    return x * 2;
}

[1,2,3,4,5].map( function mapper(v){
    return double( v );
} );
// [2,4,6,8,10]
```

Can you see that `mapper(..)` and `double(..)` have the same (or compatible, anyway) signatures? The parameter ("point") `v` can directly map to the corresponding argument in the `double(..)` call. As such, the `mapper(..)` function wrapper is unnecessary. Let's simplify with point-free style:

```js
function double(x) {
    return x * 2;
}

[1,2,3,4,5].map( double );
// [2,4,6,8,10]
```

Let's revisit an example from earlier:

```js
["1","2","3"].map( function mapper(v){
    return parseInt( v );
} );
// [1,2,3]
```

In this example, `mapper(..)` is actually serving an important purpose, which is to discard the `index` argument that `map(..)` would pass in, because `parseInt(..)` would incorrectly interpret that value as a `radix` for the parsing.

If you recall from the beginning of this chapter, this was an example where `unary(..)` helps us out:

```js
["1","2","3"].map( unary( parseInt ) );
// [1,2,3]
```

Point-free!

The key thing to look for is if you have a function with parameter(s) that is/are directly passed to an inner function call. In both of the preceding examples, `mapper(..)` had the `v` parameter that was passed along to another function call. We were able to replace that layer of abstraction with a point-free expression using `unary(..)`.

**Warning:** You might have been tempted, as I was, to try `map(partialRight(parseInt,10))` to right-partially apply the `10` value as the `radix`. However, as we saw earlier, `partialRight(..)` only guarantees that `10` will be the last argument passed in, not that it will be specifically the second argument. Since `map(..)` itself passes three arguments (`value`, `index`, `arr`) to its mapping function, the `10` value would just be the fourth argument to `parseInt(..)`; it only pays attention to the first two.

<a name="shortlongenough"></a>

Here's another example:

```js
// convenience to avoid any potential binding issue
// with trying to use `console.log` as a function
function output(txt) {
    console.log( txt );
}

function printIf( predicate, msg ) {
    if (predicate( msg )) {
        output( msg );
    }
}

function isShortEnough(str) {
    return str.length <= 5;
}

var msg1 = "Hello";
var msg2 = msg1 + " World";

printIf( isShortEnough, msg1 );         // Hello
printIf( isShortEnough, msg2 );
```

Now let's say you want to print a message only if it's long enough; in other words, if it's `!isShortEnough(..)`. Your first thought is probably this:

```js
function isLongEnough(str) {
    return !isShortEnough( str );
}

printIf( isLongEnough, msg1 );
printIf( isLongEnough, msg2 );          // Hello World
```

Easy enough... but "points" now! See how `str` is passed through? Without re-implementing the `str.length` check, can we refactor this code to point-free style?

Let's define a `not(..)` negation helper (often referred to as `complement(..)` in FP libraries):

```js
function not(predicate) {
    return function negated(...args){
        return !predicate( ...args );
    };
}

// or the ES6 => arrow form
var not =
    predicate =>
        (...args) =>
            !predicate( ...args );
```

Next, let's use `not(..)` to alternatively define `isLongEnough(..)` without "points":

```js
var isLongEnough = not( isShortEnough );

printIf( isLongEnough, msg2 );          // Hello World
```

That's pretty good, isn't it? But we *could* keep going. `printIf(..)` could be refactored to be point-free itself.

We can express the `if` conditional part with a `when(..)` utility:

```js
function when(predicate,fn) {
    return function conditional(...args){
        if (predicate( ...args )) {
            return fn( ...args );
        }
    };
}

// or the ES6 => form
var when =
    (predicate,fn) =>
        (...args) =>
            predicate( ...args ) ? fn( ...args ) : undefined;
```

Let's mix `when(..)` with a few other helper utilities we've seen earlier in this chapter, to make the point-free `printIf(..)`:

```js
var printIf = uncurry( partialRight( when, output ) );
```

Here's how we did it: we right-partially-applied the `output` method as the second (`fn`) argument for `when(..)`, which leaves us with a function still expecting the first argument (`predicate`). *That* function when called produces another function expecting the message string; it would look like this: `fn(predicate)(str)`.

A chain of multiple (two) function calls like that looks an awful lot like a curried function, so we `uncurry(..)` this result to produce a single function that expects the two `str` and `predicate` arguments together, which matches the original `printIf(predicate,str)` signature.

Here's the whole example put back together (assuming various utilities we've already detailed in this chapter are present):

<a name="finalshortlong"></a>

```js
function output(msg) {
    console.log( msg );
}

function isShortEnough(str) {
    return str.length <= 5;
}

var isLongEnough = not( isShortEnough );

var printIf = uncurry( partialRight( when, output ) );

var msg1 = "Hello";
var msg2 = msg1 + " World";

printIf( isShortEnough, msg1 );         // Hello
printIf( isShortEnough, msg2 );

printIf( isLongEnough, msg1 );
printIf( isLongEnough, msg2 );          // Hello World
```

Hopefully the FP practice of point-free style coding is starting to make a little more sense. It'll still take a lot of practice to train yourself to think this way naturally. **And you'll still have to make judgement calls** as to whether point-free coding is worth it, as well as what extent will benefit your code's readability.

What do you think? Points or no points for you?

**Note:** Want more practice with point-free style coding? We'll revisit this technique in [Chapter 4, "Revisiting Points"](ch4.md/#revisiting-points), based on newfound knowledge of function composition.

## Summary

Partial application is a technique for reducing the arity (that is, the expected number of arguments to a function) by creating a new function where some of the arguments are preset.

Currying is a special form of partial application where the arity is reduced to 1, with a chain of successive chained function calls, each which takes one argument. Once all arguments have been specified by these function calls, the original function is executed with all the collected arguments. You can also undo a currying.

Other important utilities like `unary(..)`, `identity(..)`, and `constant(..)` are part of the base toolbox for FP.

Point-free is a style of writing code that eliminates unnecessary verbosity of mapping parameters ("points") to arguments, with the goal of making code easier to read/understand.

All of these techniques twist functions around so they can work together more naturally. With your functions shaped compatibly now, the next chapter will teach you how to combine them to model the flows of data through your program.
