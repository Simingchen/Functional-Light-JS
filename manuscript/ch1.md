
# 章节1：为什么要函数式编程?

> 函数编程人员：（名词）将变量命名为“x”，将函数命名为“f”，并将代码模式命名为“zygohistomorphic prepromorphism”。
>
> James Iry @jamesiry 5/13/15
>
> https://twitter.com/jamesiry/status/598547781515485184

函数式编程（FP）并不是一个新概念。但它几乎贯穿了整个编程历史。不过，我不确定这样说是否合理，但是…直到最近几年，函数式编程才成为整个开发界的主流观念。所以我觉得函数式编程更像是学者的领域。

然而一切都在改变。不仅仅是在语言层面，甚至是在库和框架方面对函数式编程的兴趣正在与日俱增。你很可能读到了这篇文章，因为你最后也会意识到函数式编程是一个你不能再忽视的东西。或者你像我一样，你以前学过很多次函数式编程，但是很难理解其中所有的术语或数学符号。

第一章的目的是回答诸如“为什么要在代码中使用函数式编程的风格”之类的问题。“JavaScript轻量级函数式编程“与其他人对函数式编程的看法相比如何？”在我们做好基础准备之后，通过在本书的其余部分中，我们将一块一块地揭示以轻量级风格编写JS的技术和模式。

## 快照

让我们用代码的前后快照简要说明“JavaScript轻量级函数式编程”的概念。思考下列代码：

```js
var numbers = [4,10,0,27,42,17,15,-6,58];
var faves = [];
var magicNumber = 0;

pickFavoriteNumbers();
calculateMagicNumber();
outputMsg();                // The magic number is: 42

// ***************

function calculateMagicNumber() {
    for (let fave of faves) {
        magicNumber = magicNumber + fave;
    }
}

function pickFavoriteNumbers() {
    for (let num of numbers) {
        if (num >= 10 && num <= 20) {
            faves.push( num );
        }
    }
}

function outputMsg() {
    var msg = `The magic number is: ${magicNumber}`;
    console.log( msg );
}
```

现在考虑一种完全不同的风格，它可以实现相同的结果：

```js
var sumOnlyFavorites = FP.compose( [
    FP.filterReducer( FP.gte( 10 ) ),
    FP.filterReducer( FP.lte( 20 ) )
] )( sum );

var printMagicNumber = FP.pipe( [
    FP.reduce( sumOnlyFavorites, 0 ),
    constructMsg,
    console.log
] );

var numbers = [4,10,0,27,42,17,15,-6,58];

printMagicNumber( numbers );        // The magic number is: 42

// ***************

function sum(x,y) { return x + y; }
function constructMsg(v) { return `The magic number is: ${v}`; }
```

一旦你理解了函数式编程和轻量函数，第二个代码片段很可能是你阅读并脑中想着处理的方式：

> 我们首先创建一个函数 `sumOnlyFavorites(..)` 这是其他三个函数的组合。 我们结合了两个过滤器, 一个检查值是否大于或等于10，一个检查值是否小于或等于20. 然后我们使用 `sum(..)` 减少数据传输. 结果函数 `sumOnlyFavorites(..)`  作为缩减作用，用于检查一个值是否通过两个过滤器，如果通过，则将该值添加到累加器值中。
>
> 然后我们使用定义好的函数 `sumOnlyFavorites(..)` 它可以首先减少一个数字列表，然后使用另一个函数 `printMagicNumber(..)` 打印产生通过“sumOnlyFavorites”计算出数字的总和. 函数 `printMagicNumber(..)` 把最后的总数再输送到 `constructMsg(..)`, 进入 `console.log(..)`打印创建一个字符串值结果.

所有这些处理函数与函数式编程的开发人员的”对话“就好像当前相当不熟悉函数式编程的你一样，这本书帮助你像跟其他你熟悉的代码一样跟这种方式进行”对话“

关于此代码比较的其他一些简短说明:

* 对于许多读者来说，前一个片段比后一个片段更接近舒适/可读/可维护性一些。如果是这样思考的话，也完全可以。你也在一个正确的位置下想的。我相信，如果你在整本书中坚持下去，并实践我们所谈论的一切，第二个片段最终会变得更加自然，甚至更可取！

