---
layout: post
title:  "Commit 中包含大文件会导致推送时无法连接到 GitHub"
date:   2025-06-22 20:33:00 +0800
categories: git
---

前段时间的某个晚上，我想把 Logseq 的学习笔记更新推送到 GitHub，但试了几次都显示 `fatal: unable to access 'https://github.com/RiskeyL/Riskey-Learning-of-Computer-Science.git/': Recv failure: Connection was reset`。

看上去貌似是网络连接不稳定导致的，所以我先用 `ping github.com` 测试一下是否还能连通 GitHub 以及看下网络延迟情况，结果显示都正常。

**<--偏个题-->**

如果之前配置过 Git 代理（就是让 Git 通过一个代理端口去访问外网，而不是用默认的网络直接连接，也就是让 Git 走科学上网那条路），而代理配置或端口失效了，也有可能导致无法访问 GitHub。

排错流程如下：

Step 1. 使用以下命令检查是否存在 Git 代理。若没有输出，则说明没有配过或者配置已经被取消，那就可以排除这方面的原因。如果有输出，则说明之前配过，那么可以继续第二步。

```bash
git config --global --get http.proxy
git config --global --get https.proxy
```

Step 2. 使用以下命令取消 Git 代理配置，再尝试连接。

```bash
git config --global --unset http.proxy
git config --global --unset https.proxy
```

因为我之前没配过，所以这里直接跳过了。

**<--偏题结束-->**

不是网络的原因，难道和我提交的内容有关系？

于是，我用 `git show <commitHash>`看了下 commit 的内容，突然看到两个 PDF 文件（Logseq 会自动将笔记中上传的附件复制到 assets 文件夹中）... 

上网搜了下，的确有很多因为文件过大而导致 push 不成功的案例，虽然显示的报错信息和我不太一样，但可能性很大。

所以，我开始尝试排错：把这俩 PDF 文件从最近一次 commit 中删除，再重新提交、push，果然成功了。

具体步骤如下：

Step 1. 撤回最后一次 commit，并将所有修改恢复到 staged 的状态（也就是 add 之后、commit 之前）。

```bash
git reset --soft HEAD~1
```

**Note：**

    - `--soft`的同级参数有`--mixed`和`--hard`，分别指“将 commit 的内容恢复到 unstage 的状态，工作区的修改还都保留”和“丢弃 commit 的所有内容，即工作区的修改也全部丢弃”的意思。如果只是要从暂存区删除一些不打算包含在这次提交中的文件，用`--soft`最方便。

    - `HEAD`指向你当前所在的 commit（在这个 case 中，就是我们要撤回的 commit）。因此`HEAD~1`在这个命令行中的意思是：让指针后退一步，转而指向上一个 commit，即让文件状态回到最近一次提交之前的状态。`HEAD~1`也可以换成任意 commit 的 ID（hash number）。


Step 2. Unstage 两个 PDF 文件，将它们从暂存区移除。在 VS Code 等编辑器中，可以直接通过 Source Control 的界面按钮完成操作；当然，也可以使用以下命令行：

```bash
git restore --staged 文件名
```

**Note：**

    - 文件名要包含文件后缀。

    - 若文件名本身包含空格，需要用双引号包裹文件名，这样才能把整个文件名当成一个参数传递给 Git。

    - 若涉及两个或两个以上文件，用空格分隔文件名即可。

Step 3. 再次提交暂存区中的其他内容，然后推送到远程仓库。


虽然这次修正成功了，但我不想后续每次都重复“一键 add 全部、单独 unstage PDF 文件”的操作，该怎么办？

答案是：将 assets 文件夹中的 PDF 文件添加至 .gitignore 文件中，让 Git 不再跟踪本地仓库内 PDF 文件的增删变动。

具体步骤如下：

若仓库中已有 .gitignore 文件，打开并增加以下内容，然后**保存**。若没有，则需要先在仓库内的根目录下创建 .gitignore 文件，再进行编辑。保存后，我看到这两个文件在 VS Code 的 Explorer 界面中变成了灰色状态，就知道成功了。

```plaintext
# Ignore the pdf files in the assets folder
assets/*.pdf
```

**Note：**
    
    - `assets/`指“assets 文件夹内”，而`*.pdf`指“文件后缀为 .pdf 的所有文件”，`*.epub`同理。

    - 更多 .gitignore 文件中支持的语法，可参考 W3School 上 gitignore 这节课中的 [Pattern Syntax](https://www.w3schools.com/git/git_ignore.asp?remote=github)。

需要注意的是，gitignore 只能防止新文件（从未被提交过的文件）被 Git 跟踪。

在我这个 case 里，这两个 PDF 文件被我从撤回的 commit 中删除了，相当于它们从未出现在 Git 的 commit 历史中，因此我只需修改 .gitignore 文件就可以让它们之后都不被 Git 跟踪。

而对于之前已经被成功提交过、版本库中已存在的文件，还需在修改 .gitignore 文件后使用以下命令，才能让 Git 停止跟踪文件。

```bash
git rm --cached 文件名
```