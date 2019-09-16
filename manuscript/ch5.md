# 章节 5: 减少副作用影响

在[第2章](ch2.md)中，我们讨论了一个函数除了它的`return`值之外，还可以有哪些输出。到目前为止，您应该对函数的FP定义非常熟悉了，所以对这种副作用应该有印象。

我们将研究各种不同形式的副作用，并看看它们为什么对代码的质量和可读性有害。

本章的重点是:不可能编写一个没有副作用的程序。并非不可能;你当然可以。但这个程序不会做任何有用或可观察的事情。如果你写了一个没有副作用的程序，你就不能分辨出它和一个空程序的区别。

FP函数编程人员并不能消除所有副作用。相反，我们的目标是尽可能地限制它们。要做到这一点，我们首先需要完全理解它们。

## Effects on the Side, Please

因果关系:人类对周围世界最基本、最直观的观察之一。把一本书从桌子边上推下去，它掉到地上了。你不需要物理学位就能知道原因是你推了书，结果是重力把书拖到地上。这是一种明确而直接的关系。

在编程中，我们也处理因果关系。如果您调用一个函数(原因)，它将在屏幕上显示一条消息(效果)。

当阅读一个程序时，读者能够清楚地识别每一个原因和每一个结果是极其重要的。如果在程序的通读过程中不能很容易地看出因果之间的直接关系，那么程序的可读性就会下降。

考虑:

```js
function foo(x) {
    return x * 2;
}

var y = foo( 3 );
```

在这个简单的程序中，很明显，用值`3`调用foo (原因)会产生返回值`6` 的效果，然后将值赋给`y` (结果)。这里没有歧义。

现在改变下:

```js
function foo(x) {
    y = x * 2;
}

var y;

foo( 3 );
```

这个程序有完全相同的结果。但是有一个很大的区别。因果是不相交的。这种影响是间接的。这样设置 `y`就是我们所说的副作用。

注:函数在自身外部引用变量时，称为自由变量。并不是所有的自由变量引用都是不好的，但是我们要非常小心地使用它们。

如果我给您一个引用来调用一个函数`bar(..)`，但是您看不到它的代码，但是我告诉您它没有这种间接的副作用，只有一个显式的`return`值效果，那会怎么样呢?

```js
bar( 4 );           // 42
```

因为您知道`bar(..)`的内部机制不会产生任何副作用，所以现在您可以以一种更直接的方式推断任何`bar(..)`调用。但如果你不知道`bar(..)`没有副作用，要理解调用它的结果，你就必须阅读并剖析它的所有逻辑。这对读者来说是额外的精神负担。

**副作用函数的可读性较差**，因为它需要更多的阅读来理解程序。

但问题远不止于此。考虑:

```js
var x = 1;

foo();

console.log( x );

bar();

console.log( x );

baz();

console.log( x );
```

您如何确定在每个`console.log(x)`上打印哪些值?

正确答案是:一点也不。如果你不确定是否`foo()`, `bar()`, 和 `baz()`的副作用,你不能保证每一步打印的`x`具体值,除非你检查从第1行开始跟踪程序，跟踪状态的所有变化。

换句话说，最后的`console.log(x)`是不可能分析或预测的，除非您已经在心里执行了整个程序。

猜猜谁最擅长运行你的程序?JS引擎。猜猜谁不擅长运行你的程序?读你代码的人。然而，您选择在一个或多个函数调用中编写(潜在的)具有副作用的代码，这意味着您必须在一定程度上在读者的脑海中完整地执行到某一行，以便他们阅读和理解这一行。

如果`foo()`, `bar()`和`baz()`都没有副作用，那么它们就不能影响`x`”，这意味着我们不需要从心理上跟踪执行`x`发生了什么。这减少了脑力劳动，并使代码更具可读性。

### 隐藏的原因

输出，状态的变化，是副作用最常见的表现形式。但另一种损害可读性的做法是一些人所说的副作用。考虑:

```js
function foo(x) {
    return x + y;
}

var y = 3;

foo( 1 );           // 4
```

`y`没有被`foo(..)`改变，所以它的副作用和我们之前看到的不一样。但是现在，`foo(..)`的调用实际上取决于`y`的存在和当前状态。如果稍后，我们这样做:


```js
y = 5;

// ..

foo( 1 );           // 6
```

对于`foo(1)`的调用在不同的调用之间返回不同的结果，我们可能会感到惊讶吗?

`foo(..)`有一个间接的原因，这对可读性是有害的。如果没有仔细检查`foo(..)`的实现，读者无法看到是什么原因导致了输出效果。看起来参数`1`是唯一的原因，但事实并非如此。

为了提高可读性，所有决定`foo(..)`输出效果的因素都应该作为`foo(..)`的直接且明显的输入。代码的读者将清楚地看到原因和结果。

#### 固定状态

避免副作用是否意味着`foo(..)` 函数不能引用任何自由变量?

考虑这段代码:

```js
function foo(x) {
    return x + bar( x );
}

function bar(x) {
    return x * 2;
}

foo( 3 );           // 9
```
很明显，对于`foo(..)`和 `bar(..)`，唯一的直接原因是`x`参数。但是`bar(x)`调用呢?`bar`只是一个标识符，在JS中它甚至不是一个常量(也就是不可重分配的变量)。`foo(..)`函数依赖于`bar`的值——一个引用第二个函数的变量——作为一个自由变量。