* 您可能已经完成了这项任务，可能这与所提供的任何一个代码片段都有显著的或完全的不同。也没关系。这本书并没有特定性的说你应该以一种特定的方式做某件事时。目的是说明各种模式的优缺点，并使您能够做出这些决定。在本书的最后，您将如何处理这个任务可能会比现在更接近第二个片段。

* 也有可能你已经是一个经验丰富的函数式编程开发人员，正在浏览这本书的开头，看看它是否有任何有用的东西供你阅读。第二个片段肯定有一些非常熟悉的片段。但我敢打赌你会想，“嗯，我不会那样做的……”好几次。没关系，完全合理

    这不是一本传统的、规范的函数式编程书。我们的方法有时会显得很离经叛道。我们正在寻求在函数式编程明显的不可否认的好处与需要运送可操作、可维护的JS之间达成一个务实的平衡，而不必处理数学/符号/术语这座令人望而生畏的大山。这不是你独有的函数式编程，它是“js的轻量函数式编程”。

无论你出于何目的翻阅本书，欢迎加入我们！

## 信心

我有一个非常简单的前提，那就是作为一名软件开发教师，我所做的每一件事情的基础（在javascript中）：您不能信任的代码是您不理解的代码。反过来也是正确的：你不理解的代码是你不能信任的代码。此外，如果您不能信任或理解您的代码，那么您就不能对您编写的代码是否适合该任务。你运行程序，基本上就是交叉手指，祈祷没有问题发生了。

我所说的信任是什么意思？我的意思是，你可以通过阅读和推理，而不仅仅是执行来验证你理解一段代码将要做什么；而不是依赖它应该做什么的层面上。通常情况下，我们倾向于依靠运行测试来验证程序的正确性。我不是说测试不好。但是我认为我们应该渴望能够充分理解我们的代码，这样我们就知道测试在运行之前会通过。.

仅仅通过阅读代码就能让他们对我们的程序有更多信心，形成函数式编程的技术是以这样的心态设计的。理解函数式编程的人，并且有足够的自我约束在他们的程序中频繁地使用它，他们将编写代码，他们和其他人可以阅读并验证程序如他们想的一样运行。

当我们使用避免或最小化可能的错误源的技术时，信心也会大大增强。这可能是函数式编程最大卖点之一：函数式编程的程序通常比较少的错误，而且存在的错误通常在一些更明显的地方，因此更容易找到和修复。函数式编程代码趋向于更具防bug性——当然不只是为防代码错误的。

在阅读本书的过程中，您将开始对编写的代码培养更多的信心，因为您将使用已经很好证明的模式和实践；并且您将避免最常见的程序错误！

## 沟通

为什么函数式编程很重要？为了回答这个问题，我们需要后退一大步，讨论为什么编程本身很重要。

听到这个可能会让你吃惊，但我不认为代码只是计算机的一组指令。实际上，我认为代码指示计算机这几乎是一个愉快的意外。

我非常深刻地相信，代码的更重要的作用是作为与其他人交流的一种手段。

你可能从经验中知道，你花在“编码”上的大量时间实际上是花在阅读现有代码上。我们中很少有人享有这样的特权：把全部或大部分时间都花在简单地敲出所有新代码上，从不处理别人（或我们过去的自己）写的代码上。

据广泛估计，开发人员将70%的代码维护时间花在阅读上以理解它。这真让人大开眼界。居然达到了70%。难怪程序员每天编写的代码行数的平均值大约是10行。我们每天花7个小时来阅读代码，去理解这10行怎么运行！

我们需要更加关注代码的可读性。还得提一下，可读性不仅仅是字符数的减少，可读性实际上最受熟悉度的影响。
<a href="#user-content-footnote-1"><sup>1</sup></a>

如果我们要花时间来写更易于阅读和理解的代码，那么函数式编程就是这项工作的核心。函数式编程的原则是建立良好的，深入研究和审查，并可证实的。花时间学习和使用这些函数式编程原则最终将为您和其他人带来更容易识别的熟悉代码。代码熟悉度的提高以及识别的便利性将提高代码的可读性。


