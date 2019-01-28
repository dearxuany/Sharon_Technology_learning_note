# github 远程仓库错误及解决办法

## git pull 无响应
```
$ git pull origin master:master
ssh: connect to host github.com port 22: Connection timed out
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.

```
大概是ssh有问题，连不上服务器，重新设置一下。</br>
删除 ~/.ssh 重新按步骤设置ssh，结果还是不行。</br>
最后换了个网络，重新clone了目录新建了分支才能 pull，不清楚到底是哪一步解决了问题...</br>