那么，这段代码是否有一个副作用呢?

没有。尽管可以用其他函数覆盖`bar`变量的值，但在这段代码中我没有这样做，这也不是我的常见做法或先例。实际上，我的函数是常量(从不重新赋值)。

考虑:

```js
const PI = 3.141592;

function foo(x) {
    return x * PI;
}

foo( 3 );           // 9.424776000000001
```

注:JavaScript有内置函数`Math.PI`，所以我们只是使用`PI`的例子在这篇文章作为一个方便的说明。在实践中，总是使用`Math.PI`。而不是使用自定义变量!

那么前面的代码片段呢?`PI`是`foo(..)`的一个副作用吗?

有两项观察将有助于我们以合理的方式回答这个问题:

1. 想想你可能给 `foo(3)`做的每一个回调。它总是返回`9.424..`值? 是的。每一次。如果您给它相同的输入(`x`)，它总是返回相同的输出。

2. 你能不能把PI的每一个用法都替换成它的直接值，程序能不能像以前一样运行呢?能。本程序没有任何部分依赖于能够更改`PI`的值——实际上，由于它是一个`const`声明变量，所以不能重新分配它——所以这里的`PI`'变量只是为了可读性/维护性。它的值可以内联而不改变程序行为。

我的结论是:这里的`PI`变量并没有违反最小化/避免副作用的精神。前面代码片段中的`bar(x)`调用也没有。

在这两种情况下，`PI`和`bar`都不是程序状态的一部分。它们是固定的、非重分配的引用。如果它们在整个程序中没有变化，我们就不必担心跟踪它们作为变化状态。因此，它们不会损害我们的可读性。而且它们不可能是与以意想不到的方式变化的变量相关的bug的来源。

注:此处使用`const`一词，在我看来，并不能说明`PI`可以避免副作用;`var PI`也会得出同样的结论。重要的不是不能重新分配`PI`，而是不能重新分配`PI`。我们将在第6章讨论[const](ch6.md/#reassignment)。

#### 随机性

你可能从来没有考虑过，但随机性是一个副作用。使用`Math.random()`的函数不能根据其输入获得可预测的输出。任何生成唯一随机id的代码。根据定义，将被认为是程序的副作用。

在计算中，我们使用伪随机算法来生成。事实证明，真正的随机性是相当困难的，所以我们只是用一些复杂的算法来伪造它，这些算法产生的值看起来明显是随机的。这些算法计算长串的数字，但秘密是，如果你知道起始点，序列实际上是可以预测的。这个起点称为seed（种子）。

有些语言允许指定随机数生成的种子值。如果总是指定相同的种子，那么从后续的“伪随机数”中总会得到相同的输出序列。这对于测试目的非常有用，但是对于实际应用程序的使用非常危险。

在JS中，`Math.random()`计算的随机性是基于间接输入的，因为不能指定种子。因此，我们必须把内置随机数生成当作一个副作用。

### I/O 作用

最常见的(本质上也是不可避免的)副作用形式是输入/输出(I/O)。没有I/O的程序是完全没有意义的，因为它的工作不能以任何方式被观察到。有用的程序必须至少有输出，而且许多程序还需要输入。输入是副作用，输出也是副作用。

浏览器的JS程序典型输入是用户事件(鼠标、键盘)，输出是DOM。如果您在Node中工作得更多。您可能更有可能从文件系统、网络连接和/或`stdin`/`stdout`流接收输入，并将输出发送到这些流。

事实上，这些来源既可以是输入，也可以是输出，既有因果关系。以DOM为例。我们更新(副作用)DOM元素以向用户显示文本或图像，但是DOM的当前状态也是这些操作的隐式输入(副作用)。

### 副作用 Bug

可能导致bug的副作用和副作用的场景与现有的程序一样多种多样。但是让我们检查一个场景来说明这些危险，希望它们能帮助我们在自己的程序中识别出类似的错误。

考虑:

```js
var users = {};
var userOrders = {};

function fetchUserData(userId) {
    ajax( `http://some.api/user/${userId}`, function onUserData(user){
        users[userId] = user;
    } );
}

function fetchOrders(userId) {
    ajax(
        `http://some.api/orders/${userId}`,
        function onOrders(orders){
            for (let order of orders) {
                // 为每个用户保留对最新订单的引用
                users[userId].latestOrder = order;
                userOrders[order.orderId] = order;
            }
        }
    );
}

function deleteOrder(orderId) {
    var user = users[ userOrders[orderId].userId ];
    var isLatestOrder = (userOrders[orderId] == user.latestOrder);

    // 删除用户的最新订单?
    if (isLatestOrder) {
        hideLatestOrderDisplay();
    }

    ajax(
        `http://some.api/delete/order/${orderId}`,
        function onDelete(success){
            if (success) {
                // 删除用户的最新订单?
                if (isLatestOrder) {
                    user.latestOrder = null;
                }

                userOrders[orderId] = null;
            }
            else if (isLatestOrder) {
                showLatestOrderDisplay();
            }
        }
    );
}
```

我敢打赌，对于一些读者来说，其中一个潜在的bug是相当明显的。如果回调`onOrders(..)`运行在`onUserData(..)`回调之前，它将尝试向尚未设置的值(位于`users[userId]`的`user`对象)添加一个`latestOrder`属性。

因此，依赖于副作用的逻辑可能出现的一种形式的“bug”是两个不同操作的竞态条件(异步或不异步!)，我们希望这两个操作以特定的顺序运行，但在某些情况下可能以不同的顺序运行。有一些策略可以确保操作的顺序，在这种情况下，顺序非常重要。

在这会出现另外一种难以发现的bug，你会发现吗？

考虑这个调用顺序：

```js
fetchUserData( 123 );
onUserData(..);
fetchOrders( 123 );
onOrders(..);

