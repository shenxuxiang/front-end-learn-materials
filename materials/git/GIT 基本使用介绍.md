## GIT 基本使用技能介绍
在对 git 进行基本介绍之前，我们先看一张图：
![image](https://github.com/shenxuxiang/front-end-learn-materials/blob/master/images/1.png?raw=true)

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

#### 表示依据当前的远程分支新建
> git checkout -b newBranch 

#### 查看本地分支 并显示当前所处的分支
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
> `git fetch` 取回的代码保存在 `origin/<branch-name>` 中，如果要读取取回的内容需要通过 `origin/<branch-name>`:。


#### 代码合并（merge）
> 执行 `git fetch origin master` 取回代码
>
> 合并代码 `git fetch origin/master`



> 一、现在我们要拉取远程 master 最新的代码：
    
    git fetch origin master;
    
    // 也可以是 git merge origin/master
    git rebase origin/master; 

    针对上面的方法，还有更简单的： git pull origin master --rebase
    
    如果你使用的不是 rebase，可以使用 git pull origin master;
        

二、基于远程分支创建一个新的分支
    
    git fetch origin master;
    
    git checkout -b newBranch origin/master

    先同步到远程仓库最新的代码，然后通过 origin/master 就可以拿到最新代码，并创建新分支。
  