# 章节 4: 组合函数

到目前为止，我希望您对使用函数进行函数编程的含义感到更舒服。

函数式程序员把程序中的每个函数都看作是一个简单的乐高积木。他们一眼就能认出蓝色的2x2砖块，并且确切地知道它是如何工作的，以及他们可以用它做什么。当他们开始建造一个更大、更复杂的乐高模型时，因为他们需要每一块积木，他们已经有了一种本能，从他们的许多备用积木中找出哪一块来。

但有时你把蓝色的2x2块和灰色的4x1块以某种方式放在一起，你会意识到，“这是我经常需要的一块有用的砖”。

所以现在你已经想出了一个新的“片段”，两个其他片段的组合，你现在可以在任何你需要的时候得到这种片段。识别并在需要的地方使用这种蓝灰色的复合l形砖要比每次单独考虑组装两个单独的砖更有效。

函数有各种形状和大小。我们可以定义它们的特定组合来创建一个新的复合函数，这个函数在程序的各个部分都很方便。将函数一起使用的过程称为组合。

组合是指函数编程人员如何对程序中的数据流建模。从某种意义上说，它是函数编程中最基本的概念，因为没有它，您就不能声明性地对数据和状态更改建模。换句话说，如果没有组分，函数编程中的其他所有东西都会崩溃。

## 输出到输入

