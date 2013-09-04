git原理与使用
===

## git 与 svn 区别

### 维护文件模式位

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
svn import svn-std file://$PWD/svn-hello-repo -m "init svn standard layout"
svn checkout file://$PWD/svn-hello-repo/trunk svn-hello
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
svn checkout file://$PWD/svn-hello-repo/trunk svn-hello-test
cd svn-hello-test/
./a.sh
```

回到之前的工作目录, 设置可执行标志位, 却发现没法提交.

```
cd ../svn-hello
chmod +x a.sh
./a.sh
svn status
cd ..
```

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
```

回到之前的仓库, 设置可执行标志位后提交.

```
cd ../git-hello
chmod +x a.sh
./a.sh
git status
git diff
git add a.sh
git commit -m "a.sh chmod +x"
```

更新测试仓库, 发现可执行标志位被更新，可以直接执行脚本了.

```
cd ../git-hello-test
git pull
ll
./a.sh
```

> 结论: svn初次提交时保留文件模式位, 之后不能维护和修改.
git 记录和维护文件模式位.

#### svn小技巧

svn不能直接修改已提交文件的模式位, 
不过可以先删除旧文件, 再添加设置了模式位的新文件, 
从用户来看等于设置了模式位, 从 svn 来看是替换了一个文件.

```
cd ../svn-hello
svn rm --keep-local a.sh
svn add a.sh
git status
svn commit -m "a.sh chmod +x"
cd ../svn-hello-test/
svn update
ll
./a.sh
cd ..
```

### 保留分支上的提交历史

从主干拉一个分支出来, 进行一些修改, 再合并分支到主干.

#### svn

```
cd svn-hello
svn cp . file://$(cd ../svn-hello-repo; pwd)/branches/hello -m "create branch hello"
sed -i '3 c\echo goodbye' a.sh
svn commit -m 'change "bye" to "goodbye"'
svn switch file://$(cd ../svn-hello-repo; pwd)/branches/hello
sed -i '2 c\echo hello word' a.sh
svn commit -m 'change "hello" to "hello word"'
sed -i '2 c\echo hello world' a.sh
svn commit -m 'fix "word" to "world"'
svn switch file://$(cd ../svn-hello-repo; pwd)/trunk
svn merge file://$(cd ../svn-hello-repo; pwd)/branches/hello
svn commit -m "merge branch 'hello'"
```

#### git

```
git branch hello
sed -i '3 c\echo goodbye' a.sh
git commit -a -m 'change "bye" to "goodbye"'
git checkout hello
sed -i '2 c\echo hello word' a.sh
git commit -a -m 'change "hello" to "hello word"'
sed -i '2 c\echo hello world' a.sh
git commit -a -m 'fix "word" to "world"'
git checkout master
git merge hello
```

### 更聪明的合并

假设想继续在分支上开发新功能, 
那么继续之前应该把主干上的最新修改也合并到分支上来.

svn
```
svn switch file://$(cd ../svn-hello-repo; pwd)/branches/hello
svn merge file://$(cd ../svn-hello-repo; pwd)/trunk
```
主干和分支刚刚合并过, 分支再合并主干 svn 却报冲突, 这不科学, 
应该直接与刚才主干合并分支的结果一样, 而 git 正是这样的.

```
git checkout hello
git merge master
```

git 已经记录下分支 aaa 与主干 bbb 合并结果为 xxx 这样的信息, 
所以反过来合并结果也一样.
svn 却没有记录下主干版本 x 与分支版本 y 合并结果为主干版本 z 到信息,
反过来合并时又重新尝试进行合并操作, 结果还报冲突了.

git 非常好的维护了分支与合并的关系。新版本的 svn 可能再合并方面以及增强了很多, 但是 git 到分支功能比 svn 强大很多, svn 始终有很多局限性。

svn 只是用不同的子文件夹来模拟不同的分支, 分支有关的命令以文件夹(本地文件夹或完整URL)来操作.
而 git 分支是内建的一种体系, 有分支名, 分支有关的命令通过分支名来操作.

### svn - 中央仓库结构