// later

fetchOrders( 123 );
deleteOrder( 456 );
onOrders(..);
onDelete(..);
```

您是否看到`fetchOrders(..)`和`onOrders(..)`与`deleteOrder(..)`和`onDelete(..)`交织在一起?这种潜在的排序暴露了一种奇怪的情况，即状态管理的副作用。

在设置`isLatestOrder`标志和使用它来决定是否清空`users`中用户数据对象的`latestOrder` 属性之间存在时间延迟(由于回调)。在延迟期间，如果`onOrders(..)`回调触发，它可能会更改该用户的`latestOrder`引用的订单值。当`onDelete(..)`触发时，它将假定仍然需要取消`latestOrder`引用的设置。

存在bug:数据(状态)*现在可能*不同步。`latestOrder`可能没有被设置，此时它可能应该一直指向`onOrders(..)`的新订单。

这类bug最糟糕的部分是，您不会像我们处理另一个bug那样获得一个程序崩溃异常。我们只是有不正确的状态;我们的应用程序的行为被“悄悄地”破坏了。

`fetchUserData(..)`和`fetchOrders(..)`之间的顺序依赖关系相当明显，并且可以直接处理。但是`fetchOrders(..)`和`deleteOrder(..)`之间潜在的排序依赖关系就不那么明显了。这两者似乎更独立。而确保它们的顺序被保留则更加棘手，因为您事先不知道(在`fetchOrders(..)`的结果出现之前)是否真的必须强制执行排序。

是的，一旦`deleteOrder(..)`触发，您可以重新计算`isLatestOrder`标志。但现在有一个不同的问题:UI状态可能不同步。

如果您以前调用过 `hideLatestOrderDisplay()` ，你现在需要调用`showLatestOrderDisplay()`函数,但前提是一个新的`latestOrder`实际上已经被设置好了。所以你至少需要跟踪三种状态：被删除的订单是最初的“最新的”吗，是“最新的”集合吗，这两个订单不同吗？当然，这些都是可以解决的问题。但无论如何都不明显。

所有这些麻烦都是因为我们决定在代码结构中使用共享状态集上的副作用。

函数式程序员讨厌这些副作用，因为它极大地损害了我们阅读、推理、验证和最终信任代码的能力。这就是为什么他们如此认真地对待避免副作用的原则。

有多种不同的策略可以避免/修复副作用。我们将在本章后面讨论一些，其他的将在后面的章节中讨论。我可以肯定的说:带有副作用的写作通常是我们默认的写作方式，所以避免它们需要小心和有意识的努力。

## 一次就够了，谢谢

如果必须对状态进行副作用更改，对于限制潜在问题有用的一类操作是幂等性。如果值的更新是幂等的，那么数据将能够适应来自不同副作用源的多个此类更新。

如果你试图研究它，幂等性的定义可能有点令人困惑;数学家使用的含义与程序员通常使用的含义略有不同。但是，这两个观点对函数式程序员都很有用。

首先，让我们给出一个反例，它既不是数学上的幂等，也不是程序上的幂等:

```js
function updateCounter(obj) {
    if (obj.count < 10) {
        obj.count++;
        return true;
    }

    return false;
}
```

这个函数通过增加`obj.count`来通过引用修改对象。这会对这个对象产生副作用。如果多次调用`updateCounter(o)`，则`o.count`小于`10`，即程序状态每次都在变化。此外，`updateCounter(..)`的输出是一个布尔值，不适合反馈到`updateCounter(..)`的后续调用。

### 数学幂等性

从数学的角度来看，幂等性意味着一个操作，它的输出在第一次调用之后永远不会改变，如果您将该输出一次又一次地反馈到该操作中。换句话说，`foo(x)`将产生与`foo(foo(x))`和`foo(foo(foo(x)))`相同的输出。

一个典型的数学例子是`Math.abs(..)`(绝对值)。`Math.abs(-2)`等于`2`，这与`Math.abs(Math.abs(Math.abs(Math.abs(-2))))`的结果相同。其他幂等数学工具包括:

* `Math.min(..)`
* `Math.max(..)`
* `Math.round(..)`
* `Math.floor(..)`
* `Math.ceil(..)`

我们可以用同样的特征定义一些自定义数学运算:

```js
function toPower0(x) {
    return Math.pow( x, 0 );
}

function snapUp3(x) {
    return x - (x % 3) + (x % 3 > 0 && 3);
}

toPower0( 3 ) == toPower0( toPower0( 3 ) );         // true

snapUp3( 3.14 ) == snapUp3( snapUp3( 3.14 ) );      // true
```

数学形式的幂等性并不局限于数学运算。我们可以用JavaScript基本类型强制来说明这种形式的幂等性:

```js
var x = 42, y = "hello";

String( x ) === String( String( x ) );              // true

