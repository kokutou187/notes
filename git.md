---
xpora-copy-images-to: upload
---

> - 合并前保证工作目录是干净的

#### 二、Git 基础

`git config`, --list --show-origin 查看所有的配置以及所在文件，优先级：.git/config > ~/.gitconfig 或 ～/.config/git/config > /etc/gitconfig
                        --global  user.name  "kokutou" 每一个git提交都会使用这些信息
                        --global  user.email   "kokutou@xxx.com"
                        --global  core.editor  vim/emacs  设置全局的文本编辑器，默认是根据shell环境变量EDITOR决定的
                        --list  列出 git 当时能找到的所有配置
                        \<key\>  显示git的某一配置, 如 git config user.name
                        --unset  \<key\>   删除某一项配置 
                        --global alias.lgol  'log --oneline' 设置别名
                        --global user.signingkey  xxxx  设置签署的私钥

`git add` , 这是个多功能命令：可以用它开始跟踪新文件，或者把已跟踪的文件放到暂存区，还能用于合并时把有冲突的文件标记为已解决状态等。 将这个命令理解为“精确地将内容添加到下一次提交中”

`git status`   -s/--short: 简洁输出，??表示新添加未被跟踪的文件，左栏表示暂存区状态，右栏表示工作区状态

`.gitignore` 支持glob模式，匹配模式以 (/) 开头防止递归，匹配模式可以以（`/`）结尾指定目录。在模式前加上叹号（`!`）取反。/TODO只忽略当前目录下的 TODO 文件，而不忽略 subdir/TODO。build/ 忽略任何目录下名为 build 的文件夹。每个子目录都可以有自己的.gitignore文件

`git diff` 比较工作目录中当前文件和暂存区域快照之间的差异。
					--staged/--cached. 比较暂存文件与最后一次提交的文件差异
                   使用 git config diff.tool vimdiff设置图形化工具后，用git difftool图形化显示区别
                   --check 显示是否有空白错误 whitespace error

`git commit` -m 直接补全提交时候的信息
                       -a   将所有已经跟踪过的文件暂存并提交，未被跟踪的文件不会提交
                       --amend  将本次提交归入到上次提交。本质使用新提交替换旧提交。如果已经推送了最后一次提交则不建议使用。

`git rm`   从索引区和工作目录中移除文件。可以使用glob模式(shell中的正则)
                 -f   如果文件已修改就需要加上 -f 选项强制删除。
                 --cached 移除索引区的文件，但是保留工作区文件
`git mv`  重命名文件，相当于 mv  a  b,  git rm a, git add b

`git log`  -p/--patch -num  按照补丁格式/显示差异输出 num 行
                  --stat  附带一系列总结性选项
                 --oneline  简略输出
                 --pretty=format:"xxx"  指定格式输出，常用选项自查。如: %h hash %an 作者 %ar 修订日期  %s 提交说明
                 --since   --until   按照时间限制输出   --since=2.weeks/2008-01-15/"2 years one day 3 minutes ago"
                 -S  接受一个字符串参数，并且只会显示那些添加或删除了该字符串的提交。查看某一项是什么时候引入的，- S俗称鹤嘴锄pickaxe
                --author  按指定作者显示
                --decorate --graph --all 按照图画显示git log
                --no-merges 显示只有一个父亲的提交，等价于 --max-parents=1，也就是未合并的提交
                --g 输出引用日志信息
                --left-right 显示出每个提交到底处于哪一侧的分支
                -L 查看某一函数的历史。git log -L :A:zlib.c 查看zlib.c文件中A函数的变更历史

`git reset`  HEAD  \<file\>...   取消暂存的文件，本质是将HEAD指向的提交中的文件恢复到Index区，相当于 git reset --mixed HEAD \<file\>。相当于跳过下面的第一步，执行了第二步，将Head中的内容放到索引区
                      HEAD~  移动 HEAD 指向的分支到HEAD~1(分支被实际移动了，checkout不会实际移动分支)，相当于 git reset --soft HEAD~1 （1）
                      --mixed  HEAD~  更新索引，将HEAD~1中的内容放到暂存/索引区。  （2）
                      --hard     HEAD~  更新工作目录，将索引区的内容放到工作目录中。 （3）

