---
layout: post
title:  "学习用 Git 管理本地文件并托管到 GitHub"
date:   2025-06-15 00:25:00 +0800
categories: git
---

## 理论学习

用 Git 对一个本地的文本文件（比如 Markdown 文件）进行版本控制，并且将这个文本文件托管到 GitHub 远程仓库，这两件事儿分别意味着啥？

### 用 Git 对文本文件进行版本控制

这意味着，这个文本文件的每次内容变化（增、删、改，细到每一个字符）都将被 Git 完整地记录下来，就像游戏里的存档点。让你可以：

1. 不需要大记忆恢复术，就能清楚地知道自己“什么时候改了什么”。

2. 随时回到任何历史版本，有吃不完的后悔药。

3. 直观地对比任何两个版本之间的差异，删了个逗号都看得出来。 

每次修改后，你还可以写下修改的背景（改了啥？为什么改？）。后续只要扫一眼，就能快速了解每次内容变更的概要。

#### 三个小屋排排坐

你的每一次修改，都会依次经过 Git 系统里的三个小屋：工作区（Working Directory），暂存区（Staging Area），版本库（Repository）。

工作区到暂存区的传送带是 `git add`，而暂存区到版本库的传送带是 `git commit`（也就是提交）。 每一次成功提交，都相当于给当时的文件状态拍个照片，然后在版本库中形成一个新的版本。

举个栗子，假设你有两个被 Git 管理的 Markdown 文件。

有一天，你偶然被逗号星人植入了一粒逗号子，于是你开始对这两个文件里的句号感觉很不爽，并决定要把所有句号都改成逗号。然后，你撸起袖子说干就干，删除句号，打上逗号，删除句号，打上逗号...

此时，你进行的这些修改都在工作区内。Git 正闭着眼睛，完全不知道你在做什么。

两个文件的句号都被消灭后，你通过 `git add` 命令将它们送进暂存区，为正式提交到版本库做准备。这个动作相当于预告给 Git，你在这次提交中都想改啥东西。Git 这时睁开了左眼。

那有同学可能想问，为什么要在中间加个暂存区，不能在工作区修改完直接提交到版本库吗？

根据鄙人的实战经验，主要有三个使用场景，还是以栗子的形式描述：

1. 在改完其中一个文件的句号后，你突然灵机一动，想把某个逗号改成问号。但你有版本洁癖——也可能是脑袋中的那粒逗号子在捣乱——不想把这个【逗号->问号】的修改与我们神圣的【句号->逗号】的修改混在一个版本里（也就是通过一次 commit 提交）。所以，你决定先把句号都已改掉的文件 add 进暂存区，再将那个无辜的逗号改成问号。如此，同一个文件内的两种修改就可以分成两次提交、形成两个版本。

2. 因为这次消灭句号大作战总共涉及两个文件，所以你可以将先改好的文件放入暂存区，再开始改下个文件。两个文件都放入暂存区后，你可以一次打包提交。这样，单个 commit 中就同时记录了两个文件的修改。

3. （个人觉得是最重要的一个用处）在正式将修改提交至版本库之前，你可以在暂存区查看文件修改前后的 Diff，对要提交的修改进行最终检查，确认没有混入什么奇怪的东西（？

接下来，你通过 `git commit`命令将暂存区内的修改提交到版本库，Git 两只眼睛就都睁开了。于是，一个新版本、一个新的存档点、一个新的时光机站点就此诞生，完整地保存提交当时的文件状态。

简而言之，你的修改和这三个小屋的关系相当于：你是一个手艺人，在自己的工作台（工作区）上进行创作，然后在打包台（暂存台）把要卖出的作品放进箱子里，最后把箱子拿到菜鸟驿站（版本库）寄走。

#### 一些相关概念

**仓库（Repository）**

被 Git 管理的文件所在的文件夹就是一个仓库。不仅是文件，仓库内还包含文件的所有版本记录（在一个名为`.git` 的隐藏文件中）。本地仓库在你自己的电脑上，而当你把文件托管到 GitHub 这类平台上之后，就会拥有一个远程仓库，相当于本地仓库的云端备份，二者之间可以互相同步。

