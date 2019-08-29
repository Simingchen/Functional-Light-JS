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

// 改造后:
cacheResult( "http://some.api/person", { user: CURRENT_USER_ID } );
```

为此，我们可以定义一个`partialRight(..)`，而不是手动使用(两次!)`reverseArgs(..)` 封装后，它可以使用相同的双反转技巧:

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

// 改造后:
cacheResult( "http://some.api/person", { user: CURRENT_USER_ID } );
```

另一个更直接(当然也更高效)的实现`partialRight(..)`，它没有使用双重反转技巧:
```js
function partialRight(fn,...presetArgs) {
    return function partiallyApplied(...laterArgs){
        return fn( ...laterArgs, ...presetArgs );
    };
}

// ES6箭头函数格式
var partialRight =
    (fn,...presetArgs) =>
        (...laterArgs) =>
            fn( ...laterArgs, ...presetArgs );
```

这些`partialRight(..)`的实现都不能保证特定的参数将接收到特定的部分应用值;它只确保部分应用的值作为传递给原始函数的最右边(也就是最后一个)参数出现。

示例:

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

值 `"z:last"`只适用于`z`参数，当`f(..)`被调用时正好有两个参数(匹配`x` 和`y`参数)。在所有其他情况下，`"z:last"`将是最正确的参数，不管它前面有多少个参数。

## 一次一个

让我们研究一种类似于部分应用程序的技术，其中期望多个参数的函数被分解为连续的链接函数，每个函数接受一个参数并返回另一个函数来接受下一个参数。

这种技术被称为柯里化。

首先，让我们假设已经创建了一个课程版的“ajax（…）”。我们就是这样使用它的：

```js
curriedAjax( "http://some.api/person" )
    ( { user: CURRENT_USER_ID } )
        ( function foundUser(user){ /* .. */ } );
```

`(..)`的三组表示三个链接函数调用。但也许把这三个回调分开会让你更好地了解情况:

```js
var personFetcher = curriedAjax( "http://some.api/person" );

var getCurrentUser = personFetcher( { user: CURRENT_USER_ID } );

getCurrentUser( function foundUser(user){ /* .. */ } );
```

这个`curriedAjax(..)`函数不是一次接受所有参数(如`ajax(..)`)，或者先接收一些参数，然后再接收其他参数(通过`partial(..)`)，而是一次接收一个参数，每个参数都在一个单独的函数调用中。

柯里化类似于部分应用程序，在传递完所有参数之前，每个连续的局部套用调用都会将另一个参数部分应用于原始函数。

主要的区别是，`curriedAjax(..)`将返回一个函数（我们称之为`personFetcher(..)`），它只期望**下一个参数**`data`，而不是（像前面的`getPerson(..)`）可以接收所有其他参数的函数。

如果一个原始函数需要五个参数，那么该函数的柯里化形式将只接受第一个参数，并返回一个函数来接受第二个参数。它只接受第二个参数，并返回一个函数来接受第三个参数。等等。

因此，柯里化将单个高阶函数展开为一系列链式一元函数。

我们如何定义一个实用程序来实现这种柯里化?考虑:

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

// ES6箭头函数格式
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

这里的方法是将' prevArgs '中的参数集合作为空'[]'数组，并将每个接收到的' nextArg '添加到其中，调用连接' args '。而“参数。length '小于' arity '(原始' fn(..) '函数声明/预期参数的数量)，make并返回另一个' curried(..) '函数来收集下一个' nextArg '参数，将运行中的' args '集合作为它的' prevArgs '传递。一旦我们有了足够的“args”，使用它们执行原始的“fn(..)”函数。

默认情况下，此实现依赖于能够检查待处理函数的“length”属性，以了解在收集所有预期参数之前需要进行多少次柯里化。

**注意:**如果您将' curry(..) '的这个实现用于一个没有精确的' length '属性的函数，则需要传递' arity ' (' curry(..) '的第二个参数)，以确保' curry(..) '工作正常。如果函数的参数签名包含默认参数值、参数析构或变量为…args '(参见[第2章](ch2.md))。

