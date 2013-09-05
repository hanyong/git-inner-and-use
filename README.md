git原理与使用
===


## git 与 svn 初体验

### 记录和维护文件模式位

#### svn

创建 svn 标准仓库结构, 辅助创建初始化 svn 仓库.

```bash
rm -rf svn-std
mkdir svn-std svn-std/trunk svn-std/branches svn-std/tags
```

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

看一下提交历史

```
cd svn-hello
rabbitvcs log
cd ..
```

svn 只是简单把分支修改的最终结果合并到主干上, 而在分支上的修改历史都看不到了.
虽然在分支上还能查看提交历史, 这有两个问题, 
(1) 不够方便直观 
(2) 随着时间推移, 分支越来越多难以管理, 就得删除旧的分支, 分支被删除后就更加难以查看提交历史了.

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
cd git-hello
rabbitvcs log
cd ..
```

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
cd ..
```
主干和分支刚刚合并过, 分支再合并主干, svn 却报冲突了, 这不科学.
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

再看提交历史, hello 和 master 又分别发展开.

#### 结论

git 良好的维护了分支合并的关系, 智能避免冲突.
svn 分支能力很弱, 甚至导致不该有的冲突和重复报冲突.
经常使用分支合并的系统, 用了 git 之后, 冲突少了很多. 

> svn经验: 每次合并主干后重新拉分支, 不要尝试双向合并. 
这又导致分支数快速增加, 删除旧分支导致提交历史丢失.

多分支合并的时候, svn 更是彻底傻了, git 还能运筹帷幄.
明白 git 和 svn 分支实现原理的差异, 就知道 git 为何如此强大.

svn 只是用不同的子文件夹来模拟不同的分支, 分支有关的命令以文件夹(本地文件夹或完整URL)来操作.
而 git 分支是内建的一种体系, 有分支名, 分支有关的命令通过分支名来操作.


## 部署结构-分布式仓库

### svn - 中央仓库结构


