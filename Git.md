

## 1、安装

### （1）官网下载

> https://git-scm.com/downloads

### （2）Git 使用前配置

配置文件：

/etc/gitcofig：系统通用配置文件，使用 git config --system 命令；

~/.gitconfig：用户配置文件，使用 git config --global 命令；

.git/config：仓库配置文件，使用 git config 命令。

- 配置用户信息（提交人姓名&邮箱）

```
git config --global user.name YCZ
git config --global user.email 2236133845@qq.com
```

> 注：或者在 ~/.gitconfig 配置文件中配置。

- 配置编辑器

```
git config --globla core.editor vim
```

- 查看配置信息

```
git config --list

git config <key>
```

### （3）SSH免密码登录

- 生成SSH密钥对

```
ssh-keygen
```

> 秘钥存储目录：~/.ssh
>
> 私钥文件：id_rsa
>
> 公钥文件：id_rsa.pub

- 将公钥复制到代码管理库

```
cat ~/.ssh/id_rsa.pub
```



## 2、Git 基础

### 2.1 获取Git仓库

- 初始化本地仓库

```
git init
```

- 克隆远程仓库

```
git clone http://*.git
```

### 2.2 记录每次更新到仓库

- 查看当前文件状态

```
git status 

Untracked	未跟踪
Unmodified	未提交
Modified	已提交
Staged		已暂存
Unmerge		未合并（冲突）

# 状态简览
git status -s

?? 未跟踪
A  新添加到暂存区的文件
 M 已修改未暂存
M  已修改已暂存
MM 暂存后有修改

# 查看本地被忽略的文件
git status --ignored

git ls-files --others --ignored --exclude-standard
```

- 跟踪新文件 | 添加到暂存区

```
git add <filelist>
```

- 忽略文件

> .gitignore

- 查看已暂存和未暂存的修改

```
# 查看未暂存的修改
git diff

# 查看已暂存的修改
git diff --staged
```

- 提交更新（暂存区 → 本地仓库）

```
git commit -m msg

# 跳过使用暂存区（将所有已跟踪文件暂存并提交）
git commit -a
```

- 移除文件

```
# 从暂存区中移除并删除工作目录的文件，该文件以后不会被跟踪
# 如果从工作目录直接删除文件，会出现在未暂存清单中
# 删除前修改过并放入暂存区，则必须要用 -f 强制删除选项
git rm <filename>

# 仅从暂存区中国移除，不删除源文件
git rm --cached <filename>
```

- 移动文件（重命名文件）

```
git mv <from> <to>
```

### 2.3 查看提交历史

```
# 全部提交记录，最新的在前
git log

# 最近的5次提交
git log -5

# 显示每次提交的差异
git log -p

# 查看每次提交的简略统计信息
git log --stat

# 参考日志（救命稻草）
# 所有引起 HEAD 指针变动的操作，都会被记录在 git reflog 命令中
# 可以用来恢复本地错误的操作
# 存储在（本地） .git/logs/HEAD 文件中，或者是 .git/logs/refs 目录中的文件
git reflog

注：
① HEAD@{n}：表示HEAD更改历史记录，最新的更改再上面。
如：HEAD@{2}：表示HEAD指针向前移动两次的情况。
② 可以通过 HEAD@{n} 语法回退到指定的提交。
③ HEAD@{n} 与 HEAD~n 功能类似：HEAD@{n}回退的是 git reflog 命令显示的历史提交记录；HEAD~n 回退的是 git log 命令显示的历史提交记录。
```

### 2.4 撤销操作

- 修正上次提交的信息

```
git commit --amend
```

- 取消暂存的文件

```
git reset HEAD <file>

# 放弃本地修改
git reset --hard HEAD

# 回退 git log 的2步，修改内容不丢失
git reset --mixed HEAD~2

注：HEAD@{n} 与 HEAD~n 功能类似：HEAD@{n}回退的是 git reflog 命令显示的历史提交记录；HEAD~n 回退的是 git log 命令显示的历史提交记录。
```

- 撤销对文件的修改（还原成上次提交时的样子）

```
git checkout -- <file>

# 新版
git restore <file>
```

