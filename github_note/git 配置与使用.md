# git 配置与使用
## git 安装
centos7 自带 git
```
# git --version
git version 1.8.3.1
```
## 创建版本库
```
# mkdir learngit
[root@centOSlearning sunnylinux]# cd learngit
[root@centOSlearning learngit]# pwd
/home/sunnylinux/learngit
[root@centOSlearning learngit]# git init
初始化空的 Git 版本库于 /home/sunnylinux/learngit/.git/
[root@centOSlearning learngit]# ls -al
总用量 4
drwxr-xr-x.  3 root       root         18 1月  11 20:57 .
drwx------. 30 sunnylinux sunnylinux 4096 1月  11 20:56 ..
drwxr-xr-x.  7 root       root        119 1月  11 20:57 .git

```
.git的目录，这个目录是Git来跟踪管理版本库的，没事千万不要手动修改这个目录里面的文件

## 将文件提交到仓库
文件要放在之前创建的git目录下，子目录也可以</br>
要注意好好填写 -m 提交参数后面的“本次提交说明”</br>
```
# git add readme.txt
# git commit -m "wrote a readme file" 
[master（根提交） 796ce08] wrote a readme file
 Committer: root <root@centOSlearning.SharonLi>
您的姓名和邮件地址基于登录名和主机名进行了自动设置。请检查它们正确
与否。您可以通过下面的命令对其进行明确地设置以免再出现本提示信息：

    git config --global user.name "Your Name"
    git config --global user.email you@example.com

设置完毕后，您可以用下面的命令来修正本次提交所使用的用户身份：

    git commit --amend --reset-author

 1 file changed, 2 insertions(+)
 create mode 100644 readme.txt

```
修改文件（新版本）后重新提交
```
# git add readme.txt
# git commit -m "add a new line"
[master fb17fb7] add a new line
 1 file changed, 1 insertion(+)
```
查看提交历史记录(由近到远)
```
# git log
commit 54bdbdf710b4cb3583ac52f4efae0454b3229d61
Author: sharonli <root@centOSlearning.SharonLi>
Date:   Fri Jan 11 21:21:25 2019 +0800

    add one line again

commit fb17fb793e5e5f7a3e47e20f2ade9f091d12285d
Author: sharonli <root@centOSlearning.SharonLi>
Date:   Fri Jan 11 21:18:05 2019 +0800

    add a new line

commit 9abcf7a95c7dce7e3d2afebc0827ac845cca23a3
Author: sharonli <root@centOSlearning.SharonLi>
Date:   Fri Jan 11 21:10:03 2019 +0800

    wrote a readme file
    change the user name

# git log --pretty=oneline
54bdbdf710b4cb3583ac52f4efae0454b3229d61 add one line again
fb17fb793e5e5f7a3e47e20f2ade9f091d12285d add a new line
9abcf7a95c7dce7e3d2afebc0827ac845cca23a3 wrote a readme file change the user name

```

## 版本回退
在Git中，用HEAD表示当前版本，上一个版本就是HEAD^，上上一个版本就是HEAD^^，版本过多时会表示为HEAD~100
```
# 当前readme版本
Git is a version control system.
Git is free software.
Something change ...
another one change
~                                                                                        
~                         
```
回退到上一版本，注意HEAD要大写
```
# git reset --hard HEAD^  
HEAD 现在位于 fb17fb7 add a new line

# 当前readme版本
Git is a version control system.
Git is free software.
Something changes ...
~  
```
注意：回退后log里面会没有了第三版的信息
```
# git log --pretty=oneline
fb17fb793e5e5f7a3e47e20f2ade9f091d12285d add a new line
9abcf7a95c7dce7e3d2afebc0827ac845cca23a3 wrote a readme file change the user name
```
命令行如果没有关闭，可以回找前面的log中显示的commit id </br>
回退到该版本，可以输入前面几位commit id，git会自动找，如果有多个版本commit id前面几位相同，则要多输几位</br>
```
# git reset --hard 54bd
HEAD 现在位于 54bdbdf add one line again
```
如果找不到commit id，可以查git的历史命令，由近到远
```
# git reset --hard 54bd
HEAD 现在位于 54bdbdf add one line again
[root@centOSlearning learngit]# git reflog
54bdbdf HEAD@{0}: reset: moving to 54bd
fb17fb7 HEAD@{1}: reset: moving to HEAD^
54bdbdf HEAD@{2}: commit: add one line again
fb17fb7 HEAD@{3}: commit: add a new line
9abcf7a HEAD@{4}: commit (amend): wrote a readme file
87bd690 HEAD@{5}: commit (amend): wrote a readme file
796ce08 HEAD@{6}: commit (initial): wrote a readme file

```
## 工作区和暂存区
工作区：其实就是显示还没有add和commit的修改，add和commit后工作区会变为空
```
# git status
# 位于分支 master
# 尚未暂存以备提交的变更：
#   （使用 "git add <file>..." 更新要提交的内容）
#   （使用 "git checkout -- <file>..." 丢弃工作区的改动）
#
#	修改：      readme.txt
#
# 未跟踪的文件:
#   （使用 "git add <file>..." 以包含要提交的内容）
#
#	sayhello.py
修改尚未加入提交（使用 "git add" 和/或 "git commit -a"）

```
需要git add命令实际上就是把要提交的所有修改放到暂存区（Stage），然后，执行git commit就可以一次性把暂存区的所有修改提交到分支。
```
# git log --pretty=oneline
3f473421bc16c3a275a95a06386f83394c71a557 sayhello_print
79a57d4ebb679e48b5d30488a663c5283afb79fd readme.txt
54bdbdf710b4cb3583ac52f4efae0454b3229d61 add one line again
fb17fb793e5e5f7a3e47e20f2ade9f091d12285d add a new line
9abcf7a95c7dce7e3d2afebc0827ac845cca23a3 wrote a readme file change the user name
# git status
# 位于分支 master
无文件要提交，干净的工作区

```
查看工作区文件和master里最新版本的区别</br>
```
# git diff HEAD -- sayhello.py
diff --git a/sayhello.py b/sayhello.py
index b38f911..fae5a0e 100644
--- a/sayhello.py
+++ b/sayhello.py
@@ -1,3 +1,5 @@
 #! /usr/bin/python3
 
+
 print('Hello world!')
+print('git test!')

```
只有commit才会被提交到master，而add只是将修改从工作区添加到暂存区，没有add的修改不能直接被commit</br>
重复两次add后commit一次，则只提交最新的一次add</br>
