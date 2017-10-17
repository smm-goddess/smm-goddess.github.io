# git版本管理

## git介绍

git 是一个快速、可扩展的分布式版本控制系统，它具有极为丰富的命令集，对内部系统提供了高级操作和完全访问.git与你熟悉的大部分版本控制系统的差别是很大的。也许你熟悉Subversion、CVS、Perforce、Mercurial 等等，他们使用"增量文件系统" （Delta Storage systems）, 就是说它们存储每次提交(commit)之间的差异。git正好与之相反，它会把你的每次提交的文件的全部内容（snapshot）都会记录下来。 理论上，git 可以保存任何文档，但是最善于保存文本文档，因为它本来就是为解决软件源代码（也是一种文本文档）版本管理问题而开发的，提供了许多有助于文本分析的工具。对于非文本文档，git 只是简单地为其进行备份并实施版本管理。<br>
git库中由三部分组成

- git仓库就是那个.git 目录，其中存放的是我们所提交的文档索引内容
- git 可基于文档索引内容对其所管理的文档进行内容追踪，从而实现文档的版本控制
- git目录位于工作目录内。

工作环境：

1. 用户本地的目录；
2. Index（索引）：将工作目录下所有文件（包含子目录）生成快照，存放到一个临时的存储区域，git 称该区域为索引。
3. 仓库：将索引通过commit命令提交至仓库中，每一次提交都意味着版本在进行一次更新。

## git常用命令集

1. 设置全局账号：<br>
  `git config --global user.name "neal"`<br>
  `git config --global user.email "neal@gmail.com"`<br>
  执行了上面的命令后,会在你的主目录(home directory)建立一个叫 ~/.gitconfig 的文件. 内容一般像下面这样:<br>

  ```
  [user]
  name = neal
  email = neal@gmail.com
  ```

  这样的设置是全局设置,会影响此用户建立的每个项目.

2. 获得一个项目仓库:<br>

  - clone 远程仓库<br>
    `git clone {远程的git控制url} [{clone的项目文件夹}]`<br>
    例如：`git clone http://git.bravesoft.co.jp/reward-halo.git reword`<br>
    clone的项目文件夹可以省略。在默认情况下，git会把"git URL"里最后一级目录名的'.git'的后辍去掉,做为新克隆(clone)项目的目录名<br>
    例如：`git clone http://git.bravesoft.co.jp/reward-halo.git`<br>
    将会创建 reward-halo 项目文件夹。<br>
    另外，如果访问一个git URL需要用法名和密码，可以在git URL前加上用户名，并在它们之间加上@符合以表示分割，然后执行`git clone`命令，git会提示你输入密码。<br>
    例如：`git clone yutao@<http://git.bravesoft.co.jp/reward-halo.git> reword`<br>

  - 新建本地仓库<br>
    cd 到需要创建仓库的目录<br>
    运行：`git init`<br>
    运行成功后会在当前的目录创建 .git的目录，即创建成功。

3. 查看当前仓库的状态：<br>
  `git status`<br>
  _该命令在系统中还是比较重要的，可以方便我们了解项目的文件变动情况，做到很好的控制_

4. 添加到索引区域:<br>
  `git add` 生成的快照被存放到一个临时的存储区域<br>
  `git add .` 添加当前目录所有文件到索引中。<br>
  `git add` 文件名或者目录名

5. 提交索引区代码到仓库：<br>
  `git commit` 可将索引提交至仓库中，这个过程称为提交，每一次提交都意味着版本在进行一次更新<br>
  用法：<br>
  `git commit`<br>
  执行后git会自动调用系统默认的文本编辑器，要求你输入版本更新说明并保存。git不允许版本说明为空提交.请记住，输入简约的版本更新说明是非常有必要的，它可以帮助你快速回忆起对项目的重大改动。<br>
  一次提交带注释说明：<br>
  `git commit -m “version desc”`

6. 忽略项目中的临时文档：<br>
  可以将工作树中你不希望接受git管理的文档信息写到同一目录下的.gitignore文件中<br>
  例如：`echo “not_upload_file_or_dir” > .gitignore`

7. 推送到远程仓库中:<br>
  将远程仓库定义别名 origin :<br>
  `git remote add origin 远程仓库的地址`<br>
  将本地分支提交到远程分支master上 origin即上面所说的远程仓库的别名：<br>
  `git push origin master`<br>
  如果想把本地的某个分支test提交到远程仓库，并作为远程仓库的master分支，或者作为另外一个名叫test的分支，那么可以这么做。<br>
  提交本地test分支作为远程的master分支<br>
  `git push origin test:master`<br>
  提交本地test分支作为远程的test分支<br>
  `git push origin test:test`<br>
  如果想删除远程的分支呢？类似于上面，如果 : 左边的分支为空，那么将删除:右边的远程的分支<br>
  `git push origin :test`