Boolean( y ) === Boolean( Boolean( y ) );           // true
```

在本文的前面，我们探讨了一种常见的FP工具，它可以实现这种形式的幂等性:

```js
identity( 3 ) === identity( identity( 3 ) );    // true
```

某些字符串操作也是自然幂等的，比如:

```js
function upper(x) {
    return x.toUpperCase();
}

function lower(x) {
    return x.toLowerCase();
}

var str = "Hello World";

upper( str ) == upper( upper( str ) );              // true

lower( str ) == lower( lower( str ) );              // true
```

我们甚至可以用幂等的方式设计更复杂的字符串格式化操作，比如:

```js
function currency(val) {
    var num = parseFloat(
        String( val ).replace( /[^\d.-]+/g, "" )
    );
    var sign = (num < 0) ? "-" : "";
    return `${sign}$${Math.abs( num ).toFixed( 2 )}`;
}

currency( -3.1 );                                   // "-$3.10"

currency( -3.1 ) == currency( currency( -3.1 ) );   // true
```

`currency(..)`说明了一种重要的技术:在某些情况下，开发人员可以采取额外的步骤对输入/输出操作进行规范化，以确保该操作在通常不会出现的情况下是幂等的。

在可能的情况下，将副作用限制为幂等操作要比不受限制的更新好得多。

### 编程幂等性

面向编程的幂等性定义类似，但不太正式。不需要 `f(x) === f(f(x))`，这种幂等性的观点就是`f(x);`导致与`f(x); f(x);`相同的程序行为;`f(x)`换句话说，在第一次调用之后的后续调用`f(x)`的结果不会改变任何东西。

这种观点更符合我们对副作用的观察，因为这种`f(..)`操作更有可能产生幂等副作用，而不一定返回幂等输出值。

这种幂等样式经常用于HTTP操作(动词)，如GET或PUT。如果HTTP REST API正确地遵循了幂等性的规范指导，那么PUT被定义为一个完全替换资源的更新操作。因此，客户机可以发送一次或多次PUT请求(使用相同的数据)，无论如何，服务器都将具有相同的结果状态。

用编程的更具体的术语来考虑这个问题，让我们检查一些副作用操作的幂等性(或缺幂等性):

```js
// 幂等性:
obj.count = 2;
a[a.length - 1] = 42;
person.name = upper( person.name );

// 无幂等性:
obj.count++;
a[a.length] = 42;
person.lastUpdated = Date.now();
```

记住:这里幂等性的概念是指每个幂等操作(如`obj.count = 2`)可以重复多次，并且在第一次更新之后不会更改程序状态。非幂等运算每次都会改变状态。

DOM更新吗？

```js
var hist = document.getElementById( "orderHistory" );

// 幂等性:
hist.innerHTML = order.historyText;

// 无幂等性:
var update = document.createTextNode( order.latestUpdate );
hist.appendChild( update );
```

这里说明的关键区别是，幂等更新替换了DOM元素的内容。DOM元素的当前状态无关紧要，因为它被无条件地覆盖。非幂等操作向元素添加内容;隐式地，DOM元素的当前状态是计算下一个状态的一部分。

以幂等的方式定义数据操作并不总是可能的，但如果可以，它肯定有助于减少在您最不期望的时候突然出现的副作用打破预期的可能性。

## 纯函数的美好

没有副作用的函数称为纯函数。纯函数在编程意义上是幂等的，因为它不会有任何副作用。考虑:

```js
function add(x,y) {
    return x + y;
}
```

所有输入(`x`和`y`)和输出(`return ..`)都是直接的;没有自由变量引用。多次调用`add(3,4)`与只调用一次没有什么区别。`add(..)`是纯的、编程风格的幂等函数。

然而，并非所有纯函数在数学意义上都是幂等的，因为它们不必返回一个适合作为它们自己的输入进行反馈的值。考虑:

```js
function calculateAverage(nums) {
    var sum = 0;
    for (let num of nums) {
        sum += num;
    }
    return sum / nums.length;
}

calculateAverage( [1,2,4,7,11,16,22] );         // 9
```

输出`9`不是一个数组，因此不能将它传递回:`calculateAverage(calculateAverage( .. ))`。

正如我们前面讨论的，纯函数*可以*引用自由变量，只要这些自由变量不是副作用。

一些例子：

```js
const PI = 3.141592;

function circleArea(radius) {
    return PI * radius * radius;
}

function cylinderVolume(radius,height) {
    return height * circleArea( radius );
}
```

`circleArea(..)`引用自由变量`PI，但它是一个常数，所以它不是一个次要原因。`cylinderVolume(..)`引用自由变量`circleArea`，这也不是一个副作用，因为这个程序实际上把它当作一个常量，引用它的函数值。这两个函数都是纯函数。


另一个例子，一个函数仍然可以是纯的，但引用自由变量是闭包:

```js
function unary(fn) {
    return function onlyOneArg(arg){
        return fn( arg );
    };
}
```

`unary(..)`本身显然是纯的——它惟一的输入是`fn`，惟一的输出是`return`函数——但是内部函数`onlyOneArg(..)`又如何呢?

它仍然是纯的，因为`fn`永远不会改变。事实上，我们对这个事实充满信心，因为从词汇上讲，这几行是唯一可能重新分配`fn`的行。

