git原理与使用
===

## 目录

* git 与 svn 初体验
用脚本执行一些简单的操作, 展示 git 和 svn 的一些差异

* 部署结构 - 分布式仓库
* 分支
简单来看, git 与 svn 的最大差异在于分布式和分支,
接下来介绍这两块内容，及相关的一些 git 命令

* 分布式kv - 磁盘上的对象系统
简单介绍一下 git 内部原理,
这部分不属于版本控制的内容, 而是一些非常有趣的东西,
会涉及到一些底层命令, 平时可能基本用不到, 但对理解和使用 git 命令会有帮助.


## git 与 svn 初体验

### 记录和维护文件模式位

准备一个演示例子的空目录.

```bash
mkdir -p ~/tmp
cd ~/tmp
rm -rf *
```

创建 svn 标准仓库结构, 辅助创建初始化 svn 仓库.

```bash
rm -rf svn-std
mkdir svn-std svn-std/trunk svn-std/branches svn-std/tags
```

#### svn

创建一个 svn 仓库, 添加一个 shell 脚本.
提交前忘记设置可执行标志位.

```bash
svnadmin create svn-hello-repo
SVN_REPO="$PWD/svn-hello-repo"
svn import svn-std file://$SVN_REPO -m "init svn standard layout"
svn checkout file://$SVN_REPO/trunk svn-hello
cd svn-hello
cat > a.sh <<EOF
#!/bin/bash
echo hello
echo bye
EOF
svn add a.sh
svn commit -m "add a.sh"
cd ..
```

checkout 出提交的文件, 发现没有设置可执行标志位, 不能直接执行.

```
svn checkout file://$SVN_REPO/trunk svn-hello-test
cd svn-hello-test/
./a.sh
cd ..
```

回到之前的工作目录, 设置可执行标志位, 却发现没法提交.

```
clear
cd svn-hello
chmod +x a.sh
./a.sh
svn status
svn diff
cd ..
ll svn-hello svn-hello-test
```

svn-hello 设置了 "chmod +x a.sh", 却没法提交到仓库,
svn-hello-test 没法得到 "chmod +x a.sh" 的修改.

#### git

创建一个 git 仓库, 添加一个 shell 脚本.
提交前忘记设置可执行标志位.

```bash
git init git-hello
cd git-hello/
cat > a.sh <<EOF
#!/bin/bash
echo hello
echo bye
EOF
git add a.sh
git commit -m "add a.sh"
cd ..
```

clone 创建的仓库, 发现没有设置可执行标志位, 不能直接执行.

```
git clone git-hello git-hello-test
cd git-hello-test/
./a.sh
cd ..
```

回到之前的仓库, 设置可执行标志位.

```
clear
cd git-hello
ll
chmod +x a.sh
ll
./a.sh
git status
git diff
cd ..
```

注意与 svn 的区别,
"chmod +x a.sh" 被 git 检查到, 并可以将修改提交到仓库.

```
cd git-hello
git add a.sh
git commit -m "a.sh chmod +x"
cd ..
```

更新 git-hello-test , 发现可执行标志位被更新，可以直接执行脚本了.

```
cd git-hello-test
git pull
ll
./a.sh
cd ..
```

#### 结论

svn初次提交时保留文件模式位, 之后不能维护和修改.
git 记录和维护文件模式位.

#### svn小技巧

svn不能直接修改已提交文件的模式位, 
不过可以先删除旧文件, 再添加设置了模式位的新文件, 
从用户来看等于设置了模式位, 从 svn 来看是替换了一个文件.

```
cd svn-hello
svn rm --keep-local a.sh
svn add a.sh
svn status
svn commit -m "a.sh chmod +x"
cd ../svn-hello-test/
svn update
ll
./a.sh
cd ..
```

查看 git 和 svn 提交历史

```
cd git-hello
gitk --all & disown
cd ..
```

```
cd svn-hello
svn update
rabbitvcs log
cd ..
```

### 合并时保留分支上的提交历史

从主干拉一个分支出来, 主干和分支各有一些修改（不冲突）, 再合并分支到主干.

#### svn

```
cd svn-hello

svn update
svn cp . file://$SVN_REPO/branches/hello -m "create branch hello"

sed -i '2 c\echo hello world' a.sh
svn commit -m 'change "hello" to "hello word"'

svn switch file://$SVN_REPO/branches/hello
cat > b.sh <<EOF
#!/bin/bash
echo welcome
EOF
chmod +x b.sh
svn add b.sh
svn commit -m 'add b.sh'
sed -i '2 c\echo welcome to ali' b.sh
svn commit -m 'change "welcome" to "welcome to ali"'

svn switch file://$SVN_REPO/trunk
svn merge file://$SVN_REPO/branches/hello
svn commit -m "merge branch 'hello'"

cd ..
```

