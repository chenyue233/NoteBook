- 检查配置信息

  `git config --list`

- 在已存在的目录中初始化仓库

  `git init` (在该项目目录下)

- 跟踪指定文件(添加问价到暂存区)

  `git add 文件名`

- 克隆现有仓库

  `git clone [url]`

- 检查当前文件状态

  `git status`

- 忽略文件

  创建一个名为` 的文件

  文件`.gitignore` 的格式规范如下:

  - 所有空行或者以 `＃` 开头的行都会被 Git 忽略
  - 可以使用标准的 glob 模式匹配
  - 匹配模式可以以（`/`）开头防止递归
  - 匹配模式可以以（`/`）结尾指定目录
  - 要忽略指定模式以外的文件或目录，可以在模式前加上惊叹号（`!`）取反

  所谓的 glob 模式是指 shell 所使用的简化了的正则表达式。 星号（`*`）匹配零个或多个任意字符；`[abc]` 匹配任何一个列在方括号中的字符（这个例子要么匹配一个 a，要么匹配一个 b，要么匹配一个 c）；问号（`?`）只匹配一个任意字符；如果在方括号中使用短划线分隔两个字符，表示所有在这两个字符范围内的都可以匹配（比如 `[0-9]` 表示匹配所有 0 到 9 的数字）。 使用两个星号（`*`）表示匹配任意中间目录，比如 `a/**/z` 可以匹配 `a/z` , `a/b/z` 或 `a/b/c/z` 等。

  看一个 .gitignore 文件的例子

  ```
  # no .a files
  *.a
  
  # but do track lib.a, even though you're ignoring .a files above
  !lib.a
  
  # only ignore the TODO file in the current directory, not subdir/TODO
  /TODO
  
  # ignore all files in the build/ directory
  build/
  
  # ignore doc/notes.txt, but not doc/server/arch.txt
  doc/*.txt
  
  # ignore all .pdf files in the doc/ directory
  doc/**/*.pdf
  ```



- 查看已暂存和未暂存的修改

  `git diff`  比较工作目录中当前文件和暂存区域快照之间的差异，也就是修改之后还没有暂存起来的变化内容

  `git diff --staged` 对比已暂存文件与最后一次提交的文件差异

  `git diff HEAD` 显示工作版本和HEAD的差别

  `git diff topic master` or `git diff topic..master` 将两个分支上的最新提交做diff

  `git diff topic...master`   输出自topic和master分别开发以来，master分支上的changed

  `git diff test`  显示当前目录和另一个叫'test'分支的差别

  `  git diff HEAD -- ./lib` 显示当前目录下的lib目录和上次提交之间的差别（更准确的说是在当前分支下）

  ` git diff HEAD^ HEAD`  比较上次提交commit和上上次提交

  `git diff SHA1 SHA2` 比较两个历史版本之间的差异

- 提交更新

  `git commit` 这种方式会启动文本编辑器以便输入本次提交的说明

  `git commit -m'提交备注'`  可以在 `commit` 命令后添加 `-m` 选项，将提交信息与命令放在同一行

-  跳过使用暂存区域

  `git commit -a -m''`  Git 就会自动把所有已经跟踪过的文件暂存起来一并提交

- 移除文件

  `git rm 文件名`  删除文件并且git不跟踪

  `git rm --cached` git不跟踪 文件保留在本地磁盘

- 查看git日志

  `git log`   按时间先后顺序列出所有的提交，最近的更新排在最上面

  ` git log -p -2`  显示最近两次每次提交引入的差异

- 撤销操作

  `git commit --amend`  提交完了才发现漏掉了文件没有添加，或者提交信息写错了,可以使用此命令尝试重新提交

  例：提交后发现忘记了暂存某些需要的修改，可以像下面这样操作

  ```
  $ git commit -m 'initial commit'
  $ git add forgotten_file
  $ git commit --amend
  ```

- 取消暂存的文件

  `git reset HEAD 文件名`

- 撤销对文件的修改

  `git checkout --文件名`



- 查看远程仓库

  `git remote`    列出你指定的每一个远程服务器的简写。 如果你已经克隆了自己的仓库，那么至少应该能看到 origin

  `git remote -v`  显示需要读写远程仓库使用的 Git 保存的简写与其对应的 URL

- 添加远程仓库

  `git remote add <shortname> <url>`  添加一个新的远程 Git 仓库，同时指定一个你可以轻松引用的简写

- 从远程仓库中抓取与拉取

  ` git fetch [remote-name]` 访问远程仓库，从中拉取所有你还没有的数据  执行完成后，你将会拥有那个远程仓库中所有分支的引用，可以随时合并或查看。

  `git clone` 自动将其添加为远程仓库并默认以 “origin” 为简写

- 推送到远程仓库

  ` git push [remote-name] [branch-name]`  

- 查看某个远程仓库

  ``git remote show [remote-name]`` 查看某一个远程仓库的更多信息

- 远程仓库的移除已重命名

  `git remote rename`    例：将 `pb` 重命名为 `paul`  :  git remote rename pb paul

- 打标签

  `git tag` 以字母顺序列出标

  `git tag -l 'v1.8.5*'`   只对1.8.5 系列感兴趣，可以运行

  `git tag -a tagname -m "taginfo"`  创建附注标签：附注标签是存储在 Git 数据库中的一个完整对象。 它们是可以被校验的；其中包含打标签者的名字、电子邮件地址、日期时间；还有一个标签信息

  `git show tagname` 可以看到标签信息与对应的提交信息

- 后期打标签

  ```
  git log --pretty=oneline
  15027957951b64cf874c3557a0f3547bd83b3ff6 Merge branch 'experiment'
  a6b4c97498bd301d84096da251c98a07c7723e65 beginning write support
  0d52aaab4479697da7686c15f77a3d64d9165190 one more thing
  6d52a271eda8725415634dd79daabbc4d9b6008e Merge branch 'experiment'
  0b7434d86859cc7b8c3d5e1dddfed66ff742fcbc added a commit function
  4682c3261057305bdd616e23b64b0857d832627b added a todo file
  166ae0c4d3f420721acbb115cc33848dfcc2121a started write support
  9fceb02d0ae598e95dc970b74767f19372d61af8 updated rakefile
  964f16d36dfccde844893cac5b347e7b3d44abbc commit the todo
  8a5cbc430f1a9c3d00faaeffd07798508422908a updated readme
  ```

  `git tag -a v1.2 9fcebo2` 在以上的提交记录v1.2中补上标签

- 共享标签

  `git push origin [tagname]`  push命令并不会传送标签到远程仓库服务器上。 在创建完标签后你必须显式地推送标签到共享服务器上

  `git push prigin --tags` 把所有不在远程仓库服务器上的标签全部传送到那里

- 删除标签

  `git tag -d tagname`   删除掉你本地仓库上的标签

  `git push <remote> :refs/tags/<tagname>`   上述命令并不会从任何远程仓库中移除这个标签,必须使用来更新你的远程仓库

- 检出标签

  ``git checkout tagname`     查看某个标签所指向的文件版本,虽然说这会使你的仓库处于“分离头指针（detacthed HEAD）”状态

  在“分离头指针”状态下，如果你做了某些更改然后提交它们，标签不会发生变化，但你的新提交将不属于任何分支，并且将无法访问，除非确切的提交哈希。因此，如果你需要进行更改——比如说你正在修复旧版本的错误——这通常需要创建一个新分支：

  ```
  git checkout -b version2 v2.0.0
  Switched to a new branch 'version2'
  ```

  当然，如果在这之后又进行了一次提交，`version2` 分支会因为这个改动向前移动，`version2` 分支就会和 `v2.0.0` 标签稍微有些不同，这时就应该当心了

- 分支创建

  `git branch` 分支列表，分支名前的*代表`HEAD` 指针所指向的分支

  `git barnch -v `  查看每个分支的最后一次提交

  `git branch --merged` 过滤列表中已经合并到分支的分支，列表中分支名字前没有 `*` 号的分支通常可以使用 git branch -d删除掉

  `git barnch --no-merged` 包含未合并工作的分支

  `git branch name` 创建新的分支

  `git branch -av` 显示所有分支	

  `git checkout name` 分支切换

  `git checkout -b name` 创建一个分支并切换到那个分支上

  `git merge name`  合并分支，先切换到master在merge，将别的分支合并到master

  `git branch -d/-D name` 清除不需要的分支

- 变基

  `git rebae name` 提交到某一分支上的所有修改都移至另一分支上