8. 拉服务器更新拉到本地：<br>
  `git pull` 相当于是从远程获取最新版本并merge到本地<br>
  `git fetch origin master` 相当于是从远程获取最新版本到本地，不会自动merge

9. 查看版本历史<br>
  `git log`<br>
  查看该版本的详细信息<br>
  `git log –p` 显示每一次提交与其父节点提交内容之间快照的差异。<br>
  `git log -p master..origin/master`<br>
  `git show git版本号`

10. 对比不同<br>
  `git diff [filename、HEAD、branchname]` 查看当前文件的不同信息。

11. 添加标签<br>
  `git tag –a tagname`<br>
  `git tag –a tagname –m ‘version msg’`

12. 撤销操作<br>
    1. 撤销未提交的修改(git reset)<br>
    `git reset [--hard|soft|mixed|merge|keep] [<commit>或HEAD]`
      * --hard：重设（reset） index和working directory，自从`commit`以来在working directory中的任何改变都被丢弃，并把HEAD指向`commit`<br>
      例如：<br>
      `git reset --hard HEAD` #将当前修改全部回退到上一次commit提交时的状态。<br>
      `git reset --hard HEAD^` #回退到前一个提交的版本<br>
      `git reset --hard HEAD^^` #回退到前两个提交的版本 等同于 git reset --hard HEAD~2<br>
      `git reset --hard HEAD~1` #回退到前一个提交的版本，数字1代表回退的提交次数。<br>
      `git reset --hard HEAD~5` #回退到前面的5个版本<br>
      `git reset --hard commit_id` #回退到指定的版本号状态。
      * --soft：index和working directory中的内容不作任何改变，仅仅把HEAD指向`commit`<br>
      例如：回滚最近一次commit<br>
      `git commit –m “要回退的commit”`<br>
      `git reset --soft HEAD^ edit`<br>
      `git commit -a -c ORIG_HEAD`<br>
      然后使用reset之前那次commit的注释、作者、日期等信息重新提交。注意，当执行`git reset`命令时，git会把老的HEAD拷贝到文件 .git/ORIG_HEAD 中，在命令中可以使用ORIG_HEAD引用这个commit。commit 命令中 -a 参数的意思是告诉git，自动把所有修改的和删除的文件都放进stage area，未被git跟踪的新建的文件不受影响。commit命令中`-c commit` 或者 `-C <commit`意思是拿已经提交的commit对象中的信息（作者，提交者，注释，时间戳等）提交，那么这条commit命令的意思就非常清晰了，把所有更改的文件加入stage area，并使用上次的提交信息重新提交。
      * --mixed：仅reset index，但是不reset working directory。这个模式是默认模式，即当不显示告知git reset模式时，会使用mixed模式。<br>
      例如：回滚add操纵<br>
      `touch a.log`<br>
      `git add a.log`<br>
      `git reset –mixed`
      * --merge和--keep用的不多
    2. 撤销未提交的修改<br>
     `git checkout filename`
    3. 撤销当前目录所有未提交的修改<br>
    `git checkout`

## 分支管理

1. 创建分支<br>
  <br>
  `git branch 分支名`<br>
  `git checkout -b newbranch支名` 创建分支并切换到创建的分支上。

2. 查看分支list或者所在分支<br>
  `git branch` 执行后可以看到，我们现在只有一个分枝叫做'master'，代表的意思是我们正在这个主分枝上工作。

3. 分支之间的切换<br>
  `git checkout 要切换分支名`

4. 分支合并的分支
    1. 合并develop分支到master分支名称<br>
    切换到master分支<br>
    `git checkout master`<br>
    运行合并命令 `git merge develop` 该过程git会自动做合并操作，如果遇到冲突，可以通过查看冲突代码，合并冲突后提交解决冲突。
    2.  合并远程分支到当前分支<br>
    `git pull . topic/branch` <br>
5. 删除本地无用的分支<br>
  `git branch –d 需要删除的分支名称` 如果分枝还没有被合并，那么执行这个命令就会将分枝上所做的工作一并删除，git是不允许你这么干的。<br>
  如果你实在想删除的话，那么使用'-D'参数强行删除吧。<br>
  `git branch –D强制删除的分支名称`

## git开发流程详解

![git流程](/images/git.png)
