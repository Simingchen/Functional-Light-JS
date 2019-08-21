# 章节2: 函数的性质

函数式编程不仅仅是用“function”关键字进行定义来编程。如果这么简单的话，我可以在这里结束这本书！函数确实是函数式编程的核心。而且函数的方式使我们的代码*起作用*。

但是你有多确定“函数”的真正含义？

在这一章中，我们将通过探索函数的所有基本面，为本书的其余部分奠定基础。实际上，这是对所有内容的回顾，即使是非函数式编程程序员也应该了解函数。但当然，如果我们想从函数式编程的概念中得到最大的好处，我们就必须了解内部和外部的函数。

打起精神来，因为这个功能比你想象的要多得多。

## 什么是函数？

这个问题，从表面上看，似乎有一个显而易见的答案：函数是可以执行一次或多次的代码集合。

虽然这个定义看起来很合理，但它缺少一些非常重要的本质，即*函数*的核心，因为它适用于函数式编程。因此，让我们从表面开始挖掘，以更全面地理解函数。

### 简单的数学复习

我知道我答应过我们会尽量远离数学，但是在我们继续之前，请稍等片刻，因为我们很快就能从代数中观察到一些关于函数和图的基本知识。

你记得在学校里学过“f（x）”或者“y=f（x）”吗？

假设一个方程是这样定义的: <code>f(x) = 2x<sup>2</sup> + 3</code>. 那是什么意思？用图表表示这个方程意味着什么？下面是图表:

<p align="center">
    <img src="images/fig1.png" width="40%">
</p>

你能注意到的是，对于x的任何值，比如2，如果你把它插入方程，你得到11。11是什么？它是f（x）函数的返回值，前面我们说它代表y值。

换句话说，我们可以选择将输入和输出值解释为图中曲线上`（2，11）`处的点。对于我们插入的每一个'x'值，我们得到另一个'y'值，作为一个点的坐标与它配对。另一个是`（0,3）`，另一个是`（-1,5）`。把所有这些点放在一起，就得到了抛物线图，如图所示。

那么，这和函数式编程有什么关系呢？

在数学中，函数总是有输入并有输出。在函数式编程中经常听到的一个术语是’态射‘（morphism）；两个数学结构之间保持结构的一种过程抽象方法，例如与该函数的输出对应另一函数的输入。

在代数数学中，这些输入和输出通常被解释为要绘制图形的坐标的组成部分。然而，在我们的程序中，虽然很少被解释为图形上的可视绘制曲线，当时我们可以定义具有各种输入和输出的函数。

### 功能与程序

那么，为什么老是在讨论数学与图表呢？因为本质上，函数式编程就是在数学意义上接受使用函数方法作为特定程序。

您可能更习惯于将函数视为过程。有什么区别？过程是功能的任意集合。它可能有输入，也可能没有。它可能有输出（返回一个值），也可能没有。

函数接受输入，并且一定有一个“返回”值。

如果您计划进行函数编程，**您应该尽可能多地使用函数**，并尽可能避免使用过程。所有的“函数”都应该接受输入并返回输出。

为什么这么做？这有很多层面的意义，我们会在本书中揭露。

## 函数的输入

到目前为止，我们可以得出这样的结论：函数必须有输入。但让我们来深入研究函数输入是如何工作的。

您有时会听到人们将这些输入称为“参数”，有时称为“因素”。那这是怎么回事？

*参数*是您传入的值，*因素*是接收传入值的函数内的命名变量。例子：

```js
function foo(x,y) {
    // ..
}

var a = 3;

foo( a, a * 2 );
```
`“a”和“a*2”是函数“foo（…）”调用的*参数*，“x”和“y”是接收参数值的*参数*（分别为“3”和“6”（“a*2”的结果））。

**注意：**在javascript中，不要求*参数*的数量与函数*因素*的数量匹配。如果传递的*参数*多于声明接收它们的函数*参数*，那么这些值就不会受到影响。这些值可以通过几种不同的方式访问，包括以前可能听说过的“arguments”对象。如果传递的*参数*少于声明的函数*参数*，则每个不匹配的参数都将被视为“未定义”变量，这意味着它在函数的作用域内存在并可用，但只以空的“未定义”值开始。

### 默认参数

从ES6开始，参数可以声明*默认值*。如果没有传递该参数的参数，或者传递了值“undefined”，则将替换默认的赋值表达式。

想一想:

```js
function foo(x = 3) {
    console.log( x );
}

foo();                  // 3
foo( undefined );       // 3
foo( null );            // null
foo( 0 );               // 0
```
考虑有助于函数可用性的默认情况是一个很好的实践。然而，在读取和理解函数如何被调用的变化方面，默认参数可能会导致更复杂的问题。在多大程度上依赖此功能方面要谨慎。

### 计数输入

函数“预期”的参数个数由声明的参数个数决定，预期个数就是您可能希望传递给它的参数个数:

```js
function foo(x,y,z) {
    // ..
}
```
` foo（..）`需要三个参数，因为它有三个已声明的参数。这个计数有一个特殊的术语：参数数量。参数数量是指函数声明中的参数个数。“foo（…）”的参数数量为“3”。

此外，一个参数的函数也称为“一元”函数，二个参数的函数也称为“二元”函数，n个参数或更高的函数称为“n元”函数。


您可能希望在程序运行时检查函数引用以确定其参数个数。这可以通过函数引用的“length”属性来实现：

```js
function foo(x,y,z) {
    // ..
}

