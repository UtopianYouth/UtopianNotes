## 一、`Git`基础

#### 1.1 常用命令

```bash
git init	# 初始化 git 仓库
git add file_name	# 添加文件到仓库
git add .	# 添加所有的文件到仓库
git commit -m "注释"	# 提交文件到仓库（一次 commit 可以提交多个文件，m 即 message）
git diff "file_name" # 查看待提交的文件的修改内容，一般在 add 之前使用
git status # 查看当前 git 仓库的状态
git log	# 查看 git 日志
git log --pretty=oneline	# 查看简易版 git 日志
git reset --hard commit_id	# 回退 or 快进到指定版本的 git 仓库，commit_id 的位数写 5 位左右就行了
git reset --hard HEAD^	# 回退到最近一个版本的 git 仓库，HEAD 是指向当前版本的指针
git reset --hard HEAD~number	# 回退到最近第几个版本的 git 仓库
git reflog	# 记录每一次 git commit（对于不小心的版本回退操作，可以通过 git reflog 显示的每一个版本编号进行恢复）
```

#### 1.2 工作区和版本库

<center>
  <img src = "./images/git_structure.png" width = "70%">
</center>


#### 1.3 管理修改

`git commit` 命令只会提交暂存区里面修改的内容到 git 仓库中（也就是 `git add` 之后的文件）。

#### 1.4 撤销修改

```bash
git checkout -- filename  # 撤销工作区修改的文件内容
git reset HEAD filename		# 可以把暂存区的修改撤销掉，重新放回工作区
git reset --hard HEAD			# 如果修改提交到了本地仓库，直接用版本回退的功能即可
```

#### 1.5 删除文件

```bash
git rm file_name 	# 删除仓库中的文件（不要忘记 commit）
```

如果在工作区中误删了文件，且仓库中还存在误删的文件，可以使用如下命令恢复。

```bash
git checkout -- file_name			# 该命令的作用是，使仓库中的版本文件覆盖到工作区中
```

## 二、远程仓库

#### 2.1 添加远程仓库

> - 在 GitHub 中创建一个新的代码仓库
> - 在本地的 git 仓库中，运行如下指令
>
> ```bash
> git remote add origin git@github.com:TsingHeSZU/learn_git.git		# 添加（绑定）远程仓库，origin 为远程仓库在本地的映射名
> 
> git branch -M main		# 给本地仓库当前分支重命名，-M 表示重命名的意思，因为 GitHub 默认主分支名为 main
> 
> git push -u origin main		# 将本地仓库的内容推送到名为 origin 的远程仓库，-u 表示将本地 main 分支和远程 main 分支关联起来，也就是设置了"上游分支"
> ```
>
> <font color = red>注意</font>
>
> 本地仓库访问远程仓库，相当于客户端访问服务器，进行 SSH 连接即可，需要在 GitHub 的远程仓库中，配置客户端提供的 SSH 公钥，实现不用登录连接 GitHub 远程服务器。
>
> - 给绑定的远程仓库重命名
>
> ```bash
> git remote rename <old_name> <new_name>
> ```
>
> - 查看本地仓库与远程仓库的分支关系
>
> ```bash
> git branch -vv
> ```
>
> - 根据远程仓库名删除远程仓库（解除与本地仓库的绑定）
>
> ```bash
> git remote rm origin
> ```

#### 2.2 从远程仓库克隆

> 使用 `git clone ` 命令克隆远程仓库，实现协同开发
>
> ```bash
> git clone git@github.com:TsingHeSZU/git_skills.git
> ```
>
> <font color = red>注意</font>
>
> Git支持多种协议，包括 `https`，但 `ssh` 协议速度最快。

## 三、分支管理

#### 3.1 创建与合并分支

```bash
git checkout -b dev		# 创建 dev 分支，并且切换到 dev 分支上
# 上述命令相当于下面两条命令
git branch dev		# 创建分支到本地仓库
git checkout dev		# 切换本地仓库的工作分支

# 因为 checkout 不直观，建议使用如下命令
git switch -c dev		# 创建 dev 分支，并且切换到 dev 分支上
# 上述命令相当于下面两条命令
git branch dev		# 创建分支到本地仓库
git switch dev		# 切换本地仓库的工作分支

git merge branch_name		# 合并指定分支到当前的工作分支中

git branch -d branch_name		# 删除指定分支
```

