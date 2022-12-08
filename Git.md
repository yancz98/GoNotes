[toc]

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











## 9、应用

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

### （2）关于 Laravel 项目

从远程服务器克隆laravel项目到本地后的注意事项

> ①执行 composer install 安装 vendor。
>
> ②复制 .env 配置文件到根目录下。

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