**分支（Branch）**

你现在是霍格沃茨的一名学生，正在上魔药课，对照着课本按部就班地往面前的坩埚中加各种调味料，这是你的主线分支（main）。突然，你又灵机一动，想把课本上没写的东西加进去看看会发生什么。于是，你用咒语复制出了一个一模一样的坩埚（此时坩埚 2 = 坩埚 1，注意顺序），然后你开始自由发挥，这就是一个新分支。如果实验很成功，你可以把分支合入主线，也就是在坩埚 1 中同样加入坩埚 2 的东西（此时坩埚 1 = 坩埚 2）。如果实验不咋成功（最好别把教室炸了），那就当什么都没发生过，你还是可以老老实实地用原本的坩埚继续上课。分支，就是一个个平行宇宙（在现实中不清楚怎样做会有什么结果时，时常会想，如果真的能创建人生分支就好了（等等，被控制的代码们会不会也这样想？

### 将文本文件托管到 GitHub 远程仓库

这意味着，你把本地的 Git 仓库又拷贝了一份到云端。无论何时何地，只要有网络连接，你就能访问到这个文本文件的最近一次提交并推送到远端的版本。你玩的游戏从单机版变成了联网版。

每次在本地仓库提交完后，你可以将提交的内容推送（`git push`）到远程仓库——也就是让远程仓库与本地仓库保持一致；若本地文件不幸损毁或丢失，你也可以将远程仓库的最新内容反向拉取（`git pull`）到本地仓库。换了个新电脑？那更简单：把远程仓库克隆（`git clone`）到本地就成。就算你的电脑被逗号星人劫走，只要所有版本存档还在远程仓库里，就能分分钟恢复。

我一开始总有点分不清 Push（本地 -> 远端） 和 Pull（远端 -> 本地） 的同步方向（只有我是这样吗 QAQ），后来找到了一个具象的场景帮助记忆：当你在本地仓库进行各种修改的时候，你自己的坐标就在本地（好像是一句废话），而远端就在很远的云端（...）。所以当你要把本地的修改同步到远端时，实际上是在把手头的东西给推出去；而把远端的东西同步到本地时，是把外面的东西给拉进来。

## 动手操作（以 Windows 为例）

### 创建本地 Git 仓库