### 2.5 远程仓库的使用

- 设置远程仓库

> 使用 `git clone <url>` 命令克隆仓库时，默认用 origin 作为远程仓库的别名

```
# 显示所有远程仓库（别名 + URL）
git remote -v

# 显示远程仓库的详细信息
git remote show <remote-name>

# 添加远程仓库
git remote add <remote-name> <url>

# 设置远程仓库链接
git remote set-url <remote-name> <url>

# 修改远程仓库别名
git remote rename <old> <new>

# 删除远程仓库
git remote rm <name>
```

- 添加带账户密码的仓库（免密码登录）
```
git remote add origin https://account:password@gitee.com/jfdfsdd/MapDemo.git

# 存储凭证，只输一次密码，后面就不用再输
# 凭证默认存储位置：~/.git-credentials
git config --global credential.helper store --file ~/.my-credentials
```

- 从远程仓库中拉取数据

```
# 拉取远程仓库中最新的版本到本地，用户检查后决定是否合并
git fetch <remote-name>

...

git merge source

# git pull = git fetch + git merge
# 拉取远程仓库中最新的版本（直接合并）
git pull 远程仓库地址或别名 远程分支名:本地分支名

# 本地分支与远程分支同名时
git pull
```

- 推送到远程仓库

```
git push <remote-name> <branch-name>

# -u：记住推送地址及分支，下次推送时只需输入 `git push` 即可
git push -u <remote-name> <branch-name>
```

### 2.6 打标签

- 列出标签

```
git tag
```

### 2.7 Git 别名

```
# 取别名
git config --global alias.last 'log -1 HEAD'

# 获取最后一次提交
git last
```

### 2.8 其它

- 恢复记录（将git仓库中指定的更新记录恢复出来，并覆盖暂存区和工作目录）

```
git reset --hard commitID
```

- 忽略文件修改状态

```
# 忽略
git update-index --assume-unchanged file_name
# 不忽略
git update-index --no-assume-unchanged file_name
```

- 将某些提交添加到其它分支

```
# 在要附加提交的分支上执行
git cherry-pick <commit-ish ...>
```



## 3、Git 分支

> ① 主分支（master）
>
> ② 开发分支
>
> ③ 功能分支

### 3.1 分支的新建与合并

- 新建分支

```
git branch <branch-name>

# 新建分支并切换到新分支
git checkout -b <branch-name>
```

- 切换分支

```
git checkout <branch-name>
```

> 注：切换分支前，确保工作区和暂存区 clean

- 合并分支

```
git merge 来源分支
```

- 删除分支（分支被合并后才允许删除）

```
# 无法删除未合并分支
git branch -d <branch-name>

# -D 强制删除
git branch -D <branch-name>
```

### 3.2 分支管理

```
# 查看分支列表
git branch

# 查看每个分支的最后一次提交
git branch -v

# 查看已合并到当前分支的分支
git branch --merged

# 查看未合并到当前分支的分支
git branch --no-merged
```

### 3.3 远程分支

- 新建远程分支

```
# 1 创建分支并切换到新分支（本地）
git checkout -b <new_branch> <base_branch>

# 2 将本地分支推送到远程
git push <remote-name> <local_branch>:<remote_branch>

# 不指定远程分支名时，默认与本地分支同名
git push <remote-name> <local_branch>
```

- 跟踪分支

> git pull 会查找当前分支所跟踪的远程分支，并拉取数据。

```
# 新建分支并切换到指定远程分支
git checkout -b <branch> <remote-name>/<branch>
# 或
git checkout --track <remote-name>/<branch>

# 修改跟踪分支
git branch -u <remote-name>/<branch>
# 或
git branch --set-upstream-to origin/分支名

# 查看设置的所有跟踪分支
git branch -vv

# 撤销本地分支与远程分支的映射关系
git branch --unset-upstream

# 上游快捷方式
@{master} == origin/master
```

- 删除远程分支

```
# 方式1：
git push <remote-name> --delete <branch>

# 方式2：将本地空分支推送到远程分支（同样会删除远程分支）
git push <remote-name> :<remote_branch>
```

### 3.4 暂时保存分支更改