例如，一旦您学习了“map（…）”的功能，当您在任何程序中看到它时，您几乎可以立即发现并理解它。 但是每次你看到一个“for”循环，你就必须阅读整个循环才能理解它。“for”循环的语法可能是熟悉的，但实际上它所做的并不是；你每次都必须*读*才能理解。

通过拥有看一眼就能识别的代码的能力，从而减少时间去了解代码在做什么，我们的注意力被释放出来，去思考更高层次的程序逻辑；那些都是最需要我们关注的重要内容。
By having more code that's recognizable at a glance, and thus spending less time figuring out what the code is doing, our focus is freed up to think about the higher levels of program logic; this is the important stuff that most needs our attention anyway.

函数式编程 (至少，没有所有的术语来衡量它) 是制作可读代码最有效的工具之一。 这也是它如此重要的原因。

## 可读性

可读性不是一个二进制特性。这在很大程度上是描述我们与代码关系的主观因素。随着时间的推移，我们的技能和理解自然会发生变化。我曾经历过类似下图的效果，而且我也曾与许多人聊过关于这些有趣的事。

<p align="center">
    <img src="images/fig17.png" width="50%">
</p>

你可能会发现，当你读这本书的时候，你也会有类似的感受。但振作起来;如果你坚持下去，曲线就会上升!

命令式代码描述我们大多数人已经自然编写的代码;它专注于精确指导计算机“如何”做某事。而我们将学习编写的声明式代码，它遵循函数式编程原则是更专注于描述结果输出的代码。

让我们回顾本章前面介绍的两个代码片段。

第一个片段是命令式的，几乎完全集中于“如何”完成任务;它充斥着“if”语句、“for”循环、临时变量、重新分配、值突变、带有副作用的函数调用以及函数之间的隐式数据流。当然，你“可以”通过它的逻辑来查看数字是如何流动和更改到最终状态的，但它一点也不清楚或直接。

第二个片段更具声明性一些;它消除了前面提到的大多数命令式技术。注意没有显式的条件、循环、副作用、重新分配或突变;相反，它使用我们所说的函数式编程和可信的模式，如过滤、还原、转换和组合。注重从低级别的“如何”转移到高级的“结果”。

我们没有使用“if”语句来测试一个数字，而是将其委托给一个函数式编程里的实用程序，如“gte(..)”(大于或等于)去操作，然后将重点放在更重要的任务上，即将该过滤器与另一个过滤器和求和函数组合起来，得到我们想要的结果。

此外，通过第二个程序的数据流是明确的:

1. 一系列数字经过函数 `printMagicNumber(..)`.
2. 这些数字由“sumOnlyFavorites(..)”依次次处理，最后结果得到一个数字，其中得出一个我们最喜欢的数字
3. 这个总数被“constructMsg(..)”函数转换为一个带有的消息字符串。
4. 消息字符串通过`console.log(..)`打印出来.

从上面可以看到命令式代码片段更容易理解，但您可能仍然觉得这种方法很复杂，你的习惯与熟悉度对可读性的判断有深刻的影响。不过，在本书的最后，您将会潜移默化的了解到第二个代码片段的声明性方法的优点，并且熟悉性将使其可读性更强。

我知道让你相信这一点是一种信仰的飞跃。

正如我所建议的，要提高它的可读性，并最小化或消除导致bug的许多错误，需要付出更多的努力，有时还需要编写更多的代码。坦白地说，当我开始写这本书的时候，我可能还完全写不出(甚至完全理解)第二段。当我更深入学习的时候，一切都变得自然与舒适。

如果您希望使用函数式编程重构，那这就像一个神奇的银弹，能够快速地将您的代码转换为更优雅、更优雅、更聪明、更有弹性、更简洁的代码(短期内很容易实现)，万幸的是，这是一个现实的期望不难实现。

函数式编程是一种非常不同的方式来考虑代码应该如何构造，从而使数据流更加明显，并帮助读者跟随您的想法。这需要时间。这一努力非常值得，但可能是一段艰苦的旅程。