#### git

```
cd git-hello

git branch hello

sed -i '2 c\echo hello world' a.sh
git commit -a -m 'change "hello" to "hello word"'

git checkout hello
cat > b.sh <<EOF
#!/bin/bash
echo welcome
EOF
chmod +x b.sh
git add b.sh
git commit -m 'add b.sh'
sed -i '2 c\echo welcome to ali' b.sh
git commit -a -m 'b.sh change "welcome" to "welcome to ali"'

git checkout master
git merge hello

cd ..
```

看一下提交历史

```
cd svn-hello
rabbitvcs log
cd ..
```

```
cd git-hello
gitk --all & disown
cd ..
```

svn 只是简单把分支修改的最终结果合并到主干上, 
合并操作并没有使两个分支建立联系, 它们还是彼此独立的.
合并结果看不到分支上的历史, 只能在分支上查看.

```
cd svn-hello
rabbitvcs log file://$SVN_REPO/branches/hello
cd ..
```

这有两个问题, 
(1) 不够方便直观 
(2) 随着时间推移, 分支越来越多难以管理, 就得删除旧的分支, 分支被删除后就更加难以查看提交历史了.

git 的清晰的展示了分支上的修改及合并.

#### 结论
git 合并不会丢失分支上的提交历史, svn 会丢失.


### 更聪明的合并

假设想继续在分支上开发新功能, 
继续之前应该先把主干上的最新修改也合并到分支上来.

svn
```
cd svn-hello
svn switch file://$SVN_REPO/branches/hello
svn merge file://$SVN_REPO/trunk
svn commit -m "merge 'trunk'"
LANG=C svn status
cd ..
```
主干和分支刚刚合并过, 分支再合并主干, svn 却报冲突了, 这不科学.
svn 认为主干和分支都增加了文件, 而主干上的文件也是分支增加的, 
这是重复合并自己, 自己和自己冲突了, 这不科学.

双向合并的结果应该是一样的, 而 git 正是这样.

```
cd git-hello
git checkout hello
git merge master
cd ..
```

刷新提交历史, hello 和 master 重合了, 
双向合并之后的提交，重新变成两个分支的最近共同祖先,
之后 hello 可以再继续开发新的功能而不影响 master.

```
cd git-hello
git checkout master
sed -i '2 c\echo Hello, World!' a.sh
git commit -a -m 'a.sh update to "Hello, World!"'
git checkout hello
sed -i '2 c\echo Welcome to alibaba!' b.sh
git commit -a -m 'b.sh change "Welcome to alibaba!"'
cd ..
```

#### 结论

git 良好的维护了分支合并的关系, 智能避免冲突.
svn 分支能力很弱, 甚至导致不该有的冲突和重复报冲突.
经常使用分支合并的系统, 用了 git 之后, 冲突少了很多. 

> svn经验: 每次合并主干后重新拉分支, 不要尝试双向合并. 
这又导致分支数快速增加, 删除旧分支导致提交历史丢失.

svn 只是用不同的子文件夹来模拟不同的分支, 分支有关的命令以文件夹(本地文件夹或完整URL)来操作.
而 git 分支是内建的一种体系, 有分支名, 分支有关的命令通过分支名来操作.


## 部署结构-分布式仓库

svn 是典型的 "服务器-客户端" 结构,
1 个服务器对应 N 个客户端.
svn 操作服务器和操作客户端的命令也是分开的, 分别是 svnadmin 和 svn.
如创建 svn 仓库就要使用 svnadmin 命令, 而提交修改就要使用 svn 命令.

仓库保存在服务器上, 客户端只有工作拷贝(WorkingCopy)和一些 .svn 元数据.
很多 svn 命令都需要跟服务器通信.
总是客户端主动向服务器发起通信, 服务器不会主动跟客户端通信.

而 git 是每台 PC 上都有一份工作拷贝和一个完整的 .git 仓库,
简单来看可以认为 git 是把 svn 服务器上的仓库直接搬到了本地, 叫做本地仓库.
而 git 命令也相当于是把操作服务器和操作客户端的功能整合在了一起,
如创建仓库和提交修改都是使用 git 命令.
* git 整合了客户端和服务器, 消除了客户端和服务器的区别.
每个 git 节点都是等价的.

svnadmin + svn v.s. git ?

* git 工作拷贝只会和本地仓库通信, 而仓库间可以双向通信.
git 操作命令通常可分为两类
0. 操作本地仓库 
0. 仓库间同步(相互推送和拉取数据)

仓库间的相互联系可能是任意网络, 这可能会很散乱.
所以
* 通常也会选择一个仓库作为中央仓库, 所有仓库都与它通信，同步所有修改.