下面是我们将如何在前面的`ajax(..)` 示例中使用`curry(..)`:

```js
var curriedAjax = curry( ajax );

var personFetcher = curriedAjax( "http://some.api/person" );

var getCurrentUser = personFetcher( { user: CURRENT_USER_ID } );

getCurrentUser( function foundUser(user){ /* .. */ } );
```

每个调用都会对原始的`ajax(..)`调用再应用一个参数，直到提供了所有三个参数并实际调用了`ajax(..)`。

还记得我们在讨论部分应用程序时的例子吗?由于柯里化与局部应用类似，我们可以用几乎相同的方法来完成这个任务:

```js
[1,2,3,4,5].map( curry( add )( 3 ) );
// [4,5,6,7,8]
```

注意到两者之间的区别?`partial(add,3)` 与 `curry(add)(3)`。

为什么你会选择`curry(..)`而不是`partial(..)`?如果您事先知道`add(..)`是要修改的函数，但' 3 '的值还不知道，这可能会有帮助:

```js
var adder = curry( add );

// later
[1,2,3,4,5].map( adder( 3 ) );
// [4,5,6,7,8]
```

让我们看另一个数字的例子，这次把它们加在一起:

```js
function sum(...nums) {
    var total = 0;
    for (let num of nums) {
        total += num;
    }
    return total;
}

sum( 1, 2, 3, 4, 5 );                       // 15

// 现在使用柯里化:
// (5 to indicate how many we should wait for)
var curriedSum = curry( sum, 5 );

curriedSum( 1 )( 2 )( 3 )( 4 )( 5 );        // 15
```

这里的优点是，每次传入参数的调用都会产生另一个更专门化的函数，我们可以在程序的后面捕获并使用*这个*新函数。部分应用程序预先指定所有部分应用的参数，生成一个函数，该函数正在等待下一个调用**上的所有其他参数**。

如果您想使用部分应用程序一次指定一个(或多个!)参数，则必须对每个连续的部分应用的函数再次调用`partial(..)`。相比之下，柯里化的函数可以自动完成这一任务，使每次处理单个参数更加符合人体工程学。

柯里化和部分应用程序都使用闭包来随着时间记住参数，直到所有参数都被接收，然后才能调用原始函数。

### 柯里化的可视化功能

让我们更仔细地研究前一节中的`curriedSum(..)`。回想一下它的用法:curriedSum(1)(2)(3)(4)(5);5个后续(链式)函数调用。

如果我们手工定义一个`curriedSum(..)`而不是使用`curry(..)`会怎么样?

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

绝对更丑，毫无疑问。但这是一个很重要的方法来形象化一个柯里化函数。每个嵌套函数调用都返回另一个函数，该函数将接受下一个参数，并继续执行，直到指定了所有预期的参数。

当我试图破译柯里化函数时，我发现如果我能在心里把它们分解成一系列嵌套函数，这对我有很大的帮助。

事实上，为了加强这一点，让我们考虑同样的代码，但用ES6箭头函数编写:

```js
curriedSum =
    v1 =>
        v2 =>
            v3 =>
                v4 =>
                    v5 =>
                        sum( v1, v2, v3, v4, v5 );
```

现在，所有的都在一行:

```js
curriedSum = v1 => v2 => v3 => v4 => v5 => sum( v1, v2, v3, v4, v5 );
```

根据您的观点，这种将柯里化函数可视化的形式可能或多或少对您有帮助。对我来说，这有点模糊。

但我用这种方式展示它的原因是，它看起来几乎与柯里化函数的数学符号(和Haskell语法)相同!这就是那些喜欢数学符号(和/或Haskell)的人喜欢ES6箭头函数形式的原因之一。

### 为什么是柯里化和局部应用?

使用任意一种样式——柯里化(如`sum(1)(2)(3)`)或部分应用程序(如`partial(sum,1,2)(3)`)——调用站点无疑比更常见的`sum(1,2,3)`看起来更奇怪。那么，在采用FP时，我们为什么要走这个方向呢?回答这个问题有很多层次。