通常，我仍然需要多次尝试将命令式代码片段重构为更具声明性的函数式编程模式的代码，然后才能得到一些足够清晰的代码，以便以后能够理解。我发现转换到函数式编程模式是一个缓慢的迭代过程，不像从一个范例到另一个范例的二进制转换那样快速。

我还将“以后再教”测试应用到我编写的每段代码中。在我写完一段代码后，我会把它放在一边几个小时或几天，然后回来，试着用新的眼光来读它，假装我需要教别人或向别人解释它。通常，这样给别人解释的时候相当混乱，而且要不断的调整那些代码!

我不是想让你扫兴。我真希望你能解开疑惑。我很高兴我做到了。我最终可以看到上面图像解释的曲线向上弯曲的状态，改造代码以提高可读性。这些努力是值得的。

## 客观判断

大多数FP文本似乎都采取了自上而下的方法，但我们会以相反的方向：从头开始，我们将揭示基本的基本原则，我相信正式的FP使用者会承认这是他们所做一切的脚手架。但在很大程度上，我们会与大多数吓人的术语或数学符号保持一定距离，因为这些术语或数学符号很容易让学习者感到沮丧。

我相信你所说的东西不那么重要，更重要的是你要了解它是什么以及它是如何工作的。这并不是说共享术语不重要——毫无疑问，它简化了经验丰富的专业人员之间的交流。但对于学习者来说，我发现这会分散注意力。

因此，这本书将会更多地集中在基本概念，而不是花里胡哨的说些无意义的事情上。这并不是说没有术语，而是肯定会有术语。但不要过于沉溺于复杂的语言中。在必要的时候，超越他们去考虑这些想法。

我把这些不太正式的实践称为“轻量编程”，我认为真正的FP的形式主义所受害的地方是，如果你还不习惯于正式的思想，它可能是非常压倒性的。我不只是简单猜测，从我自己的故事中可以说明。即使在教过FP和写过这本书之后，我仍然可以说，FP中的术语和符号的形式主义对我来说是非常难以处理的。我试过又试，但还是觉得难以处理。

I know many FPers who believe that the formalism itself helps learning. But I think there's clearly a cliff where that only becomes true once you reach a certain comfort with the formalism. If you happen to already have a math background or even some flavors of CS experience, this may come more naturally to you. But some of us don't, and no matter how hard we try, the formalism keeps getting in the way.

So this book introduces the concepts that I believe FP is built on, but comes at it by giving you a boost from below to climb up the cliff wall, rather than condescendingly shouting down at you from the top, prodding you to just figure out how to climb as you go.

## How to Find Balance

If you've been around programming for very long, chances are you've heard the phrase "YAGNI" before: "You Ain't Gonna Need It". This principle primarily comes from extreme programming, and stresses the high risk and cost of building a feature before it's needed.

Sometimes we guess we'll need a feature in the future, build it now believing it'll be easier to do as we build other stuff, then realize we guessed wrong and the feature wasn't needed, or needed to be quite different. Other times we guess right, but build a feature too early, and suck up time from the features that are genuinely needed now; we incur an opportunity cost in diluting our energy.

YAGNI challenges us to remember: even if it's counterintuitive in a situation, we often should postpone building something until it's presently needed. We tend to exaggerate our mental estimates of the future refactoring cost of adding it later when it is needed. Odds are, it won't be as hard to do later as we might assume.

As it applies to functional programming, I would offer this admonition: there will be plenty of interesting and compelling patterns discussed in this text, but just because you find some pattern exciting to apply, it may not necessarily be appropriate to do so in a given part of your code.

This is where I will differ from many formal FPers: just because you *can* apply FP to something doesn't mean you *should* apply FP to it. Moreover, there are many ways to slice a problem, and even though you may have learned a more sophisticated approach that is more "future-proof" to maintenance and extensibility, a simpler FP pattern might be more than sufficient in that spot.

Generally, I'd recommend seeking balance in what you code, and to be conservative in your application of FP concepts as you get the hang of things. Default to the YAGNI principle in deciding if a certain pattern or abstraction will help that part of the code be more readable or if it's just introducing clever sophistication that isn't (yet) warranted.

