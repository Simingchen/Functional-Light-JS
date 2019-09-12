# 附言 A: 转换

转换是一种比我们在本书中介绍的更为先进的技术。它扩展了[第9章](ch9.md)关于列表操作的许多概念。

我并不一定要严格地将这个主题称为“功能轻量级”，但更像是一个附加功能。我将此作为附录介绍，因为您很可能需要暂时跳过讨论，待您感到相当舒服时再回过头来讨论书籍主要概念。

说实话，即使在教授了很多次关于转换的概念，并写了这一章之后，我仍然试图完全理解这种技术。所以，如果它使你陷入困境，不要感到难过。把这个附录标上书签，准备好了再来。

转换是指还原转化。

我知道这听起来像是一堆乱七八糟的字眼，让人困惑不止。但让我们看看它有多强大。实际上，我认为这是一个最好的例证，说明一旦你掌握了函数轻编程的原理，你可以做你要做的事情。

和本书的其他部分一样，我的方法是先解释为什么，然后解释如何，最后将其归结为一个简化的、可重复的“什么”。这通常与教多少人相反，但我认为你会通过这种方式更深入地学习这个话题。

## 首先弄清楚“为什么”

让我们从扩展[我们在第3章中讨论过的场景](ch3.md/#user-content-shortlongenough)开始，测试单词是否足够短和/或足够长:

```js
function isLongEnough(str) {
    return str.length >= 5;
}

function isShortEnough(str) {
    return str.length <= 10;
}
```
在[第3章，我们使用这些谓词函数](ch3.md/#user-content-shortlongenough)来测试单个单词。然后在第9章，我们学习了如何重复这样的测试[使用列表操作，如`filter(..)`](ch9.md/#filter)。例如:

```js
var words = [ "You", "have", "written", "something", "very",
    "interesting" ];

words
.filter( isLongEnough )
.filter( isShortEnough );
// ["written","something"]
```

这可能不明显，但是这种分离相邻列表操作的模式具有一些不理想的特性。当我们只处理一个由少量值组成的数组时，一切正常。但是如果数组中有很多值，每个`filter(..)`单独处理列表的速度会比我们希望的慢一些。

当我们的数组是异步/延迟的时，也会出现类似的性能问题，即随着时间的推移处理值以响应事件(参见[章节 10](ch10.md))。在这个场景中，一次只有一个值从事件流中下来，所以使用两个单独的`filter(..)`函数调用来处理这个不连续值并不是什么大问题。

但不明显的是，每个`filter(..)`方法都会产生一个单独的可观察值。将一个值从一个可观察对象注入到另一个可观察对象的开销确实会增加。这一点尤其正确，因为在这些情况下，处理成千上万的值并不罕见;即使如此小的日常开支也会迅速增加。

另一个缺点是可读性，特别是当我们需要对多个列表(或可观察对象)重复相同的系列操作时。例如:

```js
zip(
    list1.filter( isLongEnough ).filter( isShortEnough ),
    list2.filter( isLongEnough ).filter( isShortEnough ),
    list3.filter( isLongEnough ).filter( isShortEnough )
)
```

看起来重复了,是吧?

如果我们能将`isLongEnough(..)`与`isShortEnough(..)`结合起来，岂不是更好(在可读性和性能方面)?你可以手动操作:

```js
function isCorrectLength(str) {
    return isLongEnough( str ) && isShortEnough( str );
}
```

但这不是函数式编程的方式!

在[第9章，我们讨论了fusion](ch9.md/#fusion)——组合相邻的映射函数。回忆一下:

```js
words
.map(
    pipe( removeInvalidChars, upper, elide )
);
```

不幸的是，组合相邻的谓词函数不如组合相邻的映射函数那么容易。要理解原因，请考虑谓词函数的“形状”——一种描述输入和输出签名的学术方法。它接受一个值，并返回一个`true`或`false`。

如果你试了`isShortEnough(isLongEnough(str))`，它不会正常工作。`isLongEnough(..)`将返回`true`/`false`，而不是`isShortEnough(..)`期望的字符串值。结果有点让人失望。

试图组合两个相邻的reducer函数也存在类似的问题。reducer的“shape”是一个函数，它接收两个值作为输入，并返回单个组合值。一个reducer的输出作为一个单独的值，不适合输入到另一个期望有两个输入的reducer函数。

此外，`reduce(..)`的helper函数接受一个可选的`initialValue`输入。有时这可以省略，但有时必须传入。这甚至使组合更加复杂，因为一个缩减可能需要一个`initialValue`，而另一个缩减可能需要一个不同的`initialValue`。如果我们只使用某种组合的reduce调用一个`reduce(..)`调用，我们怎么可能做到这一点呢?

考虑一下这样的链式：

```js
words
.map( strUppercase )
.filter( isLongEnough )
.filter( isShortEnough )
.reduce( strConcat, "" );
// "WRITTENSOMETHING"
```
你能想象一个包含所有这些步骤的构图吗：`map(strUppercase)`, `filter(isLongEnough)`, `filter(isShortEnough)`, `reduce(strConcat)`？每个运算符的用法不同，因此它们不会直接组合在一起。我们需要把它们的运算符稍微变动一下，使它们合二为一。

希望这些观察结果已经说明了为什么简单的融合式合成不能胜任这项任务。我们需要一个更强大的技术，而转换就是这个工具。

## 下一步，怎么做？

让我们讨论如何派生 map映射、function谓词,更或者是reducer的组合。

不要过于不知所措:您不必经历编程中探索的这些心理。一旦您理解并认识到转换所解决的问题，就可以直接从FP库跳到使用 `transduce(..)`，然后继续处理应用程序的其他部分!

我们开始吧。

### 将Map/Filter表示为Reduce

我们需要执行的第一个技巧是将 `filter(..)`和`map(..)`调用表示为`reduce(..)`调用。回想一下[第9章我们是如何做到的](ch9.md/#map-as-reduce):

```js
function strUppercase(str) { return str.toUpperCase(); }
function strConcat(str1,str2) { return str1 + str2; }

function strUppercaseReducer(list,str) {
    list.push( strUppercase( str ) );
    return list;
}

function isLongEnoughReducer(list,str) {
    if (isLongEnough( str )) list.push( str );
    return list;
}

function isShortEnoughReducer(list,str) {
    if (isShortEnough( str )) list.push( str );
    return list;
}

words
.reduce( strUppercaseReducer, [] )
.reduce( isLongEnoughReducer, [] )
.reduce( isShortEnoughReducer, [] )
.reduce( strConcat, "" );
// "WRITTENSOMETHING"
```

这是一个不错的进步。我们现在有四个相邻的 `reduce(..)`调用，而不是三个不同方法的混合，它们都具有不同的基础方法。然而，我们仍然不能仅仅使用`compose(..)`组合这四个简化方法，因为它们接受两个而不是一个参数。

<a name="cheating"></a>

在[第9章，我们有点受骗了](ch9.md/#user-content-reducecheating)中，并使用`list.push(..)`进行转换，而不是创建一个全新的数组来连接。现在让我们退一步，变得更正式一点:

```js
function strUppercaseReducer(list,str) {
    return [ ...list, strUppercase( str ) ];
}

function isLongEnoughReducer(list,str) {
    if (isLongEnough( str )) return [ ...list, str ];
    return list;
}

function isShortEnoughReducer(list,str) {
    if (isShortEnough( str )) return [ ...list, str ];
    return list;
}
```

稍后，我们将再次讨论是否需要在这里创建一个新数组（例如，`[...list,str]`）来连接到该数组。

### Reducer的参数化

除了使用不同的谓词函数外，这两个过滤器reducer程序几乎是相同的。让我们参数化它，我们得到一个实用程序，可以定义任何过滤器reducer程序:

```js
function filterReducer(predicateFn) {
    return function reducer(list,val){
        if (predicateFn( val )) return [ ...list, val ];
        return list;
    };
}

var isLongEnoughReducer = filterReducer( isLongEnough );
var isShortEnoughReducer = filterReducer( isShortEnough );
```

让我们做同样的参数化`mapperFn(..)`的实用程序，以产生reducer程序:

```js
function mapReducer(mapperFn) {
    return function reducer(list,val){
        return [ ...list, mapperFn( val ) ];
    };
}

var strToUppercaseReducer = mapReducer( strUppercase );
```

Our chain still looks the same:

```js
words
.reduce( strUppercaseReducer, [] )
.reduce( isLongEnoughReducer, [] )
.reduce( isShortEnoughReducer, [] )
.reduce( strConcat, "" );
```

### 提取公共组合逻辑

仔细查看前面的`mapReducer(..)`和`filterReducer(..)`函数。您是否发现了它们共享的公共功能?

看这部分:

```js
return [ ...list, .. ];

// or
return list;
```

让我们为这个公共逻辑定义一个helper程序。但我们该怎么称呼它呢?


```js
function WHATSITCALLED(list,val) {
    return [ ...list, val ];
}
```

如果您检查那个`WHATSITCALLED(..)`函数的作用，它将接受两个值(一个数组和另一个值)，并通过创建一个新数组并将该值连接到它的末尾来“组合”它们。名字可以非常没有创意性，我们可以把它命名为`listCombine(..)`:

```js
function listCombine(list,val) {
    return [ ...list, val ];
}
```

现在让我们使用`listCombine(..)`重新定义我们的reducer函数:

```js
function mapReducer(mapperFn) {
    return function reducer(list,val){
        return listCombine( list, mapperFn( val ) );
    };
}

function filterReducer(predicateFn) {
    return function reducer(list,val){
        if (predicateFn( val )) return listCombine( list, val );
        return list;
    };
}
```

我们的链式看起来还是一样的(所以我们不会重复它)。

### 参数化组合

我们简单的`listCombine(..)`实用程序只是组合两个值的一种可能方法。让我们参数化它的使用，使我们的reducer程序更一般化:

```js
function mapReducer(mapperFn,combinerFn) {
    return function reducer(list,val){
        return combinerFn( list, mapperFn( val ) );
    };
}

function filterReducer(predicateFn,combinerFn) {
    return function reducer(list,val){
        if (predicateFn( val )) return combinerFn( list, val );
        return list;
    };
}
```

用这种方式定义我们的helper程序：

```js
var strToUppercaseReducer = mapReducer( strUppercase, listCombine );
var isLongEnoughReducer = filterReducer( isLongEnough, listCombine );
var isShortEnoughReducer = filterReducer( isShortEnough, listCombine );
```

将这些函数定义为使用两个参数而不是一个参数，这种方式更不方便进行组合，所以让我们使用`curry(..)`方法进行组合:

```js
var curriedMapReducer =
    curry( function mapReducer(mapperFn,combinerFn){
        return function reducer(list,val){
            return combinerFn( list, mapperFn( val ) );
        };
    } );

var curriedFilterReducer =
    curry( function filterReducer(predicateFn,combinerFn){
        return function reducer(list,val){
            if (predicateFn( val )) return combinerFn( list, val );
            return list;
        };
    } );

var strToUppercaseReducer =
    curriedMapReducer( strUppercase )( listCombine );
var isLongEnoughReducer =
    curriedFilterReducer( isLongEnough )( listCombine );
var isShortEnoughReducer =
    curriedFilterReducer( isShortEnough )( listCombine );
```

这看起来有点啰嗦，而且可能不是很有用。

但这对于推导的下一步是必要的。记住，我们这里的最终目标是能够使用`compose(..)`组合这些reducer函数。我们做的差不多了。

### 组合柯里化

这一步太形象化了。所以慢点读，仔细听。

让我们考虑一下之前的函数，但是没有将`listCombine(..)`函数传递给每个函数:

```js
var x = curriedMapReducer( strUppercase );
var y = curriedFilterReducer( isLongEnough );
var z = curriedFilterReducer( isShortEnough );
```

考虑这三个中间函数的形式，`x(..)`, `y(..)`, 和 `z(..)`。每个函数都期望一个组合函数，并用它生成一个reducer函数。

记住，如果我们想要这些独立的reducer函数，我们可以:

```js
var upperReducer = x( listCombine );
var longEnoughReducer = y( listCombine );
var shortEnoughReducer = z( listCombine );
```

但是如果您调用`y(z)`而不是`y(listCombine)`，会得到什么呢?基本上，当将 `z` 作为`combinerFn(..)`传递给 `y(..)`调用时会发生什么?返回的reducer函数内部看起来像这样:

```js
function reducer(list,val) {
    if (isLongEnough( val )) return z( list, val );
    return list;
}
```

看到里面的`z(..)`调用了吗?这看起来不对，因为`z(..)`函数应该只接收一个参数(`combinerFn(..)`)，而不是两个参数(`list` 和 `val`)。形式不匹配。是行不通的。

让我们看一下组合`y(z(listCombine))`。我们将其分为两个步骤:

```js
var shortEnoughReducer = z( listCombine );
var longAndShortEnoughReducer = y( shortEnoughReducer );
```

我们创建了`shortEnoughReducer(..)`，然后我们将它作为`combinerFn(..)`传递给 `y(..)`，而不是调用`y(listCombine)`;这个新调用生成`longAndShortEnoughReducer(..)`。再读几遍，直到它点击为止。

现在考虑一下:`shortEnoughReducer(..)` 和 `longAndShortEnoughReducer(..)`在内部是什么样子的?你能在脑海中看到它们吗?

```js
// shortEnoughReducer, from calling z(..):
function reducer(list,val) {
    if (isShortEnough( val )) return listCombine( list, val );
    return list;
}

// longAndShortEnoughReducer, from calling y(..):
function reducer(list,val) {
    if (isLongEnough( val )) return shortEnoughReducer( list, val );
    return list;
}
```

您是否看到`shortEnoughReducer(..)` 如何取代`listCombine(..)`中的`longAndShortEnoughReducer(..)`?为什么会这样呢?

因为`reducer(..)`的格式和`listCombine(..)`的格式是相同的。**换句话说，一个reducer函数可以作为另一个reducer函数的组合函数使用;他们就是这样组合的!`listCombine(..)`函数生成第一个reducer函数，然后可以用作组合函数来生成下一个reducer函数，以此类推。

让我们用几个不同的值来测试我们的`longAndShortEnoughReducer(..)`:

```js
longAndShortEnoughReducer( [], "nope" );
// []

longAndShortEnoughReducer( [], "hello" );
// ["hello"]

longAndShortEnoughReducer( [], "hello world" );
// []
```

`longAndShortEnoughReducer(..)`实用程序过滤掉两个不够长和不够短的值，它在同一步骤中执行这两个过滤。它是一个复合reducer函数!

再花点时间让自己明白这一点。感觉还是让我震惊。

现在，将`x(..)`添加到合成中:

```js
var longAndShortEnoughReducer = y( z( listCombine) );
var upperLongAndShortEnoughReducer = x( longAndShortEnoughReducer );
```

正如名称`upperLongAndShortEnoughReducer(..)`所暗示的，它一次完成所有三个步骤——一个映射和两个过滤器!看内部情况:

```js
// upperLongAndShortEnoughReducer:
function reducer(list,val) {
    return longAndShortEnoughReducer( list, strUppercase( val ) );
}
```

传入一个字符串`val`，用`strUppercase(..)`大写，然后传递给`longAndShortEnoughReducer(..)`。函数只有在足够长和足够短的情况下才有条件地将这个大写字符串添加到`list`中。否则，`list`将保持不变。

我的大脑花了几周的时间才完全理解这种含义。所以，如果你需要在这里停下来，花你读几遍(甚至几十遍!)的时间。

现在让我们来验证:

```js
upperLongAndShortEnoughReducer( [], "nope" );
// []

upperLongAndShortEnoughReducer( [], "hello" );
// ["HELLO"]

upperLongAndShortEnoughReducer( [], "hello world" );
// []
```

这个reducer函数是映射函数和过滤函数的合成!那太神奇了!

让我们回顾一下目前的进展:

```js
var x = curriedMapReducer( strUppercase );
var y = curriedFilterReducer( isLongEnough );
var z = curriedFilterReducer( isShortEnough );

var upperLongAndShortEnoughReducer = x( y( z( listCombine ) ) );

words.reduce( upperLongAndShortEnoughReducer, [] );
// ["WRITTEN","SOMETHING"]
```

这是很酷。让我们做得更好吧。

`x(y(z( .. )))`是一个组合型函数。让我们跳过中间的`x` / `y` / `z` 变量名，直接表示这个组合:

```js
var composition = compose(
    curriedMapReducer( strUppercase ),
    curriedFilterReducer( isLongEnough ),
    curriedFilterReducer( isShortEnough )
);

var upperLongAndShortEnoughReducer = composition( listCombine );

words.reduce( upperLongAndShortEnoughReducer, [] );
// ["WRITTEN","SOMETHING"]
```

想想组合函数中的“数据”流:

1. `listCombine(..)`作为组合函数传入，从而生成用于`isShortEnough(..)`的filter-reducer函数。
2. 生成的reducer函数然后作为组合函数传入，从而生成用于`isLongEnough(..)`的filter-reducer函数。
3. 生成的reducer函数然后作为组合函数传入，从而生成用于`strUppercase(..)`的filter-reducer函数。

在前面的代码片段中，`composition(..)`是一个复合函数，它期望组合函数能够生成一个reducer函数;这里有一个特殊的名称: 转换器。为转换提供组合功能，产生组合reducer函数:

```js
var transducer = compose(
    curriedMapReducer( strUppercase ),
    curriedFilterReducer( isLongEnough ),
    curriedFilterReducer( isShortEnough )
);

words
.reduce( transducer( listCombine ), [] );
// ["WRITTEN","SOMETHING"]
```

**注意:**我们应该对前两个代码片段中的`compose(..)`顺序进行观察，这可能会让人混淆。回想一下，在最初的示例链中，我们从`map(strUppercase)`然后`filter(isLongEnough)`最后`filter(isShortEnough)`;这些操作确实是按这个顺序进行的。但是在[章节 4](ch4.md/#user-content-generalcompose)中，我们了解到`compose(..)`通常具有按列表的相反顺序运行函数的效果。那么为什么我们不需要颠倒顺序来得到相同的期望结果呢?从每个reducer中抽象出的 `combinerFn(..)` 颠倒了有效应用操作顺序。因此，与直觉相反的是，在组合转换函数时，您实际上希望按照所需的执行顺序列出它们!

#### 列表组合: 纯函数与不纯函数

顺便提一下，让我们重温一下我们的`listCombine(..)`组合函数实现:

```js
function listCombine(list,val) {
    return [ ...list, val ];
}
```

虽然这种方法是纯的，但它对性能有负面影响:对于缩减中的每一步，我们都要创建一个全新的数组来追加值，从而有效地丢弃前面的数组。这需要创建和丢弃大量数组，这不仅不利于CPU，还会影响GC内存。

相比之下，再看看性能更好但不纯的版本:

```js
function listCombine(list,val) {
    list.push( val );
    return list;
}
```

如果单独考虑`listCombine(..)`，毫无疑问它是不纯的，这通常是我们希望避免的。然而，我们应该考虑一个更大的背景。

`listCombine(..)`根本不是我们交互的函数。我们不在程序的任何地方直接使用它;相反，我们在转换过程使用它。

回到[第5章](ch5.md)，我们断言减少副作用和定义纯函数的目标只是将纯函数暴露给我们将在整个程序中使用的函数的API级别。我们观察到，在一个纯函数的内部，它可以为了性能而欺骗所有它想要的东西，只要它不违反纯的外部契约。

`listCombine(..)` 更多的是转换的内部实现细节——事实上，它通常由转换库为您提供!——而不是在整个程序中与之正常交互的顶级方法。

我认为使用性能最佳的非纯版本`listCombine(..)`是完全可以接受的，甚至是可取的。只要确保您用代码注释来记录它是不纯的!

### 替代组合

到目前为止，这是我们通过转化得到的:

```js
words
.reduce( transducer( listCombine ), [] )
.reduce( strConcat, "" );
// WRITTENSOMETHING
```

这很好，但是我们还有最后一个关于转化的技巧。坦率地说，我认为这部分是你迄今为止所付出的所有精神努力的真正价值所在。

我们能否以某种方式“组合”这两个`reduce(..)`调用，将其简化为一个`reduce(..)`?不幸的是，我们不能`strConcat(..)`添加到`compose(..)`调用中;因为它是一个reducer函数，而不是一个期望组合的函数，所以它的形式不适合组合。

但是让我们一起来看看这两个函数:

```js
function strConcat(str1,str2) { return str1 + str2; }

function listCombine(list,val) { list.push( val ); return list; }
```

如果你仔细看，你几乎可以看到这两个功能是如何互换的。它们使用不同的数据类型进行操作，但在概念上它们做的是相同的事情:将两个值合并为一个值。

换句话说，`strConcat(..)`是一个组合函数!

这意味着如果我们的最终目标是得到一个字符串连接而不是列表，我们可以使用*it*而不是`listCombine(..)`:

```js
words.reduce( transducer( strConcat ), "" );
// WRITTENSOMETHING
```

太棒啦!这就是转化。

## 最后是什么

深呼吸。要消化的东西太多了。

先清理一下我们的大脑，让我们把注意力转回到仅仅在应用程序中使用转换函数上，而不是跳过所有的思维障碍来推导它是如何工作的。

回想一下我们前面定义的函数;为了清晰起见，我们重新命名一下:

```js
var transduceMap =
    curry( function mapReducer(mapperFn,combinerFn){
        return function reducer(list,v){
            return combinerFn( list, mapperFn( v ) );
        };
    } );

var transduceFilter =
    curry( function filterReducer(predicateFn,combinerFn){
        return function reducer(list,v){
            if (predicateFn( v )) return combinerFn( list, v );
            return list;
        };
    } );
```

还记得我们这样使用它们:

```js
var transducer = compose(
    transduceMap( strUppercase ),
    transduceFilter( isLongEnough ),
    transduceFilter( isShortEnough )
);
```

`transducer(..)`仍然需要传递给它一个组合函数(如`listCombine(..)`或`strConcat(..)`)来生成一个reducer函数，然后可以在`reduce(..)`中使用该函数。

但是为了更明确地表达所有这些转换步骤，让我们创建一个`transduce(..)` 实用程序，它为我们完成这些步骤:

```js
function transduce(transducer,combinerFn,initialValue,list) {
    var reducer = transducer( combinerFn );
    return list.reduce( reducer, initialValue );
}
```

这是我们的运行例子:

```js
var transducer = compose(
    transduceMap( strUppercase ),
    transduceFilter( isLongEnough ),
    transduceFilter( isShortEnough )
);

transduce( transducer, listCombine, [], words );
// ["WRITTEN","SOMETHING"]

transduce( transducer, strConcat, "", words );
// WRITTENSOMETHING
```

还不赖,是吧! ?看一下`listCombine(..)`和`strConcat(..)`函数作为组合函数互换使用吗?

### Transducers.js 库

最后，让我们使用 [`transducers-js` 库](https://github.com/cognitect-labs/transducers-js):

```js
var transformer = transducers.comp(
    transducers.map( strUppercase ),
    transducers.filter( isLongEnough ),
    transducers.filter( isShortEnough )
);

transducers.transduce( transformer, listCombine, [], words );
// ["WRITTEN","SOMETHING"]

transducers.transduce( transformer, strConcat, "", words );
// WRITTENSOMETHING
```

看起来几乎和上面一样。

**注意:**前面的代码片段使用了`transformers.comp(..)`，因为库提供了它，但是在本例中，来自第4章的[`compose(..)`](ch4.md/#user-content-generalcompose)将产生相同的结果。换句话说，合成本身并不是一个敏锐转换的操作。

这个代码片段中的组合函数名为`transformer`，而不是`transducer`。这是因为如果我们调用`transformer( listCombine )`(或`transformer( strConcat )`)，我们将不会像前面那样得到一个直接的转换的reducer函数。

`transducers.map(..)`和`transducers.filter(..)`是特殊的函数，它们将常规谓词或映射函数转换为生成特殊转换对象的函数;库使用这些转换对象进行转换。这种转换对象抽象的额外功能超出了我们将探讨的范围，因此请参考库文档以获得更多信息。

因为调用`transformer(..)` 会生成一个转换对象，而不是一个典型的二进制换向器函数，所以库还提供了`toFn(..)`来调整转换对象，使其可被原生数组中`reduce(..)`使用:


```js
words.reduce(
    transducers.toFn( transformer, strConcat ),
    ""
);
// WRITTENSOMETHING
```

`into(..)`是另一个函数，它可以根据指定的空/初值类型自动选择默认组合函数:

```js
transducers.into( [], transformer, words );
// ["WRITTEN","SOMETHING"]

transducers.into( "", transformer, words );
// WRITTENSOMETHING
```

当指定一个空的`[]`数组时，在内部调用的`transduce(..)`使用一个函数的默认实现，比如我们的`listCombine(..)` 。但是，当指定一个空的`""`字符串时，使用类似于我们的`strConcat(..)`的东西。这样太酷了!

如您所见，`transducers-js` 库使换能器非常简单。我们可以非常有效地利用这种技术的力量，而不必自己定义所有那些产生中间换能器的实用程序。

## 总结

转换指的是用reduce进行变换。更具体地说，转换函数是一种可组合的reducer函数。

我们使用转换来组合相邻的 `map(..)`, `filter(..)`, 和 `reduce(..)`操作。我们首先将 `map(..)`和`filter(..)`表示为`reduce(..)`，然后抽象出常见的组合操作来创建易于组合的一元生成reducer函数，从而实现这一点。

转化主要是为了提高性能，如果使用在一个纯函数是特别明显的。

但更广泛地说，转换是我们表达更声明性的函数组合的方式，否则这些函数将不能直接组合。如果像本书中所有其他技术一样使用得当，其结果将是更清晰、更可读的代码!使用转换函数的调用单个`reduce(..)`比跟踪多个`reduce(..)` 调用更容易推理。