`git checkout` [commit]  --  \<file\>  撤销之前所做的所有修改，用最近的提交来中的文件更新索引区和工作区的相应文件。--是Linux的分隔符，一般checkout后面跟分支，想跟文件就需要用--表示命令项结束了(也可以不加)。用那次提交的那个文件来更新索引和工作目录，相当于 git reset --hard [branch] file（如果reset允许）
                          -b [brA] [remote/brA]:  基于origin/brA创建并切换到brA分支。origin/brA也是在本地的别搞错。brA称为跟踪分支，origin/brA称为远程跟踪分支。<span style=
"color:red">跟踪分支和远程分支有直接关联</span>，git pull时会自动抓取合并。
                          --track  [remote/brName] 等价于上面。使用 git branch -u remote/br更新跟踪的远程分支

`git remote`  -v 显示需要读写的远程仓库的简写和对应的URL
                       add  \<shortname\>  \<url\> 添加一个新的远程git仓库
                       show  \<remote\> 显示

`git fetch` \<remote\> 拉取远程仓库数据但不合并

`git pull` \<remote\>   拉取并合并远程仓库数据

`git push` \<remote\> \<branch\> 将分支推送到远程仓库

`git tag`  -l "string" 按照特定模式查找标签
                附注标签(annotated) git tag -a vx.x -m "xxx" 是一个完整对象
                 轻量标签(lightweight)  git tag vx.x  \<版本hash\>，提交的引用
                默认标签是不会推送到远程仓库，需要 git push origin \<tagname\>
                       git push origin --tags 将不在远程仓库上的标签全部传送上去
                -d \<tagname\> 删除标签，同理远程仓库上的该标签也不会删除，执行
                      git push \<remote\> :refs/tags/\<tagname\> 或 git push origin --delete \<tagname\>来推送删除远程仓库上的标签
                -s 签署标签(这是一个附注标签)，不过在这之前需要先配置GPG个人密钥。
                          gpg --list-keys 显示个人密钥
                          gpg --gen-key  生成一个密钥 
                         git config --global user.signingkey  xxxx  设置签署的私钥
                -v \<tag-name\> 使用GPG来验证签名。但是签署者的公钥需要在你的钥匙链中



#### 三、Git 分支

`git merge` \<other branch\> 将其他分支合并到当前所在分支。`git mergetool` 使用图形化界面解决冲突
                     --squash \<brA\> 将A分支上的所有提交压缩至一个变更集，不会真的生成一个合并提交，也就说未来的提交只有一个父提交
                    --abort 尝试恢复到运行合并前的状态来解决冲突，类似于 git reset --hard HEAD
                    -Xignore-all-space/-Xignore-space-change 前者忽略空白修改，后者将一个与多个连续的空白字符视为相等。忽略但是没有修改，可能使内容更加混乱

`git branch` -v 查看每个分支的最后一次提交 --verbose
                      --merged / --no-merged \<branch\> 查看合并/未合并到指定分支(默认当前分支)的其他分支
                      -r/-a 查看远程分支/所有分支
                      -u remote/brName 更新当前分支跟踪的远程分支
                      --vv 查看设置的所有跟踪分支。这是基于本地的远程跟踪分支来看的，并没有连接远程仓库，所以执行前先执行 git fetch --all 比较好

`git fetch` \<remote\> 从远程仓库拉代码，如果远程仓库有新的分支（如 hotfix）那么本地会新建不可修改的远程跟踪分支 remote/hotfix，而不会新建一个hotfix分支，可以使用 git merge remote/hotfix将这些工作合并到自己当前所在分支。

`git push` \<remote\> \<branch\> 假设branch为master,那么他会自动展开为refs/heads/master:refs/heads/serverfix。如果本地分支名和远程分支名不同，就使用 localBranch:originBranch。
                   remote  --delete  brName 删除远程服务器上的分支
                   -u 设置上游

`git pull` 等价于 git fetch 和 git merge  

`git rebase` \<DstBr\> 将当前分支上的提交变基到目的分支，这会找到两个分支的共同祖先，然后剪切当前分支(A)到共同祖先这一段然后拼接到目的分支(B)后面，之后可以使用 git checkout B; git merge A
                       --onto master server client 取出client分支上有的提交且不在server分支上的，然后将这部分贴到master上
                      -i HEAD~3 交互式修改最后三次提交的信息。将pick改为edit表示想要修改。提交历史的显示和 git log 是相反的。删除这一行则会移除这次提交。squah 压缩提交



#### 四、服务器上的 Git

协议