foo.length;             // 3
```
在执行期间确定参数个数的一个原因是，一段代码可以从多个地方引用一个函数，并根据参数个数的不同发送不同的值。

例如，假设一个情况，一个“fn”函数引用可能需要一个、两个或三个参数，但您总是希望在最后一个位置传递一个变量“x”:

```js
// `fn` 设置为某个函数引用
// `x` 存在一些值

if (fn.length == 1) {
    fn( x );
}
else if (fn.length == 2) {
    fn( undefined, x );
}
else if (fn.length == 3) {
    fn( undefined, undefined, x );
}
```
**提示：**函数的“length”属性是只读的，声明函数时就已经确定。它被看作是一段描述函数预期用途的元数据。

需要注意的一点是，某些类型的参数列表可能会使函数的“length”属性与您可能期望的不同：


```js
function foo(x,y = 2) {
    // ..
}

function bar(x,...args) {
    // ..
}

function baz( {a,b} ) {
    // ..
}

foo.length;             // 1
bar.length;             // 1
baz.length;             // 1
```
要计算当前函数调用接收到的参数个数？这曾经是微不足道的，但现在情况稍微复杂一些。每个函数都有一个“arguments”对象（类似于数组），用于保存对传入的每个参数的引用。然后可以检查“arguments”的“length”属性，以确定实际传递了多少个:

```js
function foo(x,y,z) {
    console.log( arguments.length );
}

foo( 3, 4 );    // 2
```

从ES5（特别是严格模式）开始，“arguments”被一些人认为是不赞成使用的；许多人会避免使用它。在JS中，我们“从不”破坏向后兼容性，不管这对将来的进展有多大帮助，所以“arguments”永远不会被删除。但现在普遍建议尽可能避免使用它

但是，我建议“arguments.length”，仅用于可以需要传递的参数数量的情况。未来版本的JS可能会添加一个功能，该功能可以确定传递的参数的数量，而不需要查询“arguments.length”；如果发生这种情况，那么我们可以完全放弃使用“arguments”！

小心：**从不**按位置访问参数，如“arguments[1]”。如果必须的话，只保留对“arguments.length”的引用。

另外，如何在声明参数之外的位置访问传递的参数？我稍后回答；但首先仔细考虑一下问问自己，“我为什么要这样做？”。

这种情况应该很少发生；它不应该是您在编写函数时经常需要使用的方式。如果您发现自己处于这样的场景中，请花费额外的20分钟尝试以不同的方式设计与该函数的交互。命名这个额外的论点，即使它是例外的。

接受不确定数量参数的函数称为可变函数。有些人更喜欢这种类型的功能设计，但我认为您会发现，一般情况下，函数编程人员通常会避免这样设计。

好吧，就这一点说得够多了。

假设您确实需要以类数组的方式访问参数，可能是您访问的参数在该位置没有正式参数的需要。那我们要怎么做？

ES6可以解决这一点！可以用`…`操作符来声明我们的函数——各种各样地称为“spread”、“rest”或“gather”（我的首选项）：

```js
function foo(x,y,z,...args) {
    // ..
}
```

看到参数列表中的“…args”？这是一个ES6声明形式，它告诉引擎收集所有未分配给命名参数的剩余参数，并将它们放入名为“args”的实数数组中。` args`将始终是一个数组，即使它是空的。但它**不会**包括分配给“x”、“y”和“z”参数的值，只包括超出前三个值的其他值：

```js
function foo(x,y,z,...args) {
    console.log( x, y, z, args );
}

foo();                  // undefined undefined undefined []
foo( 1, 2, 3 );         // 1 2 3 []
foo( 1, 2, 3, 4 );      // 1 2 3 [ 4 ]
foo( 1, 2, 3, 4, 5 );   // 1 2 3 [ 4, 5 ]
```

因此，如果您*真的*想设计一个函数来解释要传入的任意数量的参数，请在末尾使用“…args”（也可以使用其他变量名称）。现在，您将拥有一个真正的、不推荐使用的、不易出错的数组来访问这些参数值。

注意值“4”位于“args”的“0”位置，而不是“3”位置。它的“length”值不包括这三个“1”、“2”和“3”值。`…args'收集其他所有的参数，不包括'x'、'y'和'z'。

您*可以*使用参数列表中的“…”扩展运算符，即使没有声明其他正式参数：

```js
function foo(...args) {
    // ..
}
```
现在，“args”将是参数的完整数组，不管它们是什么，您可以使用“args.length”来确切知道传入了多少个参数。如果你这样做的话，使用“args[1]”或“args[317]”是安全的。不过，请不要传递318个参数。

