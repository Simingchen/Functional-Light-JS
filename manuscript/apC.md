# JavaScript轻量级函数式编程
# 附录C: 关于FP的库
当你把这本书从头读到尾，请花点时间回看一下[Chapter 1](ch1.md). 这是一段很长的旅程. 我希望你已经学到了很多，并且把函数式的精髓深入到你的编程中去。

我想把这本书合上来告诉你一些常用/流行的FP库的快速入门。接下来要介绍并不是一份详尽的文档，而是一个快速使你从轻量函数式进入更广泛的FP的入门介绍。

如果可能的话，我建议你 “不要” 再发明任何轮子。如果你找到一个符合你需要的FP库，就用它吧。-- 如果你实在找不到适合自己库，那么也使用本书中的示例程序 -- 或者你也可以自己发明一个。

## 需关注的工具库（注：我的理解）

让我们展开第1章 [需要注意的FP库列表，从第1章开始](ch1.md/#libraries)，我们不可能涵盖所有这些内容（可能有很多相似），但以下应该是你要关注的库：

* [Ramda](http://ramdajs.com): 通用的FP工具库
* [Sanctuary](https://github.com/sanctuary-js/sanctuary): 类似Ramda的FP工具库
* [lodash/fp](https://github.com/lodash/lodash/wiki/FP-Guide): lodash的FP工具库
* [functional.js](http://functionaljs.com/): 通用的FP工具库
* [Immutable](https://github.com/facebook/immutable-js): facebook官方的，一个关于不可变数据的库
* [Mori](https://github.com/swannodette/mori): 基于ClojureScript不可变数据的库
* [Seamless-Immutable](https://github.com/rtfeldman/seamless-immutable): 关于不可变数据的辅助函数
* [transducers-js](https://github.com/cognitect-labs/transducers-js): Transducers
* [monet.js](https://github.com/monet/monet.js): Monadic Types

还有几十个其他优秀的库不在此列表中。仅仅因为它不在我的名单上并不意味着它不好，这里只是简单地浏览一下JavaScript中的FP。你可以[在这里](https://github.com/stoeffel/awesome-fp-js)找到更多关于FP的编程资源。

函数式编程世界中十分重要的学习资源，[Fantasy Land](https://github.com/fantasyland/fantasy-land) (简称 FL) ，与其说它是一个库，更像是一本百科全书。

Fantasy Land（FL） 不仅仅是一份为初学者准备的轻量级读物，更是一个完整而详细的 JavaScript 函数式编程路线图。为了确保最大的通用性，FL 已经成为 JavaScript 函数式编程库遵循的业内标准。

Fantasy Land （FL）与“轻量函数式编程”的概念几乎完全相反，它是 JavaScript 函数式编程世界中全面的毫无保留的一种诠释 。也就是说，当你的能力超越本书时，FL 将会成为你接下来前进的方向。我建议你将其保存在收藏夹，并在你使用本书的概念进行至少 6 个月的实战练习之后再回来。

## Ramda (0.23.0) （注：目前最新版本是0.26.1）

来自 [Ramda 文件](http://ramdajs.com/):

> Ramda函数是自动被柯里化的.
>
> Ramda函数的参数进行了优化，使其便于柯里化。要操作的数据通常跟在最后。

我发现合理设计是Ramda的优势之一。还需要注意的是，Ramda的柯里化形式(似乎与大多数库一样)是 [我们第3章讨论的“松散柯里化”](ch3.md/#user-content-loosecurry)。

回想一下，[在第3章最后的示例](ch3.md/#user-content-finalshortlong)，我们定义一个无参函数`printif(..) ` -- 在Ramda中可以这样定义：

```js
function output(msg) {
    console.log( msg );
}

function isShortEnough(str) {
    return str.length <= 5;
}

var isLongEnough = R.complement( isShortEnough );

var printIf = R.partial( R.flip( R.when ), [output] );

var msg1 = "Hello";
var msg2 = msg1 + " World";

printIf( isShortEnough, msg1 );            // Hello
printIf( isShortEnough, msg2 );

printIf( isLongEnough, msg1 );
printIf( isLongEnough, msg2 );            // Hello World
```

与第3章实现有点不同的是 [Chapter 3's approach](ch3.md/#user-content-finalshortlong)：

* 我们使用 `R.complement(..)` 代替 `not(..)` 基于`isShortEnough(..)` 创建一个相反函数 `isLongEnough(..)` 。

* 我们使用 `R.flip(..)` 代替 `reverseArgs(..)`。需要注意的是，`R.flip(..)`只交换前两个参数，而`recseArgs(..)`则反转所有参数。在这种情况下，`flip(..)`对我们来说更方便，所以我们不需要使用`ParalRight(..)`或其他的方法。

* `R.partial(..)`将其所有后续参数(函数以外)作为单个数组传入。

* 由于Ramda 使用松散柯里化，因此我们不需要使用 `R.uncurryN(..)` 来取得一个所有参数的 `printIf(..)` 方法。如果我们这样做了，它看起来就像用 `R.uncurryN( 2,.. )` 包装了 `R.partial(..)` 来调用，这看似是没必要的。

Ramda 是一个非常受欢迎和强大的函数库。如果您正在尝试将FP添加到您的代码库中，那么这是一个非常好的开始。

## Lodash/fp (4.17.4)

Lodash 是整个JS生态系统中最受欢迎的函数库。Lodash团队也发布了一个 "FP-friendly" API版本 -- ["lodash/fp"](https://github.com/lodash/lodash/wiki/FP-Guide)。

在 [第9章，我们探讨了合并独立列表操作](ch9.md/#composing-standalone-utilities) (`map(..)`, `filter(..)`, 和 `reduce(..)`)。在"lodash/fp"中，我们可以这么做 :

```js
var sum = (x,y) => x + y;
var double = x => x * 2;
var isOdd = x => x % 2 == 1;

fp.compose( [
    fp.reduce( sum )( 0 ),
    fp.map( double ),
    fp.filter( isOdd )
] )
( [1,2,3,4,5] );                    // 18
```

与我们熟悉的 `_.` 命名空间前缀不同，“lodash/fp”以 `fp.` 作为命名空间前缀定义其方法。我觉得这是个很有帮助的区别，而且比 `_.` 更容易理解！

需要注意的是，`fp.Composed(..)` (在lodash中称为`_.flowRight(..)`)接受一个函数数组，而不是单个函数参数。

lodash的稳定性、广泛的社区支持和优秀性能是你探索FP的后盾。

## Mori (0.3.2)

在[第6章](ch6.md)，我们已经简要地看过 Immutable.js 库，它可能是最著名的不可变数据结构库。

让我们看看另一个流行的库：[Mori](https://github.com/swannodette/mori) 。Mori在设计时采用了一套不同的API(表面上看更像FP)：它使用独立的函数，而不是直接对值做操作。

```js
var state = mori.vector( 1, 2, 3, 4 );

var newState = mori.assoc(
    mori.into( state, Array.from( {length: 39} ) ),
    42,
    "meaning of life"
);

state === newState;                        // false

mori.get( state, 2 );                    // 3
mori.get( state, 42 );                    // undefined

mori.get( newState, 2 );                // 3
mori.get( newState, 42 );                // "meaning of life"

mori.toJs( newState ).slice( 1, 3 );    // [2,3]
```

在这个例子中，Mori有一些有趣的地方需要指出：

* 我们使用 `vector` 代替 `list` ，主要是因为文档说它的行为更像JS的数组。

* 我们不能像操作JS数组那样在末尾随机设置值，这会引发异常报错。因此，我们必须首先使用 `morio .into(..)` 传入一个合适长度的数组来扩展 vector 的长度。当有一个43个插槽（注：索引值）(4 + 39)的vector，我们就可以使用 `mori.assoc(..)`方法将最后的索引位置 `42` 设置为`"meaning of life"`这个值。 

* 使用 `mori.into(..)` 创建一个更大的vector，然后使用 `mori.assoc(..)` 在此基础上创建另一个vector，这样的操作可能听起来效率不高。但是不可变数据结构的美妙之处在于数据没有克隆。每次进行“更改”时，新的数据结构只是跟踪与以前状态的差异。

Mori深受ClojureScript的启发。 如果你有ClojureScript语言经验（或目前正在使用！），那么应该对它的API非常熟悉。由于我没有这种经验，我觉得方法名有点奇怪，不习惯。

我真心喜欢这种独立的调用函数设计，而不是基于值的调用方法。Mori还有一些自动返回常规JS数组的函数，用起来都很方便。

## 干货: FPO

在 [第2章中，我们介绍了一种模式]（ch2.md/#named-arguments），用于处理称为“命名参数”的参数，在JS（注：ES6）中，使用对象将属性映射到析构函数的参数:

```js
function foo( {x,y} = {} ) {
    console.log( x, y );
}

foo( {
    y: 3
} );                    // undefined 3
```

接着在[Chapter 3, 我们探讨了更多](ch3.md/#order-matters) 关于析构函数的柯里化和偏函数应用，像下面这样:

```js
function foo({ x, y, z } = {}) {
    console.log( `x:${x} y:${y} z:${z}` );
}

var f1 = curryProps( foo, 3 );

f1( {y: 2} )( {x: 1} )( {z: 3} );
```

这种风格的一个好处是能够以任何顺序传参（即使是柯里化和偏函数应用！），而不必像`reverseArgs(..)`式的传参。 另一个好处是能够省略一个可选参数，只不需传一个丑陋的占位符。

在我学习FP的过程中，我经常被固定位置传参感到沮丧，因此，我非常欣赏用命名参数风格（注：传入一个对象）解决以往的问题。

有一天，我在思考FP编码风格，我想如果整个FP库都以这种风格公开其所有API方法，那会是什么样子。我开始做尝试，不断把这些尝试展示给一些人看，并得到了一些积极的反馈。

经过努力，[FPO](https://github.com/getify/fpo) (发音为“ef-poh”) 库最终诞生了。FPO意思是FP-with-Objects。

官方文档:

```js
// Ramda's `reduce(..)`
R.reduce(
    (acc,v) => acc + v,
    0,
    [3,7,9]
);  // 19

// FPO named-argument method style
FPO.reduce({
    arr: [3,7,9],
    fn: ({acc,v}) => acc + v
}); // 19
```

例如传统FP库（Ramda）的`reduce(..)`方法，初始参数的位置是固定不可变的。而FPO的`reduce(..)`方法可以按任意顺序传参，如果需要，甚至可以省略初始值。

与大多数其他FP库一样，FPO的API方法自动松柯里化，因此您不仅可以按任何顺序传参，还可以通过多次调用赋值方法来传入参数：

```js
var f = FPO.reduce({ arr: [3,7,9] });

// later

f({ fn: ({acc,v}) => acc + v });    // 19
```

最后，在`FPO.std.*`命名文件下，你会发现它们与Ramda和其他库非常相似，所有FPO的API方法也可以使用传统固定位置参数操作：

```js
FPO.std.reduce(
    (acc,v) => acc + v,
    undefined,
    [3,7,9]
);  // 19
```

如果FPO的命名参数写法对你有吸引力，你可以查看源码了解更多。 它拥有完整的测试套件和大多数你期待的FP方法，包括本文中介绍的所有内容，以帮助您更好的学习（使用）轻量级函数式编程！

## 干货 #2: fasy

FP迭代方法（`map(..)`，`filter(..)`等）几乎总是被看作同步操作，意味着立即执行所有步骤。 事实上，其他FP模式，如合成甚至转换也是迭代，并且也以这种方式执行。
		
但是如果迭代中的一个或多个步骤需要异步完成，会发生什么？自然你会想到观察者模式（见[第10章]（ch10.md/#observables）），但目前不是我们所要的。

让我快速说明一下。

想象一下，您有一个URL列表，表示您要加载到网页中的图像。 显然，提取图像是异步的。显然，这并不像你希望的那样顺序加载：

```js
var imageURLs = [
    "https://some.tld/image1.png",
    "https://other.tld/image2.png",
    "https://various.tld/image3.png"
];

var images = imageURLs.map( fetchImage );
```

`images` 数组的内容基于`fetchImage(..)`方法执行，一旦它下载完成将返回一个promise的对象。

当然，你也可以使用`Promise.all(..)` 来等待所有的图片下载完成返回一个resolve对象，通过`then()`方法返回一个接收图片对象的函数：

```js
Promise.all( images )
.then(function allImages(imgObjs){
    // ..
});
```

不幸的是，这个“技巧”只有在你要同时执行所有异步步骤（而不是串行，一个接一个）时才有效，并且只有当操作是一个`map(..)`调用时才有效。 如果你想要串行异步操作，或者你想同事使用`filter(..)`方法，这将可能返回错乱结果。

有些操作自然需要串行异步，例如异步`reduce(..)`，它显然需要一次从左到右工作;这些步骤不能同时运行，并且不能让该操作有任何意义。

正如我所说，可观察性(参见[第10章](ch10.md/#observables))不是这类任务的答案。原因是，一个可观察对象的异步协调是在单独的操作之间进行的，而不是在单个操作级别的步骤/迭代之间进行的。

另一种可视化这种区别的方法是，可观测支持“垂直异步”，而我所说的是“水平异步”。

考虑:

```js
var obsv = Rx.Observable.from( [1,2,3,4,5] );

obsv
.map( x => x * 2 )
.delay( 100 )        // <-- vertical asynchrony
.map( x => x + 1 )
.subscribe( v => console.log );
// {after 100 ms}
// 3
// 5
// 7
// 9
// 11
```

如果出于某种原因，我想确保从第一个`map(..)`处理`1`到处理`2`之间有100毫秒的延迟，那么这就是我所指的“水平异步”。没有一种明朗的方法来模拟它。

那么，我们如何跨异步操作同时支持串行迭代和并发迭代呢?

**fasy**(发音与“Tracy”相似，但带有“f”)是我为支持这类任务而构建的一个小实用程序库。你可以在这里找到更多关于它的[信息](https://github.com/getify/fasy)。

为了说明**fasy**，让我们考虑一个并发的`map(..)`与一个串行的`map(..)`:

```js
FA.concurrent.map( fetchImage, imageURLs )
.then( function allImages(imgObjs){
    // ..
} );

FA.serial.map( fetchImage, imageURLs )
.then( function allImages(imgObjs){
    // ..
} );
```

在这两种情况下，`then(..)`处理程序只会在所有获取完全完成后调用。不同之处在于，所有的获取是同时启动(也就是“并行”)，还是一次发出一个的。

你的直觉可能是同时进行总是更好的，虽然这可能是常见的，但并不总是这样。

例如，如果 `fetchImage(..)`维护一个获取图像的缓存，并在发出实际的网络请求之前检查缓存，结果会怎样?除此之外，如果“imageURLs”列表中可以有多个副本呢?在检查列表中稍后的重复图像URL之前，您肯定希望完成图像URL的第一次获取(并填充缓存)。

同样，在某些情况下不可避免地需要并发或串行异步。异步缩减总是串行的，而异步映射可能更倾向于并发，但在某些情况下也可能需要串行。这就是为什么**fasy**支持所有这些选项。

除了Observables，**fasy**将帮助您将更多fp模式和原则扩展到异步操作。

## 总结

JavaScript并不是专门为FP语言而设计的。 但是它具备设计FP的该有的核心（如函数值，闭包等）。 就像上面我们探讨过的优秀的库。

通过学习本书中的知识，你可以开始开始coding了。 找一个好的，适合的FP库并使用之。练习，练习，练习！（注：多写才是王道）

看到这里，我已经分享了我所有了解的。 我认为你完全可以称之为“轻量级函数式编程”程序员！ 现在是时候结束我们共同学习FP的“篇章”了。 但我的学习之旅仍在继续，我也希望你的确如此！
