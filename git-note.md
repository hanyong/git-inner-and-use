
如果说轻量级 tag 是命名提交, 那么 branch 就是 命名提交链. 
branch 指向提交链的头节点, 提交链增加节点时, branch 自动前移指向新的头节点.

轻量级 tag 不仅可以给提交命名, 还可以给一切 object 命名, 如 tree, blob, tag.

分支或者 tag 都只是对象引用, 或者说命名对象.
不同类型的引用行为不同, 比如 tag 和 branch 都是提交对象的引用, 
但用途和行为不一样.


## git 与 svn 区别
1. 部署结构, 分布式, 兼备客户端与服务器的功能, 不区分客户端与服务器
2. 真正的分支，轻量级分支，只是一个引用. 分支合并效果，修改记录不丢失。多次合并不冲突。


## 查询和同步远程仓库

### 跟踪和更新

* git push
push.default 影响行为, 1.x 建议修改为 upstream, 2.x 默认 simple
* git pull
* git branch --set-upstream local-branch remote-branch

### git remote 

* 更新远程信息到本地
git remote update
git remote update origin
git remote update apache70

* 查看远程仓库状态, 对比本地仓库
git remote show 

### git ls-remote

查看远程仓库上的引用(branch, tag)列表

git ls-remote
git ls-remote apache70