> 使用场景：分支临时切换

**@ 临时存储改动**

```
git stash
```

**@ 恢复改动**

```
git stash pop
```

### 3.5 变基

- 合并

```
# 将 dev 合并到 master
# 把两个分支的最新快照以及二者最近的共同祖先进行三方合并，
# 将合并结果生成一个新的快照（并提交）。
master> git merge dev
```

- 变基

```
# 将 dev 的修改变基到 master 上
# 提取 dev 中的修改，然后在 master 的基础上再应用一次。
dev> git rebase master
dev> checkout master
# 在 master 分支进行一次快速合并
master> git merge dev
```

- 合并 VS 变基

> 两者的最终结果没有任何区别；
>
> 变基使得提交历史更加整洁。



## 4、Git 服务器





## 5、其它

### （1）初始化项目

- Git 全局设置

```
git config --global user.name "YCZ" 
git config --global user.email "2236133845@qq.com" 
```
- 创建新版本库

```
git clone git@code.aliyun.com:YanChangZheng/ycz.git 
cd ycz 
touch README.md 
git add README.md 
git commit -m add README 
git push -u origin master 
```
- 已存在的文件夹或 Git 仓库

```
cd existing_folder 
git init 
git remote add origin git@code.aliyun.com:YanChangZheng/ycz.git 
git add . 
git commit -m msg
git push -u origin master
```

### （3）解决冲突

> git pull 之后，若产生了冲突，则冲突文件被自动修改：

```
<<<<<<< HEAD

文本1

=====================

文本2

>>>>>>> hash值
```

> 需手动编辑冲突后提交

### （4）GIT 忽略清单

> 只需在项目中添加配置文件：`.gitignore`

### （5）为仓库添加自述文件

> 只需在项目根目录中添加配置文件：`readme.md`

### （6）JetBrains 中配置 Git

- Setting > Version Control > Git

> 选择 Git 的安装路径：D:\Git\cmd\git.exe，测试，能得到版本信息，则配置成功。

- Setting > Tools > Terminal

> 配置 Shell Path："D:\Git\bin\sh.exe" --login -i

- Setting > Version Control > Commit

> 启用提交窗口
>
> - [x] use non-modal commit interface



## 九、Git 内部原理

### 1、`.git` 目录

```
.git
├── branches       # 分支
│   └── ...
├── config         # 仓库配置文件
├── description    # 仅供 GitWeb 程序使用
├── HEAD           # HEAD 指向目前被检出的分支
├── hooks          # 客户端或服务器的钩子脚本
│   ├── *.sample
│   └── ...
├── index          # 保存暂存区信息（尚未创建，git add 后创建）
├── info           # 包含全局性排除文件
│   └── exclude
├── objects        # 存储所有数据内容
│   ├── info
│   └── pack       # 包文件 `git gc`
└── refs           # 存储指向数据（分支、远程仓库和标签等）的提交对象的指针
    ├── heads      # head 引用
    └── tags       # tag 引用
```

注：

- 执行 `git init` 时，Git 会创建一个 `.git` 目录。这个目录包含了所有 Git 存储和操作的东西。
- 仓库配置文件：`.git/config`，用 `git config key value` 命令配置。
- 全局配置文件：`~/.gitconfig`，用 `git config --global key value` 命令配置。
- `.git/info`：包含一个全局性排除文件 `.git/info/exclude` ，放置那些不希望被记录在 `.gitignore` 文件中的忽略模式。



### 2、Git 对象

#### （1）数据对象 blob

> blob 对象仅保存了文件的内容，文件名并没有被保存。