第一个也是最明显的原因是，局部套用和局部应用程序都允许在指定单独参数的时间/空间(在整个代码库中)中分离，而传统的函数调用要求所有参数同时出现。如果您在代码中有一个位置可以知道一些参数，而在另一个位置可以确定其他参数，那么局部套用或局部应用程序非常有用。

这个答案的另一个层次，特别是对于局部套用，是当只有一个参数时，函数的组合要容易得多。所以一个最终需要三个参数的函数，如果柯里化，变成一个只需要一个，三次的函数。当我们开始组合它们时，这种一元函数会更容易处理。我们将在稍后的[第4章](ch4.md)中处理这个主题。

但是最重要的层是通用函数的专门化，以及这种抽象如何提高代码的可读性。

考虑我们运行的 `ajax(..)` 示例:

```js
ajax(
    "http://some.api/person",
    { user: CURRENT_USER_ID },
    function foundUser(user){ /* .. */ }
);
```

调用站点包含传递到实用程序的最通用版本(`ajax(..)`)所需的所有信息。潜在的易读性缺点是，URL和数据在程序中的这一点上可能不是相关的信息，但是尽管如此，这些信息仍然使调用站点混乱不堪。

现在想一想:

```js
var getCurrentUser = partial(
    ajax,
    "http://some.api/person",
    { user: CURRENT_USER_ID }
);

// 改造后

getCurrentUser( function foundUser(user){ /* .. */ } );
```

在这个版本中，我们预先定义了一个`getCurrentUser(..)`函数，该函数已经具有URL和数据预置等已知信息。这样，`getCurrentUser(..)`的调用就不会被代码中**不相关的信息所打乱。

此外，函数`getCurrentUser(..)`的语义名称比仅使用URL和数据的`ajax(..)`更准确地描述了正在发生的事情。

这就是抽象的全部含义:分离两组细节——在本例中，是获取当前用户的*方法*和使用该用户的*方法*——并在它们之间插入语义边界，这将简化每个部分的独立推理。

无论您使用柯里化还是局部应用程序，从通用函数创建专用函数都是语义抽象和提高可读性的强大技术。

### 柯里化多个参数？

到目前为止，我所给出的关于柯里化的定义和实现，我相信是最符合JavaScript精神的。

具体地说，如果我们简单地看一下在Haskell中如何使用局部套用，我们可以观察到，多个参数总是一个一个地进入一个函数，每个局部套用调用一个参数——而不是在一个参数中传输多个值的元组(类似于我们的目的中的数组)。

例如，在Haskell中:

```haskell
foo 1 2 3
```

它调用`foo`函数，并传递三个值' 1 '、' 2 '和' 3 '。但是函数在Haskell中是自动柯里化的，这意味着每个值都作为一个单独的调用进入。对应的JS应该类似于`foo(1)(2)(3)`，这与我前面介绍的`curry(..)`风格相同。

**注意:**在Haskell中，`foo (1,2,3)`不是同时作为三个单独的参数传递这三个值，而是作为一个单独的参数传递一个元组(有点像JS数组)。为了工作，需要修改' foo '来处理那个参数位置的元组。据我所知，在Haskell中不可能仅通过一个函数调用就分别传递所有三个参数;每个参数都有自己的柯里化调用。当然，对于Haskell开发人员来说，多个调用的存在是透明的，但是对于JS开发人员来说，它在语法上要明显得多。

基于这些原因，我认为我之前演示的`curry(..)`是一种忠实的改编，或者我可以称之为“严格的柯里化”。但是，需要注意的是，在大多数流行的JavaScript FP库中使用了一个更宽松的定义。

具体来说，JS 柯里化实用程序通常允许为每个调用指定多个参数。重新查看前面的“sum(..)”示例，如下所示:

```js
var curriedSum = looseCurry( sum, 5 );

curriedSum( 1 )( 2, 3 )( 4, 5 );            // 15
```

我们看到语法上稍微节省了一些'()'，并且现在有三个函数调用而不是五个函数调用带来了性能上的好处。但除此之外，使用`looseCurry(..)`与前面定义的`curry(..)`在最终结果上是相同的。我想，方便/性能因素可能是框架允许多个参数的原因。这似乎主要是品味的问题。