<font color = red>总结</font>：因为创建、合并和删除分支非常快，所以 Git 鼓励你使用分支完成某个任务，合并后再删掉分支，这和直接在 `main` 分支上工作效果是一样的，**但过程更安全**。

#### 3.2 解决合并冲突

当我们在两个不同的分支修改了同一份文件（不同内容），并且提交到 git 仓库，并且对这两个不同分支进行合并时，`git merge` 会产生合并冲突。

> **解决冲突的步骤**
>
> - 在当前分支下，手动编辑合并出错的文件（保存为想要的内容），并且保存 
> - 保存之后，使用 `git add` 和 `git commit` 两个命令重新进行提交
> - 用 `git log --graph` 命令可以看到分支合并图

#### 3.3 分支管理策略

在实际开发中，我们应该按照几个基本原则进行分支管理：

> 首先，`master / main`分支应该是非常稳定的，也就是仅用来发布新版本，平时不能在上面干活；
>
> 那在哪干活呢？干活都在`dev`分支上，也就是说，`dev`分支是不稳定的，到某个时候，比如1.0版本发布时，再把`dev`分支合并到`master / main`分支上，在`master / main`分支发布1.0版本；
>
> 你和你的小伙伴们每个人都在`dev`分支上干活，每个人都有自己的分支，时不时地往`dev`分支上合并就可以了。
>
> 所以，团队合作的分支看起来就像这样：
>
> <center>
>   <img src = "./images/branches.png">
> </center>
>
> <font color = green>小结</font>
>
> 合并分支时，加上 `--no-ff` 参数就可以用普通模式合并，合并后的历史有分支，能看出来曾经做过合并，而 `fast forward` 合并就看不出来曾经做过合并。
>
> ```bash
> git merge branch_name --no-ff -m "description"
> ```

#### 3.4 Bug 分支

场景：当前正在 `dev` 分支中开发业务逻辑，但是突然接到一个新的任务，`master / main` 主分支中存在 Bug ，需要我们去解决。

> - 由于 `dev` 分支中的业务逻辑没有完成，需要使用 `git stash` 命令来保存现场
> - 切换到 `master / main` 主分支中，创建一个新的分支 `issue_101`，这个分支用来专门解决 Bug
> - 解决完 Bug 后，`master / main` 主分支合并 `issue_101` 分支中的内容
> - 回到 `dev` 分支，利用 `git stash apply` 或者 `git stash pop` 命令恢复现场
> - 利用 `git cherry-pick` 命令将 `master / main` 主分支中，解决 Bug 的提交应用到 `dev` 分支中
> - 继续 `dev` 的业务逻辑

```bash
git stash		# 保存当前分支暂存区的现场
git stash list		# 查看当前分支的工作现场
git stash apply stash@{num}		# 恢复当前分支编号为 num 的现场
git stash drop stash@{num}		# 删除当前分支编号为 num 的现场
git stash pop stash@{num}		# 恢复当前分支编号为 num 的现场，并且删除

git cherry-pick commit_num		# 将一个分支中的 commit_num 提交应用到当前分支中，注意区分 git merge
```

#### 3.5 Feature 分支

Git 中，开发一个新功能，最好新建一个分支，如果要丢弃一个没有被合并过的分支，可以通过 `git branch -D <name>` 强行删除。

#### 3.6 多人协作

正常的，一般的开发场景中，一般是多人合作，多人合作使用 Git 的工作模式通常如下：

> 1. 首先，可以尝试用`git push origin <branch_name>`推送自己的修改（这里的 branch_name 是远程仓库的）
> 2. 如果推送失败，是因为远程分支的版本比本地分支版本新，需要先用`git pull origin <branch_name>`试图合并
> 3. 如果合并有冲突，则解决冲突（手动编辑冲突的文件），并在本地提交
> 4. 没有冲突或者解决掉冲突后，再用`git push origin <branch-name>`推送就能成功
>
> 这就是多人协作的工作模式，一旦熟悉了，就非常简单。
>
> <font color = green>远程仓库的一些小结</font>
>
> - 查看远程库信息，使用`git remote -v`
> - 本地新建的分支如果不推送到远程，对其他人就是不可见的
> - 从本地推送分支，使用`git push -u origin branch-name`（`-u` 参数自动设置上游分支），如果推送失败，先用`git pull origin branch-name`抓取远程的新提交
> - 在本地创建和远程分支对应的分支，使用`git checkout -b branch_name origin/branch_name` 或者 `git switch -c branch_name origin/branch_name`，本地和远程分支的名称最好一致
> - 建立本地分支和远程分支的关联，使用`git branch --set-upstream branch_name origin/branch_name`；
> - 从远程抓取分支，使用`git pull origin branch_name`，如果有冲突，要先处理冲突

