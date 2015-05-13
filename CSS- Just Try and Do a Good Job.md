CSS: Just Try and Do a Good Job

# CSS：尽管尝试，做的更好

//fixme 之前一句有待着重检查。

Have you ever worried that you’re approaching CSS all wrong? That you’re missing out on some new approach that makes everything easier and better? That you wish you were more confident about the state of your CSS?

你有没有担心过自己写的 CSS 都错了？有没有想过会错过了一些更加简单的新方法？是不是想在 CSS 方面更有自信呢？

I’m sure we can all empathize with Anna’s sentiment here:

 

My CSS is full of self-doubt. What class name system is best practice today, and is the cascade good or evil this week?

— Anna Debenham (@anna_debenham) November 13, 2014
 

I worry about you if you work with CSS a lot and have never felt that. You’re either scary smart or, you know.

那在这方面你和 Anna 肯定感同身受：

> 我的 CSS充满了自我怀疑。现在 class 使用什么样的名字系统更合适呢？以后什么又是最好的？什么是差的？ ——Anna Debenham [November 13, 2014](https://twitter.com/anna_debenham/status/532933212172206080)

如果你也写了很多 CSS，但是从来没有过这样的疑虑，那么就比较令人担心了。要么就是你顶级聪明，要么，[呵呵，你懂的](http://cdn.css-tricks.com/wp-content/uploads/2014/12/ISVw9.gif)。

Here’s how I’m trying to approach CSS these days: just try and do a good job. I don’t subscribe to any specific methodologies or strict rules. More like a set of loose guidelines that attempt to keep things under control, held together by the idea that I’m actively trying to do a good job.

我最近写 CSS 的方法是：尽管尝试，做的更好。我不是想要宣扬特殊的方法论或者严格的规则。这更像是一些宽松的法则，保证事情在可控的范围内，积极地尝试，然后做的更好一些。

Caveat: this is just me. I mostly work on projects where it’s mostly me touching the CSS. That applies also to 55% of you, accordinging the most recent poll running here. I’d speculate that the more people you work with, the more benefit you get to stricter rules than I adhere to.

Here’s a little elaboration on “just try and do a good job”:

警告：这是我个人的方法。我工作的项目几乎只有我自己负责 CSS。从最近 css-tricks 上的投票来看，其中55%也同样适用于你。我推测，和你一起工作的人越多，我的建议的作用就越小。 //译注：原文 csstricks 网站边栏有一个投票。

以下就是详细的法则：

Don’t be lazy. You know when you’re being lazy about something. By which I mean you slapping in a quick fix for something rather than trying to understand the problem. Or dropping some CSS into whatever file seems convenient at the moment rather than thinking about where it best belongs. Or you are avoiding creating a new pattern when it seems clear that’s what the situation calls for. Or continuing to use a pattern that is biting you. Sleep on it. Don’t rush. Do it up right.

**不要懒惰。**你什么时候偷懒了，自己心里都清楚。比如对某个问题你喜欢用快 quick fix 而不是彻底了解这个问题。//fixme：是 ide 的 quickfix 还是“草草的修正问题？”比如哪个文件起来合适就立马将 CSS 放进去而不是想想它到底该放在那里。比如当某处明显需要新的 Pattern 你却懒得再独立创建一个。//fixme:Pattern?比如死板的坚持使用一个不好的 Pattern。宁可慢一些，也坚持用正确的方法解决。

Do things how you would do them. You know what? I like light usage of child selectors in modules. .module > h2 clicks with me a lot of times. Certain strict methodologies would disagree. I don’t really care. I’m going to keep doing it until it actually causes some negative consequence, which it hasn’t yet. If it does, I’ll change, because see above.

**使用你喜欢的方式。**知道吗？在模块中我喜欢轻量使用子选择器。//fixme`.module > h2`这种形式经常出现在我的 CSS 中。严格的方法论肯定不支持这种做法，但是我可不管。在它出错之前，我会一直这么使用，但是到现在为止都是对的。如果他出错了，我再改。原因见上。//fixme 原因显而易见？

Name things how you would name them. If I let a methodology tell me how to name things, I know me, my mind starts staging a coup. I might play along for a day or two until I abort the methodology and take full control again. On projects entirely build from my WWINTT (What Would I Name This Thing?) strategy, I feel most comfortable and efficient.

**用你喜欢的方式进行命名。**如果让我根据某个规则来命名，我脑子里肯定会一团糟。我应该会花上一两天的时间来接受这个规则，并且重新进行管理。//fixme 我们的项目完全是根据我自己的爱好进行命名的。总体上来说，我感到更自在，更高效。

Use tools that are clearly useful to you. I’m not even going to name any tools because that’s not what this is about. If I can identify a tool that has clear benefits to me, I’ll use it. It could save time, produce better output, allow for better organization, solve a performance issue, automate a best practice, whatever. I’m in.

**使用自己觉得高效的工具。**我不会推荐什么工具，因为好的工具是因人而异的。如果我觉得某个工具很有用，我就会去用。只要它能节省时间，做出更好地效果，更好地组织，解决性能问题，自动做出最佳选择，不管它是什么，我用了。

The one “rule”ish thing thing I really believe in: keep your selector specificities fairly low and flat across your whole project. Harry’s specificity graph is a nice way to think about this. Specificity will trend upward, so never start high, as the ceiling can easily become problematic. Mostly: .class {}.

有一条原则是我一直以来坚信的：**在整个项目中保持选择器的低特异性。**结合 Harry 的[特异性图表](http://csswizardry.com/2014/10/the-specificity-graph/)可以很好地理解这句话。特异性会逐渐升高，如果从一开始就比较高，那么极限会很快成为一个问题。多用：`.class{}`。//fixme

Revisit sections of your site on purpose. Probably not just because you want to clean up the CSS there, but because you want to make that section of your site better in some way, for everyone. I find that every time I revisit an area, it’s an opportunity to clean up all the code that touches it. That helps me feel connected to and less afraid of old code.

**有目的性地重新访问页面的各个部分。**目的不仅仅是检查各个部分的 CSS 良好，还要试图做的更好，适用于大多数人。我发现每次我重新访问一个地方，都是做最后润色的一个机会，这让我对旧代码更有自信。