我们可以调整我们之前的柯里化实现来适应这个更宽松的定义:

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

现在，每个柯里化调用都接受一个或多个参数(作为`nextArgs`)。我们将把它作为一个练习留给感兴趣的读者来定义ES6 `=>`版本的`looseCurry(..)`类似于我们之前为`curry(..)`所做的那样。

### 请不要对我用柯里化

也有可能你有一个柯里化函数你想要取消柯里化——基本上，把一个像f(1)(2)(3)这样的函数变成像g(1,2,3)这样的函数。

这方面的标准实用程序通常被称为`uncurry(..)`。这里有一个简单的天真的实现:

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

**警告:**不要想当然地认为`uncurry(curry(f))`与`f`具有相同的行为。在一些库中，解列会得到与原始函数类似的函数，但不是所有函数;当然我们这里的例子没有。如果传递的参数与原始函数期望的一样多，那么反柯里化函数的作用(大多数情况下)与原始函数相同。然而，如果传递更少的参数，仍然会返回一个部分柯里化的函数，等待更多的参数;下面的代码片段说明了这种怪癖:

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

可能使用`uncurry(..)`更常见的情况并不是像刚才显示的那样使用手动柯里化函数，而是使用由其他一些操作集生成的柯里化函数。我们将在本章后面的[“No Points”讨论](#no-points)中演示该场景。

## 顺序的重要性

在第2章中，我们探讨了[命名参数模式](ch2.md/#named-arguments)。命名参数的一个主要优势是不需要改变参数的顺序，从而提高可读性。

我们已经看到了使用柯里化/局部应用程序分别为函数提供单独参数的优点。但缺点是这些技术传统上是基于位置参数的;因此，论点排序是一个不可避免的头痛问题。

像`reverseArgs(..)`(和其他)这样的实用程序是调整参数以使它们处于正确顺序所必需的。有时我们很幸运地定义了一个带有参数的函数，其顺序我们稍后将对其进行修改，但有时这个顺序是不兼容的，我们必须跳过一些困难才能重新排序。

令人沮丧的是，我们不仅需要使用一些实用程序来处理属性，而且使用该实用程序会给代码带来额外的干扰，使代码变得有些混乱。这些东西就像小剪纸;这里有一个，那里没有一个，但痛苦肯定会累积起来。

我们能否改进柯里化/部分应用程序，使其免于这些排序问题?让我们应用命名参数风格的技巧，并为这种适应发明一些辅助实用程序:

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

**提示：**我们甚至不需要`partialPropsRight(..)`，因为我们不需要关心映射的顺序属性；名称映射使排序问题也变得不再重要！

以下是如何使用：

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


即使使用柯里化或局部应用程序，顺序也不再重要了!现在，我们可以指定在任何有意义的序列中需要哪些参数。不再有`reverseArgs(..)`或其他讨厌的东西。这样真的是太酷了!

**提示:**如果您觉得这种类型的函数参数有用或有趣，请查看附录C中的[FPO库](apC.md/#bonus-fpo)。

### 扩展属性

不幸的是，如果我们控制了`foo(..)`的签名，并且定义它来破坏它的第一个参数，那么我们只能利用与命名参数的冲突。如果我们想将此技术与一个单独列出参数的函数一起使用（没有参数破坏！），我们无法更改该函数签名？例如：

```js
function bar(x,y,z) {
    console.log( `x:${x} y:${y} z:${z}` );
}
```

就像前面的 `spreadArgs(..)` 实用程序一样，我们可以定义一个`spreadArgProps(..)`，它将`key: value`对从对象参数中取出，并将值作为单独的参数“展开”。

不过，也有一些怪癖需要注意。使用`spreadArgs(..)`，我们处理的是数组，其中的顺序定义得很好，也很明显。然而，对于对象，属性顺序不太清晰，也不一定可靠。根据对象创建和属性设置的方式，我们不能绝对确定将产生哪些枚举顺序属性。

这样的实用程序需要一种方法来让您定义函数期望参数的顺序(例如，属性枚举顺序)。我们可以传递一个像`["x","y","z"]` 这样的数组，来告诉这个实用程序以完全相同的顺序从对象参数中提取属性。

这很好，但不幸的是，即使对于最简单的函数，它也“强制”我们添加属性名数组。至少在常见的简单情况下，有没有什么技巧可以用来检测函数参数的排列顺序?幸运的是,有的!

JavaScript函数有一个`.toString()`方法，该方法给出函数代码的字符串表示，包括函数声明签名。抛开正则表达式解析技巧，我们可以解析函数的字符串表示，并提取单独命名的参数。代码看起来有点粗糙，但已经足够完成这项工作了:

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

**注意:**本实用程序的参数解析逻辑远非无懈可击;我们使用正则表达式解析代码，这已经是一个错误的前提!但我们这里的唯一目标是处理常见的情况，这做得相当好。我们只需要对具有简单参数的函数(包括具有缺省参数值的函数)的参数顺序进行合理的缺省检测。例如，我们不需要能够解析出复杂的析构参数，因为我们不太可能将这个实用程序与这样的函数一起使用。所以，这个逻辑在80%的时间里完成了任务;它允许我们覆盖任何其他更复杂的函数签名的“比例”数组，否则将无法正确解析。这就是本书所寻求的那种务实的平衡。

让我们用我们的`spreadArgProps(..)`工具来演示:

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

虽然顺序不再是问题，但是使用这种样式定义的函数需要知道每个参数的确切名称。你不能只记住，“哦，函数作为第一个参数进入”相反，您必须记住，“函数参数称为'fn'。”约定可以创建一致性的命名，从而减轻这种负担，但是仍然需要注意。

认真考虑这些权衡


## 无参数风格

函数编程中流行的一种编码风格旨在通过删除不必要的参数-参数映射来减少一些视觉上的混乱。这种风格的正式名称是默示编程，或者更常见的名称是:无参数风格编程。这里的术语“点”（点(point)：指的是参数）指的是函数的参数输入。

**警告:**请稍等。让我们确保我们小心不要把这个讨论看作是一个无界的建议，即您不惜一切代价在FP代码中尝试免费。如果使用得当，这应该是一种提高可读性的技术。但与软件开发中的大多数事情一样，您肯定会滥用它。如果你的代码变得难以理解，因为你必须跳过这些障碍才能不扣分，那就停下来。您不会仅仅因为找到了一些聪明但深奥的方法来从代码中删除另一个参数就获得蓝丝带奖励。

让我们从一个简单的例子开始:

```js
function double(x) {
    return x * 2;
}

[1,2,3,4,5].map( function mapper(v){
    return double( v );
} );
// [2,4,6,8,10]
```

您能看到mapper(..)`和`double(..)`具有相同(或者兼容)的签名吗?参数`v`可以直接映射到`double(..)`调用中对应的参数。因此，`mapper(..)`函数包装器是不必要的。让我们用无参数风格的方式来简化:

```js
function double(x) {
    return x * 2;
}

[1,2,3,4,5].map( double );
// [2,4,6,8,10]
```

让我们回顾一下之前的一个例子:

```js
["1","2","3"].map( function mapper(v){
    return parseInt( v );
} );
// [1,2,3]
```

在本例中， `mapper(..)`实际上起到了一个重要的作用，即丢弃了`map(..)`传入的`index`参数，因为 `parseInt(..)`将错误地将该`index`值解释为解析的进制'基数'。

如果你回想一下这一章的开头， `unary(..)` 可以帮助我们解决问题的例子:

```js
["1","2","3"].map( unary( parseInt ) );
// [1,2,3]
```

这就是无参数风格！

要查找的关键问题是，是否有一个带有参数的函数直接传递给内部函数调用。在前面的两个示例中，`mapper(..)`具有传递给另一个函数调用的`v` 参数。我们能够用一个使用`unary(..)`的无参数表达式替换这个抽象层。

**警告:**您可能和我一样，尝试将`map(partialRight(parseInt,10))`应用到右侧，部分应用`10`值作为进制“基数”。然而，正如我们前面看到的，`partialRight(..)`只保证`10`是传入的最后一个参数，而不是确切地说是第二个参数。由于`map(..)`本身将三个参数(`value`, `index`, `arr`) 传递给它的映射函数，`10`值将是`parseInt(..)`的第四个参数;它只关注前两个。

<a name="shortlongenough"></a>

这是另一个例子:

```js
// 方便避免任何潜在的绑定问题
// 尝试使用`console.log`作为一个函数
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

现在假设您只想在消息足够长的情况下打印消息；换句话说，如果是运行到`!isShortEnough(..)`。你的第一个想法可能是：

```js
function isLongEnough(str) {
    return !isShortEnough( str );
}

printIf( isLongEnough, msg1 );
printIf( isLongEnough, msg2 );          // Hello World
```

很简单…但现在看现在的参数！看看`str`是如何传递的？如果不重新实现`str.length`检查，我们可以重构这段代码为无参数风格的代码吗？

让我们定义一个 `not(..)` 函数(通常在FP库中称为`complement(..)`):

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

接下来，我们使用`not(..)`来定义没有参数的`isLongEnough(..)`:
Next, let's use `not(..)` to alternatively define `isLongEnough(..)` without "points":

```js
var isLongEnough = not( isShortEnough );

printIf( isLongEnough, msg2 );          // Hello World
```

很不错，不是吗?但是我们可以继续。`printIf(..)`本身可以重构为无参数。

我们可以用一个`when(..)`工具来表达`if`条件句:

```js
function when(predicate,fn) {
    return function conditional(...args){
        if (predicate( ...args )) {
            return fn( ...args );
        }
    };
}

// ES6箭头格式
var when =
    (predicate,fn) =>
        (...args) =>
            predicate( ...args ) ? fn( ...args ) : undefined;
```

让我们将`when(..)`和我们在本章前面看到的其他一些辅助工具混合使用，以实现无参数的`printIf(..)`:

```js
var printIf = uncurry( partialRight( when, output ) );
```

我们是这样做的：我们正确地部分应用了`output`方法作为`when(..)`的第二个(`fn`)参数，这使得我们有一个函数仍然需要第一个参数(`predicate`)。*当调用该*函数时，会生成另一个期望消息字符串的函数；它看起来像是：`fn(predicate)(str)`。


一个由多个（两个）函数调用组成的链看起来非常像一个循环函数，因此我们`uncurry(..)`这个结果产生一个函数，它期望两个`str`和`predicate`参数在一起，这与原始的`printIf(predicate,str)`签名匹配。

下面是完整的例子(假设本章已经详细介绍了各种实用工具):

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
希望FP(函数编程)的无参数风格编码实践开始变得更有意义。要训练自己自然地以这种方式思考，仍然需要大量的练习。**您仍然需要判断调用**是否值得这样做，以及在多大程度上有利于代码的可读性。

你觉得怎么样?需不需要参数的风格都在于你?

**注意:**想要更多的实践无参数编码风格?我们将在[第4章，"重提参数的作用"](ch4.md/#revisiting-points)中基于函数组合的新知识重新讨论这种技术。

## 小结

部分应用程序是一种通过创建一个新的函数来减少元数（函数的预期参数数量）的技术，其中一些参数是预设的。

柯里化是一种特殊的部分应用形式，其中元数（函数的参数数量）被简化为1，具有一系列连续的链接函数调用，每个调用都接受一个参数。一旦这些函数调用指定了所有参数，就可以使用收集的所有参数执行原始函数。当然你也可以反柯里化。

其他重要的实用程序，如`unary(..)`, `identity(..)`和`constant(..)`是FP（函数编程）基本工具库的一部分。

无参数(Point-free)是一种编写代码的风格，它消除了将参数(“point”)映射到参数的不必要冗长，目的是使代码更容易阅读理解。

所有这些技术都围绕着功能进行调整，以便更自然地协同工作。现在您的函数已经形成了兼容的形式，下一章将教会您如何将它们组合起来为程序中的数据流建模。