#### 3.7 Rebase

> 暂时好像没咋看懂啊

## 四、标签管理

Git 中，标签可以理解为是某一个 commit 的映射，通过标签，可以很容易的找到版本库中的特定版本。

#### 4.1 创建标签

```bash
git tag 	# 查看本分支的所有标签
git tag <tag_name>	# 默认给最新的 commit 打上标签 <tag_name>
git tag <tag_name> <commit_num>	# 默认给 commit_num 的提交打上标签 <tag_name>
git show <tag_name>	# 查看本分支标签信息
git tag -a <tag_name> -m "info" <commit_num>	# 默认给 commit_num 的提交打上标签 <tag_name>，并且加上注释 info

```

<font color = red>注意：</font>标签总是和某个 commit 挂钩，如果这个 commit 既出现在 `master` 分支，又出现在 `dev` 分支，那么在这两个分支上都可以看到这个标签。

#### 4.2 操作标签

```bash
git tag -d <tag_name>	# 删除本地仓库中，标签名为 tag_name 的指定标签
git push origin <tag_name>	# 将本地标签 tag_name 推送给名为 origin 的远程仓库
git push origin --tags	# 将本地所有标签推送给名为 origin 的远程仓库
git push origin :refs/tags/<tag_name>	# 删除远程仓库 origin 中，名为 tag_name 的标签
```

## 五、使用 GitHub

> - 在GitHub上，可以任意Fork开源仓库
> - 自己拥有Fork后的仓库的读写权限
> - 可以推送pull request给官方仓库来贡献代码
>
> <font color = red>例子：</font>Bootstrap的官方仓库`twbs/bootstrap`、你在GitHub上克隆的仓库`my/bootstrap`，以及你自己克隆到本地电脑的仓库，他们的关系就像下图显示的那样：
>
> <center>
>   <img src = "./images/GitHub.png" width = "60%">
> </center>

## 六、使用 Gitee

国内版本的 GitHub ，其分布式服务器访问较方便且速度快。

> - 一个本地仓库既可以绑定 GitHub 的远程仓库，也可以绑定 Gitee 的远程仓库，只是注意远程仓库的取名即可。

## 七、自定义 Git

#### 7.1 忽略特殊文件

<font color= red>场景：</font>有些时候，必须把某些文件放到 Git 工作目录中，但又不能提交它们，比如保存了数据库密码的配置文件等等，每次 `git status` 都会显示 `Untracked files ... `。

> 好在Git考虑到了大家的感受，这个问题解决起来也很简单，在Git工作区的根目录下创建一个特殊的`.gitignore`文件，然后把要忽略的文件名填进去，Git就会自动忽略这些文件。
>
> <font color = red>注意：</font>GitHub 中，提供了一些指定语言项目的 `.gitignore` 模板，开源链接为：[GitHub 忽略特殊文件模板](https://github.com/github/gitignore)。

#### 7.2 配置别名（修改 git 的配置文件）

对于 Git 中，一些较为复杂的命令，我们可以通过配置别名来实现简写。

> ```bash
> git config --global alias.<new_name> <old_name>
> ```
>
> 对于上述命令，比如 `old_name == commit` 并且 `new_name == ci`，那么，`git ci` 等同于 `git commit`。
>
> <font color = green>配置文件</font>
>
> - 配置Git的时候，加上`--global`是针对当前用户起作用的，如果不加，那只针对当前的仓库起作用
> - 每个仓库的 Git 配置文件都放在 `.git/config` 文件中
> - 当前用户的 Git 配置文件放在用户主目录下的一个隐藏文件`.gitconfig`中

## 八、使用 `SourceTree`

`SourceTree` 为 Git 可视化工具。

## 九、SSH 公钥和私钥的区别