**Note:** `fn` is a reference to a function object, which is by default a mutable value. Somewhere else in the program *could*, for example, add a property to this function object, which technically "changes" the value (mutation, not reassignment). However, because we're not relying on anything about `fn` other than our ability to call it, and it's not possible to affect the callability of a function value, `fn` is still effectively unchanging for our reasoning purposes; it cannot be a side cause.

Another common way to articulate a function's purity is: **given the same input(s), it always produces the same output.** If you pass `3` to `circleArea(..)`, it will always output the same result (`28.274328`).

If a function *can* produce a different output each time it's given the same inputs, it is impure. Even if such a function always `return`s the same value, if it produces an indirect output side effect, the program state is changed each time it's called; this is impure.

Impure functions are undesirable because they make all of their calls harder to reason about. A pure function's call is perfectly predictable. When someone reading the code sees multiple `circleArea(3)` calls, they won't have to spend any extra effort to figure out what its output will be *each time*.

**Note:** An interesting thing to ponder: is the heat produced by the CPU while performing any given operation an unavoidable side effect of even the most pure functions/programs? What about just the CPU time delay as it spends time on a pure operation before it can do another one?

### Purely Relative

We have to be very careful when talking about a function being pure. JavaScript's dynamic value nature makes it all too easy to have non-obvious side causes/effects.

Consider:

```js
function rememberNumbers(nums) {
    return function caller(fn){
        return fn( nums );
    };
}

var list = [1,2,3,4,5];

var simpleList = rememberNumbers( list );
```

`simpleList(..)` looks like a pure function, as it's a reference to the inner function `caller(..)`, which just closes over the free variable `nums`. However, there's multiple ways that `simpleList(..)` can actually turn out to be impure.

First, our assertion of purity is based on the array value (referenced both by `list` and `nums`) never changing:

```js
function median(nums) {
    return (nums[0] + nums[nums.length - 1]) / 2;
}

simpleList( median );       // 3

// ..

list.push( 6 );

// ..

simpleList( median );       // 3.5
```

When we mutate the array, the `simpleList(..)` call changes its output. So, is `simpleList(..)` pure or impure? Depends on your perspective. It's pure for a given set of assumptions. It could be pure in any program that didn't have the `list.push(6)` mutation.

We could guard against this kind of impurity by altering the definition of `rememberNumbers(..)`. One approach is to duplicate the `nums` array:

```js
function rememberNumbers(nums) {
    // make a copy of the array
    nums = [...nums];

    return function caller(fn){
        return fn( nums );
    };
}
```

But an even trickier hidden side effect could be lurking:

```js
var list = [1,2,3,4,5];

// make `list[0]` be a getter with a side effect
Object.defineProperty(
    list,
    0,
    {
        get: function(){
            console.log( "[0] was accessed!" );
            return 1;
        }
    }
);

var simpleList = rememberNumbers( list );
// [0] was accessed!
```

A perhaps more robust option is to change the signature of `rememberNumbers(..)` to not receive an array in the first place, but rather the numbers as individual arguments:

```js
function rememberNumbers(...nums) {
    return function caller(fn){
        return fn( nums );
    };
}

var simpleList = rememberNumbers( ...list );
// [0] was accessed!
```

The two `...`s have the effect of copying `list` into `nums` instead of passing it by reference.

**Note:** The console message side effect here comes not from `rememberNumbers(..)` but from the `...list` spreading. So in this case, both `rememberNumbers(..)` and `simpleList(..)` are pure.

But what if the mutation is even harder to spot? Composition of a pure function with an impure function **always** produces an impure function. If we pass an impure function into the otherwise pure `simpleList(..)`, it's now impure:

```js
// yes, a silly contrived example :)
function firstValue(nums) {
    return nums[0];
}

function lastValue(nums) {
    return firstValue( nums.reverse() );
}

simpleList( lastValue );    // 5

list;                       // [1,2,3,4,5] -- OK!

simpleList( lastValue );    // 1
```

**Note:** Despite `reverse()` looking safe (like other array methods in JS) in that it returns a reversed array, it actually mutates the array rather than creating a new one.

We need a more robust definition of `rememberNumbers(..)` to guard against the `fn(..)` mutating its closed over `nums` via reference:

```js
function rememberNumbers(...nums) {
    return function caller(fn){
        // send in a copy!
        return fn( [...nums] );
    };
}
```