```shell
# ------------------------
# 底层命令：git hash-object
# ------------------------
# 可将任意数据保存于 .git/objects 目录（即对象数据库），并返回指向该数据对象的唯一键（长度为 40 个字符的校验和）
# 由 SHA-1(header + 待存储的数据) 运算而得的校验和

# version 1
$ echo 'version 1' > main.go
$ git hash-object -w main.go
83baae61804e65cc73a7201a7252750c76066a30

# version 2
$ echo 'version 2' > main.go
$ git hash-object -w main.go
1f7a7a472abf3dd9643fd615f6da379c4acb3e3a

# -------------------------------
# 对象数据库记录下了该文件的两个不同版本
# -------------------------------

# 一个文件对应一条内容。
# 校验和的前两个字符用于命名子目录，余下的 38 个字符则用作文件名。
$ tree .git/objects/
.git/objects/
├── 1f
│   └── 7a7a472abf3dd9643fd615f6da379c4acb3e3a
├── 83
│   └── baae61804e65cc73a7201a7252750c76066a30
├── info
└── pack

# -----------------------------------------
# 底层命令：git cat-file -p    从 Git 取回数据
# -----------------------------------------

# 取回 v1
$ git cat-file -p 83baae61804e65cc73a7201a7252750c76066a30
version 1

# 取回 v2
$ git cat-file -p 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a
version 2

# --------------------------------------
# 底层命令：git cat-file -t    查看对象类型
# --------------------------------------
$ git cat-file -t 83baae61804e65cc73a7201a7252750c76066a30
blob

# --------------------------------------
# 底层命令：git cat-file -t    查看对象大小
# --------------------------------------
$ git cat-file -s 83baae61804e65cc73a7201a7252750c76066a30
10
```

#### （2）树对象 tree

> 树对象解决了文件名保存的问题，也允许我们将多个文件组织到一起。

```shell
# master^{tree} 表示 master 分支上最新的提交所指向的树对象
# 做完一下操作才会有这个 tree 结构
$ git cat-file -p master^{tree} 
040000 tree 1906578c622f8547a657319a6151e08421ede398	bak
100644 blob 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a	main.go
100644 blob fa49b077972391ad58037050f2a75f74e3671e92	new.go

# Git 根据某一时刻暂存区（即 index 区）所表示的状态创建并记录一个对应的树对象，如此重复便可依次记录（某个时间段内）一系列的树对象。
# 因此，为创建一个树对象，首先需要创建一个暂存区。

# ---------------------------------------------
# 底层命令：git update-index    为单个文件创建暂存区
# ---------------------------------------------

# 将 v1 暂存
$ git update-index --add --cacheinfo 100644 83baae61804e65cc73a7201a7252750c76066a30 main.go

Options:
    --add        必须，该文件并不在暂存区中
    --cacheinfo  必须，将要添加的文件位于 Git 数据库中，而不是位于当前目录下。
    
Git 文件（即数据对象）的合法模式：
    100644  表明这是一个普通文件
    100755  表示一个可执行文件
    120000  表示一个符号链接

# ---------------------------------------------
# 底层命令：git write-tree    将暂存区内容写入树对象
# ---------------------------------------------
# 如果某个树对象此前并不存在的话，当调用此命令时，它会根据当前暂存区状态自动创建一个新的树对象。

# 创建 v1 的树对象
$ git write-tree
1906578c622f8547a657319a6151e08421ede398

# 查看 v1 树对象类型 & 内容
$ git cat-file -p 1906578c622f8547a657319a6151e08421ede398
tree
$ git cat-file -p 1906578c622f8547a657319a6151e08421ede398
100644 blob 83baae61804e65cc73a7201a7252750c76066a30	main.go  # v1

# 将 version 2 和 new.go 文件暂存
$ echo 'new file' > new.go
$ git update-index --add --cacheinfo 100644 \
1f7a7a472abf3dd9643fd615f6da379c4acb3e3a main.go
$ git update-index --add new.go    # 从当前目录暂存

# 创建 v2 + new 的树对象
$ git write-tree
6142a6bde59e4b546fc8b6752c318f7f175e6830

# 查看 v2 树对象类型 & 内容
$ git cat-file -t 6142a6bde59e4b546fc8b6752c318f7f175e6830
tree
$ git cat-file -p 6142a6bde59e4b546fc8b6752c318f7f175e6830
100644 blob 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a	main.go  # v2
100644 blob fa49b077972391ad58037050f2a75f74e3671e92	new.go   # new

# 将 v1-tree 加入到 v2-tree
# -----------------------------------------------------
# 底层命令：git read-tree --prefix=bak    将树对象读入暂存区
# -----------------------------------------------------
# 将 v1-tree 当作子树读入暂存区
$ git read-tree --prefix=bak 1906578c622f8547a657319a6151e08421ede398
$ git write-tree
1a3b9d425f870644fc663c4714e1a0deeb1b7ee7
$ git cat-file -p 1a3b9d425f870644fc663c4714e1a0deeb1b7ee7
040000 tree 1906578c622f8547a657319a6151e08421ede398	bak      # v1-tree
100644 blob 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a	main.go  # v2
100644 blob fa49b077972391ad58037050f2a75f74e3671e92	new.go   # new
```

