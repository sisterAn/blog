### 一、必备知识点


![git2](https://user-images.githubusercontent.com/19721451/56352098-1d71d700-6201-11e9-9c1b-2d1242749a49.jpeg)

![git1](https://user-images.githubusercontent.com/19721451/56352112-2498e500-6201-11e9-84f9-cae3f7f27632.png)

#### 1. 仓库

1.  **Remote:** 远程主仓库；
2.  **Repository/History：** 本地仓库；
3.  **Stage/Index：** Git追踪树,暂存区；
4.  **workspace：** 本地工作区（即你编辑器的代码）

#### 2. 注意：

- **git checkout**

  ```js
  // 丢弃工作区的修改，就是让这个文件回到最近一次git commit或git add时的状态。
  git checkout -- <文件名>
  ```

- 一般操作流程：**工作区** -> `git status`查看状态 -> `git add .`将所有修改加入**暂存区**-> `git commit -m "提交描述"`将代码提交到 **本地仓库** ->`git push`将本地仓库代码更新到 **远程仓库**

- 场景1：当你改乱了工作区某个文件的内容，想直接丢弃工作区的修改时，用命令`git checkout -- file`。

  场景2：当你不但改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改，分两步，第一步用命令`git reset HEAD file`，就回到了场景1，第二步按场景1操作。



### 二、回退到上一个commit版本

#### 1. git reset

删除指定的commit

```js
// 修改版本库，修改暂存区，修改工作区

git reset HEAD <文件名> // 把暂存区的修改撤销掉（unstage），重新放回工作区。
// git版本回退，回退到特定的commit_id版本，可以通过git log查看提交历史，以便确定要回退到哪个版本(commit 之后的即为ID);
git reset --hard commit_id 
//将版本库回退1个版本，不仅仅是将本地版本库的头指针全部重置到指定版本，也会重置暂存区，并且会将工作区代码也回退到这个版本
git reset --hard HEAD~1

// 修改版本库，保留暂存区，保留工作区
// 将版本库软回退1个版本，软回退表示将本地版本库的头指针全部重置到指定版本，且将这次提交之后的所有变更都移动到暂存区。
git reset --soft HEAD~1
```

#### 2. git revert

撤销 某次操作，此次操作之前和之后的commit和history都会保留，并且把这次撤销

作为一次最新的提交

```js
// 撤销前一次 commit
git revert HEAD
// 撤销前前一次 commit
git revert HEAD^
// (比如：fa042ce57ebbe5bb9c8db709f719cec2c58ee7ff）撤销指定的版本，撤销也会作为一次提交进行保存。
git revert commit
```

`git revert`是提交一个新的版本，将需要`revert`的版本的内容再反向修改回去，
版本会递增，不影响之前提交的内容

### 3. `git revert` 和 `git reset` 的区别

- `git revert`是用一次新的commit来回滚之前的commit，`git reset`是直接删除指定的commit。 
- 在回滚这一操作上看，效果差不多。但是在日后继续merge以前的老版本时有区别。因为`git revert`是用一次逆向的commit“中和”之前的提交，因此日后合并老的branch时，导致这部分改变不会再次出现，但是`git reset`是之间把某些commit在某个branch上删除，因而和老的branch再次merge时，这些被回滚的commit应该还会被引入。 
- `git reset` 是把HEAD向后移动了一下，而`git revert`是HEAD继续前进，只是新的commit的内容和要revert的内容正好相反，能够抵消要被revert的内容。



### 三、常用命令

### 1. 初始开发 git 操作流程

- 克隆最新主分支项目代码 `git clone 地址` 
- 创建本地分支 `git branch 分支名` 
- 查看本地分支 `git branch` 
- 查看远程分支 `git branch -a`
- 切换分支  `git checkout 分支名 ` (一般修改未提交则无法切换，大小写问题经常会有，可强制切换  `git checkout 分支名 -f`  非必须慎用)
- 将本地分支推送到远程分支 `git push <远程仓库> <本地分支>:<远程分支>` 

#### 2. git fetch

将某个远程主机的更新，全部/分支 取回本地（此时之更新了Repository）它取回的代码对你本地的开发代码没有影响，如需彻底更新需合并或使用`git pull`

#### 3. git pull

拉取远程主机某分支的更新，再与本地的指定分支合并（相当与fetch加上了合并分支功能的操作）

#### 4. git push

将本地分支的更新，推送到远程主机，其命令格式与`git pull`相似

#### 5. 分支操作

1. 使用 Git 下载指定分支命令为：`git clone -b 分支名仓库地址`
2. 创建本地分支：`git branch test`:(创建名为test的本地分支)
3. **切换分支**：`git checkout test`:(切换到test分支)
4. 创建并切换分支：`git checkout -b test`:(相当于以上两条命令的合并)
5. 查看本地分支：`git branch`
6. 查看远程仓库所有分支：`git branch -a`
7. 删除本地分支：`git branch -d test`:(删除本地test分支)
8. **分支合并**：`git merge master`:(将master分支合并到当前分支)
9. 本地分支重命名： `git branch -m oldName newName`
10. 远程分支重命名:
   1. 重命名远程分支对应的本地分支：`git branch -m oldName newName`;
   2. 删除远程分支：`git push --delete origin oldName`;
   3. 上传新命名的本地分支：`git push origin newName`;
   4. 把修改后的本地分支与远程分支关联：`git branch --set-upstream-to origin/newName`



### 四、SSH

#### 1. 查看是否生成了 SSH 公钥

```
$ cd ~/.ssh
$ ls
id_rsa      id_rsa.pub      known_hosts
```

其中 id_rsa 是私钥，id_rsa.pub 是公钥。

#### 2. 如果没有那就开始生成，设置全局的user.name与user.email

```js
git config --list // 查看是否设置了user.name与user.email，没有的话，去设置
// 设置全局的user.name与user.email
git config --global user.name "XX"
git config --global user.email "XX"
```

#### 3. 输入 ssh-keygen 即可（或`ssh-keygen -t rsa -C "email"`）

```
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/schacon/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /Users/schacon/.ssh/id_rsa.
Your public key has been saved in /Users/schacon/.ssh/id_rsa.pub.
The key fingerprint is:
```

#### 4. 生成之后获取公钥内容，输入 cat ~/.ssh/id_rsa.pub 即可， 复制 ssh-rsa 一直到 .local这一整段内容

```
$ cat ~/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAklOUpkDHrfHY17SbrmTIpNLTGK9Tjom/BWDSU
GPl+nafzlHDTYW7hdI4yZ5ew18JH4JW9jbhUFrviQzM7xlELEVf4h9lFX5QVkbPppSwg0cda3
Pbv7kOdJ/MTyBlWXFCR+HAo3FXRitBqxiX1nKhXpHAZsMciLq8V6RjsNAQwdsdMFvSlVK/7XA
t3FaoJoAsncM1Q9x5+3V0Ww68/eIFmb1zuUFljQJKprrX88XypNDvjYNby6vw/Pb0rwert/En
mZ+AW4OZPnTPI89ZPmVMLuayrD2cE86Z/il8b+gw3r3+1nKatmIkjn2so1d01QraTlMqVSsbx
NrRFi9wrf+M7Q== schacon@agadorlaptop.local
```

#### 5. 打开 GitLab 或者 GitHub，点击头像，找到设置页

#### 6. 左侧找到 SSH keys 按钮并点击，输入刚刚复制的公钥即可



### 五、暂存 

`git stash` 可用来暂存当前正在进行的工作，比如想 pull 最新代码又不想 commit ， 或者另为了修改一个紧急的 bug ，先 stash，使返回到自己上一个 commit,，改完 bug 之后再 stash pop , 继续原来的工作；

- 添加缓存栈： `git stash` ;
- 查看缓存栈： `git stash list` ;
- 推出缓存栈： `git stash pop` ;
- 取出特定缓存内容： `git stash apply stash@{1}` ;