So is `simpleList(..)` reliably pure yet!? **Nope.** :(

We're only guarding against side effects we can control (mutating by reference). Any function we pass that has other side effects will have polluted the purity of `simpleList(..)`:

```js
simpleList( function impureIO(nums){
    console.log( nums.length );
} );
```

In fact, there's no way to define `rememberNumbers(..)` to make a perfectly pure `simpleList(..)` function.

Purity is about confidence. But we have to admit that in many cases, **any confidence we feel is actually relative to the context** of our program and what we know about it. In practice (in JavaScript) the question of function purity is not about being absolutely pure or not, but about a range of confidence in its purity.

The more pure, the better. The more effort you put into making a function pure(r), the higher your confidence will be when you read code that uses it, and that will make that part of the code more readable.

## There or Not

So far, we've defined function purity both as a function without side causes/effects and as a function that, given the same input(s), always produces the same output. These are just two different ways of looking at the same characteristics.

But a third way of looking at function purity, and perhaps the most widely accepted definition, is that a pure function has referential transparency.

Referential transparency is the assertion that a function call could be replaced by its output value, and the overall program behavior wouldn't change. In other words, it would be impossible to tell from the program's execution whether the function call was made or its return value was inlined in place of the function call.

From the perspective of referential transparency, both of these programs have identical behavior as they are built with pure functions:

```js
function calculateAverage(nums) {
    var sum = 0;
    for (let num of nums) {
        sum += num;
    }
    return sum / nums.length;
}

var numbers = [1,2,4,7,11,16,22];

var avg = calculateAverage( numbers );

console.log( "The average is:", avg );      // The average is: 9
```

```js
function calculateAverage(nums) {
    var sum = 0;
    for (let num of nums) {
        sum += num;
    }
    return sum / nums.length;
}

var numbers = [1,2,4,7,11,16,22];

var avg = 9;

console.log( "The average is:", avg );      // The average is: 9
```

The only difference between these two snippets is that in the latter one, we skipped the `calculateAverage(nums)` call and just inlined its output (`9`). Since the rest of the program behaves identically, `calculateAverage(..)` has referential transparency, and is thus a pure function.

### Mentally Transparent

The notion that a referentially transparent pure function *can be* replaced with its output does not mean that it *should literally be* replaced. Far from it.

The reasons we build functions into our programs instead of using pre-computed magic constants are not just about responding to changing data, but also about readability with proper abstractions. The function call to calculate the average of that list of numbers makes that part of the program more readable than the line that just assigns the value explicitly. It tells the story to the reader of where `avg` comes from, what it means, and so on.

What we're really suggesting with referential transparency is that as you're reading a program, once you've mentally computed what a pure function call's output is, you no longer need to think about what that exact function call is doing when you see it in code, especially if it appears multiple times.

That result becomes kinda like a mental `const` declaration, which as you're reading you can transparently swap in and not spend any more mental energy working out.

Hopefully the importance of this characteristic of a pure function is obvious. We're trying to make our programs more readable. One way we can do that is to give the reader less work, by providing assistance to skip over the unnecessary stuff so they can focus on the important stuff.

The reader shouldn't need to keep re-computing some outcome that isn't going to change (and doesn't need to). If you define a pure function with referential transparency, the reader won't have to.

### Not So Transparent?

What about a function that has a side effect, but this side effect isn't ever observed or relied upon anywhere else in the program? Does that function still have referential transparency?

Here's one:

```js
function calculateAverage(nums) {
    sum = 0;
    for (let num of nums) {
        sum += num;
    }
    return sum / nums.length;
}

var sum;
var numbers = [1,2,4,7,11,16,22];

var avg = calculateAverage( numbers );
```

Did you spot it?

`sum` is an outer free variable that `calculateAverage(..)` uses to do its work. But, every time we call `calculateAverage(..)` with the same list, we're going to get `9` as the output. And this program couldn't be distinguished in terms of behavior from a program that replaced the `calculateAverage(nums)` call with the value `9`. No other part of the program cares about the `sum` variable, so it's an unobserved side effect.

Is a side cause/effect that's unobserved like this tree:

> If a tree falls in the forest, but no one is around to hear it, does it still make a sound?

By the narrowest definition of referential transparency, I think you'd have to say `calculateAverage(..)` is still a pure function. However, because we're trying to avoid a strictly academic approach in favor of balancing it with pragmatism, I also think this conclusion needs more perspective. Let's explore.

#### Performance Effects

You'll generally find these kind of side-effects-that-go-unobserved being used to optimize the performance of an operation. For example:

```js
var cache = [];

function specialNumber(n) {
    // if we've already calculated this special number,
    // skip the work and just return it from the cache
    if (cache[n] !== undefined) {
        return cache[n];
    }

    var x = 1, y = 1;

    for (let i = 1; i <= n; i++) {
        x += i % 2;
        y += i % 3;
    }

    cache[n] = (x * y) / (n + 1);

    return cache[n];
}

specialNumber( 6 );             // 4
specialNumber( 42 );            // 22
specialNumber( 1E6 );           // 500001
specialNumber( 987654321 );     // 493827162
```

This silly `specialNumber(..)` algorithm is deterministic and thus pure from the definition that it always gives the same output for the same input. It's also pure from the referential transparency perspective -- replace any call to `specialNumber(42)` with `22` and the end result of the program is the same.

However, the function has to do quite a bit of work to calculate some of the bigger numbers, especially the `987654321` input. If we needed to get that particular special number multiple times throughout our program, the `cache`ing of the result means that subsequent calls are far more efficient.

Don't be so quick to assume that you could just run the calculation `specialNumber(987654321)` once and manually stick that result in some variable/constant. Programs are often highly modularized and globally accessible scopes are not usually the way you want to share state between those independent pieces. Having `specialNumber(..)` do its own caching (even though it happens to be using a global variable to do so!) is a more preferable abstraction of that state sharing.

The point is that if `specialNumber(..)` is the only part of the program that accesses and updates the `cache` side cause/effect, the referential transparency perspective observably holds true, and this might be seen as an acceptable pragmatic "cheat" of the pure function ideal.

But should it?

Typically, this sort of performance optimization side effecting is done by hiding the caching of results so they *cannot* be observed by any other part of the program. This process is referred to as memoization. I always think of that word as "memorization"; I have no idea if that's even remotely where it comes from, but it certainly helps me understand the concept better.

Consider:

```js
var specialNumber = (function memoization(){
    var cache = [];

    return function specialNumber(n){
        // if we've already calculated this special number,
        // skip the work and just return it from the cache
        if (cache[n] !== undefined) {
            return cache[n];
        }

        var x = 1, y = 1;

        for (let i = 1; i <= n; i++) {
            x += i % 2;
            y += i % 3;
        }

        cache[n] = (x * y) / (n + 1);

        return cache[n];
    };
})();
```

We've contained the `cache` side causes/effects of `specialNumber(..)` inside the scope of the `memoization()` IIFE, so now we're sure that no other parts of the program *can* observe them, not just that they *don't* observe them.

That last sentence may seem like a subtle point, but actually I think it might be **the most important point of the entire chapter**. Read it again.

Recall this philosophical musing:

> If a tree falls in the forest, but no one is around to hear it, does it still make a sound?

Going with the metaphor, what I'm getting at is: whether the sound is made or not, it would be better if we never create a scenario where the tree can fall without us being around; we'll always hear the sound when a tree falls.

The purpose of reducing side causes/effects is not per se to have a program where they aren't observed, but to design a program where fewer of them are possible, because this makes the code easier to reason about. A program with side causes/effects that *just happen* to not be observed is not nearly as effective in this goal as a program that *cannot* observe them.

If side causes/effects can happen, the writer and reader must mentally juggle them. Make it so they can't happen, and both writer and reader will find more confidence over what can and cannot happen in any part.

## Purifying

The first best option in writing functions is that you design them from the beginning to be pure. But you'll spend plenty of time maintaining existing code, where those kinds of decisions were already made; you'll run across a lot of impure functions.

If possible, refactor the impure function to be pure. Sometimes you can just shift the side effects out of a function to the part of the program where the call of that function happens. The side effect wasn't eliminated, but it was made more obvious by showing up at the call-site.

Consider this trivial example:

```js
function addMaxNum(arr) {
    var maxNum = Math.max( ...arr );
    arr.push( maxNum + 1 );
}

var nums = [4,2,7,3];

addMaxNum( nums );

nums;       // [4,2,7,3,8]
```

The `nums` array needs to be modified, but we don't have to obscure that side effect by containing it in `addMaxNum(..)`. Let's move the `push(..)` mutation out, so that `addMaxNum(..)` becomes a pure function, and the side effect is now more obvious:

```js
function addMaxNum(arr) {
    var maxNum = Math.max( ...arr );
    return maxNum + 1;
}

var nums = [4,2,7,3];

nums.push(
    addMaxNum( nums )
);

nums;       // [4,2,7,3,8]
```

**Note:** Another technique for this kind of task could be to use an immutable data structure, which we cover in the next chapter.

But what can you do if you have an impure function where the refactoring is not as easy?

You need to figure what kind of side causes/effects the function has. It may be that the side causes/effects come variously from lexical free variables, mutations-by-reference, or even `this` binding. We'll look at approaches that address each of these scenarios.

### Containing Effects

If the nature of the concerned side causes/effects is with lexical free variables, and you have the option to modify the surrounding code, you can encapsulate them using scope.

Recall:

```js
var users = {};

function fetchUserData(userId) {
    ajax( `http://some.api/user/${userId}`, function onUserData(user){
        users[userId] = user;
    } );
}
```

One option for purifying this code is to create a wrapper around both the variable and the impure function. Essentially, the wrapper has to receive as input "the entire universe" of state it can operate on.

```js
function safer_fetchUserData(userId,users) {
    // simple, naive ES6+ shallow object copy, could also
    // be done w/ various libs or frameworks
    users = Object.assign( {}, users );

    fetchUserData( userId );

    // return the copied state
    return users;


    // ***********************

    // original untouched impure function:
    function fetchUserData(userId) {
        ajax(
            `http://some.api/user/${userId}`,
            function onUserData(user){
                users[userId] = user;
            }
        );
    }
}
```

**Warning:** `safer_fetchUserData(..)` is *more* pure, but is not strictly pure in that it still relies on the I/O of making an Ajax call. There's no getting around the fact that an Ajax call is an impure side effect, so we'll just leave that detail unaddressed.

Both `userId` and `users` are input for the original `fetchUserData`, and `users` is also output. The `safer_fetchUserData(..)` takes both of these inputs, and returns `users`. To make sure we're not creating a side effect on the outside when `users` is mutated, we make a local copy of `users`.

This technique has limited usefulness mostly because if you cannot modify a function itself to be pure, you're not that likely to be able to modify its surrounding code either. However, it's helpful to explore it if possible, as it's the simplest of our fixes.

Regardless of whether this will be a practical technique for refactoring to pure functions, the more important take-away is that function purity only need be skin deep. That is, the **purity of a function is judged from the outside**, regardless of what goes on inside. As long as a function's usage behaves pure, it is pure. Inside a pure function, impure techniques can be used -- in moderation! -- for a variety of reasons, including most commonly, for performance. It's not necessarily, as they say, "turtles all the way down".

Be very careful, though. Any part of the program that's impure, even if it's wrapped with and only ever used via a pure function, is a potential source of bugs and confusion for readers of the code. The overall goal is to reduce side effects wherever possible, not just hide them.

### Covering Up Effects

Many times you will be unable to modify the code to encapsulate the lexical free variables inside the scope of a wrapper function. For example, the impure function may be in a third-party library file that you do not control, containing something like:

```js
var nums = [];
var smallCount = 0;
var largeCount = 0;