> Reminder, any extensibility point that’s never used isn’t just wasted effort, it’s likely to also get in your way as well
>
> Jeremy D. Miller @jeremydmiller 2/20/15
>
> https://twitter.com/jeremydmiller/status/568797862441586688

Remember, every single line of code you write has a reader cost associated with it. That reader may be another team member, or even your future self. Neither of those readers will be impressed with overly clever, unnecessary sophistication just to show off your FP prowess.

The best code is the code that is most readable in the future because it strikes exactly the right balance between what it can/should be (idealism) and what it must be (pragmatism).

## Resources

I have drawn on a great many different resources to be able to compose this text. I believe you, too, may benefit from them, so I wanted to take a moment to point them out.

### Books

Some FP/JavaScript books that you should definitely read:

* [Professor Frisby's Mostly Adequate Guide to Functional Programming](https://drboolean.gitbooks.io/mostly-adequate-guide/content/ch1.html) by [Brian Lonsdorf](https://twitter.com/drboolean)
* [JavaScript Allongé](https://leanpub.com/javascriptallongesix) by [Reg Braithwaite](https://twitter.com/raganwald)
* [Functional JavaScript](http://shop.oreilly.com/product/0636920028857.do) by [Michael Fogus](https://twitter.com/fogus)

### Blogs/sites

Some other authors and content you should check out:

* [Fun Fun Function Videos](https://www.youtube.com/watch?v=BMUiFMZr7vk) by [Mattias P Johansson](https://twitter.com/mpjme)
* [Awesome FP JS](https://github.com/stoeffel/awesome-fp-js)
* [Kris Jenkins](http://blog.jenkster.com/2015/12/what-is-functional-programming.html)
* [Eric Elliott](https://medium.com/@_ericelliott)
* [James A Forbes](https://james-forbes.com/)
* [James Longster](https://github.com/jlongster)
* [André Staltz](http://staltz.com/)
* [Functional Programming Jargon](https://github.com/hemanth/functional-programming-jargon#functional-programming-jargon)
* [Functional Programming Exercises](https://github.com/InceptionCode/Functional-Programming-Exercises)

### Libraries

The code snippets in this book largely do not rely on libraries. Each operation that we discover, we'll derive how to implement it in standalone, plain ol' JavaScript. However, as you begin to build more of your real code with FP, you'll soon want a library to provide optimized and highly reliable versions of these commonly accepted utilities.

By the way, you need to check the documentation for the library functions you use to ensure you know how they work. There will be a lot of similarities in many of them to the code we build on in this text, but there will undoubtedly be some differences, even between popular libraries.

Here are a few popular FP libraries for JavaScript that are a great place to start your exploration with:

* [Ramda](http://ramdajs.com)
* [lodash/fp](https://github.com/lodash/lodash/wiki/FP-Guide)
* [functional.js](http://functionaljs.com/)
* [Immutable.js](https://github.com/facebook/immutable-js)

[Appendix C takes a deeper look at these libraries](apC.md/#stuff-to-investigate) and others.

## Summary

You may have a variety of reasons for starting to read this book, and different expectations of what you'll get out of it. This chapter has explained why I want you to read the book and what I want you to get out of the journey. It also helps you articulate to others (like your fellow developers) why they should come on the journey with you!

Functional programming is about writing code that is based on proven principles so we can gain a level of confidence and trust over the code we write and read. We shouldn't be content to write code that we anxiously *hope* works, and then abruptly breathe a sigh of relief when the test suite passes. We should *know* what it will do before we run it, and we should be absolutely confident that we've communicated all these ideas in our code for the benefit of other readers (including our future selves).

This is the heart of Functional-Light JavaScript. The goal is to learn to effectively communicate with our code but not have to suffocate under mountains of notation or terminology to get there.

The journey to learning functional programming starts with deeply understanding the nature of what a function is. That's what we tackle in the next chapter.

----

<a name="footnote-1"><sup>1</sup></a>Buse, Raymond P. L., and Westley R. Weimer. “Learning a Metric for Code Readability.” IEEE Transactions on Software Engineering, IEEE Press, July 2010, dl.acm.org/citation.cfm?id=1850615.