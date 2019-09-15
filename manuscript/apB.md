# 附言 B: 卑微的monoid（单子）

从这个附录我开始承认:在开始编写这个附录之前，我对单子是什么并不了解。我们犯了很多错误才了解到。如果您不相信我的话，请访问[这本书在github](https://github.com/getify/Functional-Light-JS)查看这个附录的提交历史!

我在书中包含了单子的主题，因为它是每个开发人员在学习FP时都会遇到的过程的一部分，正如我在本书中所写的那样。

我们基本上是以简短地浏览一下单子来结束这本书，而大多数FP文献几乎都是以单子开始的!在我的“轻量函数”编程中，我没有遇到需要显式地按照单子进行思考的情况，所以这就是为什么本文比主体核心更有价值。但这并不是说单子没有用处或不流行——它们非常流行。

在JavaScript 函数编程世界里有一个笑话，几乎每个人都必须编写自己的教程或博客文章来介绍单子是什么，就像单独编写单子是一种仪式一样。多年来，单子被各种各样地描述为墨西哥卷饼、洋葱和其他各种古怪的概念抽象。我希望这里不要发生那种愚蠢的事情!

> 单子说白了不过就是自函子范畴上的一个幺半群而已

我们以这句引言作为序言，所以回到这里似乎是合适的。但不，我们不会讨论单子，内函子，或范畴理论。这句话不仅羞涩难懂，而且毫无帮助。

我希望你们从这次讨论中得到的是不要再害怕‘monad’这个词或者这个概念——我已经害怕了很多年了!——当你看到他们的时候能够认出他们。你可能，只是可能，偶尔会用到它们。

## 类型

FP中有一个很大的领域是我们在整本书中都没有涉及的:类型理论。我不打算深入研究类型理论，因为坦白地说，我没有资格这么做。即使我做了，你也不会感激的。

但我要说的是monad基本上是一种值类型。

数字`42`有一个值类型(数字!)，它带来了我们所依赖的某些特性和功能。字符串`42`可能看起来非常相似，但是在我们的程序中它有不同的用途。

在面向对象编程中，当您有一组数据(甚至是单个离散值)，并且您有一些想要与之绑定的行为时，您将创建一个对象/类来表示该“类型”。实例就是该类型的成员。这种实践通常被称为“数据结构”。

我将在这里宽松地使用数据结构的概念，我们可能会发现，在程序中为某个值定义一组行为和约束，并将它们与该值捆绑到一个抽象中，是非常有用的。这样，当我们在程序中处理一个或多个这样的值时，它们的行为是免费的，并且会使处理它们更加方便。所谓方便，我的意思是对代码的读者来说更具有声明性和可接近性!

monad是一种数据结构。这是一个类型。它是一组专门设计用来使使用值的工作变得可预测的行为。

回想一下在[第9章，我们讨论了函数变量](ch9.md/#a-word-functors):一个值和一个类似于map的实用程序，用于对其所有构成数据成员执行操作。monad（单子）是包含一些附加行为的函数变量。

## 零散接口

实际上，monad并不是一种单一的数据类型，它更像是一组相关的数据类型。它是一种根据不同值的需要实现不同的接口。每个实现都是不同类型的单子。

例如，您可能会读到关于"Identity Monad"、"IO Monad"、"Maybe Monad"、"Either Monad"或其他各种Monad。它们都定义了基本的单子行为，但是它根据每种单子类型的用例扩展或覆盖了交互。

它不仅仅是一个接口，因为使一个对象成为monad的不仅仅是某些API方法的存在。这些方法的相互作用有一定的保证，这是monadic的。这些众所周知的不变量对于使用单子是至关重要的，通过熟悉提高可读性;否则，它只是一个特殊的数据结构，必须被完全读取才能被读者理解。

事实上，对于这些单子方法的名称，甚至没有一个统一的协议，这是一个真正的接口所要求的;monad更像是一个松散的接口。有些人将某个方法称为`bind(..)`，有些人将其称为`chain(..)`，有些人将其称为`flatMap(..)`，依此类推。

因此monad是一个对象数据结构，具有足够的方法(实际上是任何名称或类型的方法)，至少满足monad定义的主要行为需求。每一种单子在最小值以上都有一种不同的扩展。但是，因为它们在行为上都有重叠，所以将两种不同的单子放在一起使用仍然是直接和可预测的。

在这个意义上，单子有点像一个接口。

## Just monad

您将遇到的许多其他单子下面的基本单子称为Just。它只是一个简单的单子包装任何常规(又名，非空)的值。

由于monad是一种类型，您可能认为我们应该将`Just`定义为要实例化的类。这是一种有效的方法，但它在我不想处理的方法中引入了“this”绑定问题;相反，我将坚持使用一个简单的函数方法。

下面是一个基本的实现:

```js
function Just(val) {
    return { map, chain, ap, inspect };

    // *********************

    function map(fn) { return Just( fn( val ) ); }

    // aka: bind, flatMap
    function chain(fn) { return fn( val ); }

    function ap(anotherMonad) { return anotherMonad.map( val ); }

    function inspect() {
        return `Just(${ val })`;
    }
}
```

**注:**`inspect(..)`方法仅用于演示目的。它在单子意义上没有直接作用。

您将注意到，无论`val`值是多少，实例`Just(..)`都会保持不变。所有monad方法都创建新的monad实例，而不是修改monad的值本身。

如果现在这些都没有意义，不要担心。我们不会过分关注monad设计背后的细节或数学/理论。相反，我们将更专注于说明我们可以用它们做什么。

### 使用Monad（单子）方法工作

所有monad实例都有`map(..)`, `chain(..)`(也称为`bind(..)`或`flatMap(..)`)和`ap(..)`方法。这些方法及其行为的目的是提供一种标准化的方法来实现多个monad实例之间的交互。

让我们首先看看单子`map(..)`函数。就像数组上的`map(..)`(参见[第9章](ch9.md/#map))用它的值调用mapper函数并生成一个新的数组一样，monad的`map(..)`用monad的值调用mapper函数，无论返回什么都被包装在一个新的Just monad实例中:

```js
var A = Just( 10 );
var B = A.map( v => v * 2 );

B.inspect();                // Just(20)
```

单子`chain(..)`函数和`map(..)`做的是一样的，但它会从它的新单子中打开结果值。然而，相对于非正式地考虑“展开”monad，更正式的解释应该是`chain(..)`将monad扁平易懂。考虑:

```js
var A = Just( 10 );
var eleven = A.chain( v => v + 1 );

eleven;                     // 11
typeof eleven;              // "number"
```

`eleven`是实际的原始数字`11`，而不是包含该值的单子。

为了从概念上将这个`chain(..)`方法与我们已经学过的东西联系起来，我们将指出，许多monad实现将这个方法命名为`flatMap(..)`。现在，回想一下[第9章`flatMap(..)`](ch9.md/#user-content-flatmap)(与`map(..)`相比)使用数组做了什么:

```js
var x = [3];

map( v => [v,v+1], x );         // [[3,4]]
flatMap( v => [v,v+1], x );     // [3,4]
```

看出不同了吗?mapper函数`v => [v,v+1]`产生一个`[3,4]` 数组，它最终位于外部数组的第一个位置，因此我们得到`[[3,4]]`。但是`flatMap(..)`将内部数组展平到外部数组中，因此我们得到的只是 `[3,4]` 。

monad的`chain(..)`(通常称为`flatMap(..)`)也是如此。不是像`map(..)`那样获得一个monad来保存值，而是`chain(..)`将monad扁平化到基础值中。实际上，`chain(..)`通常实现得更高效，而不是创建中间的单子只是为了立即使它变扁平化，只是走捷径而不是首先创建单子。无论哪种方式，最终结果都是一样的。

以这种方式说明 `chain(..)`的一种方法是结合`identity(..)`实用程序(参见[第3章](ch3.md/#one-on-one))，以便有效地从单子中提取一个值:

```js
var identity = v => v;

A.chain( identity );        // 10
```

`A.chain(..)`使用`A`中的值调用`identity(..)`，而不管`identity(..)`返回什么值(在本例中是`10`)，都会直接返回，不需要任何中间的单子。换句话说，从前面的`Just(..)`代码清单中，我们实际上不需要包含可选的`inspect(..)` ，因为`chain(identity)`实现了相同的目标;这纯粹是为了便于调试，因此我们学习单子。

在这一点上，希望`map(..)`和`chain(..)`对您来说都是合理的。

相比之下，monad的`ap(..)`方法乍一看可能没有那么直观。这看起来像是一种奇怪的交互扭曲，但设计背后有深刻而重要的理由。让我们花点时间把它分解一下。

`ap(..)`获取一个monad中包装的值，并使用另一个monad的 `map(..)`将其“应用”到另一个monad。到目前为止还好。

然而，`map(..)` 总是需要一个函数。这意味着你调用的monad`ap(..)`必须包含一个函数作为它的值，以传递给另一个monad`map(..)`。

困惑吗?是啊，不是你想的那样。我们将尝试简要说明，但只是希望这些事情困惑你的一段时间，直到你有了更多的探索了解和monads练习。

我们将`A`定义为包含值 `10`的单子， `B`定义为包含值`3`的单子:

```js
var A = Just( 10 );
var B = Just( 3 );

A.inspect();                // Just(10)
B.inspect();                // Just(3)
```

现在，我们如何创建一个新的monad，其中的值`10`和`3`已经添加在一起，比如通过`sum(..)`函数?事实证明， `ap(..)`可以提供帮助。

为了使用`ap(..)`，我们说首先需要构造一个包含函数的单子。具体地说，我们需要一个函数本身包含`A`中的值(通过闭包记住)。让我们先理解一下。

要从`A`生成一个包含值函数的单子，我们调用`A.map(..)`，给它一个curried函数“记住”提取的值(参见[Chapter 3](ch3.md/#one-at-a-time)作为它的第一个参数。我们将这个新的功能包含单子`C`:

```js
function sum(x,y) { return x + y; }

var C = A.map( curry( sum ) );

C.inspect();
// Just(function curried...)
```

想想如何运行的。`sum(..)`函数期望有两个值来完成它的工作，我们通过`A.map(..)` 将`10`提取出来并传递给它，从而得到第一个值。`C`现在保存了通过闭包记住`10`的函数。

现在，要获得第二个值(`3`内`B`)传递给在`C`等待的curried函数:

```js
var D = C.ap( B );

D.inspect();                // Just(13)
```

值`10`来自`C`，`3`来自`B`，`sum(..)`将它们加到`13`中，并将其封装在monad`D`中。让我们把这两个步骤放在一起，这样你就能更清楚地看到他们之间的联系:

```js
var D = A.map( curry( sum ) ).ap( B );

D.inspect();                // Just(13)
```

为了说明`ap(..)` 正在帮助我们做什么，我们可以通过以下方法获得相同的结果:

```js
var D = B.map( A.chain( curry( sum ) ) );

D.inspect();                // Just(13);
```

当然，这只是一个组合(见[第4章](ch4.md)):

```js
var D = compose( B.map, A.chain, curry )( sum );

D.inspect();                // Just(13)
```

酷吧! ?

如果到目前为止关于monad方法的讨论还不清楚，请回去重新阅读。如果解释还是难懂，那就坚持看完上面的。monad很容易让开发者困惑，这就是它的本质!

## Maybe 单子实例

在FP中很常见的是覆盖著名的单子，比如Maybe。实际上，Maybe monad是另外两个更简单的monad的特定组合:Just和Nothing。

我们已经看到Just;Nothing是一个包含空值的单子。也许是一个单子要么持有一个Just或一个Nothing。

下面是一个最简单的实现Maybe:

```js
var Maybe = { Just, Nothing, of/* aka: unit, pure */: Just };

function Just(val) { /* .. */ }

function Nothing() {
    return { map: Nothing, chain: Nothing, ap: Nothing, inspect };

    // *********************

    function inspect() {
        return "Nothing";
    }
}
```

**注:** `Maybe.of(..)`(有时称为`unit(..)`或`pure(..)`)是`Just(..)`的别名。

与`Just()`实例不同，`Nothing()`实例对所有monadic方法都有无操作定义的含义。因此，如果这样一个单子实例出现在任何monadic运算中，它的作用基本上是短路而不发生任何行为。请注意，这里没有强制执行“空”的含义——您的代码将决定这一点。稍后会详细介绍。

在Maybe中，如果一个值是非空的，它由`Just(..)`实例表示;如果它是空的，则由`Nothing()`实例表示。

但是这种monad表示的重要性在于，无论我们有一个`Just(..)`实例还是一个`Nothing()`实例，我们都将使用相同的API方法。

抽象的功能在于隐式地封装了行为/无操作对偶性。

### 不同的Maybe

JavaScript的许多实现可能都包含一个检查(通常在`map(..)`中)来查看值是否为`null`/`undefined`，如果是，则跳过该行为。事实上，也许被鼓吹为有价值，正是因为它自动短路其行为与封装的空值检查。

下面是Maybe通常的表达方式:

```js
// instead of unsafe `console.log( someObj.something.else.entirely )`:

Maybe.of( someObj )
.map( prop( "something" ) )
.map( prop( "else" ) )
.map( prop( "entirely" ) )
.map( console.log );
```

换句话说，如果在链上的任何一点我们得到一个`null`/`undefined`值，那么可能会神奇地切换到无操作定义模式——现在是一个 `Nothing()`monad实例!——停止对链的其他部分做任何事情。这使得嵌套属性访问在某些属性丢失/为空时不会抛出JS异常。这很酷，而且是一个非常有用的抽象概念!

但是…这种Maybe不是一个纯粹的单子

Monad的核心是它必须对所有值都有效，并且不能对值进行任何检查——甚至不能进行空检查。所以其他的实现都是为了方便而偷工减料。这并不是什么大事，但是当涉及到学习一些东西的时候，你应该先以最纯粹的形式来学习，然后再去改变规则。

我提供的Maybe monad的早期实现与其他Maybe的主要区别在于它没有空检查。此外，我们将`Maybe`表示为`Just(..)`/`Nothing()`的松散组合。

所以等等。如果我们没有自动短路，为什么Maybe有用呢?!?这似乎就是它的全部意义。

不要害怕!我们可以简单地在外部提供空检查，而Maybe monad的其他短路行为将正常工作。下面是如何实现以前的嵌套属性访问(`someObj.something.else.entirely`)，但更“正确”:

```js
function isEmpty(val) {
    return val === null || val === undefined;
}

var safeProp = curry( function safeProp(prop,obj){
    if (isEmpty( obj[prop] )) return Maybe.Nothing();
    return Maybe.of( obj[prop] );
} );

Maybe.of( someObj )
.chain( safeProp( "something" ) )
.chain( safeProp( "else" ) )
.chain( safeProp( "entirely" ) )
.map( console.log );
```

我们创建了一个`safeProp(..)`来执行空检查，如果是空检查，则选择`Nothing()` monad实例，或者将值包装在`Just(..)`实例中(通过`Maybe.of(..)`)。然后，我们不再使用`map(..)`，而是使用`chain(..)`，它知道如何“打开”`safeProp(..)`返回的单子。

当遇到空值时，我们会得到相同的链短路。我们只是不把这种逻辑嵌入Maybe中。

monad的好处，可能更具体地说，是我们的 `map(..)`和`chain(..)`方法具有一致的和可预测的交互，不管哪种monad返回。这都很酷!

## Humble 单子实例

既然我们对“Maybe”和它的作用有了更多的了解，我将对它进行一点小小的改动——并在我们的讨论中加入一些自我幽默——通过发明Maybe+Humble monad。从技术上讲，`MaybeHumble(..)`本身不是一个单子，而是一个工厂函数，它生成一个可能的单子实例。

Humble是一个公认的人为设计的数据结构包装器，它可能用于跟踪`egoLevel`数字的状态。具体地说，`MaybeHumble(..)`—生成的monad实例只有在其自我级别值足够低(小于`42`!)到被认为是humble时才会进行肯定的操作;否则它就是一个`Nothing()`、无操作定义。这听起来很像Maybe;真的非常像!

下面是我们的Maybe+Humble monad的工厂函数:

```js
function MaybeHumble(egoLevel) {
    // accept anything other than a number that's 42 or higher
    return !(Number( egoLevel ) >= 42) ?
        Maybe.of( egoLevel ) :
        Maybe.Nothing();
}
```

您会注意到，这个工厂函数有点像`safeProp(..)`，因为它使用一个条件来决定应该选择`Just(..)`还是 `Nothing()`作为Maybe的一部分。

让我们来说明一些基本用法:

```js
var bob = MaybeHumble( 45 );
var alice = MaybeHumble( 39 );

bob.inspect();              // Nothing
alice.inspect();            // Just(39)
```

如果Alice赢得了一个大奖，现在对自己更自豪了呢?

```js
function winAward(ego) {
    return MaybeHumble( ego + 3 );
}

alice = alice.chain( winAward );
alice.inspect();            // Nothing
```

`MaybeHumble( 39 + 3 )`调用创建一个`Nothing()`monad实例从 `chain(..)`调用返回，因此现在Alice不再符合humble的条件。

在，让我们一起使用几个单子:

```js
var bob = MaybeHumble( 41 );
var alice = MaybeHumble( 39 );

var teamMembers = curry( function teamMembers(ego1,ego2){
    console.log( `Our humble team's egos: ${ego1} ${ego2}` );
} );

bob.map( teamMembers ).ap( alice );
// Our humble team's egos: 41 39
```

回顾前面`ap(..)`的用法，我们现在可以解释这段代码是如何工作的。

因为`teamMembers(..)`是curried，所以`bob.map(..)`调用传递到`bob`，并创建一个monad实例，其中封装了剩余的函数。在monad调用`ap(alice)`，调用`alice.map(..)` ，并将函数从monad传递给它。其效果是，`bob`和`alice`monad的数值都提供给了`teamMembers(..)`函数，打印出了如下所示的消息。

然而，如果其中一个或两个单子实际上是`Nothing()` 实例:

```js
var frank = MaybeHumble( 45 );

bob.map( teamMembers ).ap( frank );
// ..no output..

frank.map( teamMembers ).ap( bob );
// ..no output..
```

`teamMembers(..)`永远不会被调用(也不会打印消息)，因为`frank`是一个`Nothing()`实例。这就是Maybe monad的力量，而我们的`MaybeHumble(..)`允许我们进行选择。太酷了!

### Humility实例

再举一个例子来说明我们的Maybe+Humble数据结构的行为:

```js
function introduction() {
    console.log( "I'm just a learner like you! :)" );
}

var egoChange = curry( function egoChange(amount,concept,egoLevel) {
    console.log( `${amount > 0 ? "Learned" : "Shared"} ${concept}.` );
    return MaybeHumble( egoLevel + amount );
} );

var learn = egoChange( 3 );

var learner = MaybeHumble( 35 );

learner
.chain( learn( "closures" ) )
.chain( learn( "side effects" ) )
.chain( learn( "recursion" ) )
.chain( learn( "map/reduce" ) )
.map( introduction );
// Learned closures.
// Learned side effects.
// Learned recursion.
// ..nothing else..
```

不幸的是，学习过程似乎被缩短了。你看，我发现学习一堆东西而不与他人分享会让你的自我膨胀得太厉害，对你的技能没有好处。

让我们尝试一个更好的学习方法:

```js
var share = egoChange( -2 );

learner
.chain( learn( "closures" ) )
.chain( share( "closures" ) )
.chain( learn( "side effects" ) )
.chain( share( "side effects" ) )
.chain( learn( "recursion" ) )
.chain( share( "recursion" ) )
.chain( learn( "map/reduce" ) )
.chain( share( "map/reduce" ) )
.map( introduction );
// Learned closures.
// Shared closures.
// Learned side effects.
// Shared side effects.
// Learned recursion.
// Shared recursion.
// Learned map/reduce.
// Shared map/reduce.
// I'm just a learner like you! :)
```

学习时分享。这是学得更多、学得更好的最好方法。

## 总结

什么是单子?monad是具有封装行为的值类型、接口和对象数据结构。

但这些定义都不是特别有用。这里有一个更好的尝试:** monad是您如何以更声明的方式围绕一个值去组织行为方式。

就像这本书里的其他内容一样，在有用的地方使用单子，但不要仅仅因为别人在FP中谈论它们就使用单子。monad并不是万能的灵丹妙药，但如果使用得比较保守，它们确实提供了一些实用功能。