function generateMoreRandoms(count) {
    for (let i = 0; i < count; i++) {
        let num = Math.random();

        if (num >= 0.5) {
            largeCount++;
        }
        else {
            smallCount++;
        }

        nums.push( num );
    }
}
```

The brute-force strategy to *quarantine* the side causes/effects when using this utility in the rest of our program is to create an interface function that performs the following steps:

1. Capture the to-be-affected current states
2. Set initial input states
3. Run the impure function
4. Capture the side effect states
5. Restore the original states
6. Return the captured side effect states

```js
function safer_generateMoreRandoms(count,initial) {
    // (1) Save original state
    var orig = {
        nums,
        smallCount,
        largeCount
    };

    // (2) Set up initial pre-side effects state
    nums = [...initial.nums];
    smallCount = initial.smallCount;
    largeCount = initial.largeCount;

    // (3) Beware impurity!
    generateMoreRandoms( count );

    // (4) Capture side effect state
    var sides = {
        nums,
        smallCount,
        largeCount
    };

    // (5) Restore original state
    nums = orig.nums;
    smallCount = orig.smallCount;
    largeCount = orig.largeCount;

    // (6) Expose side effect state directly as output
    return sides;
}
```

And to use `safer_generateMoreRandoms(..)`:

```js
var initialStates = {
    nums: [0.3, 0.4, 0.5],
    smallCount: 2,
    largeCount: 1
};

