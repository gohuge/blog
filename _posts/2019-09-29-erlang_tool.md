---
layout: post
title: Erlang 5种必备工具
date: 2019-09-29 11:32:20
img: mac.jpg
tags: [erlang]
---

## 在Shell中工作
与许多其他语言一样，与Erlang VM交互的最常见方法是通过Erlang Shell。您可以使用它来测试代码，学习如何使用Erlang以及调试生产系统。Erlang shell是一个功能强大的东西，但是您可以使用下面将向您展示的三个工具为其添加更多功能。

1. user_default
通常，要在Erlang中的模块外部评估函数，必须在其名称前加上相应的模块名称。例如，如果要汇总列表，则必须执行以下操作：
```
1> lists:sum([1, 2, 3]).
6
2>
```
但是您会注意到，对于Shell中的某些功能，您不需要显式定义。例如：
```
1> c(your_module).
{ok,your_module}
2> h().
1: c(your_module)
-> {ok,your_module}
ok
3> e(1).
{ok,your_module}
4>
```
这些功能住在哪里？它们在shell_default模块中定义。您也可以添加自己的默认功能。为此，您必须创建并加载名为的模块user_default。例如，我喜欢在使用Erlang shell时运行bash命令；我不想离开外壳只是为了复制文件。Erlang / OTP为您提供了以下功能：os:cmd/1，但这不会打印出命令的输出。相反，此命令将以字符串形式返回该输出。您必须打印字符串才能正确查看输出。

因此，使用user_default，我在user_default.erl文件中添加了以下几行：
```
-module(user_default).
-export([cmd/1].

cmd(Cmd) -> io:format("~s~n", [os:cmd(Cmd)]).
```
现在，我可以运行，执行和打印os:cmd/1使用shell 的结果：
```
1> c(user_default).
{ok, user_default}
2> cmd("cat user_default.erl").
-module(user_default).
-export([cmd/1].

cmd(Cmd) -> io:format("~s~n", [os:cmd(Cmd)]).

ok
3>
```

2. 〜/ .erlang
但是，然后，我必须手动加载user_default打开的每个外壳程序。~/.erlang营救！~/.erlang是一个几乎像~/.bashrcbash 一样工作的文件。启动后，外壳程序将读取~/.erlang并评估它在那里找到的每个表达式，就好像它们已输入到外壳程序一样。

因此，我在该文件中添加了以下几行：
```
UD = "/path/to/my/user_default".
shell_default:c(UD).

~/.erlang实际上，我要比这复杂得多，但是shell_default应该可以让您了解我的修改要点。现在，我打开的每个外壳都支持cmd/1。
```

3. erlang-history
关于Erlang shell的最烦人的事情之一是，它不能很好地跟踪命令历史。实际上，默认设置将在关闭外壳程序时清除所有命令历史记录。我已经看到许多开发人员通过使用一个erlang表达式笔记本来解决此限制，每次他们从中复制并粘贴到控制台中。（其中一些人使用user_default和/或~/.erlang为此。）

但这仍然不是真正的解决方案，对不对？为了正确解决此问题，请在安装新版本的Erlang / OTP之后立即安装erlang-history。对于Erlang / OTP发行版来说，这是一个很小的，几乎看不见的黑客，其目的很简单：在Erlang Shell中跟踪先前的命令并让您重用它们。简单，却非常有用。

**管理您的应用**
当您跳过简单的示例并进入OTP应用程序领域时，使用构建工具可以更好地工作，该工具可以帮助您组织代码，管理依赖关系，构建发行版，运行测试等。

使用Erlang时，可以拥抱Makefile，也可以尝试使其离得尽可能远。

4. erlang.mk
如果使用Makefiles构建和管理项目对您来说很自然，那么您可能会喜欢erlang.mk。erlang.mk基本上是一个很大的Makefile脚本，您可以将其包含在自己的Makefile中，例如：
```
PROJECT=your_app
include erlang.mk
```
这样，您就可以访问许多命令，例如

$ make 建立你的项目
$ make tests 运行您的测试套件
$ make rel 产生发行
您可以使用找到整个列表$ make help，当然，可以通过扩展Makefile添加更多内容。

erlang.mk还提供了插件基础结构，您可以在其中添加构建工具，例如hexer.mk或elvis.mk。

最好的事情erlang.mk是，它为构建几乎所有现有的Erlang库提供了必要的支持，而与库所有者用来维护它的工具无关。随着erlang.mk您可以包括带有内置应用螺纹钢，rebar3，erlang.mk（预期：P），即席Makefile中，和其他人在你的托管。您可以从github，hex.pm，bitbucket，本地文件系统和许多其他地方下载deps 。显然erlang.mk，Erlang的用途更加广泛。

5. rebar3

现在，如果Makefiles不是您的事，并且您希望使用Erlang / OTP团队赞助的官方构建工具（自2016年3月起），那么rebar3是您的理想选择。

rebar3完全用Erlang编写，其想法是您可以在不向项目中添加任何Makefile的情况下使用它。

一旦你安装它在你的系统，你应该添加rebar.config在你的项目的根文件夹中的文件。最小的看起来像这样（尽管以下选项是可选的）：
```
{deps, []}.
{erl_opts, [debug_info]}.
{cover_enabled, true}.
```