> 1. 本地协议（Local protocol），远程版本库就是同一主机上的另一个目录。
>
>    ```shell
>    $ git clone /srv/git/project.git
>    # 指定file://开头的会触发网路传输资料的进程，传输效率较低
>    $ git clone file:///srv/git/project.git  
>    ```
>
> 2. SSH 可以鉴权
> 3. http 兼具2、4特点
> 4. git 公开的，无鉴权，用于只读文件



#### 五、分布式Git

`git request-pull` \<remote/br\> \<remoteB\> 生成一个请求(通过邮件发送给项目维护者)，要求将当前分支（在remoteB中）的更改拉入remote/br中

`git merge-base` \<brA\> \<brB\> 找出A B分支的共同祖先

`git describe` \<br\> 生成构建号，生成一个字符串， 它由最近的标签名、自该标签之后的提交数目和你所描述的提交的部分 SHA-1 值（前缀的 `g` 表示 Git）构成

`git archive` \<brA\> --prefix='xxx/' | gzip > \`git describe brA\`.tar.gz 准备发布，创建一个brA的快照，通常是master

`git shortlog` 制作提交简报，如 git shortlog --no-merges master --not v1.0.1



#### 七、Git工具

> HEAD～2 代表第一父提交的第一父提交，即祖父提交
> HEAD^2 代表第二父提交。所以HEAD~等价于HEAD^
> HEAD~1^2 父提交的第二父提交
>
> A..B/^A  B/B --not A 在B分支中而不在A分支中的部分，HEAD会代替留空的一边
>                                     双点只能有两个对象，但是后两种语法可以有多个对象
> A...B 三点语法可以找出被A或B其中之一包含但是不被两者同时包含的提交

`git reflog` 查看HEAD的引用日志

`git show` br@{yesterday/2.months.ago} 查看br分支在昨天/两个月前时候的引用日志（指向哪个提交）

`git stash` 贮藏当前的工作内容（包括索引区和工作目录）
                     list  查看贮藏的东西
                     apply  默认使用最近的贮藏，可以指定 apply stash@{indexNum}
                                 --index 还尝试恢复索引区的内容，即原封不动地恢复索引区和工作目录，不然全部恢复到索引区
                     drop  stash@{indexNum} 删除某个贮藏
                     --keep-index 贮藏并且保留在索引区中
                     -u 贮藏任何未跟踪文件，但不包括忽略的文件除非使用 -a/--all就全包括
                     branch  \<brName\> 以指定的分支名创建一个新分支，检出贮藏工作时所在的提交

`git clean` 移除没有忽略的未被追踪的文件
                     -d 移除工作目录空的子目录
                     -n/--dry-run  做一次演习看看会删除啥，别配合-f

`git grep` \<pattern\> \<file\> 默认查找工作目录的文件
                   -n/--line-number 输出找到的匹配行的行号
                   -c/--count 输出有匹配的每个文件包含了多少匹配
                   -p/--show-function 显示每一个匹配的字符串所在的方法或函数
                   --break 在不同文件中打印一个空行，方便查看
                   --heading 展示文件名在匹配的之上而不是在每行的开头
                   -e 下一个参数是匹配模式
                    --and/or/not 确保多个匹配出现在同一行中 git grep -e A --and -e B fileName

`git filter-branch` 使用脚本改写大量提交
                                     --tree-filter 'rm -f passwd.txt' HEAD 从整个提交历史中移除 passwd.txt 文件
                                     --all 在所有分支上运行

```shell
# 全局修改邮箱地址，常用于开始工作时忘记运行 git config 来设置名字和邮箱地址
git filter-branch --commit-filter '
        if [ "$GIT_AUTHOR_EMAIL" = "schacon@localhost" ];
        then
                GIT_AUTHOR_NAME="Scott Chacon";
                GIT_AUTHOR_EMAIL="schacon@example.com";
                git commit-tree "$@";
        else
                git commit-tree "$@";
        fi' HEAD
```

 

`git blame` 查看每行是谁以及什么时候写的
                     -L a,b 限制行数在a到b之间
                     -C 分析你正在标注的文件，并尝试找出文件中从别的地方复制过来的代码片段

`git bisect` 二分查找bug，能够在数百个提交中快速定位bug存在，有点复杂以后研究

`git bundle` create xx.bundle HEAD master 创建一个打包文件，加上HEAD饮用后解压出来后会自动检出到master分支，不然需要在 git clone -b branchName xx.bundle destDir 指定检出的分支名，使用 git clone xx.bundle destDir 将打的包克隆到指定目录						 