我们已经看到了一些合成的例子。例如，我们在第3章中讨论[`unary(..)`](ch3.md/#user-content-unary)时包含了这样一个表达式:[' [..].map(unary(parseInt))](ch3.md/#user-content-mapunary)。想想发生了什么。

要将两个函数组合在一起，请将第一个函数调用的输出作为第二个函数调用的输入。在`map(unary(parseInt))`中，`unary(parseInt)`调用返回一个值(一个函数);该值直接作为参数传递给 `map(..)`，后者返回一个数组。

后退一步考虑并结合可视化数据的概念流:

```txt
arrayValue <-- map <-- unary <-- parseInt
```

`parseInt` 是`unary(..)`的输入值. `unary(..)` 的输出值是`map(..)`的输入值. `map(..)` 的输出值是 `arrayValue`. 这就是 `map(..)` 与 `unary(..)`的组合.

**注意:**这里的从右到左的方向是有意这样的，尽管在您的学习过程中这一点可能看起来很奇怪。我们稍后会详细解释。

把这个数据流想象成糖果工厂的传送带，每个操作都是冷却、切割和包装糖果过程中的一个步骤。我们将在本章中使用糖果工厂的比喻来解释什么是组合。

<p align="center">
    <img src="images/fig2.png">
</p>

让我们一步一步地研究一下实际的作文。考虑一下您的程序中可能具有的这两个实用程序:

```js
function words(str) {
    return String( str )
        .toLowerCase()
        .split( /\s|\b/ )
        .filter( function alpha(v){
            return /^[\w]+$/.test( v );
        } );
}

function unique(list) {
    var uniqList = [];

    for (let v of list) {
        // value not yet in the new list?
        if (uniqList.indexOf( v ) === -1 ) {
            uniqList.push( v );
        }
    }

    return uniqList;
}
```

`words(..)`将字符串分割成单词数组。`unique(..)`获取一个单词列表，并对其进行筛选，使其不包含任何重复的单词。

要使用这两个实用程序来分析一段文本字符串:

```js
var text = "To compose two functions together, pass the \
output of the first function call as the input of the \
second function call.";

var wordsFound = words( text );
var wordsUsed = unique( wordsFound );

wordsUsed;
// ["to","compose","two","functions","together","pass",
// "the","output","of","first","function","call","as",
// "input","second"]
```

我们将`words(..)`的数组输出命名为`wordsFound`。`unique(..)`的输入也是一个数组，因此我们可以将 `wordsFound`传递给它。

回到糖果工厂的装配线:第一台机器将融化的巧克力作为“输入”，它的“输出”是一块成形并冷却的巧克力。下一台机器沿着装配线向下一点，它的“输入”是一块巧克力，“输出”是一块切碎的巧克力糖。接下来，生产线上的一台机器从传送带上取下小块巧克力糖，并输出包装好的糖果，准备打包运输。

<img src="images/fig3.png" align="right" width="9%" hspace="20">

糖果工厂在这一过程中相当成功，但与所有企业一样，管理层一直在寻找增长的途径。

为了满足更多糖果生产的需求，他们决定拿出传送带装置，把三台机器一台台叠起来，其中一台的输出阀直接连接到下一台的输入阀上。再也不会有大块巧克力在传送带上缓慢而嘈杂地从一台机器传到另一台机器的浪费空间了。

这一创新为工厂节省了很多空间，所以管理层很高兴他们能每天生产更多的糖果!

与此改进的糖果生产配置等价的代码是跳过中间步骤(前面代码片段中的`wordsFound`变量)，只使用两个函数调用:

```js
var wordsUsed = unique( words( text ) );
```

**注意:**虽然我们通常从左到右读取函数调用`unique(..)`和`words(..)`，但是操作的顺序实际上会从右到左，或者从内到外。首先运行`words(..)`，然后运行`unique(..)`。稍后，我们将讨论一种模式，它将执行顺序与从左到右的自然读取匹配，称为`pipe(..)`。

堆叠的机器工作得很好，但是到处都挂着电线有点笨重。他们创建的机器栈越多，工厂的地板就越杂乱。组装和维护所有这些机器堆栈的工作非常耗时。

<img src="images/fig4.png" align="left" width="15%" hspace="20">

一天早上，糖果厂的一位工程师有了一个好主意。她认为如果她做一个外部盒子来隐藏所有的电线会更有效率;在内部，所有的三个机器都连接在一起，而在外部，一切都是干净整洁的。在这台新机器的顶部是一个阀门，可以将融化的巧克力倒入机器，底部是一个阀门，可以吐出包装好的巧克力糖果。这样太棒了!

这台单台复合机器更容易移动和安装在工厂需要的任何地方。工厂里的工人们更开心，因为他们不再需要摆弄三台机器上的按钮和刻度盘;他们很快就喜欢用单台花哨的机器。

与代码相关:我们现在意识到`words(..)` 和 `unique(..)`以特定的执行顺序(想想:复合乐高)的配对可以在应用程序的其他几个部分中使用。所以，让我们定义一个复合函数来组合它们:

```js
function uniqueWords(str) {
    return unique( words( str ) );
}
```

`uniqueWords(..)` 接受一个字符串并返回一个数组。它由两个函数组成:`unique(..)` 和 `words(..)`;它创建了这个数据流:

```txt
wordsUsed <-- unique <-- words <-- text
```

您现在可能已经认识到:糖果工厂设计中正在展开的革命是功能组合。

### 机器制造

糖果厂生意兴隆，由于节省了不少空间，他们现在有足够的空间来尝试制造新型糖果。在早期成功的基础上，管理层热衷于不断为他们不断增长的糖果品种发明新的神奇的复合机器。

但是工厂的工程师们很难跟上步伐，因为每次需要制造一种新型的高级复合机器时，他们都要花费相当多的时间来制造新的外箱，并将各个机器安装到其中。

因此，工厂的工程师联系一个工业机器供应商寻求帮助。他们惊奇地发现这个供应商提供**机器制造**机器!听起来不可思议的是，他们购买了一台机器，可以把工厂里的几台较小的机器——比如冷却巧克力的机器和切割巧克力的机器——自动连接起来，甚至用一个干净的大盒子把它们包起来。这一定会使糖果厂真正腾飞!

<p align="center">
    <img src="images/fig5.png" width="50%">
</p>

回到代码领域，让我们考虑一个名为`compose2(..)`的实用程序，它自动创建两个函数的组合，与我们手动创建的方法完全相同:

```js
function compose2(fn2,fn1) {
    return function composed(origValue){
        return fn2( fn1( origValue ) );
    };
}

// ES6 箭头格式
var compose2 =
    (fn2,fn1) =>
        origValue =>
            fn2( fn1( origValue ) );
```

您是否注意到，我们将参数顺序定义为`fn2,fn1`，而且它是第一个运行的第二个列出的函数(又名`fn1`参数名)，然后是第一个列出的函数(`fn2`)?换句话说，函数从右到左组成。

这似乎是一个奇怪的选择，但有一些原因。大多数典型的FP库都将它们的`compose(..)`定义为按顺序从右向左工作，所以我们坚持这个约定。

但是为什么呢?我认为最简单的解释(但可能不是历史上最准确的解释)是，我们列出它们是为了匹配手工编写代码的顺序，或者更确切地说，是从左到右阅读时遇到的顺序。

`unique(words(str))` 以从左到右的顺序列出函数`unique, words`，因此我们让我们的`compose2(..)`实用程序也按这个顺序接收它们。执行顺序是从右到左，但是代码顺序是从左到右。密切关注那些在你脑海中清晰的东西。

现在，对糖果机更有效的定义是:

```js
var uniqueWords = compose2( unique, words );
```

### 成分变异

看起来`<-- unique <-- words`组合是这两个函数可以组合的惟一顺序。但实际上我们可以把它们按相反的顺序组合在一起，从而创建一个具有不同用途的实用程序:

```js
var letters = compose2( words, unique );

var chars = letters( "How are you Henry?" );
chars;
// ["h","o","w","a","r","e","y","u","n"]
```

这是因为`words(..)` 实用程序出于值类型安全的考虑，首先使用`String(..)`将其输入强制转换为字符串。因此，`unique(..)`返回的数组——现在是`words(..)`的输入——变成了字符串`"H,o,w, ,a,r,e,y,u,n,?"`，然后`words(..)`中的其他行为将处理该字符串到 `chars` 数组中。

诚然，这是一个人为的例子。但重点是函数组合并不总是单向的。有时我们把灰砖放在蓝砖的上面，有时我们把蓝砖放在上面。

糖果工厂最好小心，如果他们试图把包装好的糖果放进机器，混合和冷却巧克力!

## 一般组成

如果我们能定义两个函数的组合，我们就可以继续支持组合任意数量的函数。组成的任意数量函数的常规数据可视化流如下所示：

```txt
finalValue <-- func1 <-- func2 <-- ... <-- funcN <-- origValue
```

<p align="center">
    <img src="images/fig6.png" width="50%">
</p>

现在糖果厂拥有所有机器中最好的一台:这台机器可以取任意数量的独立的小机器，然后吐出一台大而别致的机器，按顺序执行每一步。这可真是个糖果店啊!这是威利·旺卡的梦想!

我们可以实现一个通用的 `compose(..)` 实用程序如下:

<a name="generalcompose"></a>

```js
function compose(...fns) {
    return function composed(result){
        // copy the array of functions
        var list = [...fns];

        while (list.length > 0) {
            // take the last function off the end of the list
            // and execute it
            result = list.pop()( result );
        }

        return result;
    };
}

// ES6 箭头格式
var compose =
    (...fns) =>
        result => {
            var list = [...fns];

            while (list.length > 0) {
                // take the last function off the end of the list
                // and execute it
                result = list.pop()( result );
            }

            return result;
        };
```

**警告：**`fns` 是一个集合的参数数组，不是传入的数组，因此 `compose(..)`是本地的。可能会有人认为 `[...fns]` 是不必要的。但是，在这个特定的实现中，内部的`composed(..)`函数中的`.pop()`正在改变列表，因此如果我们每次都不进行复制，则返回的 `composed(..)` 函数只能可靠地使用一次。我们将在[Chapter 6](ch6.md/#user-content-hiddenmutation)中重新讨论此危害。

现在我们来看一个组成两个以上函数的例子。回顾我们的`uniqueWords(..)` 组合示例，让我们在组合中添加一个`skipShortWords(..)`：

```js
function skipShortWords(words) {
    var filteredWords = [];

    for (let word of words) {
        if (word.length > 4) {
            filteredWords.push( word );
        }
    }

    return filteredWords;
}
```

让我们定义 `biggerWords(..)`，其中包括`skipShortWords(..)`。手动合成相当于`skipShortWords( unique( words( text ) ) )`，所以让我们用`compose(..)`来实现:

```js
var text = "To compose two functions together, pass the \
output of the first function call as the input of the \
second function call.";

var biggerWords = compose( skipShortWords, unique, words );

var wordsUsed = biggerWords( text );

wordsUsed;
// ["compose","functions","together","output","first",
// "function","input","second"]
```

要对组合做一些更有趣的事情，让我们使用这是我们在第3章中首先看到的[`partialRight(..)`](ch3.md/#user-content-partialright)。我们可以构建一个`compose(..)`本身的函数式编程中局部应用，预先指定第二个和第三个参数(`unique(..)` 和 `words(..)`;我们将其称为`filterWords(..)`。

然后，我们可以通过调用`filterWords(..)`多次完成合成，但分别使用不同的第一个参数：

```js
// 注意：使用`<=4'检查而不是`>4'检查
// `skipShortWords(..)` 使用
function skipLongWords(list) { /* .. */ }

var filterWords = partialRight( compose, unique, words );

var biggerWords = filterWords( skipShortWords );
var shorterWords = filterWords( skipLongWords );

biggerWords( text );
// ["compose","functions","together","output","first",
// "function","input","second"]

shorterWords( text );
// ["to","two","pass","the","of","call","as"]
```

花点时间考虑一下`compose(..)`上正确的部分应用程序给了我们什么。它允许我们提前指定构图的第一步，然后用不同的后续步骤（`biggerWords(..)`和`shorterWords(..)`）创建该构图的特殊变体。这是FP最强大的技巧之一！

您也可以`curry(..)`组合而不是部分应用程序，但是由于从右到左排序，您可能更经常希望`curry( reverseArgs(compose), ..)`而不是仅仅`curry( compose, ..)`本身。

**注意：**因为`curry(..)` （至少[我们在第3章中实现它的方式](ch3.md/#user-content-curry)）依赖于检测元数（`length`）或手动指定它，并且`compose(..)` 是一个可变函数，所以需要手动指定预期的元素，如 `curry(.. , 3)`。

### 替代实施

虽然您可能永远不会实现自己的`compose(..)`以在生产中使用，而只是按照提供的方式使用库的实现，但我发现理解它在封面下的工作方式实际上有助于很好地巩固一般的fp概念。

因此，让我们检查一下`compose(..)`的一些不同实现选项。我们还将看到每个实现都有一些优缺点，特别是性能。

我们将在第9章中详细研究[`reduce(..)`](ch9.md/#reduce)，但现在只需知道它将一个列表（数组）缩减为一个有限值。就像一个奇特的环。

例如，如果对一个数字列表（例如`[1,2,3,4,5,6]`）进行加法归约，则会在进行加法运算时循环它们。减法将把`1` 加到`2`，把结果加到`3`，然后把结果加到`4`，依此类推，最终得到`21`。

`compose(..)`的原始版本使用循环并立即计算要传递给下一个调用的作为一个调用的参数。这是一个函数列表的缩减，所以我们可以用`reduce(..)`做同样的事情:

<a name="composereduce"></a>

```js
function compose(...fns) {
    return function composed(result){
        return [...fns].reverse().reduce( function reducer(result,fn){
            return fn( result );
        }, result );
    };
}

// ES6 箭头格式
var compose = (...fns) =>
    result =>
        [...fns].reverse().reduce(
            (result,fn) =>
                fn( result )
            , result
        );
```

**注意:**`compose(..)`的这个实现使用`[...fns].reverse().reduce(..)` 来从右到左减少。我们将在[第9章重温 `compose(..)`](ch9.md/#user-content-composereduceright)中使用`reduceRight(..)`代替。

注意，`reduce(..)` 循环在每次运行最终的`composed(..)`函数时发生，并且每个中间的 `result(..)` 作为下一个调用的输入传递给下一个迭代。

这种实现的优点是代码更加简洁，并且使用了一个众所周知的函数编程结构:`reduce(..)`。这个实现的性能也类似于最初的for循环版本。

但是，这种实现受到限制，因为外部组合函数(也就是组合中的第一个函数)只能接收单个参数。大多数其他实现都将所有参数传递给第一个调用。如果合成中的每个函数都是一元的，这没什么大不了的。但是，如果需要向第一个调用传递多个参数，则需要不同的实现。

为了修正第一次调用的单参数限制，我们仍然可以使用`reduce(..)`，但是会产生一个延迟计算函数包装:

```js
function compose(...fns) {
    return fns.reverse().reduce( function reducer(fn1,fn2){
        return function composed(...args){
            return fn2( fn1( ...args ) );
        };
    } );
}

// ES6 箭头格式
var compose =
    (...fns) =>
        fns.reverse().reduce( (fn1,fn2) =>
            (...args) =>
                fn2( fn1( ...args ) )
        );
```

注意，我们直接返回`reduce(..)`调用的结果，它本身是一个函数，而不是一个计算结果。*那个*函数允许我们传入尽可能多的参数，将它们全部传递到组合中的第一个函数调用，然后在每个后续调用中弹出每个结果。

与计算运行结果并将其作为`reduce(..)`循环进行传递不同，此实现在组合时预先运行一次 `reduce(..)`循环，并延迟所有函数调用计算——称为延迟计算。还原的每个部分结果都是一个依次封装的函数。

当您调用最终的组合函数并提供一个或多个参数时，大型嵌套函数的所有级别，从最内部的调用到最外部的调用，都是反向连续执行的(不是通过循环)。

性能特征可能与以前基于 `reduce(..)`的实现不同。在这里， `reduce(..)`只运行一次，以生成一个大型复合函数，然后这个复合函数调用简单地执行它的所有嵌套函数的每个调用。在前一个版本中，每次调用都会运行`reduce(..)` 。

对于哪种实现更好，您的优势可能会有所不同，但是请记住，后一种实现并不像前一种实现那样在参数计数方面受到限制。

我们还可以使用递归定义`compose(..)`。`compose(fn1,fn2, .. fnN)`看起来像:

```txt
compose( compose(fn1,fn2, .. fnN-1), fnN );
```

**注意:**我们将在[第8章](ch8.md)中更全面地讨论递归，所以如果这种方法看起来令人困惑，现在不要担心。或者，去读那一章，然后回来再读这篇笔记。:)

下面是我们如何用递归实现`compose(..)`:

```js
function compose(...fns) {
    // 完成最后两个参数
    var [ fn1, fn2, ...rest ] = fns.reverse();

    var composedFn = function composed(...args){
        return fn2( fn1( ...args ) );
    };

    if (rest.length == 0) return composedFn;

    return compose( ...rest.reverse(), composedFn );
}

// ES6 箭头格式
var compose =
    (...fns) => {
        // pull off the last two arguments
        var [ fn1, fn2, ...rest ] = fns.reverse();

        var composedFn =
            (...args) =>
                fn2( fn1( ...args ) );

        if (rest.length == 0) return composedFn;

        return compose( ...rest.reverse(), composedFn );
    };
```

我认为递归实现的好处主要是概念性的。我个人发现，用递归的方式考虑重复操作要比在循环中跟踪运行的结果容易得多，所以我更喜欢代码以这种方式来表示它。

另一些人对递归方法更加气馁，把握度会没有那么强。我希望你能做好判断。

## 重新排序组成

我们在前面讨论了标准的`compose(..)`实现了从右到左的顺序。这样做的好处是，以与手动组合时相同的顺序列出参数。

缺点是它们是以执行的相反顺序列出的，这可能会让人感到困惑。使用`partialRight(compose, ..)`来预先指定要在组合中执行*第一个*函数也更尴尬。

相反的顺序，从左到右组合，有一个共同的名称:`pipe(..)`。这个名称据说来自Unix/Linux领域，其中多个程序通过(' | '操作符)将第一个的输出作为第二个的输入串在一起，以此类推(即， `ls -la | grep "foo" | less`)。

`pipe(..)`与`compose(..)`是相同的，只是它按照从左到右的顺序处理函数列表:

```js
function pipe(...fns) {
    return function piped(result){
        var list = [...fns];

        while (list.length > 0) {
            // 从列表中获取第一个函数
            // 然后执行
            result = list.shift()( result );
        }

        return result;
    };
}
```

实际上，我们可以将`pipe(..)`定义为`compose(..)`的参数反转：

```js
var pipe = reverseArgs( compose );
```

就是这么简单!

回想一下前面的一般构图：

```js
var biggerWords = compose( skipShortWords, unique, words );
```

为了用 `pipe(..)`来表达这一点，我们只需颠倒列出它们的顺序：

```js
var biggerWords = pipe( words, unique, skipShortWords );
```

`pipe(..)`的优点是它按执行顺序列出函数，这有时可以减少读者的困惑。阅读代码 `pipe( words, unique, skipShortWords )`，并认识到它首先执行`words(..)`，然后执行“unique（…）”，最后执行`skipShortWords(..)`，这样可能更简单。

如果您想要部分应用执行的*第一个*函数，`pipe(..)`也很方便。前面我们使用了`compose(..)`的右偏应用程序来实现这一点。

对比:

```js
var filterWords = partialRight( compose, unique, words );

// 对比

var filterWords = partial( pipe, words, unique );
```

您可能还记得我们在[第3章中`partialRight(..)`](ch3.md/#user-content-partialright)第一次实现时，它使用`reverseArgs(..)`，就像我们现在使用的`pipe(..)`一样。两种方法得到的结果都是一样的。

*在本例中*，使用`pipe(..)`的性能优势是，因为我们没有试图保留`compose(..)`的从右到左的参数顺序，所以我们不需要像在`partialRight(..)`中那样反转参数顺序。所以`partial(pipe, ..)`在这里比`partialRight(compose, ..)`更有效。

## 提取概念

抽象在我们对构成的推理中起着重要作用，所以让我们更详细地研究它。

类似于局部应用和局部套用(参见[第3章](ch3.md/#some now-some-later))允许从一般化函数发展到专门化函数，我们可以通过提取两个或多个任务之间的通用性来抽象。一般部分定义一次，避免重复。要执行每个任务的专门化，一般部分是参数化的。

例如，考虑以下代码(显然是人为设计的):

```js
function saveComment(txt) {
    if (txt != "") {
        comments[comments.length] = txt;
    }
}

function trackEvent(evt) {
    if (evt.name !== undefined) {
        events[evt.name] = evt;
    }
}
```
这两个实用程序都在数据源中存储值。这是普遍性。其特殊之处在于，其中一个函数将值放在数组的末尾，而另一个函数将值设置在对象的属性名处。

所以让我们提取一下：

```js
function storeData(store,location,value) {
    store[location] = value;
}

function saveComment(txt) {
    if (txt != "") {
        storeData( comments, comments.length, txt );
    }
}

function trackEvent(evt) {
    if (evt.name !== undefined) {
        storeData( events, evt.name, evt );
    }
}
```

在对象(或数组)上引用属性并设置其值的一般任务被抽象为它自己的函数 `storeData(..)`。虽然这个实用程序现在只有一行代码，但是您可以设想在这两个任务中常见的其他一般行为，比如生成一个惟一的数字ID或使用该值存储时间戳。

如果我们在多个地方重复常见的一般行为，我们将面临更改某些实例而忘记更改其他实例的维护风险。在这种抽象中有一个原则在起作用，通常被称为“不要重复自己”(DRY)。

对于任何给定的任务，DRY（“不要重复自己”原则）都力求在程序中只有一个定义。另一个激励枯燥编码的格言是，程序员通常很懒，不想做不必要的工作。

抽离提取可能会做的太过。考虑：

```js
function conditionallyStoreData(store,location,value,checkFn) {
    if (checkFn( value, store, location )) {
        store[location] = value;
    }
}

function notEmpty(val) { return val != ""; }

function isUndefined(val) { return val === undefined; }

function isPropUndefined(val,obj,prop) {
    return isUndefined( obj[prop] );
}

function saveComment(txt) {
    conditionallyStoreData( comments, comments.length, txt, notEmpty );
}

function trackEvent(evt) {
    conditionallyStoreData( events, evt.name, evt, isPropUndefined );
}
```

为了避免重复`if`语句，我们将条件转移到一般抽象中。我们还假设将来我们*可能*会在程序的其他地方检查非空字符串或非`undefined`值，所以我们最好使用DRY（“不要重复自己”原则）

这段代码“更”DRY（“不要重复自己”原则），但有些过分，抽离的太过细。程序员必须小心地将适当的抽象级别应用到程序的每个部分，不多也不少。

关于我们在本章中对函数组合的更深入的讨论，它的好处似乎是这种DRY（“不要重复自己”原则）的抽象。但是我们不要急着下结论，因为我认为组合实际上在我们的代码中起到了更重要的作用。

此外，**即使只有一次出现的重构也是有帮助的**。

### 分离使注意力集中

除了泛化和专门化，我认为抽象还有另一个更有用的定义，如下面这句话所示:

> ... 抽象提取是一个过程，程序员通过这个过程将一个名称与一个可能很复杂的程序片段关联起来，然后可以根据它的函数目的来考虑它，而不是根据该函数是如何实现的来考虑它。通过隐藏不相关的细节，抽象降低了概念的复杂性，使得程序员可以在任何特定的时间专注于程序文本的可管理子集。
>
> Michael L. Scott, 程序设计语言<a href="#user-content-footnote-1"><sup>1</sup></a>

这段话的意思是，抽象——一般来说，将一些代码提取到它自己的函数中——主要目的是将两部分功能分离开来，这样就可以独立地关注每一部分。

请注意，在这个意义上的抽象并不是真正打算“隐藏”细节，就像我们“从未”检查过的函数黑盒子一样。

在这句话中，“无关紧要”，就隐藏的东西而言，不应该被认为是一个绝对的定性判断，而是相对于你在任何特定时刻想要关注的东西而言。换句话说，当我们把X和Y分开，如果我想关注X, Y在那一刻是不相关的。另一时刻，如果我想关注Y，此时X是不相关的。

**我们不是为了隐藏细节而抽象;我们正在分离细节以提高对其他的注意**

回想一下，在这本书的开头，我曾说过FP（函数编程）的目标是创建更易于阅读和理解的代码。一种有效的方法是将复杂的代码分解成独立的、更简单的代码片段。这样，读者就不会在寻找另一部分的细节时被其中一部分的细节分散注意力。

我们更高的目标不是只执行一次，就像DRY（“不要重复自己”原则）心态一样。事实上，有时候我们会在代码中重复我们自己。

正如我们在[第3章中断言](ch3.md/#why-currying-and-partial-application)的那样，抽象的主要目标是实现单独的东西。我们正在努力提高专注度，因为这提高了可读性。

通过分离两个概念，我们在它们之间插入了一个语义边界，这使我们能够专注于独立于其他方面的每一方面。在许多情况下，语义边界类似于函数的名称。函数的实现主要关注“如何”计算某些东西，而按名称使用该函数的调用站点主要关注“如何”处理其输出。我们把“如何”从“什么”中抽象出来，使它们是独立的，可以分开推理的。

描述此目标的另一种方法是使用命令式和声明式编程风格。命令式代码主要关注于显式地说明“如何”完成任务。声明性代码声明结果应该是什么，并将实现留给其他一些责任。

声明性代码将*what*从*how*中抽象出来。典型的声明式编码在可读性上优于命令式编码，尽管没有程序(当然机器代码1和0除外)完全是其中之一。程序员必须在它们之间寻求平衡。

ES6增加了许多语法功能，可以将旧的命令式操作转换为新的声明形式。也许其中最明显的就是破坏。析构是一种赋值模式，描述如何将复合值(对象、数组)拆分成其组成值。

下面是数组解构的一个例子:

```js
function getData() {
    return [1,2,3,4,5];
}

// 命令式
var tmp = getData();
var a = tmp[0];
var b = tmp[3];

// 声明式
var [ a ,,, b ] = getData();
```

*what*的方式将数组的第一个值赋给`a`，第四个值赋给`b`。*how*的方式获取对数组的引用(`tmp`)，并在赋值给`a`和`b`时手动引用索引`0` 和 `3`。

数组析构是否隐藏了赋值?这取决于你的观点。我断言它只是简单地将“what”和“how”分隔开。JS引擎仍然执行任务，但它可以防止您被“如何”完成任务分心。

相反，你读`[ a ,,, b ] = ..`并且可以看到该模式仅仅告诉你将会发生什么。数组析构是声明性抽象的一个例子。

### 抽象构图

这和函数复合有什么关系?函数组合也是声明性抽象。

回想一下前面的`shorterWords(..)`示例。让我们来比较一下命令式和声明式的定义:

```js
// 命令式
function shorterWords(text) {
    return skipLongWords( unique( words( text ) ) );
}

// 声明式
var shorterWords = compose( skipLongWords, unique, words );
```

声明性表单主要关注*what*——这三个函数将数据从字符串传输到更短的单词列表——而将*how*留给`compose(..)`的内部部分。

在更大的意义上，`shorterWords = compose(..)`解释了如何定义`shorterWords(..)`，在代码的其他地方留下这行声明性代码，只关注“what”层:

```js
shorterWords( text );
```

构图是从所需的步骤中抽象出一个简短单词的列表。

相比之下，如果我们没有使用构图抽象呢？

```js
var wordsFound = words( text );
var uniqueWordsFound = unique( wordsFound );
skipLongWords( uniqueWordsFound );
```

甚至是这样:

```js
skipLongWords( unique( words( text ) ) );
```

这两个版本中的任何一个都演示了与先前的声明式风格相反的更命令式风格。读者在这两个片段中的注意力不可避免地与“如何”联系在一起，而较少地与“什么”联系在一起。

函数组合不仅仅是用DRY（“不要重复自己”原则）保存代码。即使`shorterWords(..)` 的用法只出现在一个地方，也没有需要避免的重复!——将“how”与“what”分隔开来，仍然可以改进我们的代码。

组合是一个强大的抽象工具，它可以将命令式代码转换为更可读的声明性代码。

## 重新看参数 

现在我们已经全面介绍了组合(这是一个在FP的许多领域都非常有用的技巧)，让我们通过重温[第3章，"No Points"无参数风格编程](ch3.md/#no-points) 来观察它的实际应用，使用一个重构起来稍微复杂一些的场景:

```js
// given: ajax( url, data, cb )

var getPerson = partial( ajax, "http://some.api/person" );
var getLastOrder = partial( ajax, "http://some.api/order", { id: -1 } );

getLastOrder( function orderFound(order){
    getPerson( { id: order.personId }, function personFound(person){
        output( person.name );
    } );
} );
```

我们要删除的“参数”是“order”和“person”参数引用。

让我们从尝试从`personFound(..)` 函数中获得 `person` 参数开始。为此，我们首先定义:

```js
function extractName(person) {
    return person.name;
}
```

请考虑此操作可以用通用术语表示:按名称从任何对象中提取任何属性。让我们将这样的实用程序称为`prop(..)`：


```js
function prop(name,obj) {
    return obj[name];
}

// ES6箭头格式
var prop =
    (name,obj) =>
        obj[name];
```

在处理对象属性时，我们还定义了另一个实用程序:`setProp(..)`，用于在对象上设置属性值。

但是，我们要小心，不要只是修改一个现有对象，而是创建一个对象的克隆来进行修改，然后返回它。这样做的原因将在[第5章](ch5.md)中详细讨论。

<a name="setprop"></a>

```js
function setProp(name,obj,val) {
    var o = Object.assign( {}, obj );
    o[name] = val;
    return o;
}
```

现在，要定义一个`extractName(..)`来从对象中提取`"name"` 属性，我们将部分应用 `prop(..)`:

```js
var extractName = partial( prop, "name" );
```

**注意:**不要错过这里的 `extractName(..)`实际上还没有提取任何内容。我们部分地应用了`prop(..)` 来创建一个函数，该函数等待从传入的对象中提取 `"name"`属性。我们也可以用`curry(prop)("name")`来做。

接下来，让我们将示例的嵌套查找调用的重点缩小到以下内容:

```js
getLastOrder( function orderFound(order){
    getPerson( { id: order.personId }, outputPersonName );
} );
```

我们如何定义`outputPersonName(..)` ?为了可视化我们需要什么，考虑一下所需的数据流:

```txt
output <-- extractName <-- person
```

`outputPersonName(..)` 需要是一个函数，它接受(对象)值，将其传递给`extractName(..)`，然后将该值传递给`output(..)`。

希望您认识到这是一个`compose(..)`操作。因此，我们可以将`outputPersonName(..)`定义为:

```js
var outputPersonName = compose( output, extractName );
```

我们刚刚创建的`outputPersonName(..)`函数是提供给`getPerson(..)`的回调函数。因此，我们可以定义一个名为`processPerson(..)`的函数，使用`partialRight(..)`预先设置回调参数:

```js
var processPerson = partialRight( getPerson, outputPersonName );
```

让我们用我们的新函数重新构建嵌套查找示例:

```js
getLastOrder( function orderFound(order){
    processPerson( { id: order.personId } );
} );
```

嘿，我们进展不错！

但我们需要继续下去，去掉`order`。下一步是观察`personId`可以通过`prop(..)`从对象(如`order`)中提取出来，就像我们在`person`对象上提取`name`一样:

```js
var extractPersonId = partial( prop, "personId" );
```

构造对象(形式为`{ id: .. }`)，需要传递给`processPerson(..)`，让我们创建另一个实用程序，在对象中包装一个指定属性名的值，称为`makeObjProp(..)`:

```js
function makeObjProp(name,value) {
    return setProp( name, {}, value );
}

// ES6箭头格式
var makeObjProp =
    (name,value) =>
        setProp( name, {}, value );
```

**提示：**此实用程序在ramda库中称为`objOf(..)`。

就像我们使用`prop(..)`生成`extractName(..)`一样，我们将部分应用`makeObjProp(..)`来构建一个函数`personData(..)`，它使我们的数据对象:

```js
var personData = partial( makeObjProp, "id" );
```

若要使用`processPerson(..)`来查找附加到“order”值的人员，我们需要的操作的数据概念流是:

```txt
processPerson <-- personData <-- extractPersonId <-- order
```

因此，我们将再次使用 `compose(..)`来定义`lookupPerson(..)`实用程序：

```js
var lookupPerson =
    compose( processPerson, personData, extractPersonId );
```

还有…就这样！把整个例子放在一起，没有任何“参数”：

```js
var getPerson = partial( ajax, "http://some.api/person" );
var getLastOrder =
    partial( ajax, "http://some.api/order", { id: -1 } );

var extractName = partial( prop, "name" );
var outputPersonName = compose( output, extractName );
var processPerson = partialRight( getPerson, outputPersonName );
var personData = partial( makeObjProp, "id" );
var extractPersonId = partial( prop, "personId" );
var lookupPerson =
    compose( processPerson, personData, extractPersonId );

getLastOrder( lookupPerson );
```

哇。这就是无参数编程。`compose(..)`在两个地方都非常有用!

我认为在这种情况下，尽管推导最终答案的步骤有点长，但是最终的结果是可读性更好的代码，因为我们已经显式地调用了每个步骤。

即使你不喜欢看到/命名所有这些中间步骤，你也可以保持无参数的方式，但是不需要单独的变量就可以将表达式连接在一起:

```js
partial( ajax, "http://some.api/order", { id: -1 } )
(
    compose(
        partialRight(
            partial( ajax, "http://some.api/person" ),
            compose( output, partial( prop, "name" ) )
        ),
        partial( makeObjProp, "id" ),
        partial( prop, "personId" )
    )
);
```

这个代码段当然没有那么冗长，但是我认为它的可读性比前面的代码段要差，在前面的代码段中，每个操作都是它自己的变量。不管怎样，构图帮助了我们的无分风格。

## Summary

Function composition is a pattern for defining a function that routes the output of one function call into another function call, and its output to another, and so on.

Because JS functions can only return single values, the pattern essentially dictates that all functions in the composition (except perhaps the first called) need to be unary, taking only a single input from the output of the previous function.

Instead of listing out each step as a discrete call in our code, function composition using a utility like `compose(..)` or `pipe(..)` abstracts that implementation detail so the code is more readable, allowing us to focus on *what* the composition will be used to accomplish, not *how* it will be performed.

Composition is declarative data flow, meaning our code describes the flow of data in an explicit, obvious, and readable way.

In many ways, composition is the most important foundational pattern, in large part because it's the only way to route data through our programs aside from using side effects; the next chapter explores why such should be avoided wherever possible.

----

<a name="footnote-1"><sup>1</sup></a>Scott, Michael L. “Chapter 3: Names, Scopes, and Bindings.” Programming Language Pragmatics, 4th ed., Morgan Kaufmann, 2015, pp. 115.
