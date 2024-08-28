# git 放弃本地修改

https://blog.csdn.net/andyzhaojianhui/article/details/80818934
##未使用 git add 缓存代码时。  
可以使用 git checkout -- filepathname (比如： git checkout -- readme.md  ，不要忘记中间的 “--” ，不写就成了检出分支了！！)。放弃所以的文件修改可以使用 git checkout .  命令。  

此命令用来放弃掉所有还没有加入到缓存区（就是 git add 命令）的修改：内容修改与整个文件删除。但是此命令不会删除掉刚新建的文件。因为刚新建的文件还没已有加入到 git 的管理系统中。所以对于git是未知的。自己手动删除就好了。


##已经使用了  git add 缓存了代码。

可以使用  git reset HEAD filepathname （比如： git reset HEAD readme.md）来放弃指定文件的缓存，放弃所以的缓存可以使用 git reset HEAD . 命令。

此命令用来清除 git  对于文件修改的缓存。相当于撤销 git add 命令所在的工作。在使用本命令后，本地的修改并不会消失，而是回到了如（一）所示的状态。继续用（一）中的操作，就可以放弃本地的修改。


##已经用 git commit  提交了代码。

可以使用 git reset --hard HEAD^ 来回退到上一次commit的状态。此命令可以用来回退到任意版本：git reset --hard  commitid 

你可以使用 git log 命令来查看git的提交历史。git log 的输出如下,之一这里可以看到第一行就是 commitid：