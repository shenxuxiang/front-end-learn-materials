## GIT 基本使用技能介绍
在对 git 进行基本介绍之前，我们先看一张图：
![image](https://github.com/shenxuxiang/front-end-learn-materials/blob/master/materials/git/images/1.png?raw=true)

关于图中的 `Workspace`，`Index`，`Repository`，`Remote` 分别代表的是：我们的工作区、本地暂存区、本地仓库、以及远程仓库

#### 工作区 (Workspace)
> 本地项目存放文件的位置。可以理解成图上的 workspace

#### 暂存区 (Index/Stage) 
> 顾名思义就是暂时存放文件的地方，是通过 `git add` 命令将工作区的文件添加到暂存区。

#### 本地仓库（Repository）
> 通常情况下，我们使用 `git commit` 命令可以将暂存区的文件添加到本地仓库。

#### 远程仓库（Remote）
> 举个例子，当我们使用 GitHub 托管我们项目时，它就是一个远程仓库。通常我们使用 `git clone` 命令将远程仓库代码拷贝下来，本地代码更新后，通过 `git push` 托送给远程仓库。


## 一、项目初始化

#### 本地 GIT 初始化
> git init

#### 克隆远程代码
> git clone [url]

#### 查看远程主机
> git remote

#### 本地项目与远程仓库进行连接
> git remote add origin [url]

#### 本地项目与远程仓库断开连接
> git remote rm origin

#### 修改远程仓库的地址
> git remote set-url origin [url]



## 仓库设置、查看

#### 查看远程仓库地址详情
> git remote -v

#### 查看配置 （user.name 和 user.emial可以查看）
> git config --list （当前项目）
>
> git config --global --list （全局）

#### 修改配置
> git config [--global] -e
>
> 打出上面的命令行以后，就进入了配置界面(具体操作同 vim)，此时就可以查看当前的项目仓库的信息，同时也可以修改。
>
> `--global` 表示查看和修改全局的。正对所有的 git 命令来说，只要添加了 `--global` 都是正对全局的。

#### 单独修改 user.name 和 user.email
> git config --global user.name 'you name'
>
> git config --global user.email 'you email'
>
> 上面是修改全局配置的 name 和 email
>
> 注意，一些情况下，如果远程仓库的 name、email 与你本地项目不匹配的话，在执行 `git push` 时将抛出异常

#### 使用密码时，密码存储在本地
> git config [--global] credential.helper [store | cache]
>
> store 存放在硬盘中  cache 放在缓存中

#### 使用密钥
> 首先，我们需要确认本地是否已经存在密钥。
>
> 执行 `ls ~/.ssh` 命令查看是否存在 `id_rsa`、`id_rsa.pub` 这两个文件
>
> 如果不存在，我们需要执行 `ssh-keygen` 命令（根据提示，直接回车就行）。完成后就可以通过上面的命令查看。
>
> 我们将 publicKey 复制到 github 上，具体操作路径: setting >> SSH and GPG keys > new SSH key 完成。
>
> 之后我们在第一次进行操作时还是会让我们输入密码，之后就不需要了。


## 二、分支创建和查看

#### 切换分支
> git checkout dev

#### 依据远程某个分支，创建一个新分支
> git checkout -b newBranch [remoteName]/[branchName]
>
> 一般为了确保远程分支 [remoteName]/[branchName] 内容最新，在创建之前应该执行 `git fetch [remoteName] [branchName]`
>
> 或者可以直接拉取所有分支 `git fetch`

#### 表示依据当前的分支新建
> git checkout -b newBranch 

#### 查看本地分支，并显示当前所处的分支
> git branch

#### 查看远程分支，并显示当前本地分支和远程那个分支是映射关系
> git branch -r

#### 查看本地和远程所有分支
> git branch -a

#### 查看哪些分支已经合并到当前分支
> git branch --merged

#### 删除本地分支，但是不能删除当前分支 (-D 表示强制删除，相当于 --delete --force)
> git branch -d <branchname>


## 三、拉取远程仓库代码、合并代码

#### 拉取远程仓库代码
![image](https://github.com/shenxuxiang/front-end-learn-materials/blob/master/materials/git/images/7.png?raw=true)
> git fetch <remote-name>
>
> 也可以取回特定的分支 `git fetch <remote-name> <remote-branch-name>`
>
> 我们也可以基于远程仓库的分支来创建一个新的分支
>
> git fetch origin <remove-branch-name>:<local-branch-name>
>
> 例如，依赖远程 master 分支创建 dev 分支: `git fetch origin master:dev`
> 
> 同 `git checout -b dev  [<remote-name>/<remote-branch-name>]` 是一样的效果
> 
> `git fetch` 取回的代码保存在 `origin/<branch-name>` 中，如果要读取取回的内容需要通过 `origin/<branch-name>`。

#### 代码合并（merge）
> git fetch origin master
>
> git merge origin/master
>
> 还可以使用更简单的方式: `git pull origin master` 可以理解为是上面两行命令的合并；
>
> 如果你想合并本地的两个分支，则可以这样使用: `git merge dev` 表示将合并本地的 dev 分支。

#### 代码合并（rebase）
> git fetch origin master
>
> git rebase origin/master
>
> 还可以使用更简单的方式: `git pull origin master --rebase`
>
> 代码在合并的过程如果有冲突，需要先解决冲突，然后再执行: `git add .; git rebase --comtinue`
>
> 如果不想继续合并，可以执行 `git rebase --abort` 取消合并

#### 区分 merge 和 rebase
> `git merge` 在合并分支时，会产生一条新的合并记录，类似 `Merge branch branchA into branchB` 的一条提交信息。如下图:
![image](https://github.com/shenxuxiang/front-end-learn-materials/blob/master/materials/git/images/3.png?raw=true)
>
> 假设我们现在有2条分支，一个为 `master`，一个为 `dev`，都基于初始的提交 `feat: 添加了materials 文件` 进行检出分支。
>
> 之后，`master` 分支增加了 `a.js`，并提交，`dev` 也增加 `b.js`，并提交。
>
> 此时，我们执行 `git pull origin master` 合并代码，执行 `git log --graph` 查看提交记录。如下图
![image](https://github.com/shenxuxiang/front-end-learn-materials/blob/master/materials/git/images/11.png?raw=true)
>
> 现在我们看看 rebase 的合并记录，如下图
> 
![image](https://github.com/shenxuxiang/front-end-learn-materials/blob/master/materials/git/images/12.png?raw=true)
>
> rebase 翻译为变基，他的作用和 merge 很相似，用于把一个分支的修改合并到当前分支上。并且不会产生新的合并记录。如下图
>
![image](https://github.com/shenxuxiang/front-end-learn-materials/blob/master/materials/git/images/2.jpg?raw=true)


## 四、代码提交

#### 将代码提交到暂存区
> git add [filename] 将指定文件提交到暂存区
>
> git add . 表示将所有的文件都添加到暂存区
>
> 如果你不修改某个文件，则可以执行 `git checkout [filename]` 则可以撤销文件的修改。
>
> 当让，你也可以撤销所有的文件 `git checkout .`


#### 撤销暂存区中的文件
> git reset [filename] 将指定文件从暂存区中撤销
>
> git reset . 将所有文件从暂存区中撤销

#### 将暂存区中的文件提交到本地仓库
> git commit -m [message]
>
> 如果我们希望修改 commit-message 中的消息，那么你可以使用 `git commit --amend`
>
> 此时界面会切换到 vim 界面，然后你就可以对 commit message 进行修改。修改完成后保存并退出。就可以了。
>
> 其实我们可以将 `--amend` 理解为是将上次的和这次的 commit 进行合并，并修改 commit-message。

#### 撤销本地仓库中的提交
> `git reset --soft HEAD^` 表示撤销最近一次的 `commit` 操作，我们一般称之为‘软回退’。
>
> 也可以指定 commit-id: `git reset --soft [commit-id]`，此时 GIT 就会撤销此 commit 之后的所有提交（保留之前的提交）
>
> 注意，‘软回退’是将已经提交从本地仓库的代码撤回到本地的暂存区，
> 
> 也就是说‘软回退’后，你的代码是处于 `git add .` 之后的状态，（软回退命令并不会修改我们的代码，只是修改 GIT 提交状态）
>
> 基于 `git reset --soft HEAD^` 来理解，此时代码处于上一次 commit 之后，再执行 `git add .` 命令后的状态。
>
> 基于 `git reset --soft [commit-id]` 来理解，此时代码处于此 commit 之后，再执行 `git add .` 命令后的状态。



#### 提交到远程仓库
> 如果本地分支与远程分支已经关联了，则可以直接使用 `git push` 提交代码，
>
> 如果没有关联，我们可以执行 `git push -u origin [local-branch]:[remove-branch]`
>
> 如果远程分支名与本地分支名同名，执行 `git push -u origin [local-branch]` 此时的远程分支名可以省略。
>
> 如果不相关联，则可以将 [-u] 去掉就可以了。

#### 如何删除远程分支
> git push origin [blank]:<remote-branch>
>
> git push origin -d [remote-branch]
>
> 以上两种方法都可以将远程的分支进行删除。


## 五、GIT场景回顾

#### stash
> 当你正在开发项目时，这时有一个 `hot-fix` 需要及时修复，这时候你需要怎么做？？？
>
> 此时有人可能会想先 commit，然后切换分支去修改，这是一种方法，但不是最好的。
>
> 最好的方法是使用 `git stash`，然后切换分支去修线上bug，处理完成后再切换回原来分支；
>
> 此时，你再通过 `git statsh pop` 将代码取回到最新的状态，并清楚 stash 列表
>
> `git stash list` 命令可以将当前的 stash 栈信息打印出来，你只需要将找到对应的版本号，
>
> 使用 `git stash apply stash@{1}` 就可以将你指定版本号为 stash@{1} 的工作取出来，
>
> 当你将所有的栈都应用回来的时候执行 `git stash clear` 来将栈清空。


#### blame
> 如果你想知道文件的某个位置是谁修改的，什么时候修改的，那么这个命令可以帮助你；
>
> 比如说，我想知道 index.js 文件的第160-170 行是谁什么时候修改的，可以执行 `git blame -L 160,+10 .index.js`

#### clean
> git clean -df 可以帮助你清除 untracked 文件

#### prune
> 与远程仓库代码进行一次同步，如果本地仓库很久没有与远程仓库同步了。可以执行 `git remote prune origin`
>
> 该命令执行后，如果远程的某些分支已经被删除，那么此时本地与之对应的分支也将会被删除，
>
> 但是，当前正激活的分支不会被删除。


## 六、代码会滚

#### 代码硬回退
> git reset --hard HEAD^
>
> git reset --hard [commit-id]
>
> 上面两种方法，都能使代码回退到之前的版本。前者是回退到最近的一次 commit 位置，后者回退到指定的 commit 位置
>
> 代码硬回退之后，本地代码在这之后提交都将被核销，并且我们在 log 中也无法查看到在这之后提交的 commit-log
>
> 但是，我想说的是 `git reflog` 命令，却能帮我们查看到所有的操作：
>
![image](https://github.com/shenxuxiang/front-end-learn-materials/blob/master/materials/git/images/8.png?raw=true)
>
> 上图是我的提交记录，然后执行 `git reset --hard e5e60adb532774021983dc51eff0651496fdf99a`
> 此时，我们的代码就会滚到了【删除了 setupproxy js文件】这个位置。
> 执行 `git log` 然后你会发现，日志中已经没有了关于 hello 的日志。但是你可以通过 `git reflog` 可以找到

![image](https://github.com/shenxuxiang/front-end-learn-materials/blob/master/materials/git/images/9.png?raw=true)
>
> 通过上面的案例我们可以联想到，在团队协作中，如果都是基于同一个分支进行打包和发布，
>
> 在线上出现问题时，需要将代码即是会滚。如果使用 git reset --hard 来解决问题，很显然不是很切合实际，
>
> 因为，一旦你使用硬回退来会滚代码时，所有在你之后的 commit 都将会撤销。


#### revert
> git revert [commit-id]
>
> `git revert` 与 `git reset --hard` 有什么区别？？？
>
> `git revert` 是用一次新的 commit 来回滚之前指定的 commit，此次提交之前的 commit 都会被保留；
>
> `git reset` 是回到某次提交，在此 `commit-id` 之前的提交都将保留，但是此 `commit-id` 之后的修改都会被删除。

