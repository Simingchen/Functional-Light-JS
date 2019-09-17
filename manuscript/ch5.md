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

注:`fn`是对函数对象的引用，默认情况下，函数对象是一个可变值。例如，在程序*could*的其他地方，向这个函数对象添加一个属性，该属性在技术上“改变”了值(突变，而不是重新分配)。然而，因为我们不依赖于`fn`的任何东西，除了我们调用它的能力，而且它不可能影响函数值的可调用性，对于我们的推理目的来说，`fn`仍然是有效不变的;它不可能是一个次要原因。

表达函数纯度的另一种常见方法是:给定相同的输入，它总是产生相同的输出。如果您将`3`传递给`circleArea(..)`，它将始终输出相同的结果(`28.274328`)。

如果一个函数在每次给定相同的输入时都能产生不同的输出，那么它就是不纯的。即使这样一个函数总是`return`相同的值，如果它产生间接输出副作用，程序状态每次被调用时都会改变;这是不纯的。

不纯函数是不受欢迎的，因为它们使所有调用都难以推理。一个纯函数的调用是完全可预测的。当阅读代码的人看到多个`circleArea(3)`调用时，他们就不需要花费额外的精力来计算每次的输出是什么了。

注:值得思考的一件有趣的事情是:CPU在执行任何给定操作时产生的热量，即使是最纯粹的函数/程序，也会产生不可避免的副作用吗?那么CPU的时间延迟呢，因为它在一个纯操作上花费了时间，然后它才能执行另一个操作吗？

### 相对的纯

当我们讨论纯函数时，我们必须非常小心。JavaScript的动态值特性使得它很容易产生不明显的副作用。

考虑:

```js
function rememberNumbers(nums) {
    return function caller(fn){
        return fn( nums );
    };
}

var list = [1,2,3,4,5];

var simpleList = rememberNumbers( list );
```

`simpleList(..)`看起来像一个纯函数，因为它是对内部函数`caller(..)`的引用，后者只是在自由变量`nums`上关闭。然而，`simpleList(..)`实际上有多种方法是不纯的。

首先，我们对纯度的判断是基于数组值(list和nums都引用了)不变:

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

当我们改变数组时，`simpleList(..)`调用会改变它的输出。那么，`simpleList(..)`是纯的还是不纯的呢?这取决于你的观点。对于给定的一组假设它是纯的。它可以在任何没有`list.push(6)`突变的程序中是纯的。

我们可以通过改变`rememberNumbers(..)`的定义来防止这种不纯。一种方法是复制`nums`数组:

```js
function rememberNumbers(nums) {
    // 复制数组
    nums = [...nums];

    return function caller(fn){
        return fn( nums );
    };
}
```

但一个更为棘手的隐藏副作用可能正在潜伏:

```js
var list = [1,2,3,4,5];

// 使`list[0]`成为具有副作用的getter
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

一个可能更健壮的选项是改变`rememberNumbers(..)`的签名，首先不接收数组，而是将数字作为单独的参数:

```js
function rememberNumbers(...nums) {
    return function caller(fn){
        return fn( nums );
    };
}

var simpleList = rememberNumbers( ...list );
// [0] was accessed!
```

两个`...`的作用是将`list`复制到`nums`中，而不是通过引用传递它。

注:这里控制台消息的副作用不是来自于`rememberNumbers(..)`，而是来自于`...list`的扩展。因此，在这种情况下，`rememberNumbers(..)` 和`simpleList(..)`都是纯的。

但如果这种突变更难发现呢?纯函数与非纯函数的复合总是生成非纯函数。如果我们将一个不纯函数传递给纯的`simpleList(..)`，它现在是不纯的:

```js
// 看，一个愚蠢的人为的例子 :)
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

注:尽管`reverse()`看起来很安全(就像JS中的其他数组方法一样)，因为它返回一个反向数组，但它实际上是在修改数组，而不是创建一个新的数组。

