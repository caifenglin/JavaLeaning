

## 版本控制

### 什么是版本控制

版本控制是一种记录一个或若干文件内容变化，以便将来查阅特定版本修订情况的系统。 除了项目源代码，你可以对任何类型的文件进行版本控制。

### 为什么要版本控制

有了它你就可以将某个文件回溯到之前的状态，甚至将整个项目都回退到过去某个时间点的状态，你可以比较文件的变化细节，查出最后是谁修改了哪个地方，从而找出导致怪异问题出现的原因，又是谁在何时报告了某个功能缺陷等等。

### 本地版本控制系统

许多人习惯用复制整个项目目录的方式来保存不同的版本，或许还会改名加上备份时间以示区别。 这么做唯一的好处就是简单，但是特别容易犯错。 有时候会混淆所在的工作目录，一不小心会写错文件或者覆盖意想外的文件。

为了解决这个问题，人们很久以前就开发了许多种本地版本控制系统，大多都是采用某种简单的数据库来记录文件的历次更新差异。

![本地版本控制系统1](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-3/本地版本控制系统.png)

### 集中化的版本控制系统

接下来人们又遇到一个问题，如何让在不同系统上的开发者协同工作？ 于是，集中化的版本控制系统（Centralized Version Control Systems，简称 CVCS）应运而生。 

集中化的版本控制系统都有一个单一的集中管理的服务器，保存所有文件的修订版本，而协同工作的人们都通过客户端连到这台服务器，取出最新的文件或者提交更新。

![集中化的版本控制系统1](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-3/集中化的版本控制系统.png)

这么做虽然解决了本地版本控制系统无法让在不同系统上的开发者协同工作的诟病，但也还是存在下面的问题：

- **单点故障：** 中央服务器宕机，则其他人无法使用；如果中心数据库磁盘损坏有没有进行备份，你将丢失所有数据。本地版本控制系统也存在类似问题，只要整个项目的历史记录被保存在单一位置，就有丢失所有历史更新记录的风险。
- **必须联网才能工作：** 受网络状况、带宽影响。

### 分布式版本控制系统

于是分布式版本控制系统（Distributed Version Control System，简称 DVCS）面世了。 Git 就是一个典型的分布式版本控制系统。

这类系统，客户端并不只提取最新版本的文件快照，而是把代码仓库完整地镜像下来。 这么一来，任何一处协同工作用的服务器发生故障，事后都可以用任何一个镜像出来的本地仓库恢复。 因为每一次的克隆操作，实际上都是一次对代码仓库的完整备份。

![分布式版本控制系统](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-3/分布式版本控制系统.png)

分布式版本控制系统可以不用联网就可以工作，因为每个人的电脑上都是完整的版本库，当你修改了某个文件后，你只需要将自己的修改推送给别人就可以了。但是，在实际使用分布式版本控制系统的时候，很少会直接进行推送修改，而是使用一台充当“中央服务器”的东西。这个服务器的作用仅仅是用来方便“交换”大家的修改，没有它大家也一样干活，只是交换修改不方便而已。

分布式版本控制系统的优势不单是不必联网这么简单，后面我们还会看到 Git 极其强大的分支管理等功能。

## 认识 Git

### Git 简史

Linux 内核项目组当时使用分布式版本控制系统 BitKeeper 来管理和维护代码。但是，后来开发 BitKeeper 的商业公司同 Linux 内核开源社区的合作关系结束，他们收回了 Linux 内核社区免费使用 BitKeeper 的权力。 Linux 开源社区（特别是 Linux 的缔造者 Linus Torvalds）基于使用 BitKeeper 时的经验教训，开发出自己的版本系统，而且对新的版本控制系统做了很多改进。 

### Git 与其他版本管理系统的主要区别

 Git 在保存和对待各种信息的时候与其它版本控制系统有很大差异，尽管操作起来的命令形式非常相近，理解这些差异将有助于防止你使用中的困惑。