safer_generateMoreRandoms( 5, initialStates );
// { nums: [0.3,0.4,0.5,0.8510024448959794,0.04206799238...

nums;           // []
smallCount;     // 0
largeCount;     // 0
```

That's a lot of manual work to avoid a few side causes/effects; it'd be a lot easier if we just didn't have them in the first place. But if we have no choice, this extra effort is well worth it to avoid surprises in our programs.

**Note:** This technique really only works when you're dealing with synchronous code. Asynchronous code can't reliably be managed with this approach because it can't prevent surprises if other parts of the program access/modify the state variables in the interim.

### Evading Effects

When the nature of the side effect to be dealt with is a mutation of a direct input value (object, array, etc.) via reference, we can again create an interface function to interact with instead of the original impure function.

Consider:

```js
function handleInactiveUsers(userList,dateCutoff) {
    for (let i = 0; i < userList.length; i++) {
        if (userList[i].lastLogin == null) {
            // remove the user from the list
            userList.splice( i, 1 );
            i--;
        }
        else if (userList[i].lastLogin < dateCutoff) {
            userList[i].inactive = true;
        }
    }
}
```

Both the `userList` array itself, plus the objects in it, are mutated. One strategy to protect against these side effects is to do a deep (well, just not shallow) copy first:

```js
function safer_handleInactiveUsers(userList,dateCutoff) {
    // make a copy of both the list and its user objects
    let copiedUserList = userList.map( function mapper(user){
        // copy a `user` object
        return Object.assign( {}, user );
    } );

    // call the original function with the copy
    handleInactiveUsers( copiedUserList, dateCutoff );

    // expose the mutated list as a direct output
    return copiedUserList;
}
```

The success of this technique will be dependent on the thoroughness of the *copy* you make of the value. Using `[...userList]` would not work here, since that only creates a shallow copy of the `userList` array itself. Each element of the array is an object that needs to be copied, so we need to take extra care. Of course, if those objects have objects inside them (they might!), the copying needs to be even more robust.

### `this` Revisited

Another variation of the via-reference side cause/effect is with `this`-aware functions having `this` as an implicit input. See [Chapter 2, "What's This"](ch2.md/#whats-this) for more info on why the `this` keyword is problematic for FPers.

Consider:

```js
var ids = {
    prefix: "_",
    generate() {
        return this.prefix + Math.random();
    }
};
```

Our strategy is similar to the previous section's discussion: create an interface function that forces the `generate()` function to use a predictable `this` context:

```js
function safer_generate(context) {
    return ids.generate.call( context );
}

// *********************

safer_generate( { prefix: "foo" } );
// "foo0.8988802158307285"
```

These strategies are in no way fool-proof; the safest protection against side causes/effects is to not do them. But if you're trying to improve the readability and confidence level of your program, reducing the side causes/effects wherever possible is a huge step forward.

Essentially, we're not really eliminating side causes/effects, but rather containing and limiting them, so that more of our code is verifiable and reliable. If we later run into program bugs, we know that the parts of our code still using side causes/effects are the most likely culprits.

## Summary

Side effects are harmful to code readability and quality because they make your code much harder to understand. Side effects are also one of the most common *causes* of bugs in programs, because juggling them is hard. Idempotence is a strategy for restricting side effects by essentially creating one-time-only operations.

Pure functions are how we best avoid side effects. A pure function is one that always returns the same output given the same input, and has no side causes or side effects. Referential transparency further states that -- more as a mental exercise than a literal action -- a pure function's call could be replaced with its output and the program would not have altered behavior.

Refactoring an impure function to be pure is the preferred option. But if that's not possible, try encapsulating the side causes/effects, or creating a pure interface against them.

No program can be entirely free of side effects. But prefer pure functions in as many places as that's practical. Collect impure functions side effects together as much as possible, so that it's easier to identify and audit these most likely culprits of bugs when they arise.
