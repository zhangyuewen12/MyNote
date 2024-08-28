在我们使用git作为版本控制工具进行代码管理之后，经常性的会碰到一个问题：git commit后，如何撤销commit，下面详细讲一下。

git add newFiles

git commit -m '新增xx页面'

执行commit后，还没执行push时，想要撤销这次的commit，该怎么办？

解决方案：
我们可以使用命令：git reset --soft HEAD^  这样就成功撤销了commit。

使用git reset --hard HEAD^  这样连add也撤销了。

*注：reset 命令只能回滚最新的提交，无法满足保留最后一次提交只回滚之前的某次提交。

命令解释：

HEAD^ 表示上一个版本，即上一次的commit，几个^代表几次提交，如果回滚两次就是HEAD^^。
也可以写成HEAD~1，如果进行两次的commit，想要都撤回，可以使用HEAD~2。
--soft
不删除工作空间的改动代码 ，撤销commit，不撤销add
--hard

删除工作空间的改动代码，撤销commit且撤销add
如果commit后面的注释写错了，先别急着撤销，可以运行git commit --amend 
进入vim编辑模式，修改完保存即可