#### （3）提交对象 commit

> 如果你做完了以上所有操作，那么现在就有了三个树对象，分别代表我们想要跟踪的不同项目快照。
>
> 然而问题依旧：若想重用这些快照，你必须记住所有三个 SHA-1 哈希值。并且，你也完全不知道是谁保存了这些快照，在什么时刻保存的，以及为什么保存这些快照。而以上这些，正是提交对象（commit object）能为你保存的基本信息。

```shell
# -----------------------------------------
# 底层命令：git commit-tree    创建一个提交对象
# -----------------------------------------

# 创建 v1 的提交对象
# 注意：只能为 tree-object 创建提交对象
$ git commit-tree -m 'version 1' 1906578c622f8547a657319a6151e08421ede398
6b5de9a4044ad1adeddee8ce55803b4da6950f76  # 每次都不一样（携带时间戳）

# 查看提交对象的类型 & 内容
$ git cat-file -t 6b5de9a4044ad1adeddee8ce55803b4da6950f76
commit
$ git cat-file -p 6b5de9a4044ad1adeddee8ce55803b4da6950f76
tree 1906578c622f8547a657319a6151e08421ede398
author root <root@localhost.localdomain> 1678777819 +0800
committer root <root@localhost.localdomain> 1678777819 +0800

version 1

# 创建 v2 & v3 的提交对象（它们分别引用各自的上一个提交作为其父提交对象）
$ git commit-tree -m 'version 2' 6142a6bde59e4b546fc8b6752c318f7f175e6830
0348de75892186425e593cc53c55a9c198b0e546  # 每次都不一样（携带时间戳）
$ git commit-tree -m 'version 3' 1a3b9d425f870644fc663c4714e1a0deeb1b7ee7
3dc6836374bf679f9bf4ccc1b7b890fdc982e6ca  # 每次都不一样（携带时间戳）

# 这三个提交对象分别指向之前创建的三个树对象快照
# 以上即为 `git add` 和 `git commit` 的底层实现

# 可以通过 git log 查看 Git 的提交历史了
$ git log --stat  #（理论上可以）
fatal: bad default revision 'HEAD'
# 需更新引用（超纲）
# 将 master 指向某一系列提交之首
git update-ref refs/heads/master 3dc6836374bf679f9bf4ccc1b7b890fdc982e6ca
```

#### （4）对象存储

> Git 仓库的所有对象都会有个头部信息一并被保存。
>
> 头信息 header = "blob #{content.length}\0"
>
> 如：`blob 10\0version 1`

### 3、Git 引用（分支）

> Git 引用：用一个文件来保存 SHA-1 值，然后给该文件取个简单的名字，用这个名字指针替代原始的 SHA-1 值。

```shell
# 若要创建一个新引用来帮助记忆最新提交所在的位置，从技术上讲我们只需简单地做如下操作：（不建议直接编辑引用文件）
$ echo 3dc6836374bf679f9bf4ccc1b7b890fdc982e6ca > .git/refs/heads/master

# ----------------------------------
# 底层命令：git update-ref    更新引用
# ----------------------------------

# 更新引用 master
git update-ref refs/heads/master 3dc6836374bf679f9bf4ccc1b7b890fdc982e6ca

注：
    这基本就是 Git 分支的本质：一个指向某一系列提交之首的指针或引用。
    运行类似于 `git branch <branch>` 这样的命令时，Git 实际上会运行 update-ref 命令，取得当前所在分支最新提交对应的 SHA-1 值，并将其加入你想要创建的任何新引用中。
```

#### HEAD 引用

