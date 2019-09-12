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

Both filter reducers are almost identical, except they use a different predicate function. Let's parameterize that so we get one utility that can define any filter-reducer:

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

Let's do the same parameterization of the `mapperFn(..)` for a utility to produce any map-reducer:

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

### Extracting Common Combination Logic

Look very closely at the preceding `mapReducer(..)` and `filterReducer(..)` functions. Do you spot the common functionality shared in each?

This part:

```js
return [ ...list, .. ];

// or
return list;
```

Let's define a helper for that common logic. But what shall we call it?

```js
function WHATSITCALLED(list,val) {
    return [ ...list, val ];
}
```

If you examine what that `WHATSITCALLED(..)` function does, it takes two values (an array and another value) and it "combines" them by creating a new array and concatenating the value onto the end of it. Very uncreatively, we could name this `listCombine(..)`:

```js
function listCombine(list,val) {
    return [ ...list, val ];
}
```

Let's now re-define our reducer helpers to use `listCombine(..)`:

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

Our chain still looks the same (so we won't repeat it).

### Parameterizing the Combination

Our simple `listCombine(..)` utility is only one possible way that we might combine two values. Let's parameterize the use of it to make our reducers more generalized:

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

To use this form of our helpers:

```js
var strToUppercaseReducer = mapReducer( strUppercase, listCombine );
var isLongEnoughReducer = filterReducer( isLongEnough, listCombine );
var isShortEnoughReducer = filterReducer( isShortEnough, listCombine );
```

Defining these utilities to take two arguments instead of one is less convenient for composition, so let's use our `curry(..)` approach:

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

That looks a bit more verbose, and probably doesn't seem very useful.

But this is actually necessary to get to the next step of our derivation. Remember, our ultimate goal here is to be able to `compose(..)` these reducers. We're almost there.

### Composing Curried

This step is the trickiest of all to visualize. So read slowly and pay close attention here.

Let's consider the curried functions from earlier, but without the `listCombine(..)` function having been passed in to each:

```js
var x = curriedMapReducer( strUppercase );
var y = curriedFilterReducer( isLongEnough );
var z = curriedFilterReducer( isShortEnough );
```

Think about the shape of all three of these intermediate functions, `x(..)`, `y(..)`, and `z(..)`. Each one expects a single combination function, and produces a reducer function with it.

Remember, if we wanted the independent reducers from all these, we could do:

```js
var upperReducer = x( listCombine );
var longEnoughReducer = y( listCombine );
var shortEnoughReducer = z( listCombine );
```

But what would you get back if you called `y(z)`, instead of `y(listCombine)`? Basically, what happens when passing `z` in as the `combinerFn(..)` for the `y(..)` call? That returned reducer function internally looks kinda like this:

```js
function reducer(list,val) {
    if (isLongEnough( val )) return z( list, val );
    return list;
}
```

See the `z(..)` call inside? That should look wrong to you, because the `z(..)` function is supposed to receive only a single argument (a `combinerFn(..)`), not two arguments (`list` and `val`). The shapes don't match. That won't work.

Let's instead look at the composition `y(z(listCombine))`. We'll break that down into two separate steps:

```js
var shortEnoughReducer = z( listCombine );
var longAndShortEnoughReducer = y( shortEnoughReducer );
```

We create `shortEnoughReducer(..)`, then we pass *it* in as the `combinerFn(..)` to `y(..)` instead of calling `y(listCombine)`; this new call produces `longAndShortEnoughReducer(..)`. Re-read that a few times until it clicks.

Now consider: what do `shortEnoughReducer(..)` and `longAndShortEnoughReducer(..)` look like internally? Can you see them in your mind?

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

Do you see how `shortEnoughReducer(..)` has taken the place of `listCombine(..)` inside `longAndShortEnoughReducer(..)`? Why does that work?

Because **the shape of a `reducer(..)` and the shape of `listCombine(..)` are the same.** In other words, a reducer can be used as a combination function for another reducer; that's how they compose! The `listCombine(..)` function makes the first reducer, then *that reducer* can be used as the combination function to make the next reducer, and so on.

Let's test out our `longAndShortEnoughReducer(..)` with a few different values:

```js
longAndShortEnoughReducer( [], "nope" );
// []

longAndShortEnoughReducer( [], "hello" );
// ["hello"]

longAndShortEnoughReducer( [], "hello world" );
// []
```

The `longAndShortEnoughReducer(..)` utility is filtering out both values that are not long enough and values that are not short enough, and it's doing both these filterings in the same step. It's a composed reducer!

Take another moment to let that sink in. It still kinda blows my mind.

Now, to bring `x(..)` (the uppercase reducer producer) into the composition:

```js
var longAndShortEnoughReducer = y( z( listCombine) );
var upperLongAndShortEnoughReducer = x( longAndShortEnoughReducer );
```

As the name `upperLongAndShortEnoughReducer(..)` implies, it does all three steps at once -- a mapping and two filters! What it kinda look likes internally:

```js
// upperLongAndShortEnoughReducer:
function reducer(list,val) {
    return longAndShortEnoughReducer( list, strUppercase( val ) );
}
```

A string `val` is passed in, uppercased by `strUppercase(..)` and then passed along to `longAndShortEnoughReducer(..)`. *That* function only conditionally adds this uppercased string to the `list` if it's both long enough and short enough. Otherwise, `list` will remain unchanged.

It took my brain weeks to fully understand the implications of that juggling. So don't worry if you need to stop here and re-read a few (dozen!) times to get it. Take your time.

Now let's verify:

```js
upperLongAndShortEnoughReducer( [], "nope" );
// []

upperLongAndShortEnoughReducer( [], "hello" );
// ["HELLO"]

upperLongAndShortEnoughReducer( [], "hello world" );
// []
```

This reducer is the composition of the map and both filters! That's amazing!

Let's recap where we're at so far:

```js
var x = curriedMapReducer( strUppercase );
var y = curriedFilterReducer( isLongEnough );
var z = curriedFilterReducer( isShortEnough );

var upperLongAndShortEnoughReducer = x( y( z( listCombine ) ) );

words.reduce( upperLongAndShortEnoughReducer, [] );
// ["WRITTEN","SOMETHING"]
```

That's pretty cool. But let's make it even better.

`x(y(z( .. )))` is a composition. Let's skip the intermediate `x` / `y` / `z` variable names, and just express that composition directly:

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

Think about the flow of "data" in that composed function:

1. `listCombine(..)` flows in as the combination function to make the filter-reducer for `isShortEnough(..)`.
2. *That* resulting reducer function then flows in as the combination function to make the filter-reducer for `isLongEnough(..)`.
3. Finally, *that* resulting reducer function flows in as the combination function to make the map-reducer for `strUppercase(..)`.

In the previous snippet, `composition(..)` is a composed function expecting a combination function to make a reducer; `composition(..)` here has a special name: transducer. Providing the combination function to a transducer produces the composed reducer:

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

**Note:** We should make an observation about the `compose(..)` order in the previous two snippets, which may be confusing. Recall that in our original example chain, we `map(strUppercase)` and then `filter(isLongEnough)` and finally `filter(isShortEnough)`; those operations indeed happen in that order. But in [Chapter 4](ch4.md/#user-content-generalcompose), we learned that `compose(..)` typically has the effect of running its functions in reverse order of listing. So why don't we need to reverse the order *here* to get the same desired outcome? The abstraction of the `combinerFn(..)` from each reducer reverses the effective applied order of operations under the hood. So counter-intuitively, when composing a transducer, you actually want to list them in desired order of execution!

#### List Combination: Pure vs. Impure

As a quick aside, let's revisit our `listCombine(..)` combination function implementation:

```js
function listCombine(list,val) {
    return [ ...list, val ];
}
```

While this approach is pure, it has negative consequences for performance: for each step in the reduction, we're creating a whole new array to append the value onto, effectively throwing away the previous array. That's a lot of arrays being created and thrown away, which is not only bad for CPU but also GC memory churn.

By contrast, look again at the better-performing but impure version:

```js
function listCombine(list,val) {
    list.push( val );
    return list;
}
```

Thinking about `listCombine(..)` in isolation, there's no question it's impure and that's usually something we'd want to avoid. However, there's a bigger context we should consider.

`listCombine(..)` is not a function we interact with at all. We don't directly use it anywhere in the program; instead, we let the transducing process use it.

Back in [Chapter 5](ch5.md), we asserted that our goal with reducing side effects and defining pure functions was only that we expose pure functions to the API level of functions we'll use throughout our program. We observed that under the covers, inside a pure function, it can cheat for performance sake all it wants, as long as it doesn't violate the external contract of purity.

`listCombine(..)` is more an internal implementation detail of the transducing -- in fact, it'll often be provided by the transducing library for you! -- rather than a top-level method you'd interact with on a normal basis throughout your program.

Bottom line: I think it's perfectly acceptable, and advisable even, to use the performance-optimal impure version of `listCombine(..)`. Just make sure you document that it's impure with a code comment!

### Alternative Combination

So far, this is what we've derived with transducing:

```js
words
.reduce( transducer( listCombine ), [] )
.reduce( strConcat, "" );
// WRITTENSOMETHING
```

That's pretty good, but we have one final trick up our sleeve with transducing. And frankly, I think this part is what makes all this mental effort you've expended thus far, actually worth it.

Can we somehow "compose" these two `reduce(..)` calls to get it down to just one `reduce(..)`? Unfortunately, we can't just add `strConcat(..)` into the `compose(..)` call; because it's a reducer and not a combination-expecting function, its shape is not correct for the composition.

But let's look at these two functions side by side:

```js
function strConcat(str1,str2) { return str1 + str2; }

function listCombine(list,val) { list.push( val ); return list; }
```

If you squint your eyes, you can almost see how these two functions are interchangeable. They operate with different data types, but conceptually they do the same thing: combine two values into one.

In other words, `strConcat(..)` is a combination function!

That means we can use *it* instead of `listCombine(..)` if our end goal is to get a string concatenation rather than a list:

```js
words.reduce( transducer( strConcat ), "" );
// WRITTENSOMETHING
```

Boom! That's transducing for you. I won't actually drop the mic here, but just gently set it down...

## What, Finally

Take a deep breath. That was a lot to digest.

Clearing our brains for a minute, let's turn our attention back to just using transducing in our applications without jumping through all those mental hoops to derive how it works.

Recall the helpers we defined earlier; let's rename them for clarity:

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

Also recall that we use them like this:

```js
var transducer = compose(
    transduceMap( strUppercase ),
    transduceFilter( isLongEnough ),
    transduceFilter( isShortEnough )
);
```

`transducer(..)` still needs a combination function (like `listCombine(..)` or `strConcat(..)`) passed to it to produce a transduce-reducer function, which can then be used (along with an initial value) in `reduce(..)`.

But to express all these transducing steps more declaratively, let's make a `transduce(..)` utility that does these steps for us:

```js
function transduce(transducer,combinerFn,initialValue,list) {
    var reducer = transducer( combinerFn );
    return list.reduce( reducer, initialValue );
}
```

Here's our running example, cleaned up:

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

Not bad, huh!? See the `listCombine(..)` and `strConcat(..)` functions used interchangeably as combination functions?

### Transducers.js

Finally, let's illustrate our running example using the [`transducers-js` library](https://github.com/cognitect-labs/transducers-js):

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

Looks almost identical to above.

**Note:** The preceding snippet uses `transformers.comp(..)` because the library provides it, but in this case our [`compose(..)` from Chapter 4](ch4.md/#user-content-generalcompose) would produce the same outcome. In other words, composition itself isn't a transducing-sensitive operation.

The composed function in this snippet is named `transformer` instead of `transducer`. That's because if we call `transformer( listCombine )` (or `transformer( strConcat )`), we won't get a straight-up transduce-reducer function as earlier.

`transducers.map(..)` and `transducers.filter(..)` are special helpers that adapt regular predicate or mapper functions into functions that produce a special transform object (with the transducer function wrapped underneath); the library uses these transform objects for transducing. The extra capabilities of this transform object abstraction are beyond what we'll explore, so consult the library's documentation for more information.

Because calling `transformer(..)` produces a transform object and not a typical binary transduce-reducer function, the library also provides `toFn(..)` to adapt the transform object to be useable by native array `reduce(..)`:

```js
words.reduce(
    transducers.toFn( transformer, strConcat ),
    ""
);
// WRITTENSOMETHING
```

`into(..)` is another provided helper that automatically selects a default combination function based on the type of empty/initial value specified:

```js
transducers.into( [], transformer, words );
// ["WRITTEN","SOMETHING"]

transducers.into( "", transformer, words );
// WRITTENSOMETHING
```

When specifying an empty `[]` array, the `transduce(..)` called under the covers uses a default implementation of a function like our `listCombine(..)` helper. But when specifying an empty `""` string, something like our `strConcat(..)` is used. Cool!

As you can see, the `transducers-js` library makes transducing pretty straightforward. We can very effectively leverage the power of this technique without getting into the weeds of defining all those intermediate transducer-producing utilities ourselves.

## Summary

To transduce means to transform with a reduce. More specifically, a transducer is a composable reducer.

We use transducing to compose adjacent `map(..)`, `filter(..)`, and `reduce(..)` operations together. We accomplish this by first expressing `map(..)`s and `filter(..)`s as `reduce(..)`s, and then abstracting out the common combination operation to create unary reducer-producing functions that are easily composed.

Transducing primarily improves performance, which is especially obvious if used on an observable.

But more broadly, transducing is how we express a more declarative composition of functions that would otherwise not be directly composable. The result, if used appropriately as with all other techniques in this book, is clearer, more readable code! A single `reduce(..)` call with a transducer is easier to reason about than tracing through multiple `reduce(..)` calls.
