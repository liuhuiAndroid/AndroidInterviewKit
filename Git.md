#### merge 和 rebase 有什么区别



#### 如何提交 pull request

- 在新的分支进行开发
- 定期将本地分支的更新推送到远程仓库的同名分支
- 定期从远程主分支拉取最新代码到本地，以保持更新
- 在GitHub或其他托管服务上创建一个Pull Request，请求将你的分支合并到主分支。
- 在PR中清楚地描述所做的更改。

[如何从本地把项目上传到github&&如何把github项目通过clone复制下来，详细教程](http://www.cnblogs.com/chengxs/p/6297659.html)

#### 初始化仓库

```
git init
git add README.md
git commit -m "first commit"
git remote add origin https://github.com/liuhuiAndroid/CustomView.git
git push -u origin master
```

#### 切换到tag

```
git checkout v3.8.0
git checkout -b branch tag // 创建一个基于指定tag的分支

git checkout -b code-analysis v3.8.0 //例子
git push origin code-analysis:code-analysis //提交本地分支到远程分支，没有则创建
```

#### 删除本地分支 

```
git branch -D br 
```

#### 切换分支 

```
git checkout master
```

#### 查看远程分支地址

```
 git remote -v
```

#### 忽略文件

.gitignore

#### 设置大小写敏感

```
git config core.ignorecase false  
```

```
在本地新建一个分支： git branch index-swiper
切换到你的新分支: git checkout index-swiper
将新分支发布在github上： git push origin index-swiper
在本地删除一个分支： git branch -d index-swiper
在github远程端删除一个分支： git push origin :index-swiper(分支名前的冒号代表删除)

合并index-swiper分支到主分支：
git checkout master
git merge origin/index-swiper
git push
```

#### 修改commit

git commit --amend

#### 常见错误

windows 使用git 需要在环境变量配置：/git安装目录/bin/，否则idea中git无法使用

#### 提交回滚

想fork一个github上的项目，然后回滚到某个简单的版本然后提交到我的远程分支。

git reset --hard c146cb0
git commit -m "revert to version 2.3.0"
git push -u origin master -f    强制提交

#### 如何将多个本地 commit 合并

[参考：git如何使用SourceTree合并多次本地提交记录](https://www.jb51.net/program/286044a27.htm)

1. 选择上一次的远程推送 -> 右键 -> 交互式变基xxx的子提交

2. 选择：用以前的提交来squash

   在软件开发中，"squash" 是一个版本控制的术语，通常用于合并和整理提交历史记录。具体来说，"用以前的提交来squash" 意味着将多个先前的提交（也称为"commit"）合并为一个单一的提交，以减少提交历史中的混乱和冗余。

3. 选择：编辑信息，并提交

4. 推送

#### SourceTree 修改 commit 信息



