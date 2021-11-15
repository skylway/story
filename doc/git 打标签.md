## git 打标签

## 一、如何用 git 打 tag

> 版本号格式：X.Y.Z（X主版本号，Y次版本号，Z补丁版本号）
>比如我们用的 Go 语言，目前是 1.13.10。它还是 Go 1，每次升级都保证是兼容的，13的版本号是新 feature，而最末尾的版本号是修复。如果末尾的版本号为0，说明当前的版本上了之后还没有修复过问题。


### 1、创建 v1 分支

```
$ git checkout -b v1
$ git push -u origin v1
```

### 2、在 v1 分支修改代码（调试阶段）

```
$ git commit -m "update testmod"
$ git tag v1.0.0-beta
$ git push --tags origin v1
```

### 3、v1 分支修改正式封包后

```
$ git commit -m "update testmod"
$ git tag v1.0.0
$ git push --tags origin v1
```

### 4、其它记录
```
$ git tag                 #查看本地的tag
$ git tag -a release-1.0.0 -m 'goee全链路监控系统第一个版本'   #在本地创建一个tag，并备注
$ git tag -d release-1.0.0          #删除本地tag
$ git show release-1.0.0         #查看tag的详细信息，可以看到tag是根据哪个commit打出来的

```