1. [下载](https://git-scm.com/downloads)并安装 Git。安装完成后，在终端中输入`git -v`或 `git --version`，若回车后显示 Git 版本，则说明安装成功。

2. 通过以下任一方式，在终端中进入目标文件夹（where 存放着你想要用 Git 进行版本管理的文件）。

    - 右键点击文件夹，选择**在终端中打开**。
    - 打开终端，输入`cd 目标文件夹的完整路径`。示例：`cd C:\Users\7a8y`

3. 在终端中输入`git init`。一个`.git` 文件悄么声地出现在了你的目标文件夹内，而后者就此变成了一个 Git 仓库。

4. 配置你的用户名和邮箱——告诉 Git 你是谁，便于 Git 记录每次提交的信息：谁在什么时间改了啥？

    - 配置用户名：`git config --global user.name "用户名"`。示例：`git config user.name "Riskey"`
    - 配置邮箱：`git config --global user.email "邮箱"`。示例：`git config user.email "7a8y@163.com"`

    **Note**：

        - `--global`指该设置对当前操作系统用户（也就是你）的所有 Git 仓库都生效，也可以换成`--local`和`--system`，分别是对当前 Git 仓库生效和对这台计算机上所有用户及所有仓库都生效。

        - 输错了没关系，重新输入一遍带正确信息的命令即可。

5. 在终端中输入`git add 文件名`将目标文件加入暂存区。

    **Note**：
    
        - 添加多个文件时，只需用空格分隔文件名即可；也可以输入`git add .`，一次性添加文件夹内的所有文件。
        
        - 如果只想添加特定类型的文件，可以输入`git add *.文件后缀`。示例：`git add *.md`

6.在终端中输入`git commit -m "注释"`，创建版本库内的第一个版本~

到这儿，你就完全可以用 Git 管理你的目标文件了。如果你不习惯用命令行进行 git 操作，可以使用 [Visual Studio Code](https://code.visualstudio.com/) 的内置图形化界面。

### 连接 GitHub 远程仓库

1. 注册 GitHub 账号并创建一个新仓库。

2. 添加 SSH Key 并关联到 GitHub。GitHub 支持通过 HTTPS 和 SSH 两种方式建立远程仓库与本地仓库的连接，但更推荐 SSH，因为只需配置一次，后续无需每次连接都认证。

    1. 在终端中输入`ls ~/.ssh`，检查是否已经存在 SSH Key。若没有，继续进行第二步；若有，则无需重复创建，直接跳到第四步。

    2. 输入`ssh-keygen -t ed25519 -C "邮箱"`，创建 SSH Key（包含公钥和私钥）。如果提示不支持 ed25519，则改输入`ssh-keygen -t rsa -b 4096 -C "邮箱"`。然后，连按两次回车就行。这一步会创建一对公钥和私钥文件（默认在 ~/.ssh 里），公钥用于上传给 GitHub，私钥留存在本机用于证明你的身份。邮箱只是作为备注方便区分，可以写注册 GitHub 使用的邮箱。

    3. 打开 Git Bash（安装 Git 时已同步安装），依次输入`eval "$(ssh-agent -s)"`和`ssh-add ~/.ssh/id_ed25519`（rsa 版本：`ssh-add ~/.ssh/id_rsa`），启动 ssh-agent 并添加私钥。ssh-agent 相当于一个密码保险箱，通过把秘钥添加进 ssh-agent，之后和 GitHub 每次通信时就无需输入私钥密码啦。

    4. 在终端中输入`cat ~/.ssh/id_ed25519.pub`（rsa 版本：`cat ~/.ssh/id_rsa.pub`），选中全部内容复制到剪切板。

    5. 在 GitHub 中，点击**右上角个人头像** > **设置** > **SSH 和 GPG 秘钥** > **添加 SSH 密钥**。把公钥内容复制到**密钥**字段中，标题可以随便写，能分辨这把钥匙来自哪台设备就行，最后点**添加**。这一步相当于把你的公钥登记到 GitHub，这样以后你推送、拉取文件时，GitHub 才能识别你的本地身份，确认你有权限对该远程仓库进行操作。

    6. 输入`ssh -T git@github.com`，测试一下能否成功连接。第一次会提示“Are you sure you want to continue connecting (yes/no)?”，输入 yes。若连接正常，会显示 “Hi 用户名! You've successfully authenticated, but GitHub does not provide shell access.”

3. 进入刚创建的 GitHub 仓库，点击绿色**代码**按钮，复制仓库的 SSH 地址。

4. 在终端中回到你的目标文件夹，输入`git remote add origin SSH 地址`，建立本地仓库与远端仓库的连接。这一步相当于给本地仓库绑定一个名为 origin 的远程仓库，地址是 “SSH 地址”。

5. 输入 `git push -u origin main`，将本地仓库的 main 分支上的最新内容（也就是你之前提交的）推送到远程仓库的 main 分支。这里` -u `的作用是绑定本地 main 和远程 main，之后你只需输入 `git push` 或 `git pull`，Git 就会自动把本地的 main 分支推送到/拉取自远端的 main 分支（相当于网购时的默认地址），不然每次都得重复指定远程仓库和分支名。

好了，现在刷新一下你的 GitHub 仓库页面，本地仓库的内容应该已经全部都在里面了。之后你在本地仓库每次提交后，只要再多输入一次`git push`，就能把本地的修改同步至 GitHub 远端仓库啦！

这是我的第一篇博文，主要是为了记录我自己的学习和实践过程，希望对其他人也能有一点点帮助:p