下面我们主要说一个关于 Git 其他版本管理系统的主要差别：**对待数据的方式**。

**Git采用的是直接记录快照的方式，而非差异比较。我后面会详细介绍这两种方式的差别。**

大部分版本控制系统（CVS、Subversion、Perforce、Bazaar 等等）都是以文件变更列表的方式存储信息，这类系统**将它们保存的信息看作是一组基本文件和每个文件随时间逐步累积的差异。**

具体原理如下图所示，理解起来其实很简单，每个我们对提交更新一个文件之后，系统记录都会记录这个文件做了哪些更新，以增量符号Δ(Delta)表示。

<div align="center">  
<img src="https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-3deltas.png" width="500px"/>
</br>
</div>

**我们怎样才能得到一个文件的最终版本呢？**

很简单，高中数学的基本知识，我们只需要将这些原文件和这些增加进行相加就行了。

**这种方式有什么问题呢？**

比如我们的增量特别特别多的话，如果我们要得到最终的文件是不是会耗费时间和性能。

Git 不按照以上方式对待或保存数据。 反之，Git 更像是把数据看作是对小型文件系统的一组快照。 每次你提交更新，或在 Git 中保存项目状态时，它主要对当时的全部文件制作一个快照并保存这个快照的索引。 为了高效，如果文件没有修改，Git 不再重新存储该文件，而是只保留一个链接指向之前存储的文件。 Git 对待数据更像是一个 **快照流**。



![](Git.assets/2019-3snapshots-1563432577767.png)



### Git 的三种状态

Git 有三种状态，你的文件可能处于其中之一：

1. **已提交（committed）**：数据已经安全的保存在本地数据库中。
2. **已修改（modified）**：已修改表示修改了文件，但还没保存到数据库中。
3. **已暂存（staged）**：表示对一个已修改文件的当前版本做了标记，使之包含在下次提交的快照中。

由此引入 Git 项目的三个工作区域的概念：**Git 仓库(.git directoty) **、**工作目录(Working Directory)** 以及 **暂存区域(Staging Area)** 。

<div align="center">  
<img src="https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-3areas.png" width="500px"/>
</div>

**基本的 Git 工作流程如下：**

1. 在工作目录中修改文件。
2. 暂存文件，将文件的快照放入暂存区域。
3. 提交更新，找到暂存区域的文件，将快照永久性存储到 Git 仓库目录。

## Git 使用快速入门

### 获取 Git 仓库

有两种取得 Git 项目仓库的方法。

1. 在现有目录中初始化仓库: 进入项目目录运行  `git init`  命令,该命令将创建一个名为 `.git` 的子目录。
2. 从一个服务器克隆一个现有的 Git 仓库: `git clone [url]` 自定义本地仓库的名字: `git clone [url]` directoryname 

### 记录每次更新到仓库

1. **检测当前文件状态** : `git status`
2. **提出更改（把它们添加到暂存区**）：`git add filename ` (针对特定文件)、`git add *`(所有文件)、`git add *.txt`（支持通配符，所有 .txt 文件）
3. **忽略文件**：`.gitignore` 文件
4. **提交更新:** `git commit -m "代码提交信息"` （每次准备提交前，先用 `git status` 看下，是不是都已暂存起来了， 然后再运行提交命令 `git commit`）
5. **跳过使用暂存区域更新的方式** : `git commit -a -m "代码提交信息"`。 `git commit` 加上 `-a` 选项，Git 就会自动把所有已经跟踪过的文件暂存起来一并提交，从而跳过 `git add` 步骤。
6. **移除文件** ：`git rm filename`  （从暂存区域移除，然后提交。）
7. **对文件重命名** ：`git mv README.md README`(这个命令相当于`mv README.md README`、`git rm README.md`、`git add README` 这三条命令的集合)

### 推送改动到远程仓库