### 参数数组

如果您想将一个数组作为参数传递给函数调用，该怎么办？

```js
function foo(...args) {
    console.log( args[3] );
}

var arr = [ 1, 2, 3, 4, 5 ];

foo( ...arr );                      // 4
```

上面“…”被使用了，但现在不仅在参数列表中；它也在调用函数的参数列表中使用。在这种情况下，它有相反的行为。在参数列表中，我们说它*收集*参数个数。在参数列表中，`...`是*展开*了数组的参数。所以“arr”的内容实际上是作为“foo（…）”调用的单个参数展开的。那这和仅仅传递一个对整个“arr”数组的引用有什么不同吗？

还有，多个值和`…`扩展参数可以交错：

```js
var arr = [ 2 ];

foo( 1, ...arr, 3, ...[4,5] );      // 4
```
思考这个“…”：在值列表位置，它是*展开*的功能。在赋值位置的参数列表收集参数，赋值给使用参数。

无论您调用哪种行为，`…`都会使处理参数数组更加容易。“slice（…）”、“concat（…）”和“apply（…）”的日子已经变得没有用了，这些方法都需要数组参数值。

**提示：**实际上，这些方法并不是完全无用的。在本书的整个代码中，我们将有一些地方依赖它们。但是，在大多数地方，`…`将会更加声明性地可读，因此更可取。

### 参数的解构

考虑上一节中的变量“foo（..）”：

```js
function foo(...args) {
    // ..
}

foo( ...[1,2,3] );
```

如果我们想改变这种交互，让函数的调用者传递一个数组，而不是单个参数值，该怎么办？试下这两种用法：

```js
function foo(args) {
    // ..
}

foo( [1,2,3] );
```

很简单。但是，如果现在我们想给传入数组中前两个值中的每一个都提供一个参数名呢？我们不再声明单个参数，我们似乎实现不了。

感谢ES6给出了答案。解构是一种为您希望看到的结构类型（对象、数组等）声明*模式*的方法，以及如何处理其各个部分的分解（分配）。

想一想:

<a name="funcparamdestr"></a>

```js
function foo( [x,y,...args] = [] ) {
    // ..
}

foo( [1,2,3] );
```

你看到`[ .. ]`参数列表的括号了吗？这称为数组参数解构。

在这个例子中，解构函数告诉引擎在这个分配位置需要一个数组。该模式表示取数组的第一个值并将其赋给名为“x”的局部参数变量，将第二个值赋给“y”，剩下的值将赋给“args”。

### 声明风格的重要性

思考下我们刚才被解构的“foo（…）”，我们可以手动处理参数：

