# A tmux Primer

![](https://dmiessler.cachefly.net/images/tmuxpanes.jpg)

There are 4,257 tutorials on tmux. That’s a rough number that I just made up. This one is designed to take you from “wtf tmux” to “omg tmux” with extreme haste.

有关 Tmux 的教程多达 4257 篇，这个数还是我粗略统计的。而我这篇入门教程的目的是让你从“我去，tmux 是神马”极速地转变到“Tmux 真牛B”！

Let’s get started.

那么，我们就开始吧！

## Why Tmux

## 为什么要用 Tmux

tmux is useful to people in different ways. To me, it’s most useful as a way to maintain persistent working states on remote servers—allowing you to detach and re-attach at will.

tmux 在很多方面都很有用。对我而言，由于 tmux 允许随时随地断开或重新接入会话（Session），所以它最大的作用就是在远程服务器上持久地保存工作状态。

You could, for example, have a session on your server for hacking on a node REST API (my current project), and call it “nodeapi”. And let us say that you are compiling something for it that will take two hours (work with me), but you’re currently working at a coffee shop and you have to leave. tmux lets you simply detach from that session and come back to it later.

例如，你可以在服务器上新建一个会话并命名为“nodeapi”，然后用它来挖掘 node REST API 的漏洞（这是我现在的项目）。或者假设你正在咖啡店里工作，需要编译一些代码，而编译要花费 2 个小时才能完成（如果是和我一起工作的话），这时你又不得不离开咖啡店。如果使用了 tmux，你就可以轻松地断开当前的会话，并于稍后方便时重新接入该会话，继续工作。

That’s handy.

这真是太方便了。

Others like to focus on how you can use tmux to have multiple panes within multiple windows within multiple tabs within multiple sessions. I don’t do that. I like fewer of those—as few as possible, actually—and this guide will be focused on a simple persistent-remote-sessions model.

“如何使用 tmux 才能打开多个会话，如何在会话中打开多个标签（Tab），如何在标签中打开多个窗口（Window），又如何在窗口中打开多个窗格（Pane）”，也许有人对这些操作更感兴趣。而我是不会这样做的，因为我不喜欢打开太多的——实际上是尽可能少地打开——这些东西。因此，这篇入门教程主要讲解的也是作为简单的可持久化远程会话模型的 tmux。

#### A remote computing lifestyle

#### 远程计算（操作？）的生活方式

**Mobility** is a central theme for tmux users. Many developers do all of their work from the server, and simply connect in from $wherever to do it. tmux (and similar tools) allow you to work from a coffee shop in SF, start something building on the server, disconnect to take a flight, and then pick up that same task on the ground in NYC when you land.

机动性是 tmux 用户的主题。有很多开发者都是在服务器上进行所有工作的，他们只是简单地从某处连接上服务器就可以工作了。有了 tmux（或者其他类似的工具），你就可以先坐在旧金山的某个咖啡店里开始在服务器上进行构建工作，然后断开会话去赶飞机，待飞机降落到纽约市后再继续进行刚刚的工作。

A related advantage to this mobile approach is the fact that your client machine is not too terribly important. You can upgrade your laptop, clone a repo with your vim/tmux dotfiles in it, and you’re back to your optimum **computing environment** in minutes rather than days.

tmux 带来的另一个好处是在移动办公中，作为客户端的计算机不再那么重要了。只需要升级你的笔记本，然后从版本库中克隆出 vim 和 tmux 的配置文件，就可以进入最优的操作环境了。而且这一切只需要短短的几分钟。

Anyway, those are some reasons that people love tmux, but you don’t have to make this lifestyle change in order to see its benefits.

总之，这些就是人们喜爱 tmux 的原因。当然即使你的生活不是四处奔波，也一样能体验到 tmux 带来的好处。

### What about screen?

### 那么 screen 呢？

Good question. tmux is a lot like screen, only better. The short answer for how it’s better is that tmux is better designed to perform the same functions. Screen gets you there (kind of) but does so precariously.

问得好。tmux 和 screen 很像，但比 screen 更好。要问好在哪里，简单的回答就是虽然与 screen 的功能相同，但是 tmux 设计得更好。screen 能在某种程度上达到目的，但是很不稳定。

Here are a few of the key advantages of tmux over screen:

以下是一些 tmux 胜过 screen 的优点：

* Screen is a largely dead project, and its code has significant issues
* Tmux is an active project with an active codebase
* Tmux is built to be truly client/server; screen emulates this behavior
* Tmux supports both emacs and vim shortcuts
* Tmux supports auto-renaming windows
* Tmux is highly scriptable
* Window splitting is more advanced in tmux

* screen 的项目大体上已经终止了，并且代码中有大量的问题
* tmux是一个活跃的项目，并且其代码库经常更新
* tmux使用的是真正的客户端/服务器模型，而 screen 只是模拟了这种模型的行为
* tmux 同时支持 emacs 和 vim 的快捷键
* tmux 支持自动重命名窗口
* tmux 可以高度的脚本化
* tmux 的窗口分割功能更加先进

Enough about that. Use tmux.

这些优点已经足够了吧，开始使用 tmux 吧。

## Basics

## 基础

Now is a good time to mention that there is a universal tmux shortcut that lets you quickly perform many tasks.

首先要告诉诸位的是 tmux 中的一个universal的快捷键，用这个快捷键就可以执行很多任务。

### The tmux shortcut

### tmux 的快捷键

By default, tmux uses Ctrl-b as its shortcut activation chord, which enables you perform a number of functions quickly. Here are a few of the basics:

tmux 默认使用 `Ctrl-b` 作为激活快捷键（操作）的开关，激活后就可以快速调用大量的功能。下面就是一些基础的功能：

First you hit:

首先按下

`$ Ctrl-b`

…followed by a number of options that we’ll talk about below. But get ready to use that Ctrl-b combo. Also consider remapping CAPSLOCK to CONTROL within your operating system; it makes the pinky walk for Ctrl-b quite nice.

接下来就是一些后面将会讲解的option。不过先不要着急，先为使用组合键 `Ctrl-b` 做一点准备。不妨在操作系统中将键盘上的 CAPSLOCK 键映射为 Ctrl 键，这样就可以小拇指的移动更加舒服。

### Invocation

### 运行 tmux

Right then. Let’s start by running tmux. You want to do this from the system that you want to detach and re-attach to—which for me is usually a remote server.

好了，下面让我们从运行 tmux 开始。请在你希望在断开会话后依然可以重新连接的系统（对我来说这通常是远程服务器）上运行如下的命令：

`$ tmux`

Simple enough. You now have a tmux session open that you can disconnect from and come back to later.

很简单对吧。现在就开启了一个 tmux 的会话，你可以断开这个会话并在稍后再重新接入。

### Show Sessions

### 显示所有会话

Since the idea of tmux is having multiple sessions open, and being able to disconnect and reconnect to them as desired, we need to be able to see them quickly.

由于 tmux 的理念是可以开启多个会话，并且可以随心地断开会话后重新接入，为此我们首先需要能立刻看到这些会话。

```
# Via shortcut (by default Ctrl-b)

$ Ctrl-b s
```

```
# 通过快捷键（默认 Ctrl-b）

$ Ctrl-b s
```

```
# 通过tmux命令

$ tmux ls
```

Either way you get the same thing:

上面两种方法的效果相同，都可以得到类似下面的结果：

`0: 1 windows (created Thu Nov 28 06:12:52 2013) [80x24] (attached)`

### Create a new session

### 新建会话

Now we’re going to create a new session. You can do this with just the new command, or by providing an argument to it that serves as the session name. I recommend providing a session name, since organization is rather the point of tmux.

下面我们就来新建一个新的会话。可以使用 `new` 命令新建会话，并且可以以参数的形式传递一个会话名给该命令。我建议在新建时要提供一个会话名以便于日后的管理。

```
$ tmux new -s session-name
```


```
# Without naming the new session (not recommended) 
# 新建会话但并不指定名字 (不推荐) 

$ tmux new
```

### Attaching to an existing session

### 接入一个之前的会话

Since we’re going to be creating sessions with names, and we may have more than one, we want to be able to attach to them properly. There are a couple ways of doing this.

既然我们已经创建了多个带有名称的会话，那么就会想要properly接入它们吧，有几种方法可以实现：

You can simply type `tmux a` and it’ll connect you to the first available session.

可以简单地输入 `tmux a`，这样可以接入第一个可用的会话。

```
$ tmux a
```

Or you can attach to a specific session by providing an argument.

或者可以通过参数指定一个要接入的会话：

```
$ tmux a -t session-name
```

### Detaching from a session

### 从会话中断开

You can detach from an existing session (so you can come back to it later) by sending the detach command.

可以使用`detach`命令断开已有的会话（因此稍后才会有重新接入会话这么一说）。

`$ tmux detach`

Or you can use the shortcut.

也可以使用快捷键断开会话：

`$ Ctrl-b d`

### Killing a session

### 关闭会话

There are times when you’ll want to destroy a session. This can be done using the following syntax, which is much the same as attachment:

要关闭会话的话，可以使用如下的命令，该命令和接入会话时所使用的命令很像：

`$ tmux kill-session -t session-name`

[ NOTE: You can kill windows the same way, but using kill-window instead. You can also kill tmux altogether with killall tmux. ]

提示：关闭窗口时也可以用类似的方法，只不过要把 kill-session 换成 kill-window。另外，还可以使用 tmux killall 同时关闭 tmux。

## Configuration

## 配置

As with most things in tech, you can get pretty silly with your tmux config. The common things to tinker with are:

与其他工具一样，一旦配置好了 tmux，使用起来就将会非常顺手。下面就给出几个通常需要配置的项目：

* The primary tmux shortcut
* Your status bar
* Your various keyboard shortcuts

* tmux 的主要快捷键
* 屏幕下方的状态条
* 自定义的各种快捷键

I went pretty Spartan with mine.

我使用了一些相当简单的配置：

```
# Set a Ctrl-b shortcut for reloading your tmux config
# 设置一个 Ctrl-b 后面的快捷键，用于重新加载 tmux 的配置文件
bind r source-file ~/.tmux.conf

# Rename your terminals
# 重命名终端
set -g set-titles on
set -g set-titles-string '#(whoami)::#h::#(curl ipecho.net/plain;echo)'

# Status bar customization
# 自定义状态条
set -g status-utf8 on
set -g status-bg black
set -g status-fg white
set -g status-interval 5
set -g status-left-length 90
set -g status-right-length 60
set -g status-left "#[fg=Green]#(whoami)#[fg=white]::#[fg=blue]\
(hostname -s)#[fg=white]::##[fg=yellow]#(curl ipecho.net/plain;echo)"

set -g status-justify left
set -g status-right '#[fg=Cyan]#S #[fg=white]%a %d %b %R' 
```

One thing worth noting here is that I use ipecho.net to get my current WAN IP4 WAN address instead of icanhazip as most other tutorials have. It’s just faster and less prone to error, from my experience.

这里有一点值得注意，我使用 ipecho.net 而不是 icanhazip 来获取计算机当前的 IP 地址（IPv4）。虽然有很多教程使用的是 icanhazip，但是凭借我的经验，ipecho.net 的速度更快，更稳定。

[ My current, updated configuration can be found here if you’re interested. ]

提示：如果你感兴趣，可以从[这里](这里)查看我最新的配置。

## Advanced

## 高级功能

That covers how I usually use tmux, but I do often make use of some of the more powerful features.

我平时常用的功能就是这些了。不过，我也会经常使用一些 tmux 中更强大的功能。

### Windows and Panes

### 窗口和窗格

![tmuxpanes]()

One of these features is the ability to break your session into more discreet(discrete?) components, called windows and panes. These are good for organizing multiple varied activities in a logical way.

这些高级功能之一就是 tmux 可以将一个会话分割成若干个称为窗口（Window）和窗格（Pane）的相互分离的组件。这种逻辑上的分割非常适合于用户安排各种各样的活动。

Let’s look at how they relate to each other.

下面就来看一看这几个概念之间的关系。

#### Nesting

#### 层次结构

![](tmuxnesting)

tmux sessions have windows, and windows have panes. Below you can see how how I conceptualize them—although if anyone has a more authoritative or useful hierarchy I’ll happily embrace it.

一个会话包含多个窗口，一个窗口包含多个窗格。下图就是我对这些概念的理解。当然如果诸位有更权威或者更实用的层次结构，我很乐意洗耳恭听。

* Sessions are for an overall theme, such as work, or experimentation, or sysadmin.
* Windows are for projects within that theme. So perhaps within your experimentation session you have a window titled noderestapi, and one titled lua sample.
* Panes are for views within your current project. So within your sysadmin session, which has a logs window, you may have a few panes for access logs, error logs, and system logs.

* 会话用于大的工作内容，例如日常工作，实验或是系统管理。
* 窗口适用于这些大工作中的项目。例如，在用于实验的会话中可能有一叫做 noderestapi 的窗口，有一个叫做 lua 的窗口。
* 窗格适用于浏览当前的项目。例如，在系统管理的会话中有一个叫做 logs 的窗口，在这个窗口中可以打开多个窗格分别用于浏览 access log，error log和system log。

It’s also possible to create panes within a session without first creating a separate window. I do this sometimes. Hopefully it isn’t as horrible as it sounds right after reading about nesting. As I said in the beginning, I incline towards simplicity with my use of tmux.

我们也可以在会话中直接创建窗格，而不需要先创建一个分离的窗口。我有时会这样做。当阅读完“层次结构”这一小节，希望我的这种做法没有听起来那样恐怖。正如我在一开始谈到的，我倾向于简化 tmux 的使用。

#### Navigating with panes

#### 在窗格间移动光标

There’s a default way to navigate between panes, but I don’t know what it is. I’m a vim guy, so I navigate within my panes using the h, j, k, and l keys like so:

虽然有默认的在窗格间移动光标的方法，但是我并不清楚是什么。因为我习惯用 vim，所以我会用h，j，k和l键在窗格间移动光标。为此，要加入如下的配置：

```
# Remap window(pane?) navigation to vim
# 用 vim 的方式在窗格间移动光标
unbind-key j
bind-key j select-pane -D
unbind-key k
bind-key k select-pane -U
unbind-key h
bind-key h select-pane -L
unbind-key l
bind-key l select-pane -R
```

## Recommendations

A few thoughts that may help you in your tmux travels:

以下几条建议也许会有助于诸位的 tmux 之旅：

Consider using as few sessions and windows as possible. Humans aren’t as good at multitasking as we think we are, and while it feels powerful to have 47 panes open it’s usually not as functional as you’d imagine.

1.  尽可能少打开会话和窗口。人类没有我们自认为的那样善于多任务处理。虽然打开 47 个窗格显得功能很强大，但是这并没有我们想象的那样实用。

When you do use windows and panes, take the time to name them. They are indeed useful, but switching between sessions and windows is supremely annoying when they’re all labeled 0, 1, and 2.

2.  当确实要使用窗口和窗格时，花一点时间为它们起个有意义的名字。这非常有用，如果只是用0、1、2这样的名字，切换会话或窗口时就会非常麻烦。

Start with a basic config and get used to it before you get silly. I’ve seen multiple people spend hours configuring vim or tmux only to confuse themselves and abandon the project altogether. Start simple.

3.  从基础的配置、操作开始，别把自己搞糊涂了。我曾遇到过很多人，他们花费大量的时间配置 vim 或 tmux，而最终带来的结果却是不但把自己绕进去了，项目也没有进展。

## Shortcut Reference

## 快捷键参考

Now a `Ctrl-b` options reference:

按下 `Ctrl-b` 后的快捷键如下：

### Basics

### 基础

*  `?` get help

*  `?` 获取帮助信息

### Session management

### 会话管理

*   `s` list sessions
*   `$` rename the current session
*   `d` detach from the current session

*   `s` 列出所有会话
*   `$` 重命名当前的会话
*   `d` 断开当前的会话

### Windows

### 窗口

*   `c` create a new window
*   `,` rename the current window
*   `w` list windows
*   `%` split horizontally
*   `"` split vertically
*   `n` change to the next window
*   `p` change to the previous window
*   `0` to `9` select windows 0 through 9

*   `c` 创建一个新窗口
*   `,` 重命名当前窗口
*   `w` 列出所有窗口
*   `%` 水平分割窗口
*   `"` 竖直分割窗口
*   `n` 选择下一个窗口
*   `p` 选择上一个窗口
*   `0~9` 选择0~9对应的窗口

### Panes

### 窗格

*   `%` create a horizontal pane
*   `"` create a vertical pane
*   `h` move to the left pane. *
*   `j` move to the pane below *
*   `l` move to the right pane *
*   `k` move to the pane above *
*   <span style="color:red">`k` move to the pane above * （好像重复了）</span>
*   `q` show pane numbers
*   `o` toggle between panes
*   `}` swap with next pane
*   `{` swap with previous pane
*   `!` break the pane out of the window
*   `x` kill the current pane

*   `%` 创建一个水平窗格
*   `"` 创建一个竖直窗格
*   `h` 将光标移入左侧的窗格
*   `j` 将光标移入下方的窗格
*   `l` 将光标移入右侧的窗格
*   `k` 将光标移入上方的窗格
*   `q` 显示窗格的编号
*   `o` 在窗格间切换
*   `}` 与下一个窗格交换位置
*   `{` 与上一个窗格交换位置
*   <span style="color:red">`!` 使窗格脱离窗口</red>
*   `x` 关闭当前窗格

### Miscellaneous

*   `t` show the time in current pane

*   `t` 在当前窗格显示时间

I hope this has been helpful. 

希望这篇文章有助于你理解 tmux。

> [ If you liked this, check out my other technical primers here. ]

> 如果喜欢这篇文章，那么不妨再读一读我写的其他入门级技术文章。

## Resources

## 资源

1.  The man page.
2.  A thousand other great tutorials.


1.  man 手册
2.  大量精彩教程