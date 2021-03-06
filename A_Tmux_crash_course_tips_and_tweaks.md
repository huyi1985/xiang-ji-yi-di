主要介绍了*nix平台上文本三剑客zsh, tmux和vim中其中Tmux和vim的技巧和集成知识。【网路冷眼】推荐 **[阅读原文 »](http://tangosource.com/blog/a-tmux-crash-course-tips-and-tweaks/)**

----------------------

# A Tmux crash course: tips and tweaks.

# Tmux 简明教程：技巧和配置

> By Tonatiuh Nuñez in Development, FeaturedPosted May 13, 2015 at 2:26 pm
> 

## ~ Intro

## ~ 简介

If you are one of those devs who uses the terminal a lot and ends up with way too many tabs open, or practices pair programming, then this post is for you. During the last months, I’ve started using Tmux a lot. Since I’ve found it to be very useful, I thought I would write a post where I share a few recommendations and pro-tips. I’ll show you what Tmux is and how to use it in combination with Vim to make a more effective and elegant use of the Terminal.

有些开发者经常要使用终端模拟器工作，导致最终打开了过多的标签页。如果你也是他们当中的一员，或者你正在实践结对编程，那么我推荐你读一读这篇文章。从上个月开始，我开始大量使用 Tmux 并且发现 Tmux 非常实用，所以我想应该写一篇文章，与诸位分享一些有关使用 Tmux 的建议和专业方案。本文将先介绍 Tmux 是什么，然后讲解如何使用 Tmux，才能使其同 Vim 结合起来，打造出更高效、更优雅的终端模拟器。

So, this is what we’ll cover:

本文将会包含以下内容：

* Tmux basics.
* The best of Tmux
	* Windows
	* Panes
	* Sessions
	* Fast text navigation and copying
	* And a very neat pair programming feature
* Tweaks to improve Vim integration.
	* Colorscheme background
	* Static cursor shape
	* Indentation at pasting
* A few extras to enhance the Tmux experience.
	* Tmuxinator for session automation.
	* Changing your color of your Tmux bar.

1. Tmux 的基础
1. Tmux 中最棒的功能
	1. 窗口（Window）
	1. 窗格（Pane）
	1. 会话（Session）
	1. 快速在文本间移动光标或复制文本
	1. 非常**轻巧**的结对编程功能
1. 调整 Tmux 以增强其同 Vim 的集成度
	1. 调整背景的配色方案
	1. 调整光标的形状
	1. 调整粘贴时的文本缩进
1. 其他能够提升 Tmux 体验的工具或技巧
	1. 用 Tmuxinator 自动创建会话
	1. 改变 Tmux 状态栏的颜色
 
An important thing to bear in mind, this is the tool stack I had installed while writing this post, I tested what I say here with these versions:

请注意，在撰写本文的过程中，我安装了以下这一组软件，并在测试时使用了这些版本：

- Tmux 1.9a
- Vim 7.4
- iTerm 2.1
- Mac OS (Mavericks and Yosemite)
 
Let’s start!

让我们开始吧！
 
## ~ The Basics

## 基础知识

### What is Tmux?

### 什么是Tmux？

Tmux is a tool that allows running multiple terminal sessions through a single terminal window. It allows you to have terminal sessions running in the background and attach and detach from them as needed, which is very useful. Later on, we will see how to make the most out of that feature.

Tmux 是一个工具，用于在一个终端窗口中运行多个终端会话。不仅如此，你还可以通过 Tmux 使终端会话运行于后台或是按需接入、断开会话，这个功能非常实用。稍后，我们将会看到如何最大限度地利用这个功能。

Here’s a screenshot of a Tmux session:

如图所示，这就一个是 Tmux 的会话：

![image1](http://tangosource.com/wp-content/uploads/2015/05/image1.png)

What you see in the image:

从图中我们可以看出：

- Left: Vim
- Right: a system’s shell
- Bottom-left: the Tmux session name (“pomodoro-app”)
- Bottom-middle: the current Tmux windows (“app log”, “editor” and “shell”)
- Bottom-right: the current date

- 左侧：Vim
- 右侧：系统 Shell
- 左下方：Tmux 会话的名字（“pomodoro-app”）
- 下方的中部：当前会话中的 Tmux 窗口（“app log”、“editor”和 “shell”）
- 右下方：当前的日期

### How to install Tmux?

### 如何安装 Tmux？

#### In Mac OS:

#### 在 Mac OS 中安装：

* Install Homebrew
1. 安装 Homebrew

```
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
[Here’s](https://raw.githubusercontent.com/Homebrew/install/master/install) more info.

有关安装 homebrew 的详细的信息可以参考[这里](https://raw.githubusercontent.com/Homebrew/install/master/install)。

* Install Tmux:
2. 安装 Tmux

```
$ brew install tmux
```

#### In Ubuntu:

#### 在 Ubuntu 中安装：

Run this on the terminal:

在终端模拟器中输入如下命令：

```
$ sudo apt-get install tmux
```

### The Tmux prefix

### Tmux 的快捷键前缀（Prefix）

In order to isolate its own keyboard shortcuts from other shortcuts, Tmux provides a shortcut prefix. When you want to trigger a Tmux shortcut you will press the Tmux prefix and then the Tmux shortcut key. The prefix that Tmux uses by default is Ctrl-b (“Ctrl” key in combination with the “b” key). For instance, let’s say you want to trigger the shortcut that lists the current Tmux sessions, which is the ｀s｀ key. Here is what you will need to do:

为了使自身的快捷键和其他软件的快捷键互不干扰，Tmux 提供了一个快捷键前缀。当想要使用快捷键时，需要先按下快捷键前缀，然后再按下快捷键。Tmux 所使用的快捷键前缀默认是组合键 `Ctrl-b`（同时按下 `Ctrl` 键和 `b` 键）。例如，假如你想通过快捷键列出当前 Tmux 中的会话（对应的快捷键是 s），那么你只需要做以下几步：

* Press Ctrl-b keys (Tmux prefix)
* Release Ctrl-b keys
* Press the s key

1. 按下组合键 `Ctrl-b` (Tmux 快捷键前缀)
1. 放开组合键 `Ctrl-b` 
1. 按下 `s` 键
 
A few recommendations:
这里有一些小建议：

If you have not already mapped the `ctrl` key to the `caps-lock` key and vice-versa I suggest you do it.

首先我建议对调 `Ctrl` 键和 `Caps-Lock` 键的功能。

Calling ctrl from the caps-lock key is very practical. This is because when coding you need to call ctrl very frequently. Moreover, it is a lot easier/quicker given the caps-lock key aligns with the default position of your fingers in the keyboard.

通过按下 `Caps-Lock` 键来代替 `Ctrl` 键将会非常实用。因为在编码过程中，你需要频繁地按下 `Ctrl` 键，而由于 `Caps-Lock` 与手指在键盘的起始位置处于同一直线，所以按下 `Caps-Lock` 键会更加容易、便捷。
 
I recommend changing the Tmux prefix to Ctrl-a . Once the `Ctrl` key has been set to the `Caps-Lock` key, it gets a lot easier/quicker to call Ctrl-a instead of Ctrl-b, because the new prefix keys are very close to each other on the keyboard.

其次，我建议将 Tmux 的快捷键前缀变为 `Ctrl - a`。用 `Caps-Lock` 键替代了 `Ctrl` 键之后，由于 `Caps-Lock` 键与 `a` 键离得更近，所以按下 `Ctrl - a` 就将会比按下 `Ctrl - b` 更容易、更便捷。

Here is what you need to add in your ~/.tmux.conf file to change the prefix to Ctrl-a:

若要将快捷键前缀变更为 `Ctrl - a` ，请将以下配置加入到 Tmux 的配置文件 `~/.tmux.conf` 中：

```
unbind C-b
set -g prefix C-a
```

### The config file

### Tmux 的配置文件

`~/.tmux.conf` is a file that Tmux reads every time a new session opens. It’s where the customizations for Tmux need to be placed. Suggestion: in the case that you need (**and chances are you will**) to apply a new change made to the without opening a new session, you can add the following line to the ~/.tmux.conf file:

每当开启一个新的会话时，Tmux 都会先读取 `~/.tmux.conf` 这个文件。该文件中存放的就是对 Tmux 的配置。

小提示：如果你不希望直到开启一个新的会话，新的配置项才能生效，那么你可以将下面这一行配置加入到文件 `~/.tmux.conf` 中。

```
# bind a reload key
bind R source-file ~/.tmux.conf \; display-message "Config reloaded.."
```

This way, once you have added a new change to the `~/.tmux.conf` file, just press `Ctrl-b r` to reload your configuration without having to open a new Tmux session.

这样配置了之后，每当向 `~/.tmux.conf` 文件中添加了新的配置，只需要按下 `Ctrl-b r` 就可以重新加载配置并使新的配置生效，从而免去了开启一个新的会话。
 
## ~ The best of Tmux

## Tmux 中最棒的功能

Quick note: the screenshot shown here may differ slightly from what you see by default when you install Tmux. This is because I modified the status bar. If you want to do the same follow the steps on the “Pimp your Tmux bar” section of this post.

提示：下面这张图也许与你使用 Tmux 时看到的画面略有不同。这是因为我修改了 Tmux 的状态栏。如果你也想修改成和一样的，那么可以参照“美化 Tmux 的状态栏”这一节中的步骤。

### Panes

### 窗格

I like the idea of dividing the screen **vertically**, so that on one side of the screen I have Vim and on the other side I have the output of my code. I could even have another console if I wanted to. Here’s how Tmux makes it happen:

我认为沿竖直方向分割屏幕是个不错的主意，这样我就可以在一边使用 Vim，而在另一边输出代码的运行结果了。甚至有时我还会再打开一个控制台。下面我就要讲解如何利用 Tmux 实现这一切。

![image2](http://tangosource.com/wp-content/uploads/2015/05/image2.png)

What you see in the image:

从图中可以看出：

- Left side: Vim (at the top: a Ruby class file, at the bottom: a test file for the Ruby class).
- Right side: a bash session.

* 左侧：Vim（左上方是一个 Ruby 的类文件，左下方是针对这类编写的测试文件）
* 右侧：一个 Bash 的会话

A vertical `pane` is easy to create. Once you have launched a new Tmux session just press `Ctrl-b %` and a new vertical pane will appear. Also, if what you need is a horizontal division then press `Ctrl-b "`. Navigating through Tmux panes is easy, just press the Tmux prefix and then any of the direction arrows depending on which pane you want to go.

要创建一个竖直放置的窗格并不困难。待开启了一个 Tmux 会话之后，只需再按下 `Ctrl-b %` ，一个竖直放置的窗格就出现了。另外，若要把屏幕沿水平方向分割，则只需要按下 `Ctrl-b "`。在 Tmux 的窗格间移动光标也很简单，只需要先按下 Tmux 的快捷键前缀，然后再按下对应的方向键就可以让光标进入到目标窗格了。

### Windows

### 窗口

In Tmux, a window is a container for one or more panes. Tmux windows allow you to arrange multiple panes inside windows depending on what you need. For instance, in my case I usually have one window called “server” for running the app’s server (where I can see the log), another window called “editor” (where I do the coding). One pane for Vim and another for running the code tests. And another window called “shell”, which is a bash shell where I can run system commands. Tmux windows are useful since they allow users to allocate panes inside of them, **to see more about each pane by going to the window that contains it**. This is an efficient use of the available screen space.

在Tmux中，窗口就是个容器。你可以将多个窗格放置在窗口中，至于如何排列这些窗格，也是完全取决于你的需要。例如，我经常是这样做，先开启一个叫作“server”的窗口用于运行应用程序的服务器（在这个窗口中可以看到服务器的日志），然后开启另一个叫作“editor”的窗口用于编写代码。在这个窗口中有两个窗格，一个用于 Vim，一个用于运行测试代码。最后再开启一个叫作“shell”的窗口用于通过 Bash shell 运行命令。Tmux 的窗口很实用，因为它允许用户在里面创建窗格，<span style="color: red">// 只要进入了窗口就可以查看每个窗格的更多信息</span>，这样可以有效地利用屏幕的空间。

The list of existent windows in a Tmux session displays at the bottom of the screen. Here is an example of how Tmux displays (by default) the list of windows created. In this case, there are three windows: “server”, “editor” and “shell”):

在 Tmux 的会话中，现有的窗口将会列在屏幕下方。下图所示的就是在默认情况下 Tmux 列出现有窗口的方式。这里一共有三个窗口，分别是“server”、“editor”和“shell”。

![image3](http://tangosource.com/wp-content/uploads/2015/05/image3.png)

In order to create a new window you need to press `Ctrl-b c`. To navigate through windows press `Ctrl-b` followed by the index number of the window you want to go. The index number displays next to the name.

若要创建一个窗口，只需要按下`Ctrl-b c`；若要切换窗口，只需要先按下`Ctrl-b`，然后再按下想切换的窗口所对应的数字，该数字会紧挨着窗口的名字显示。

### Sessions

### 会话

A Tmux session can contain multiple windows. Sessions are a neat feature; I can create a Tmux session that is **exclusive to** a particular project. To create a new session just run the following command on your terminal:

一个 Tmux 会话中可以包含多个窗口。会话提供的功能很简单，例如可以为一个特定的项目创建一个专用的 Tmux 会话。若要创建一个新的会话，只需要在终端模拟器运行如下的命令：

```
$ tmux new -s <name-of-my-session>
```

If I need to work on a different project I will just create a new session for that. Although the focus will be on the new session, the original session will remain alive. This allows me to get back to it later, and continue where I left off. To create a new session press `Ctrl-b :` and then enter the following command:

假设我还需要开发另一个项目，于是我就会为此再新建一个会话。虽然进入了新的会话，但是原来的会话并没有消失。所以我可以在稍后回到之前的会话继续工作。若要创建一个新的会话，只需要按下 `Ctrl-b :` ，然后输入如下的命令：

```
new -s <name-of-my-new-session>
```

Tmux sessions remain alive until you restart your machine or you explicitly kill the session. As long as you don’t restart your machine, you can jump from a project’s session to another as you need it.

除非显式地关闭会话，否则 Tmux 的会话在重启计算机之前都不会消失。只要还没有重启计算机，你都可以自由地从一个项目的会话跳转到另一个。
 
### Navigation through Tmux sessions

### 在 Tmux 的会话间切换

To get a list of the existing sessions, press `Ctrl-b s`. Here is an example of what Tmux will show you:

若要获取现有会话的列表，可以按下`Ctrl-b s`。下图所示的就是会话的列表：

![image4](http://tangosource.com/wp-content/uploads/2015/05/image4.png)

Each session listed has an ID number, starting from zero. In order to go to that session type the session’s ID number in your keyboard. In the case that you are not in Tmux but you have one or more sessions running just use:This command will take you back to your Tmux sessions.

列表中的每个会话都有一个 ID，该 ID 是从 0 开始的。按下对应的 ID 就可以进入会话。如果你已经创建了一个或多个会话，但是还没有运行 Tmux，那么可以输入如下命令以接入已开启的会话。

```
$ tmux attach
```

### Fast text navigation and copying

### 在文本间快速移动光标，复制文本

I always disliked the fact that to copy content from iTerm2 quickly you need to use the keyboard plus the mouse. I think that there should be a quicker way to do it without having to use the mouse. Fortunately, **Tmux allows for that, since it is run from the command line, where the use of the mouse is not allowed.**

在 iTerm2 中，要想快速地复制内容就不得不键盘和鼠标一起用，这一点我一直很不喜欢。我想一定会有不需要用鼠标的、更快捷的复制方法。幸运的是，Tmux就提供了只用键盘就可以完成复制的功能，这源于 Tmux 是从命令行启动的，而在命令行界面是无法使用鼠标的。

#### Navigate text

#### 在文本间移动光标

Tmux allows for text navigation in a way that is very similar to Vim. You know, where the k key goes one line up, w moves one word forwards, and so on. **Yet** you can increase Tmux’s similarity with Vim by telling it to use the `vi` mode. Here is what you need to add in your `~/.tmux.conf` file to accomplish this:

在 Tmux 中可以使用与 Vim 极为相似的方式在文本间移动光标。正如你熟知的那样，用 `k` 键可以将光标移动到上一行，用 `w` 键可以向后移动一个单词等等。而且还可以通过把 Tmux 设为 vi 模式，使其与 Vim 的操作更加接近。为此，需要将以下配置加入到文件 `~/.tmux.conf` 中。

```
# Use vim keybindings in copy mode
setw -g mode-keys vi
```

### Send copied text to System’s clipboard

### 将复制下来的文本发送到系统的剪贴板中

By default, when you copy text from Tmux, the text is only available to be pasted inside that same Tmux session. In order to make that text available to be pasted anywhere, you have to tell Tmux to copy to system’s clipboard. Here’s how to do it:

在默认情况下，当从 Tmux 中复制文本时，复制下来的文本只能粘贴到同一个 Tmux 会话中。若要使复制下来的文本可以粘贴到任何位置，就需要让 Tmux 将文本复制到系统的剪贴板。为此，我们需要这样做：

Install retach-to-user-namespace, this is very easy with brew, just run the following command:

安装 retach-to-user-namespace。用 brew 安装的话将会非常简单，只需要运行下面这条命令：

```
$ brew install reattach-to-user-namespace
```

Update `~/.tmux.conf` file

在配置文件 `~/.tmux.conf` 中加入以下内容：

```
# invoke reattach-to-user-namespace every time a new window/pane opens
set-option -g default-command "reattach-to-user-namespace -l bash"
```

### Select and copy text

Now that the `vi` mode is set and `rettach-to-user-namespace` installed, let’s see how to copy text from a Tmux session. Let’s say you’ve run the `ifconfig` command because you wanted to copy your ip address. Now, follow these steps to copy that text:

既然已经设置成了 vi 模式，也安装了 rettach-to-user-namespace，下面就让我们来看看如何从 Tmux 的会话中复制文本吧。假设要复制的是 IP 地址，于是我们先运行了 `ifconfig` 命令。接下来就请跟随以下的步骤：

Enter copy mode: `Ctrl-b [`. You’ll see a short highlighted text appear at the top right of the screen as in the following image (“*[0/0]*”).

首先按下 `Ctrl-b [` 进入复制模式。然后应该就可以看到一小段高亮的文本出现在了屏幕的右上角 (“*[0/0]*”)（如下图所示）。
 
![image5](http://tangosource.com/wp-content/uploads/2015/05/image5.png)

Start moving across the text as you would do in Vim: with j, k, l, h, etc..

接下来就可以像在 Vim 中一样用 `j`、`k`、`l` 和 `h` 等键在文本间移动光标了。

Once you get to the text you want to copy press the spacebar and start selecting text (exactly as you would do it in Vim).

把光标移动到想复制的文本上后再按下空格键就可以开始选择文本了（这和在 Vim 中复制文本的步骤一模一样）。

Once text is selected press the enter key

选择完要复制的文本后再按下回车键。
 
Now that you have copied your IP address, paste it wherever you want.

这样 IP 地址就复制下来并可以粘贴到任何地方了。

### Make text copying even more Vim-like

### 让复制文本的操作更像 Vim

You can use `v` key to select text and `y` to copy it, just add the following to your `~/.tmux.conf` file:

还可以将 Tmux 设置为用 `v` 键选择文本，用 `y` 键复制文本。为此只需要将下面的配置项加入到配置文件 `~/.tmux.conf` 中。

```
# start selecting text typing 'v' key (once you are in copy mode)
bind-key -t vi-copy v begin-selection
# copy selected text to the system's clipboard
bind-key -t vi-copy y copy-pipe "reattach-to-user-namespace pbcopy"
```

### Effective pair programming

### 高效的结对编程

You can share the address of a Tmux session with someone else, and that person can connect to the session via `SSH`. Since the connection runs over `SSH`, it will be very lightweight. For the user connecting to the remote Tmux session, it will feel as if the session is running locally, providing the internet connection is fast enough.

你可以将 Tmux 会话的地址分享给他人，这样他们就可以通过 SSH 接入这个会话了。由于会话是建立在 SSH 之上的，所以不会产生**额外的开销**。通过使用高速的互联网，对于那些连接到远程会话上的用户而言，他们会觉得这个会话就是运行在本地的。

#### Tmux with Tmate

#### 在Tmux 中使用 Tmate

Tmate is a tool that makes it very easy to create a Tmux session and share it with someone else over the internet. To share a new Tmux session using Tmate these are the steps you have to follow:

Tmate 是一个工具，不但可以用于轻松创建 Tmux 会话，而且还能通过互联网把该会话共享给其他人。若要使用 Tmate 共享 Tmux 会话，请按照以下步骤操作：

* Install Homebrew:

1. 安装 Homebrew

```
$ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

* Install Tmate

1. 安装 Tmate

```
$ brew update             && \
  brew tap nviennot/tmate && \ 
  brew install mate
```

* Launch a new session with Tmate:

1. 使用 Tmate 开启一个新的会话

```
$ tmate
```

* Copy the SSH URL given by Tmate on the Tmux session. An example is showed in the following image (message at the bottom: “*[tmate] Remote session: ssh …*”):

1. 从 Tmux 的会话中复制由 Tmate 产生的 SSH  URL。如下图所示，请注意屏幕下方的信息“[tmate] Remote session: ssh …”：

![image6](http://tangosource.com/wp-content/uploads/2015/05/image6.png)

* Ask the other person to access via *SSH* using the URL you just copied.

1. 利用刚刚复制下来的 URL 就可以邀请其他人通过 SSH 访问你的会话了。

Now that you know how to make good use of Tmux’s pair programming feature, you can further improve the interactivity of your session by doing a voice call via your preferred provider.

了解了如何利用 Tmux 的结对编程功能之后，还可以再利用您所喜爱的运营商提供的语音服务进一步加强会话交互性。
 
## ~ Tweaks for Vim integration

## 调整 Tmux 以增强其同 Vim 的集成度

### Colorscheme background

### 调整背景的配色方案

When I first opened Vim through Tmux I found the colors weren’t being correctly applied. The background color was being displayed only where characters appeared. Here is an example:

当我第一次通过 Tmux 打开 Vim 时，我发现 Vim 的颜色没有正确显示。正如下图所示，只有有字符的地方才有背景色。

![image7](http://tangosource.com/wp-content/uploads/2015/05/image7.png)

This issue is due to Vim’s need of setting a different term parameter when ran through Tmux. To set the right term parameter just add the following lines to your `~/.vimrc` file:

这个问题是由于当通过 Tmux 运行了 Vim 以后，Vim 需要<span style="color: red">设置一个不同的 term 参数</span>。为了正确设置 term 参数，需要将以下这行配置加入到 VIM 的配置文件 `~/.vimrc` 中。

```
if exists('$TMUX')
  set term=screen-256color
endif
```

After updating the ~/.vimrc file, the color scheme is displayed correctly:

在更新了配置文件 ~/.vimrc 以后，颜色应该就可以正确显示了。

![image8](http://tangosource.com/wp-content/uploads/2015/05/image8.png)

### Static cursor shape

### 调整光标的形状

By default when Vim is run through Tmux, the cursor shape is always the same regardless of the current Vim mode (insert, visual, etc). This makes it hard to identify the current Vim mode. To fix this, you need Tmux to tell iTerm to update the cursor shape. You can do it by adding the following to your ~/.vimrc file:

在默认情况下，当通过 Tmux 运行 Vim 时，无论当前 Vim 是处于插入模式、可视模式还是其他模式，光标的形状都是一样的。这样就很难判断当前的 Vim 模式是什么。若要填补这个缺陷，就需要让 Tmux 通知 iTerm 更新光标的形状。为此，需要将以下配置加入到文件 ~/.vimrc 中。

```
if exists('$ITERM_PROFILE')
  if exists('$TMUX') 
    let &t_SI = "\<Esc>[3 q"
    let &t_EI = "\<Esc>[0 q"
  else
    let &t_SI = "\<Esc>]50;CursorShape=1\x7"
    let &t_EI = "\<Esc>]50;CursorShape=0\x7"
  endif
end
```

*Thanks to Andy Fowler who originally [shared this tip](https://gist.github.com/andyfowler/1195581).*

在这里我要感谢 Andy Fowler，是他最先分享了调整光标的形状[这个技巧](https://gist.github.com/andyfowler/1195581)。

### Indentation at pasting

### 调整粘贴时的文本缩进

Sometimes, when pasting text into Vim the indentation of the text is changed, which is a problem when you paste a large amount of text. This issue can be prevented by executing `:set nopaste` before pasting. However, there is a better way to do this. By adding the following to your `~/.vimrc` file, Vim will automatically prevent auto-indenting the text when pasting:

在 Vim 中粘贴文本时可能会遇到这样的问题，有时文本的缩进会发生变化，特别是在粘贴大量的文本时，这个问题会更加明显。虽然可以通过在粘贴前执行 `:set nopaste` 来解决这个问题，但是这里还有一种更好的解决方法。就是把下面这段配置加入到配置文件 `~/.vimrc` 中，这样 Vim 就会自动地阻止粘贴文本时的自动缩进。

```
" for tmux to automatically set paste and nopaste mode at the time pasting (as
" happens in VIM UI)
 
function! WrapForTmux(s)
  if !exists('$TMUX')
    return a:s
  endif
 
  let tmux_start = "\<Esc>Ptmux;"
  let tmux_end = "\<Esc>\\"
 
  return tmux_start . substitute(a:s, "\<Esc>", "\<Esc>\<Esc>", 'g') . tmux_end
endfunction
 
let &t_SI .= WrapForTmux("\<Esc>[?2004h")
let &t_EI .= WrapForTmux("\<Esc>[?2004l")
 
function! XTermPasteBegin()
  set pastetoggle=<Esc>[201~
  set paste
  return ""
endfunction
 
inoremap <special> <expr> <Esc>[200~ XTermPasteBegin()
```

*Thanks to Marcin Kulik who originally [shared this tip](https://coderwall.com/p/if9mda/automatically-set-paste-mode-in-vim-when-pasting-in-insert-mode).*

在这里我要感谢 Marcin Kulik，是他最先分享了[这个技巧](https://coderwall.com/p/if9mda/automatically-set-paste-mode-in-vim-when-pasting-in-insert-mode)。
 
## ~ Nice extras

## 其他能够提升 Tmux 体验的工具或技巧

### Tmuxinator (automate sessions for your projects)

### Tmuxinator （为项目自动创建会话）

Let’s say you start coding for application A and you always create a Tmux session with three windows for that: “servers”, “editor” (for the project’s code) and “shell” (to run system commands). At certain point of the day you need to temporarily switch to application B, and you will create a Tmux session with the same windows arrangement you had for application A, with a few slight differences (such as the directory and some commands). With Tmuxinator, you can declare the configuration for each Tmux session and then create them with a single command! Pretty sweet.

假设你正在开发应用程序 A。在开发过程中，经常要创建 Tmux 会话，会话中包含“server”、“editor”（用于编写代码）和“shell”（用于运行系统命令）这 3 个窗口。不仅如此，在一天之中的某个特定的时间你还需要临时进入到应用程序 B 的开发工作中。于是你又不得不创建另一个会话，虽然有略微的不同（比如目录和某些命令），但是会话中还是要包含应用程序 A 中的那 3 个窗口。但是有了 Tmuxinator，你就可以为每个Tmux 会话声明一个配置，然后用 1 条命令就能创建出这个会话了。这功能太棒了，不是吗。

Tmuxinator is a gem that allows you to automate the creation of Tmux sessions. It is done specifying the details of the sessions in the configuration files and then creating the sessions with a command.
Let’s see how to install Tmuxinator and how to add the configuration to start a project session. Install the Tmuxinator gem by running the following command:

Tmuxinator 是一个 Ruby 的 gem 包，可用于创建 Tmux 的会话。它的工作方式是先在配置文件中定义会话中的细节，然后用 1 条命令创建出这些会话。下面就让我们看看如何安装 Tmuxinator 以及如何添加配置来为指定项目开启一个会话。可以通过运行如下命令安装 Tmuxinator 的 gem 包。

```
$ gem install tmuxinator
```

Now that Tmuxinator is installed you can run the tmuxinator or mux commands from the system’s shell. Let’s create the configuration file for your first application as described above (three windows: “servers”, “editor” and “shell”), tell Tmuxinator to create and open the config file for this project:

安装好了 Tmuxinator 以后，就可以在系统 Shell 中运行 `tmuxinator` 或 `mux` 命令了。下面就让我们为上述的应用程序（有 3 个窗口，分别是“servers”, “editor” 和 “shell”）来创建一个配置文件吧。下面这条命令的作用是为这个项目创建并打开一个配置文件。

```
$ tmuxinator new project_a
```

At this point the file `~/.tmuxinator/project_a.yml` should have been automatically opened. In order to accomplish what is needed for project A you need to update the content of project_a.yml to:

按下回车键后，就会自动打开文件 `~/.tmuxinator/project_a.yml`。为了实现项目 A 所需的配置，你需要把 `project_a.yml` 的内容更新为：

```
name: project_a
root: <the-folder-of-project-A>
 
windows:
  - server: <command-to-start-application-server>
 
  - editor:
    layout: even-horizontal
    panes:
      - vim
      - <command-to-launch-tests-guard>
 
  - shell: ''
```

Once you have added the configuration to the Yaml file for project A, just run the following command to start the Tmux session:

一旦将上面的配置添加到了项目 A 的 Yaml 文件中，只需要运行下面这条命令就可以启动 Tmux 的会话了。

```
$ tmuxinator start project_a
```

Or, if you prefer, use the Tmuxinator alias:

当然如果愿意的话，你也可以使用 Tmuxinator 命令的别名：

```
$ mux start project_a
```

There you have it. Now, in order to start coding for project A just run the Tmuxinator command.

大功告成了。现在，每当想进入项目 A 的编码工作时，就只需要运行 Tmuxinator 命令。

*Find the official documentation in the [Tmuxinator repo](https://github.com/tmuxinator/tmuxinator).

可以到[这里](https://github.com/tmuxinator/tmuxinator)查看Tmuxinator的官方文档。

### Pimp your Tmux bar

### 美化 Tmux 的状态栏

By default, the Tmux bar looks like the following (green bar at the bottom of the image):

默认情况下，Tmux的状态栏看起来是下图这个样子：

![image9](http://tangosource.com/wp-content/uploads/2015/05/image9.png)

You can change its appearance if you want. In my case I like something cleaner as in:

我们可以根据需要改变状态栏的外观。对我来说，我喜欢下图这种清爽的外观。

![image10](http://tangosource.com/wp-content/uploads/2015/05/image10.png)

In order to accomplish it, I use the following settings in my `~/.tmux.conf` file:

为了达到上图的效果，我将如下的配置加入到了配置文件 ~/.tmux.conf 中。

```
# Status bar
  # colors
  set -g status-bg black
  set -g status-fg white
 
  # alignment
  set-option -g status-justify centre
 
  # spot at left
  set-option -g status-left '#[bg=black,fg=green][#[fg=cyan]#S#[fg=green]]'
  set-option -g status-left-length 20
 
  # window list
  setw -g automatic-rename on
  set-window-option -g window-status-format '#[dim]#I:#[default]#W#[fg=grey,dim]'
  set-window-option -g window-status-current-format '#[fg=cyan,bold]#I#[fg=blue]:#[fg=cyan]#W#[fg=dim]'
 
  # spot at right
  set -g status-right '#[fg=green][#[fg=cyan]%Y-%m-%d#[fg=green]]'
```

## ~ Conclusion

## 总结

In summary, we revised basic functionality and the most useful features Tmux offers. We also reviewed a few tweaks and extras. What are your impressions about Tmux so far? Are there any other features/tweaks that you have found useful? Let us know in the comments section below.

在这篇文章中我们先介绍了 Tmux 的基本功能，然后介绍了 Tmux 中最棒的几个功能。这之后介绍了一些配置以及几个能够提升 Tmux 体验的工具。至此，诸位对 Tmux 的印象如何呢？找到了其他什么有用的功能或配置了吗？如果有的话欢迎留言告诉我们。

Thanks for reading!

感谢您阅读本文！

TN.（应该是作者的名字吧）