```js
function foo(params) {
    var x = params[0];
    var y = params[1];
    var args = params.slice( 2 );

    // ..
}
```
但在这里，我们强调一个我们在[第1章](ch1.md/#readability)中提到的原则：声明性代码比命令式代码更有效地通信。

声明性代码（例如，前一个“foo（…）”代码段中的解构函数，或“…”运算符用法）将重点放在代码的结果上。

命令式代码（如后一段中的手动处理的函数）更关注如何获得结果。如果你以后读到这样的命令式代码，你必须关注执行所有的代码，以理解期望的结果。变得在那里被“编码”，很容易被细节所模糊。

前面的“foo（…）”被认为更具可读性，因为解构的方法隐藏了不必要的参数输入的细节；读者可以自由地只关注处理这些参数。这显然是最重要的关注点，所以读者应该集中精力来最全面地理解代码。

无论我们使用的语言和库/框架的允许程度怎样，只要可能，**我们都应该努力实现声明性、自解释的代码。**


## 命名参数

正如我们可以解构数组参数一样，我们也可以解构对象参数：

```js
function foo( {x,y} = {} ) {
    console.log( x, y );
}

foo( {
    y: 3
} );                    // undefined 3
```

我们将一个对象作为单个参数传入，它被分解为两个单独的参数变量“x”和“y”，它们从传入的对象中分配相应属性名的值。“x”属性不在对象上并不重要；它只是像您所期望的那样，以一个带有“undefined”的变量结束。

但是我希望您注意的是,参数对象解构的一部分对象被传递到“foo（…）”。

对于“foo（undefined，3）”这样的普通调用，位置表明了如何映射到函数的参数上；我们将“3”放在第二个位置，以将其分配给“y”参数。但是在参数解构的这种新的调用上，一个简单的对象参数值“3”对应分配给哪个参数（`Y`）。

我们不用解释那个调用的“x”，因为我们不关心“x”的使用。我们可以省略它，就不必做一些分散注意力的事情，比如将“undefined”作为位置占位符传递。

有些语言有一个明确的特性：命名参数。换句话说，在调用中，标记输入值以指示它映射到哪个参数。javascript没有命名参数，但参数对象解构是下一个最好的方法。

使用对象析构函数传递潜在多个参数的方法对函数式编程的好处是，只接受一个参数的函数更容易与另一个函数的单个输出组合。在[第4章](ch4.md)中有阐述的更详细。

### 无序参数

另一个重要的好处是，命名参数由于被指定为对象属性，所以没有从根本上进行排序。这意味着我们可以按照我们想要的任何顺序输入：

```js
function foo( {x,y} = {} ) {
    console.log( x, y );
}

foo( {
    y: 3
} );                    // undefined 3
```

我们只是简单地省略了'x'参数。当然我们也可以指定一个'x'参数，可以放在'y'的后面。这样调用的时候不用考虑x为“undefined”的有序占位符而忽略了参数。

从可读性的角度来看，命名参数更灵活，更具吸引力，尤其是当相关函数可以接受三个、四个或更多输入时。

**提示：**如果这种类型的函数参数对您有用或感兴趣，请查看 [附录C的函数式编程对象](apC.md/#bonus-fpo)。

## 函数的输出

让我们把注意力从函数的输入转移到函数的输出吧。

在JavaScript中，函数总是返回一个值。这三个函数都具有相同的“返回”行为：

```js
function foo() {}

function bar() {
    return;
}

function baz() {
    return undefined;
}
```

如果没有“return”，或者只有空的“return；”，则“undefined”值是隐式的“return”。

但是尽可能多地遵循函数编程定义的精神——使用函数而不是过程——我们的函数应该总是有输出，这意味着它们应该显式地“返回”一个值，而不是返回隐式的返回“undefined”。

“return”语句只能返回单个值。因此，如果函数需要返回多个值，唯一可行的选择是将它们收集到数组或对象这样的复合值中：

```js
function foo() {
    var retValue1 = 11;
    var retValue2 = 31;
    return [ retValue1, retValue2 ];
}
```
然后，我们从“foo（）”返回的两个对应项分配给“x”和“y”：
Then, we'll assign `x` and `y` from two respective items in the array that comes back from `foo()`:

```js
var [ x, y ] = foo();
console.log( x + y );           // 42
```
将多个值收集到数组（或对象）中以返回，然后将这些值解构回不同的赋值，是透明地表示函数的多个输出的一种方法。

**提示:**如果我不建议您花时间来考虑一个需要多个输出的函数是否可以重构来避免这种情况，或者将其分解为两个或更多更小的单用途函数，那就是我的疏忽了。有时是可能的，有时不是;但你至少应该考虑一下这种情况的存在。

### 提前返回

“return”语句不仅返回函数的值。它也是一个流控制结构；它在该点结束了函数的执行。因此，具有多个“return”语句的函数具有多个可能的退出点，这意味着如果有多条路径可以生成该输出，则可能难以读取函数以了解其输出行为。

想一想:

```js
function foo(x) {
    if (x > 10) return x + 1;

    var y = x / 2;

    if (y > 3) {
        if (x % 2 == 0) return x;
    }

    if (y > 1) return y;

    return x;
}
```
小测验：如果不作弊，也不在浏览器中运行此代码，那么'foo（2）'返回什么？那么“foo（4）”呢？还有“foo（8）”？还有“foo（12）”？

你对自己的答案有信心吗?为了得到这些答案，你付了多少精神税?前两次我都想错了!

我认为可读性也是一部分问题，我们使用“return”不仅返回不同的值，而且作为流控制结构在某些情况下提前退出函数的执行。显然有更好的方法来编写流控制(“if”逻辑，等等)，但是我也认为有一些方法可以使输出路径更加明显。

**注：**小测验的答案是'2`、'2`、'8`和'13`。

考虑下该版本的代码：

```js
function foo(x) {
    var retValue;

    if (retValue == undefined && x > 10) {
        retValue = x + 1;
    }

    var y = x / 2;

    if (y > 3) {
        if (retValue == undefined && x % 2 == 0) {
            retValue = x;
        }
    }

    if (retValue == undefined && y > 1) {
        retValue = y;
    }

    if (retValue == undefined) {
        retValue = x;
    }

    return retValue;
}
```
这个版本无疑更加冗长。但我认为遵循这种逻辑稍微简单一些，因为可以设置“retValue”的每个分支都由检查它是否已经设置的条件“保护”。

我们使用常规流控制(' if '逻辑)来确定' retValue '的赋值，而不是从函数提前'返回'。最后，我们简单地返回“retValue”。

我并不是无条件地说您应该总是有一个单一的“返回”，或者您永远不应该做提前的“返回”，但是我确实认为您应该注意“返回”的流控制部分，它在函数定义中创建了更多的含义。试着找出最明确的表达逻辑的方法;这通常是最好的方法。

### Un`return`ed Outputs 取消返回的输出

您可能已经在编写的大多数代码中使用了一种技术，甚至可能没有考虑太多，就是让一个函数通过简单地改变自身外部的变量来输出其部分或全部值。

还记得我们在本章前面<code>f(x) = 2x<sup>2</sup> + 3</code>的函数吗？我们可以在JS中这样定义它：

```js
var y;

function f(x) {
    y = (2 * Math.pow( x, 2 )) + 3;
}

f( 2 );

y;                      // 11
```

我知道这是一个愚蠢的例子；我们可以很容易地将值返回，而不是从函数中将其设置为“y”：

```js
function f(x) {
    return (2 * Math.pow( x, 2 )) + 3;
}

var y = f( 2 );

y;                      // 11
```

两个函数都完成了相同的任务，所以我们有理由选择这个版本而不是另一个版本吗？回答：**是的，当然可以。**

解释差异的一种方法是，后一个版本中的“返回”表示显式输出，而前一个版本中的“y”赋值是隐式输出。这么说的话，估计你知道怎么做了；通常，开发人员更喜欢显式模式而不是隐式模式。

但是，在外部作用域中更改变量，正如我们在“foo（…）”中使用“y”赋值所做的那样，这只是实现隐式输出的一种方法。一个更微妙的例子是通过引用对非本地值进行更改。

想一想:

```js
function sum(list) {
    var total = 0;
    for (let i = 0; i < list.length; i++) {
        if (!list[i]) list[i] = 0;

        total = total + list[i];
    }

    return total;
}

var nums = [ 1, 3, 9, 27, , 84 ];

sum( nums );            // 124
```

这个最明显是sum函数的输出`124`，我们显式地‘返回’。但是您发现了其他输出吗？尝试该代码，然后检查“nums”数组。现在你发现区别了吗？

数组位置“4”中现在有一个“0”而不是“undefined”空值，看起来无害的' list[i] = 0 '操作最终影响了外部的数组值，即使我们在本地的“list”参数变量上操作。

为什么？因为从函数中创建了异常输出，'list'保存了'nums'引用的引用副本，而不是`[1,3,9，..]`数组值的值副本。通常javascript使用数组、对象和函数的引用和引用副本。

## 函数的功能

函数可以接收和返回任何类型的值。接收或返回一个或多个其他函数值的函数具有特殊名称：高阶函数。

想一想:

```js
function forEach(list,fn) {
    for (let v of list) {
        fn( v );
    }
}

forEach( [1,2,3,4,5], function each(val){
    console.log( val );
} );
// 1 2 3 4 5
```

` foreach（..）`是一个高阶函数，因为它接收一个函数作为了参数。

高阶函数也可以输出另一个函数，例如：

```js
function foo() {
    return function inner(msg){
        return msg.toUpperCase();
    };
}

var f = foo();

f( "Hello!" );          // HELLO!
```

`return`不是“输出”内部函数的唯一方法：

```js
function foo() {
    return bar( function inner(msg){
        return msg.toUpperCase();
    } );
}

function bar(func) {
    return func( "Hello!" );
}

foo();                  // HELLO!
```

根据定义，将其他函数视为值的函数是高阶函数。函数式编程人员一直在写这些！

### 作用域的保持

在所有编程中，尤其是在函数式编程中，最强大的功能之一就是一个函数在另一个函数的作用域中时的行为。当内部函数引用外部函数中的变量时，这称为闭包。

实用性的定义:

> 闭包是指当一个函数从它自己的作用域之外记住和访问变量时，即使这个函数是在另一个作用域中执行的。

想一想:

```js
function foo(msg) {
    var fn = function inner(){
        return msg.toUpperCase();
    };

    return fn;
}

var helloFn = foo( "Hello!" );

helloFn();              // HELLO!
```

'foo（..）'作用域内的'msg'参数变量在内部函数内被引用。当执行“foo（..）”并创建内部函数时，它捕获对“msg”变量的访问，并且即使在“return”之后仍然保留该访问。

一旦我们定义了“hellofn”，对内部函数“foo（…）”的引用就结束了，它的作用域似乎应该消失了，这意味着“msg”变量将不再存在。但并不是这样，因为内部函数在“msg”上有一个闭包，使其保持活动状态。只要内部函数（现在由另一个作用域中的“hellofn”引用）保持不变，封闭的“msg”变量就会一直存在。

让我们再看几个实际中的闭包示例：

```js
function person(name) {
    return function identify(){
        console.log( `I am ${name}` );
    };
}

var fred = person( "Fred" );
var susan = person( "Susan" );

fred();                 // I am Fred
susan();                // I am Susan
```

在参数为“name”的内部函数“identify（）”存在闭包。

闭包启用的访问不仅限于读取变量的原始值——它不仅仅是一个快照，而是一个活动链接。您可以更新该值，并且新的状态将一直保留到下一次访问：

```js
function runningCounter(start) {
    var val = start;

    return function current(increment = 1){
        val = val + increment;
        return val;
    };
}

var score = runningCounter( 0 );

score();                // 1
score();                // 2
score( 13 );            // 15
```

**警告：**我们将在本书后面更深入地探讨这个问题，这个使用闭包来记住更改（`val`）的状态的示例可能是您希望尽可能避免的。

如果有一个操作需要两个输入，其中一个现在知道，另一个稍后将被指定，则可以使用闭包记住第一个输入：

```js
function makeAdder(x) {
    return function sum(y){
        return x + y;
    };
}

// 我们已经知道“10”和“37”分别作为第一个输入
var addTo10 = makeAdder( 10 );
var addTo37 = makeAdder( 37 );

// 稍后，我们将指定第二个输入
addTo10( 3 );           // 13
addTo10( 90 );          // 100

addTo37( 13 );          // 50
```

通常，“sum（..）”函数会同时使用“x”和“y”输入将它们添加到一起。但在本例中，我们首先接收并记住（通过闭包）x值，而y值则在后面单独指定。

**注:**这种在连续函数调用中指定输入的技术在函数式编程中非常常见，有两种形式:局部应用和局部套用。我们将在[第三章](ch3.md/#some-now-some-later)中更深入地研究它们。

当然，由于函数只是JS中的值，我们可以通过闭包来记住函数值:

```js
function formatter(formatFn) {
    return function inner(str){
        return formatFn( str );
    };
}

var lower = formatter( function formatting(v){
    return v.toLowerCase();
} );

var upperFirst = formatter( function formatting(v){
    return v[0].toUpperCase() + v.substr( 1 ).toLowerCase();
} );

lower( "WOW" );             // wow
upperFirst( "hello" );      // Hello
```

函数式编程鼓励我们创建简单的函数来封装这种行为，而不是在代码中到处分发/重复“toUpperCase()”和“toLowerCase()”逻辑。

具体来说，我们创建了两个简单的一元函数“lower(..)”和“upperFirst(..)”，因为这些函数将更容易与程序其余部分中的其他函数连接起来。

**提示：**您是否发现“upperfirst（..）”如何使用“lower（..）”？

我们将在本文的其余部分大量使用闭包。它可能只是所有函数式编程中最重要的基础实践，如果不是作为一个整体进行编程的话。相信你很满意！

## 语法

在我们从这个函数入门开始之前，让我们花点时间来讨论它们的语法。

与本文的许多其他部分相比，本节中的讨论大多是意见和偏好，无论您是否同意此处提出的观点或采取相反的观点。这些想法是非常主观的，尽管许多人似乎对它们有相当绝对的感觉。

不过，最后你要做决定。

### 命名

从语法上讲，函数声明需要包含一个名称：

```js
function helloMyNameIs() {
    // ..
}
```

但是函数表达式可以有命名和匿名两种形式：

```js
foo( function namedFunctionExpr(){
    // ..
} );

bar( function(){    // <-- 看这, 未进行命名!
    // ..
} );
```

顺便问一下，匿名到底是什么意思？具体地说，函数有一个“name”属性，它保存函数语法上给定的名称的字符串值，例如“hellomyname”或“namedfunctionexpr”。JS环境中的控制台/开发人员工具最显著地使用此“name”属性来列出函数参与堆栈跟踪时的列表（通常来自异常）。

匿名函数通常显示为`(anonymous function)`。

如果您必须从异常的堆栈跟踪中调试JS程序，那么您可能会感到看到`（匿名函数）`一行接一行出现的痛苦。并没有给开发人员任何关于异常来源路径的线索。它对开发人员没有任何帮助。

如果你是想使用命名函数表达式，一定要定义名称。因此，如果您使用像“handleprofileclicks”这样的好名称而不是“foo”，您将得到更多有用的堆栈跟踪。

从ES6开始，在某些情况下，匿名函数表达式由定义的名字辅助
想一想:

```js
var x = function(){};

x.name;         // x
```

如果引擎能够猜出您*可能*想要函数取什么名称，它将继续执行并执行此操作。

但要注意，并非所有的句法形式都能从名称推断中受益。函数表达式出现的最常见地方可能是作为函数调用的参数：

```js
function foo(fn) {
    console.log( fn.name );
}

var x = function(){};

foo( x );               // x
foo( function(){} );    //
```

当不能从直接的周围语法推断出名称时，它仍然是一个空字符串。这样的函数将在堆栈跟踪中报告为“匿名函数”（anonymous function）。

除了调试问题外，对正在命名的函数还有其他好处。首先，句法名称（又称词汇名称）对于内部自引用很有用。自引用对于递归（同步和异步）是必需的，并且对事件处理程序也很有帮助。

考虑这些不同的场景：

```js
// 同步递归：
function findPropIn(propName,obj) {
    if (obj == undefined || typeof obj != "object") return;

    if (propName in obj) {
        return obj[propName];
    }
    else {
        for (let prop of Object.keys( obj )) {
            let ret = findPropIn( propName, obj[prop] );
            if (ret !== undefined) {
                return ret;
            }
        }
    }
}
```

```js
// 异步递归：
setTimeout( function waitForIt(){
    // does `it` exist yet?
    if (!o.it) {
        // try again later
        setTimeout( waitForIt, 100 );
    }
}, 100 );
```

```js
// 事件处理解除绑定
document.getElementById( "onceBtn" )
    .addEventListener( "click", function handleClick(evt){
        // 解除绑定
        evt.target.removeEventListener( "click", handleClick, false );

        // ..
    }, false );
```

在所有这些情况下，命名函数的词法名称从内部来说是一个有用且可靠的自引用。

此外，即使在只有一个线性函数的简单情况下，命名它们也会使代码更易于解释，因此对于以前没有阅读过它的人来说，更容易阅读：

```js
people.map( function getPreferredName(person){
    return person.nicknames[0] || person.firstName;
} )
// ..
```

函数命名为“getpreferredname（..）”告诉读者一些关于映射操作要做什么的事情，而不仅仅是从其代码中看是显而易见的。此名称标签有助于代码更易于阅读。

匿名函数表达式常见的另一个地方是立即调用的函数表达式（IIFES）：

```js
(function(){

    // 看，这是立即调用函数

})();
```
实际上，您不会看到IIFEs在函数表达式中使用名称，但它们应该使用名称。为什么？出于所有相同的原因，我们刚刚讨论了：堆栈跟踪调试、可靠的自引用和可读性。如果你想不出你的生活的其他名字，至少要用“IIFE”这个词：

```js
(function IIFE(){

    // 你已经知道我是立即调用函数

})();
```

我的意思是，为什么**命名函数总是比匿名函数更可取，原因有很多。**事实上，我想说的是，基本上没有比匿名函数更好的情况了。他们只是没有任何优势比他们的命名对手。

编写匿名函数是非常容易的，因为这样我们就少了一个名字来花心思去计算。

我将诚实;我和其他人一样对此感到内疚。我不喜欢纠结于命名。我为函数想到的头几个名字通常都不好。我得一遍又一遍地重新考虑这个名字。我宁愿使用一个好的匿名函数表达式。

但我们正在用写作的简单性来换取阅读的痛苦。这不是一个好的权衡。懒惰或缺乏创造性，以至于不想为函数指定名称，这是使用匿名函数的常见但糟糕的借口。

**为每个函数命名。**如果你坐在那里手足无措，想不出一个适合你写的函数的好名字，我强烈建议你还没有完全理解这个函数的用途——或者它太宽泛或太抽象了。您需要返回并重新设计函数，直到这一点变得更加清晰。到那时，名字就会变得更加明显。

在我的实践中，如果我没有一个好的函数名可以使用，我最初将其命名为' TODO '。我确信，在提交代码之前搜索“TODO”注释时，我至少会捕捉到这一点。

我可以从我自己的经验中证明，在努力为某个东西命名时，我通常会更好地理解它，甚至经常重构它的设计以提高可读性和可维护性。

这次投资很值得。

### Functions Without `function`

So far we've been using the full canonical syntax for functions. But you've no doubt also heard all the buzz around the ES6 `=>` arrow function syntax.

Compare:

```js
people.map( function getPreferredName(person){
    return person.nicknames[0] || person.firstName;
} );

// vs.

people.map( person => person.nicknames[0] || person.firstName );
```

Whoa.

The keyword `function` is gone, so is `return`, the parentheses (`( )`), the curly braces (`{ }`), and the innermost semicolon (`;`). In place of all that, we used a so-called fat arrow symbol (`=>`).

But there's another thing we omitted. Did you spot it? The `getPreferredName` function name.

That's right; `=>` arrow functions are lexically anonymous; there's no way to syntactically provide it a name. Their names can be inferred like regular functions, but again, the most common case of function expression values passed as arguments won't get any assistance in that way. Bummer.

If `person.nicknames` isn't defined for some reason, an exception will be thrown, meaning this `(anonymous function)` will be at the top of the stack trace. Ugh.

Honestly, the anonymity of `=>` arrow functions is a `=>` dagger to the heart, for me. I cannot abide by the loss of naming. It's harder to read, harder to debug, and impossible to self-reference.

But if that wasn't bad enough, the other slap in the face is that there's a whole bunch of subtle syntactic variations that you must wade through if you have different scenarios for your function definition. I'm not going to cover all of them in detail here, but briefly:

```js
people.map( person => person.nicknames[0] || person.firstName );

// multiple parameters? need ( )
people.map( (person,idx) => person.nicknames[0] || person.firstName );

// parameter destructuring? need ( )
people.map( ({ person }) => person.nicknames[0] || person.firstName );

// parameter default? need ( )
people.map( (person = {}) => person.nicknames[0] || person.firstName );

// returning an object? need ( )
people.map( person =>
    ({ preferredName: person.nicknames[0] || person.firstName })
);
```

The case for excitement over `=>` in the FP world is primarily that it follows almost exactly from the mathematical notation for functions, especially in FP languages like Haskell. The shape of `=>` arrow function syntax communicates mathematically.

Digging even further, I'd suggest that the argument in favor of `=>` is that by using much lighter-weight syntax, we reduce the visual boundaries between functions which lets us use simple function expressions much like we'd use lazy expressions -- another favorite of the FPer.

I think most FPers are going to wave off the concerns I'm sharing. They love anonymous functions and they love saving on syntax. But like I said before: you decide.

**Note:** Though I do not prefer to use `=>` in practice in my production code, they are useful in quick code explorations. Moreover, we will use arrow functions in many places throughout the rest of this book -- especially when we present typical FP utilities -- where conciseness is preferred to optimize for the limited physical space in code snippets. Make your own determinations whether this approach will make your own production-ready code more or less readable.

## What's This?

If you're not familiar with the `this` binding rules in JavaScript, I recommend checking out my book *You Don't Know JS: this & Object Prototypes*. For the purposes of this section, I'll assume you know how `this` gets determined for a function call (one of the four rules). But even if you're still fuzzy on *this*, the good news is we're going to conclude that you shouldn't be using `this` if you're trying to do FP.

**Note:** We're tackling a topic that we'll ultimately conclude we shouldn't use. Why!? Because the topic of `this` has implications for other topics covered later in this book. For example, our notions of function purity are impacted by `this` being essentially an implicit input to a function (see [Chapter 5](ch5.md)). Additionally, our perspective on `this` affects whether we choose array methods (`arr.map(..)`) versus standalone utilities (`map(..,arr)`) (see [Chapter 9](ch9.md)). Understanding `this` is essential to understanding why `this` really should *not* be part of your FP!

JavaScript `function`s have a `this` keyword that's automatically bound per function call. The `this` keyword can be described in many different ways, but I prefer to say it provides an object context for the function to run against.

`this` is an implicit parameter input for your function.

Consider:

```js
function sum() {
    return this.x + this.y;
}

var context = {
    x: 1,
    y: 2
};

sum.call( context );        // 3

context.sum = sum;
context.sum();              // 3

var s = sum.bind( context );
s();                        // 3
```

Of course, if `this` can be input into a function implicitly, the same object context could be sent in as an explicit argument:

```js
function sum(ctx) {
    return ctx.x + ctx.y;
}

var context = {
    x: 1,
    y: 2
};

sum( context );
```

Simpler. And this kind of code will be a lot easier to deal with in FP. It's much easier to wire multiple functions together, or use any of the other input wrangling techniques we will get into in the next chapter, when inputs are always explicit. Doing them with implicit inputs like `this` ranges from awkward to nearly impossible depending on the scenario.

There are other tricks we can leverage in a `this`-based system, including prototype-delegation (also covered in detail in *You Don't Know JS: this & Object Prototypes*):

```js
var Auth = {
    authorize() {
        var credentials = `${this.username}:${this.password}`;
        this.send( credentials, resp => {
            if (resp.error) this.displayError( resp.error );
            else this.displaySuccess();
        } );
    },
    send(/* .. */) {
        // ..
    }
};

var Login = Object.assign( Object.create( Auth ), {
    doLogin(user,pw) {
        this.username = user;
        this.password = pw;
        this.authorize();
    },
    displayError(err) {
        // ..
    },
    displaySuccess() {
        // ..
    }
} );

Login.doLogin( "fred", "123456" );
```

**Note:** `Object.assign(..)` is an ES6+ utility for doing a shallow assignment copy of properties from one or more source objects to a single target object: `Object.assign( target, source1, ... )`.

In case you're having trouble parsing what this code does: we have two separate objects `Login` and `Auth`, where `Login` performs prototype-delegation to `Auth`. Through delegation and the implicit `this` context sharing, these two objects virtually compose during the `this.authorize()` function call, so that properties/methods on `this` are dynamically shared with the `Auth.authorize(..)` function.

*This* code doesn't fit with various principles of FP for a variety of reasons, but one of the obvious hitches is the implicit `this` sharing. We could be more explicit about it and keep code closer to FP-friendly style:

```js
// ..

authorize(ctx) {
    var credentials = `${ctx.username}:${ctx.password}`;
    Auth.send( credentials, function onResp(resp){
        if (resp.error) ctx.displayError( resp.error );
        else ctx.displaySuccess();
    } );
}

// ..

doLogin(user,pw) {
    Auth.authorize( {
        username: user,
        password: pw
    } );
}

// ..
```

From my perspective, the problem is not with using objects to organize behavior. It's that we're trying to use implicit input instead of being explicit about it. When I'm wearing my FP hat, I want to leave `this` stuff on the shelf.

## Summary

Functions are powerful.

But let's be clear what a function is. It's not just a collection of statements/operations. Specifically, a function needs one or more inputs (ideally, just one!) and an output.

Functions inside of functions can have closure over outer variables and remember them for later. This is one of the most important concepts in all of programming, and a fundamental foundation of FP.

Be careful of anonymous functions, especially `=>` arrow functions. They're convenient to write, but they shift the cost from author to reader. The whole reason we're studying FP here is to write more readable code, so don't be so quick to jump on that bandwagon.

Don't use `this`-aware functions. Just don't.

You should now be developing a clear and colorful perspective in your mind of what *function* means in Functional Programming. It's time to start wrangling functions to get them to interoperate, and the next chapter teaches you a variety of critical techniques you'll need along the way.
