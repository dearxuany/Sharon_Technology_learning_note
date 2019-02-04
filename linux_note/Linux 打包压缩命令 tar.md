# Linux 打包压缩命令 tar
## tar 语法
```
tar [-z|-j|-J] [cv] [-f 新建的文件名.扩展名] filename   打包压缩，但不会删除源文件
tar [-z|-j|-J] [tv] [-f 已被打包压缩的文件名]           查看已打包压缩的文件里面的文件名
tar [-z|-j|-J] [xv] [-f 已被打包压缩的文件名] [-C 目录] 解压
```
选项与参数：
```
-c  ：建立打包档案，可搭配 -v 来察看过程中被打包的档名(filename)
-t  ：察看打包档案的内容含有哪些档名，重点在察看『档名』就是了；
-x  ：解打包或解压缩的功能，可以搭配 -C (大写) 在特定目录解开
      特别留意的是， -c, -t, -x 不可同时出现在一串指令列中
      
-z  ： gzip  *.tar.gz
-j  ： bzip2 *.tar.bz2
-J  ： xz    *.tar.xz
      特别留意， -z, -j, -J 不可以同时出现在一串指令列中
      
-v  ：在压缩/解压缩的过程中，将正在处理的档名显示出来！
-f filename：-f 后面要立刻接要被处理的档名！建议 -f 单独写一个选项囉！(比较不会忘记)
-C 目录    ：这个选项用在解压缩，若要在特定目录解压缩，可以使用这个选项

-p(小写) ：保留备份资料的原本权限与属性，常用于备份(-c)重要的设定档
-P(大写) ：保留绝对路径，亦即允许备份资料中含有根目录存在之意；(保留的话解压时会覆盖掉同名源文件的数据)
--exclude=FILE：在压缩的过程中，不要将 FILE 打包！
```

## 打包压缩
常用：打包压缩文件到指定目录（指定目录直接写在 *.tar.gz 文件名前面）
```
$ pwd
/home/sunnylinux/test/tartest
$ ls
tar_file  tar_test_file

# gz格式，打包当前目录中的tar_test_file到tar_file目录下，含源目录信息
# tar -czf /目的路径/文件名.tar.gz /原路径/文件名
$ tar -czf /home/sunnylinux/test/tartest/tar_file/tar_test_file.tar.gz ./tar_test_file  

# 打包指定目录/home/sunnylinux/pythontest下的 python3_script 目录，到./tar_file/，不含源目录信息
# tar -czf /目的路径/文件名.tar.gz -C /源路径 文件名
$ tar -czf ./tar_file/python_script.tar.gz -C /home/sunnylinux/pythontest python3_script 
```
仅打包，不压缩  
```
tar -cv -f /tmp/etc.tar /etc
```
一次性打包到移动磁盘
```
tar -cv -f /dev/st0 /home /root /etc
```
打包压缩 tarball （不删除源文件，注意是否去除根目录，是否是绝对路径）
```
time tar -zpcv -f ./test.tar.gz ./test  # 利用gzip压缩，计时
time tar -jpcv -f ./test.tar.bz2 ./test # 利用bzip2压缩，计时
time tar -Jpcv -f ./test.tar.xz ./test # 利用xz压缩。计时
```
查看已打包压缩的文件中的文件名（注意是否存在根目录）
```
tar -tvz -f ./test.tar.gz
tar -tvj -f ./test.tar.bz2
tar -tvJ -f ./test.tar.xz
```
压缩某一目录下的文件，并排除某些不需要的文件
```
tar -jvc -f testexcludetar.tar.bz2 test --exclude=*.txt  # 压缩test目录并排除txt格式文件
tar -jvc -f testexcludetar2.tar.bz2 test --exclude=cptest # 压缩test目录并排除子目录cptest下的文件（不要打成/cptest会出错）
```
压缩备份比某时刻新的文件
```
tar -jvc -f testnewertar.tar.bz2 --newer-mtime="2018/03/06" test # 压缩mtime比2018/03/06新的文件，not dumped表示没有被备份的文件
```
## 解压
常见：解压到目标目录
```
tar -xvz -f ./test.tar.gz -C 目标目录 
tar -xvj -f ./test.tar.bz2 -C 目标目录 
tar -xvJ -f ./test.tar.xz -C 目标目录 

[sunnylinux@centOSlearning tartest]$ tar -xvzf ./tar_file/python_script.tar.gz -C ./tar_test_file
```
解压到本目录
```
# 不推荐，解压到指定目录(解压后原压缩文件不会被删除)，要注意路径问题

[sunnylinux@centOSlearning tartest]$ ls
tar_file  tar_test_file  tar_test_file.tar.gz
[sunnylinux@centOSlearning tartest]$ tar -xvzf ./tar_test_file.tar.gz
./tar_test_file/
./tar_test_file/tar_test.txt
[sunnylinux@centOSlearning tartest]$ ls
tar_file  tar_test_file  tar_test_file.tar.gz

[sunnylinux@centOSlearning tartest]$ tar -xvzf ./tar_file/python_script.tar.gz
[sunnylinux@centOSlearning tartest]$ ls
python3_script  tar_file  tar_test_file  tar_test_file.tar.gz
```
仅解压其中一个文件
```
# 先搜索关键字（查看压缩包中是否存在该文件）
tar -jtv -f ./test3.tar.bz2 | grep 'mvtest'  # 'mvtest' 为要单一打开的文件名

# 解压该文件 
tar -jxv -f 打包档.tar.bz2 待解开档名
tar -jxv -f ./test3.tar.bz2 test/mvtest
```