中央仓库只是为了同步统一所有开发人员的本地仓库, 不需要在上面开发, 
因此通常使用一种特殊的仓库类型, 没有工作拷贝, 叫做 bare 仓库.
通常在中央仓库上只进行第二类操作: 仓库间同步.

除了没有工作拷贝, 中央仓库跟普通仓库也是一样的.
一个普通仓库可以与 0 个或多个仓库建立连接.

* git 命令
大家都比较熟悉, 可以直接跳过, 有疑问的命令可以讲一下.

概念: upstream 和 push.default

## 分布式kv - 磁盘上的对象系统

删除遗留文件

```
rm -rf git-hello*
```

创建一个空仓库, 查看仓库内容

```
git init git-hello
cd git-hello
ls .git
find .git/objects/ -type f
```

这是一个没有数据的 kv 系统. 仓库内容输出如下.

```
observer.hany@ali-59375nm:~/tmp/git-hello$ ls .git
branches  config  description  HEAD  hooks  info  objects  refs
observer.hany@ali-59375nm:~/tmp/git-hello$ find .git/objects/ -type f
observer.hany@ali-59375nm:~/tmp/git-hello$ 
```

添加一个文件, 查看仓库变化.

```
echo "hello world" > a.txt
git add a.txt
ls .git
find .git/objects/ -type f
```

.git 仓库下多了一个 "index" 文件, kv 系统多了一个对象.

```
observer.hany@ali-59375nm:~/tmp/git-hello$ ls .git
branches  config  description  HEAD  hooks  index  info  objects  refs
observer.hany@ali-59375nm:~/tmp/git-hello$ find .git/objects/ -type f
.git/objects/3b/18e512dba79e4c8300dd08aeb37f8e728b8dad
```

计算一下文件的 SHA1, 与 git 的 key 并不相等

```
observer.hany@ali-59375nm:~/tmp/git-hello$ sha1sum a.txt 
22596363b3de40b06f981fb85d82312e8c0ed511  a.txt
```

因为 git 对象包含元数据, k 不是简单的文件 SHA1.

修改文件内容后重新添加, 
```
echo "bye" >> a.txt
git add a.txt
ls .git
find .git/objects/ -type f
```

.git 目录无变化, kv 系统中增加了一个对象.

```
observer.hany@ali-59375nm:~/tmp/git-hello$ ls .git
branches  config  description  HEAD  hooks  index  info  objects  refs
observer.hany@ali-59375nm:~/tmp/git-hello$ find .git/objects/ -type f
.git/objects/90/d6d2fbdec0f3667fc0eb7d784843717aaccaf1
.git/objects/3b/18e512dba79e4c8300dd08aeb37f8e728b8dad
```

提交工作拷贝, 再观察 .git 仓库.

```
git commit -m "add a.txt"
find .git/objects/ -type f
```

又多增加了 2 个对象

```
observer.hany@ali-59375nm:~/tmp/git-hello$ find .git/objects/ -type f
.git/objects/4f/d9e45478c1fe4fcebc77c5ad42cf7e3764ccb1
.git/objects/90/d6d2fbdec0f3667fc0eb7d784843717aaccaf1
.git/objects/3b/18e512dba79e4c8300dd08aeb37f8e728b8dad
.git/objects/c1/de456523c5c4ef403d5f8efe95b9bbfa4ef3c7
```

查看新增的两个对象

```
git cat-file -p '4fd9e45478c1fe4fcebc77c5ad42cf7e3764ccb1'
git cat-file -p c1de456523c5c4ef403d5f8efe95b9bbfa4ef3c7
```

新增的两个对象一个是 tree 对象, 一个是提交对象.

```
observer.hany@ali-59375nm:~/tmp/git-hello$ git cat-file -p '4f/d9e45478c1fe4fcebc77c5ad42cf7e3764ccb1'
fatal: Not a valid object name 4f/d9e45478c1fe4fcebc77c5ad42cf7e3764ccb1
observer.hany@ali-59375nm:~/tmp/git-hello$ git cat-file -p '4fd9e45478c1fe4fcebc77c5ad42cf7e3764ccb1'
tree c1de456523c5c4ef403d5f8efe95b9bbfa4ef3c7
author 涧泉 <observer.hany@alibaba-inc.com> 1378399788 +0800
committer 涧泉 <observer.hany@alibaba-inc.com> 1378399788 +0800

add a.txt
observer.hany@ali-59375nm:~/tmp/git-hello$ git cat-file -p c1de456523c5c4ef403d5f8efe95b9bbfa4ef3c7
100644 blob 90d6d2fbdec0f3667fc0eb7d784843717aaccaf1	a.txt
```