- 如果你还没有克隆现有仓库，并欲将你的仓库连接到某个远程服务器，你可以使用如下命令添加：·`git remote add origin <server>` ,比如我们要让本地的一个仓库和 Github 上创建的一个仓库关联可以这样`git remote add origin https://github.com/Snailclimb/test.git` 
- 将这些改动提交到远端仓库：`git push origin master` (可以把 *master* 换成你想要推送的任何分支)

  如此你就能够将你的改动推送到所添加的服务器上去了。

### 远程仓库的移除与重命名

- 将 test 重命名位 test1：`git remote rename test test1`
- 移除远程仓库 test1:`git remote rm test1`

### 查看提交历史

在提交了若干更新，又或者克隆了某个项目之后，你也许想回顾下提交历史。 完成这个任务最简单而又有效的工具是 `git log` 命令。`git log` 会按提交时间列出所有的更新，最近的更新排在最上面。

**可以添加一些参数来查看自己希望看到的内容：**

只看某个人的提交记录：

```shell
git log --author=bob
```

### 撤销操作

有时候我们提交完了才发现漏掉了几个文件没有添加，或者提交信息写错了。 此时，可以运行带有 `--amend` 选项的提交命令尝试重新提交：

```console
git commit --amend
```

取消暂存的文件

```console
git reset filename
```

撤消对文件的修改:

```
git checkout -- filename
```

假如你想丢弃你在本地的所有改动与提交，可以到服务器上获取最新的版本历史，并将你本地主分支指向它：

```
git fetch origin
git reset --hard origin/master
```



### 分支

分支是用来将特性开发绝缘开来的。在你创建仓库的时候，*master* 是“默认的”分支。在其他分支上进行开发，完成后再将它们合并到主分支上。

我们通常在开发新功能、修复一个紧急 bug 等等时候会选择创建分支。单分支开发好还是多分支开发好，还是要看具体场景来说。

创建一个名字叫做 test 的分支

```console
git branch test
```

切换当前分支到 test（当你切换分支的时候，Git 会重置你的工作目录，使其看起来像回到了你在那个分支上最后一次提交的样子。 Git 会自动添加、删除、修改文件以确保此时你的工作目录和这个分支最后一次提交时的样子一模一样）

```console
git checkout test
```

<div align="center">  
<img src="https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-3切换分支.png" width="500px"/>
</div>

你也可以直接这样创建分支并切换过去(上面两条命令的合写)

```console
git checkout -b feature_x
```

切换到主分支

```
git checkout master
```

合并分支(可能会有冲突)

```java
 git merge test
```

把新建的分支删掉

```
git branch -d feature_x
```

将分支推送到远端仓库（推送成功后其他人可见）：

```
git push origin 
```



###  使用.gitignore文件删除掉已经提交的文件