我们需要对`rememberNumbers(..)`有一个更强有力的定义，以防止`fn(..)`通过引用将它的关闭值更改为`nums`:

```js
function rememberNumbers(...nums) {
    return function caller(fn){
        // 传递一个拷贝值!
        return fn( [...nums] );
    };
}
```

`simpleList(..)`也是如此？不 :(

我们只是在预防我们可以控制的副作用(通过引用进行变异)。我们传递的任何有其他副作用的函数都会污染`simpleList(..)`的纯度:

```js
simpleList( function impureIO(nums){
    console.log( nums.length );
} );
```

事实上，没有办法定义`rememberNumbers(..)`来生成一个完全纯的`simpleList(..)`函数。

纯就是自信。但我们必须承认，在很多情况下，我们所感受到的任何自信，实际上都与我们所处的环境以及我们对它的了解有关。在实践中(在JavaScript中)，函数纯度的问题不在于是否绝对纯净，而在于对函数纯度的一系列信心。

越纯净越好。您在使函数 pure(r)方面投入的精力越多，当您阅读使用它的代码时，您的信心就会越高，这将使代码的这一部分更具可读性。

## 有或者无

到目前为止，我们已经将函数纯度定义为一个没有副作用的函数，以及一个给定相同输入始终产生相同输出的函数。这只是看待相同特征的两种不同方式。

引用透明性是这样一种断言，即函数调用可以被它的输出值替换，而整个程序行为不会发生变化。换句话说，不可能从程序的执行中看出函数调用是被执行的，还是它的返回值内联在函数调用的位置上。

从引用透明性的角度来看，这两个程序都具有相同的行为，因为它们都是用纯函数构建的:

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

这两个代码段之间唯一的区别是，在后一个代码段中，我们跳过了`calculateAverage(nums)`调用，而只是将它的输出内联起来(`9`)。由于程序的其他部分行为相同，`calculateAverage(..)`具有引用透明性，因此是一个纯函数。

### 精神上的透明清晰

引用透明的纯函数*可以用其输出替换*的概念并不意味着它*应该按字面意思替换。远非如此。

我们在程序中构建函数而不是使用预先计算好的神奇常量，这不仅仅是为了响应不断变化的数据，还与适当抽象的可读性有关。计算该数字列表平均值的函数调用使程序的这一部分比仅显式赋值的行更具可读性。它向读者讲述了`avg`的由来、含义等等。

我们真正表明引用透明性是,当你阅读程序,一旦你精神上计算纯函数调用的输出是什么,你不再需要思考,确切的函数调用是做什么当你看到它的代码,特别是如果它出现很多次了。

这个结果就像一个精神上的`const`声明，当你读它的时候，你可以透明地切换进去，而不需要花费更多的精神能量。

希望这个纯函数特征的重要性是显而易见的。我们正努力使我们的程序更具可读性。我们可以做的一种方式是减少读者的工作量，通过提供帮助来跳过不必要的内容，这样他们就可以专注于重要的内容。

读者不应该不断地重新计算一些不会改变(也不需要)的结果。如果定义了具有引用透明性的纯函数，读者就不必这样做了。

### 不透明的?

如果一个函数有副作用，但是这个副作用在程序的任何其他地方都没有被观察到或依赖，那该怎么办?该函数仍然具有引用透明性吗?

这有个例子:

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

你发现了吗?

`sum`是一个外部自由变量， `calculateAverage(..)`使用它来完成工作。但是，每次调用具有相同列表的 `calculateAverage(..)`时，我们将得到输出`9`。这个程序在行为方面无法与用值`9`替换`calculateAverage(nums)` 调用的程序区别开来。程序的其他部分不关心`sum`变量，所以这是一个未观察到的副作用。

是否像这棵树一样无法观察到的副作用:

> 如果一棵树倒在森林里，但周围没有人听见，它还会发出声音吗（可以认为他发出声音了吗）？

根据对引用透明性最狭义的定义，我认为您不得不说`calculateAverage(..)`仍然是一个纯函数。然而，因为我们试图避免一个严格的学术方法，以平衡它与实用主义，我也认为这个结论需要更多的视角。让我们探索。

#### 性能影响

您通常会发现这些副作用，这些副作用被用于优化操作的性能。例如:

```js
var cache = [];

function specialNumber(n) {
    // 如果我们已经计算过这个特殊的数，
    // 跳过，从缓存中返回它
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

这样没头脑的`specialNumber(..)`算法是确定性的，因此从定义上看，它总是为相同的输入提供相同的输出。从引用透明性的角度来看，它也是纯的——将任何对`specialNumber(42)`的调用替换为 `22`，程序的最终结果是相同的。

然而，该函数必须做相当多的工作来计算一些较大的数字，特别是`987654321`输入。如果我们需要在整个程序中多次获得这个特定的特殊数字，结果的“缓存”意味着后续调用的效率要高得多。

不要急于假设您可以只运行一次计算`specialNumber(987654321)`，然后手动将结果粘贴到某个变量/常量中。程序通常是高度模块化的，全局可访问范围通常不是您希望在这些独立部分之间共享状态的方式。让`specialNumber(..)`执行自己的缓存(即使它恰好使用全局变量来执行缓存!)是对状态共享的更可取的抽象。

关键是，如果`specialNumber(..)`是程序中唯一访问和更新“缓存”端因果关系的部分，那么引用透明透视图显然是正确的，这可能被视为对纯函数理想的一种可接受的实用“欺骗”。

但应该吗?

通常，这种性能优化的副作用是通过隐藏结果的缓存来实现的，这样它们就“不能”被程序的任何其他部分观察到。这个过程称为记忆。我一直认为这个词是“memorization”;我不知道它的来源，但它确实帮助我更好地理解这个概念。

考虑:

```js
var specialNumber = (function memoization(){
    var cache = [];

    return function specialNumber(n){
        // 如果我们已经计算过这个特殊的数，
    	// 跳过，从缓存中返回它
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

我们已经将`specialNumber(..)`的`cache`副作用包含在`memoization()`IIFE（立即执行函数表达式）的范围内，所以现在我们确定程序的其他部分*不能*干扰它们，而不仅仅是它们*不*干扰它们。

最后这句话似乎是一个微妙的观点，但实际上我认为它可能是**整章最重要的一点**。可以再读一遍。

回想一下这个哲学思考:

> 如果一棵树倒在森林里，但周围没有人听到，它还会发出声音吗（可以认为他发出声音了吗）?

用这个比喻，我想说的是:不管声音有没有发出来，如果我们不创造一个场景，让树在没有我们的情况下倒下，那就更好了;当树倒下时，我们总是能听到声音。

减少副作用的目的本身并不是要让一个程序不被观察到，而是要设计一个可以减少副作用的程序，因为这使代码更容易推理。在这个目标中，一个带有“恰好”没有被观察到的“副作用”的程序远不如一个“不能”观察到它们的程序有效。

如果有可能产生副作用，作者和读者必须在心理上同时处理它们。让它们不可能发生，这样作家和读者都会对任何部分能发生和不能发生的事情有更多的信心。

## 重构不纯函数

编写函数的第一个最佳选择是从头开始设计函数。但是您将花费大量的时间来维护现有的代码，这些类型的决策已经做出;你会遇到很多不纯函数。

如果可能，将不纯函数重构为纯函数。有时，您可以将副作用从函数转移到程序中函数调用发生的部分。副作用并没有被消除，但是在调用点出现的副作用更加明显，更容易发现与理解。

考虑这个简单的例子:

```js
function addMaxNum(arr) {
    var maxNum = Math.max( ...arr );
    arr.push( maxNum + 1 );
}

var nums = [4,2,7,3];

addMaxNum( nums );

nums;       // [4,2,7,3,8]
```

需要修改`nums`数组，但我们不必通过将它包含在`addMaxNum(..)`中来掩盖这种副作用。让我们将`push(..)`突变移出，这样`addMaxNum(..)`就变成了一个纯函数，副作用现在更明显了:

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

注:这类任务的另一种技术可能是使用不可变的数据结构，我们将在下一章中讨论。

但是，如果您有一个不纯的函数，而重构并不容易，那么您可以做什么呢?

你需要弄清楚函数有什么样的副作用。这可能是因为副作用来自不同的词汇自由变量，引用的变异，甚至`this`绑定。我们将研究处理这些场景的方法。

### 包含影响

如果副作用的问题性质是词法自由变量，并且您可以选择修改周围的代码，那么您可以使用作用域封装它们。

回顾:

```js
var users = {};

function fetchUserData(userId) {
    ajax( `http://some.api/user/${userId}`, function onUserData(user){
        users[userId] = user;
    } );
}
```

重构这段代码的一个选项是在变量和非纯函数周围创建一个包装器。本质上，包装器必须接收它可以操作的状态的“所有引用参数”作为输入。

```js
function safer_fetchUserData(userId,users) {
    // 简单、朴素的ES6+浅对象复制，也可以
    // 使用各种库或框架
    users = Object.assign( {}, users );

    fetchUserData( userId );

    // 返回复制的状态
    return users;


    // ***********************

    // 原始未变动的不纯函数：
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

警告: `safer_fetchUserData(..)`更纯粹一些，但并不完全纯粹，因为它仍然依赖于发出Ajax调用的I/O操作。Ajax调用是一个不纯的副作用，这是无法回避的事实，因此我们将不处理这个细节。

 `userId` 和`users` 都是原始`fetchUserData`的输入，`users`也是输出。 `safer_fetchUserData(..)`接受这两个输入，并返回 `users`。为了确保 `users`发生突变时不会对外部产生副作用，我们创建了`users`的本地副本。

这种技术的用处有限，主要是因为如果不能将函数本身修改为纯函数，那么也不太可能修改其周围的代码。然而，如果可能的话，研究它是有帮助的，因为它是我们的修复中最简单的。

无论这是否是重构纯函数的实用技术，更重要的是函数的纯性只需要表面功夫。也就是说，函数的**纯度是从外部**判断的，而不管函数内部发生了什么。只要函数的使用行为是纯的，它就是纯的。在纯函数内部，可以使用不纯的技术——适度使用!——出于各种各样的原因，包括最常见的表现原因。正如他们所说，这并不一定是“turtles all the way down”原则（总是惊人的相似）。

不过要非常小心。程序中任何不纯的部分，即使是用纯函数封装的，并且只通过纯函数使用，也会给代码的读者带来潜在的bug和混淆。总的目标是尽可能减少副作用，而不仅仅是隐藏它们。

### 掩盖副作用

很多时候，您将无法修改代码来将词法自由变量封装在包装器函数的范围内。例如，不纯函数可能位于您无法控制的第三方库文件中，其中包含如下内容:

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

当我们在程序使用这个工具时，“隔离”副作用的暴力策略是创建一个执行以下步骤的接口函数:

1. 捕获要受影响的当前状态
2. 设置初始输入状态
3. 运行非纯函数
4. 捕捉副作用状态
5. 恢复初始状态
6. 返回捕获的副作用状态

```js
function safer_generateMoreRandoms(count,initial) {
    // (1) 保存原始状态
    var orig = {
        nums,
        smallCount,
        largeCount
    };

    // (2) 设置初始预副作用状态
    nums = [...initial.nums];
    smallCount = initial.smallCount;
    largeCount = initial.largeCount;

    // (3) 当心不纯性!
    generateMoreRandoms( count );

    // (4) 俘获副作用状态
    var sides = {
        nums,
        smallCount,
        largeCount
    };

    // (5) 恢复初始状态
    nums = orig.nums;
    smallCount = orig.smallCount;
    largeCount = orig.largeCount;

    // (6) 直接将副作用状态显示为输出
    return sides;
}
```

然后使用 `safer_generateMoreRandoms(..)`:

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

这需要大量的手工工作来避免一些副作用;如果我们一开始就没有它们，事情会简单得多。但是如果我们别无选择，那么为了避免程序中出现意外，这种额外的努力是非常值得的。

注意:这项技术只在处理同步代码时才有效。异步代码不能用这种方法可靠地进行管理，因为如果程序的其他部分在此期间访问/修改状态变量，它就不能防止意外。

### 规避效应

当要处理的副作用的本质是通过引用直接输入值(对象、数组等)的突变时，我们可以再次创建一个接口函数来与之交互，而不是原始的不纯函数。

考虑:

```js
function handleInactiveUsers(userList,dateCutoff) {
    for (let i = 0; i < userList.length; i++) {
        if (userList[i].lastLogin == null) {
            // 从列表中删除用户
            userList.splice( i, 1 );
            i--;
        }
        else if (userList[i].lastLogin < dateCutoff) {
            userList[i].inactive = true;
        }
    }
}
```

`userList`数组本身以及其中的对象都发生了变化。防止这些副作用的一个策略是先做一个深度的(好吧，只是不要做浅的)复制:

```js
function safer_handleInactiveUsers(userList,dateCutoff) {
    // 复制列表及其用户对象
    let copiedUserList = userList.map( function mapper(user){
        // 复制一个`user`对象
        return Object.assign( {}, user );
    } );

    // 使用副本调用原始函数
    handleInactiveUsers( copiedUserList, dateCutoff );

    // 将修改后的列表作为直接输出公开
    return copiedUserList;
}
```

这项技术的成功将取决于您对值的“复制”是否彻底。使用`[...userList]` 这里不能实现，因为这只创建了一个浅拷贝的`userList`数组本身。数组中的每个元素都是一个需要复制的对象，因此我们需要格外小心。当然，如果这些对象中有对象(可能有!)，那么复制需要更加健壮的方法。

### 重新审视`this`

另一个导致副作用的原因是函数将`this`作为隐式输入。参见[第2章，"This指的是什么"](ch2.md/#whats-this)了解更多关于为什么`this`关键字的信息。

考虑:

```js
var ids = {
    prefix: "_",
    generate() {
        return this.prefix + Math.random();
    }
};
```

我们的策略类似于上一节的讨论:创建一个接口函数，强制`generate()`函数使用一个可预测的`this`上下文:

```js
function safer_generate(context) {
    return ids.generate.call( context );
}

// *********************

safer_generate( { prefix: "foo" } );
// "foo0.8988802158307285"
```

这些策略绝不是万无一失的;对副作用最安全的保护就是不去做。但是，如果您试图提高程序的可读性和可信度，那么尽可能减少副作用是一个巨大的进步。

本质上，我们并没有真正消除副作用，而是包含并限制它们，这样我们的代码中就有更多是可验证和可靠的。如果我们后来遇到程序错误，我们知道代码中仍然使用副作用的部分最有可能是罪魁祸首。

## 总结

副作用对代码的可读性和质量是有害的，因为它们使代码更难理解。副作用也是程序中bug最常见的“原因”之一，因为处理它们很困难。幂等性本质上是通过创建一次操作来限制副作用的策略。

纯功能是我们避免副作用的最佳方式。一个纯函数总是在给定相同输入的情况下返回相同的输出，并且没有副作用。引用透明性进一步指出，纯函数的调用可以用它的输出替换，程序不会改变行为，这更像是一种脑力练习，而不是文字操作。

将不纯函数重构为纯函数是首选。但是如果这是不可能的，尝试封装含有副作用的函数，或者创建一个纯接口来处理它们。

没有一个程序可以完全没有副作用。但在实际应用中更喜欢纯函数。尽可能多地收集不纯函数的副作用，以便在出现bug时更容易识别和审计这些最有可能的罪魁祸首。
