# 本地工作区状态



![Git 下文件生命周期图。](https://git-scm.com/book/en/v2/images/lifecycle.png)



- Untracked：一个新创建的文件就是未被跟踪的。
- Unmodified：一个已经在仓库里、但是没有被修改过的文件。
- Modified：已经在仓库里且被修改过还没有add过的文件。
- Staged：git add之后的文件。放到了暂存区中。



# 常见（忘）命令



- 查看工作区状态

```bash
# 查看文件的状态
# 常见状态：
# 1.Changes to be committed: 下面的文件已经被添加到暂存区 （git add后）
# 2.Changes not staged for commit: 文件被修改过，但是还没有放到暂存区。
# 3.
git status
```

```bash
# 简化状态输出信息
# 输出格式：状态 文件名
# 状态包括：M、MM、A、??; 
# M=修改过未add，
# MM有部分修改过的已经add，也有部分未add，
# A=已经add，
# ??=新创建的文件未add
git status -s / git status --short
```



- 查看暂存区（Staged）和没有add的文件之间区别

```bash
# 查看修改过的文件和之前暂存区的区别
# 如果 git add . 之后 执行 git diff 将不会输出任何信息，因为都add了。
git diff
```

```bash
# 查看暂存起来的变化
# 已暂存文件和最后一次提交的差异
git diff --staged / git diff --cached
```



- 提交

```bash
# 结合git add
# 将自动把所有已经跟踪过的文件暂存起来一并提交，（我觉得不好）
git commit -a 
```



- 移除文件

```bash
# 把文件从实际目录下移除，同时也从暂存区里移除
# -f：用于删除之前修改过或已经放到暂存区的文件
git rm -f file
```

```bash
# 把文件从暂存区里移除，保留本地文件
# 也就是这个文件我不跟踪了
# 常用场景：当你忘记添加 .gitignore 文件，不小心把一个很大的日志文件或一堆 .a 这样的编译生成文件添加到暂存区时
git rm --cached file
```



- 移动文件

```bash
# 跟Shell一样，可以用来重命名
# 实际上一条mv命令 = mv file1 file2; git rm file1; git add file2;
git mv file1 file2
```



- 日志输出

```bash
# 查看最近的三条log
git log -3
```

```bash
# 直观的输出多个commit之间的联系
git log --graph
```

> 当项目比较庞大，commit次数很多的时候，需要用各种筛选策略定位需要的日志





## 远程仓库操作

```bash
# 查看远程仓库的简写和URL
git remote -v

$ git remote -v
origin  https://gitee.com/kicc/JavaGuide.git (fetch)
origin  https://gitee.com/kicc/JavaGuide.git (push)
```



- **拉取到本地**

```bash
# 运行 git remote add <shortname> <url> 添加一个新的远程 Git 仓库，同时指定一个方便使用的简写
$ git remote add pb https://github.com/paulboone/ticgit
$ git remote -v
origin	https://github.com/schacon/ticgit (fetch)
origin	https://github.com/schacon/ticgit (push)
pb	https://github.com/paulboone/ticgit (fetch)
pb	https://github.com/paulboone/ticgit (push)

# 现在你可以在命令行中使用字符串 pb 来代替整个 URL。
# 可以执行 git fetch <remote>来拉取。 
git fetch pb
```

> 须注意 `git fetch` 命令只会将数据下载到你的本地仓库——它并不会自动合并或修改你当前的工作。合并需要手动。

```bash
# 如果【当前】分支设置了跟踪远程分支，git pull 命令来自动抓取后合并该远程分支到当前分支。
git pull
```



- **推送到远程**

```bash
# 这个命令很简单：git push <remote> <branch>,把本地分支推送到服务器
git push origin master
```

> 只有当你有所克隆服务器的写入权限，并且之前没有人推送过时，这条命令才能生效。
>
> 如果push时，其他协作者已经push过了。则必须先fetch最新的内容进行合并后才能push。



- **远程仓库的重命名与移除**

```bash
# 你可以运行 git remote rename 来修改一个远程仓库的简写名
$ git remote rename pb paul
$ git remote
origin
paul
```



```bash
# 移除一个远程仓库，这个远程仓库相关的远程跟踪分支以及配置信息都被删除。
$ git remote remove paul
$ git remote
origin
```



```bash
1
```



```bash
1
```



```bash
1
```



```bash
1
```



```bash
1
```

