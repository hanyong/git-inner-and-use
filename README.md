git原理与使用
===

## git 与 svn 区别

### 文件模式位

#### svn

创建一个 svn 仓库, 添加一个 shell 脚本.
提交前忘记设置可执行标志位.

```bash
svnadmin create svn-hello-repo
svn checkout file://$PWD/svn-hello-repo svn-hello
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
svn checkout file://$PWD/svn-hello-repo svn-hello-test
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
```

### svn - 中央仓库结构