[在Windows系统里创建.gitignore文件](https://www.cnblogs.com/xbj-2016/p/6859493.html)

\1. 在项目根目录下面创建gitignore.txt文件

\2. 把你需要排除的文件名保存到gitignore.txt文件

\3. 在项目根目录下面按住Shift键并邮件然后选择“在此处打开命令窗口”

\4. 执行命令 `ren gitignore.txt .gitignore`

顺序依次执行如下命令：

**First commit any outstanding code changes**, and then, run this command:



```
git rm -r --cached .
```



This removes any changed files from the *index*(staging area), then just run:

```
git `add` .
```

Commit it:

`git `commit` -m `".gitignore is now working"``

最后提交到远程仓库，会发现.gitignore文件起作用了。





# Git-Commit-Log规范（Angular 规范）

## Commit message目的

- 生成 CHANGELOG.md
- 识别不重要的提交
- 在浏览 Git 历史时提供更多信息

## Angular 规范

- 恢复(Revert)
- 信息头部
   \- 必需<type>
- \- 必需<scope>
- \- <subject>文本
- 信息主体
- 信息尾部
   \- 重大更改(Breaking changes)
- \- 引用讨论(Referencing issues)

标准格式如下：

```
<type>(<scope>): <subject> #header
// 空一行
<body>
// 空一行
<footer> 
```

> Header 是必需的，Body 和 Footer 可以省略。

### 格式讲解

### Header

Header部分只有一行，包括三个字段：type（必需）、scope（可选）和subject（必需）。

#### type

用于说明本次commit的类别，只允许使用下面7个标识

- `feat`：新功能（feature）
- `fix`：修补bug
- `docs`：文档（documentation）
- `style`： 格式（不影响代码运行的变动）
- `refactor`：重构（即不是新增功能，也不是修改bug的代码变动）
- `test`：增加测试
- `chore`：构建过程或辅助工具的变动

> 如果type为feat和fix，则该 commit 将肯定出现在 Change log 之中。其他情况（docs、chore、style、refactor、test）由你决定，要不要放入 Change log，建议是不要。

#### scope

用于说明 commit 影响的范围，比如数据层、控制层、视图层等等，视项目不同而不同。

#### subject

是 commit 目的的简短描述，不超过50个字符。

```
以动词开头，使用第一人称现在时，比如change，而不是changed或changes
第一个字母小写
结尾不加句号（.）
```

### Body

用于对本次 commit 的详细描述，可以分成多行。下面是一个范例。

```
More detailed explanatory text, if necessary.  Wrap it to 
about 72 characters or so. 

Further paragraphs come after blank lines.

- Bullet points are okay, too
- Use a hanging indent
```

有两个注意点：

1. 使用第一人称现在时，比如使用change而不是changed或changes。
2. 应该说明代码变动的动机，以及与以前行为的对比。

### Footer

Footer 部分只用于两种情况。

#### 不兼容变动

如果当前代码与上一个版本不兼容，则 Footer 部分以`BREAKING CHANGE`开头，后面是对变动的描述、以及变动理由和迁移方法。

```
BREAKING CHANGE: isolate scope bindings definition has changed.

To migrate the code follow the example below:

Before:

scope: {
  myAttr: 'attribute',
}

After:

scope: {
  myAttr: '@',
}

The removed `inject` wasn't generaly useful for directives so there should be no code using it.
```

#### 关闭 Issue

如果当前 commit 针对某个issue，那么可以在 Footer 部分关闭这个 issue 。

```
Closes #234
```

也可以一次关闭多个 issue 。

```
Closes #123, #245, #992
```

### Revert（可忽视）

还有一种特殊情况，如果当前 commit 用于撤销以前的 commit，则必须以`revert:`开头，后面跟着被撤销 Commit 的 Header。

```
revert: feat(pencil): add 'graphiteWidth' option

This reverts commit 667ecc1654a317a13331b17617d973392f415f02.
```

Body部分的格式是固定的，必须写成This reverts commit <hash>.，其中的hash是被撤销 commit 的 SHA 标识符。

如果当前 commit 与被撤销的 commit，在同一个发布（release）里面，那么它们都不会出现在 Change log 里面。如果两者在不同的发布，那么当前 commit，会出现在 Change log 的Reverts小标题下面。

## 提交频率

关于什么时候提交一次：

每次你写完一个功能的时候，就应该做一次提交（这个提交是提交到本地的git库中）

当然，这里的写完表示的是你的这个功能是没有问题的；



## 示例

- docs(guide/migration): fix typo (misceallenous --> miscellaneous)

- fix(ngStyle): correctly remove old style when new style value is invalid

- chore(*): update jQuery from 3.2.1 to 3.4.0

- test(ngAnimate): ensure that `blockTransitions` can be spied upon


## 参考

>  - https://www.jianshu.com/p/c7e40dab5b05
>  -  https://blog.csdn.net/terrychinaz/article/details/72676721
>  -  https://github.com/angular/angular.js/commits/master


# 推荐阅读

- [Git - 简明指南](http://rogerdudler.github.io/git-guide/index.zh.html)
- [图解Git](http://marklodato.github.io/visual-git-guide/index-zh-cn.html)
- [猴子都能懂得Git入门](https://backlog.com/git-tutorial/cn/intro/intro1_1.html)
- https://git-scm.com/book/en/v2
