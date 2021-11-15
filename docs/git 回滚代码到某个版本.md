### git 回滚代码到某个版本


目前项目是用git做版本控制，最近由于同事不小心把测试分支代码合并到线上，故项目需要回滚代码，总结如下：
```
回滚命令：
$ git reset --hard HEAD^         回退到上个版本
$ git reset --hard HEAD~3        回退到前3次提交之前，以此类推，回退到n次提交之前
$ git reset --hard commit_id     退到/进到 指定commit的sha码
```
```
强推到远程：
$ git push origin HEAD --force
```

做完以上操作其实远程分支已经是回滚成功了，但由于多人协作开发，有些同事本地分支存在之前版本代码，直接使用git pull只会做合并操作，是不会覆盖回滚本地分支版本库的。
有两种方式操作：

1，将本地分支强制删除，重新拉起远程最新分支既可以恢复版本代码操作，代码如下：
```
git checkout AAA //随便切换到一个正常分支
git branch -D XXX //删除本地分支
git branch -a  //查看分支
git fetch //更新下远程跟踪分支
git pull origin XXX //重新拉取
git chekcout XXX //切换到分支
```

2，网上还教人一种简单方式：
```
git fetch --all
git reset --hard origin/XXX 把HEAD指向重新重置为远端最新的版本
```


备忘一：删除本地分支和远程分支：
```
git branch -d test
git push origin --delete test
```



[Git如何回滚一次错误的合并][1]
```
git revert -m 1 f57d2306289beada7f45d7221c0a8c014e96371a
恢复之前的回滚
git checkout master
git revert rev3
git merge dev
```



  [1]: https://juejin.im/post/5b5ab8136fb9a04f834659ba