> Git 如何知道最新提交的 SHA-1 值呢？ 答案是 HEAD 文件。
>
> `.git/HEAD` 文件：通常是一个符号引用（指向其他引用的指针），指向目前所在的分支。

```shell
# 正常情况下，HEAD 指向 master 引用
# 然而在某些罕见的情况下，HEAD 文件可能会包含一个 git 对象的 SHA-1 值。 
# 当你在检出一个标签、提交或远程分支，让你的仓库变成 “分离 HEAD” 状态时，就会出现这种情况。
$ cat .git/HEAD
ref: refs/heads/master

# 当我们执行 git commit 时，该命令会创建一个提交对象，并用 HEAD 文件中那个引用所指向的 SHA-1 值设置其父提交字段。

# 查看 HEAD 引用的值
$ git symbolic-ref HEAD
refs/heads/master

# 设置 HEAD 引用的值
$ git symbolic-ref HEAD refs/heads/test
```

#### 标签引用

> 标签对象：类似提交对象，主要的区别在于，标签对象通常指向一个提交对象，而不是一个树对象。
>
> 它像是一个永不移动的分支引用——永远指向同一个提交对象。

```shell
# 创建一个轻量标签
$ git update-ref refs/tags/v1.0 cac0cab538b970a37ea1e769cbbde608743bc96d

# 创建一个附注标签
$ git tag -a v1.1 1a410efbd13591db07496601ebc7a059dd55cfe9 -m 'test tag'

```

#### 远程引用

> 如果你添加了一个远程版本库并对其执行过推送操作，Git 会记录下最近一次推送操作时每一个分支所对应的值，并保存在 refs/remotes 目录下。
>
> 远程引用（refs/remotes/）和分支（refs/heads/）之间最主要的区别在于，远程引用是只读的。 
>
> 虽然可以 git checkout 到某个远程引用，但是 Git 并不会将 HEAD 引用指向该远程引用。因此，你永远不能通过 commit 命令来更新远程引用。
>
> Git 将这些远程引用作为记录远程服务器上各分支最后已知位置状态的书签来管理。

### 4、包文件

> 手动执行 git gc 命令让 Git 对对象进行打包。
>
> 当版本库中有太多的松散对象，或者向远程服务器执行推送时，Git 都会自动打包。 

```shell
# 手动打包
$ git gc
Counting objects: 8, done.
Compressing objects: 100% (4/4), done.
Writing objects: 100% (8/8), done.
Total 8 (delta 1), reused 0 (delta 0)

# 打包后
$ tree .git/objects/
.git/objects/
├── 19
│   └── 5a60ad34ac46dfcd53e779aed8e0e33c26946c
├── info
│   └── packs
└── pack
    ├── pack-c39d225178055e0ed0698ccebfb4787807936e95.idx
    └── pack-c39d225178055e0ed0698ccebfb4787807936e95.pack

# --------------------------------------
# 底层命令：git verify-pack 查看已打包的内容
# --------------------------------------
$ git verify-pack -v .git/objects/pack/pack-c39d225178055e0ed0698ccebfb4787807936e95.idx
# 解析：                   |    SHA-1    | type |size|  |  |
cb070c8923e4834bb945764f859246101d17cc03 commit 268 173 12
2819caa241044f7a41666bc2bc1ae72993d4f8f2 commit 31 42 185 1 cb070c8923e4834bb945764f859246101d17cc03
856fa4b281d296680517cfe9056d92bf013e6435 commit 169 114 227
6142a6bde59e4b546fc8b6752c318f7f175e6830 tree   69 75 341
1f7a7a472abf3dd9643fd615f6da379c4acb3e3a blob   10 19 416
fa49b077972391ad58037050f2a75f74e3671e92 blob   9 18 435
1906578c622f8547a657319a6151e08421ede398 tree   35 45 453
83baae61804e65cc73a7201a7252750c76066a30 blob   10 19 498
非 delta：7 个对象
链长 = 1: 1 对象
.git/objects/pack/pack-c39d225178055e0ed0698ccebfb4787807936e95.pack: ok

```

### 5、引用规范



### 6、传输协议

