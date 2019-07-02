```
git config --global user.name "liu-dongyu"
git config --global user.email "liudongyu.mail@gmail.com"
git config -l  # 查看所有配置
```

```
# 缓存用户名密码
git config credential.helper 'cache --timeout=3600'
git config --global credential.helper store # 长期缓存
```

```
# clone特定分支
git clone -b <branch> <remote_repo>
```

```
git add <file> # 将工作文件修改提交到本地暂存区
git add . # 将所有修改过的工作文件提交暂存区
```

```
git rm <file> # 从版本库中删除文件
git rm <file> --cached # 从版本库中删除文件，但不删除文件
```

```
# 查看最近一次提交的内容
git show
```

```
# 查看工作去状态
git status
```

```
# 抛弃工作区
git checkout -- <file>
git checkout -- .
```

```
# 回退版本
git log # 查看版本 commit code
git reset --hard # 恢复最近一次提交过的状态
git reset --hard 871ffcd6d7b7b56366528cee098e33dfea9550fb
```

```
# 拉取更新
git pull origin <originName>
```

```
# 推送跟新
git push origin <originName>
```

```
# 拉取其他分支的更新，合并到自己的分支
git fetch origin <otherOriginName>
git rebase origin/<otherOriginName>
```

```
# 将pull操作默认为rebase
git config --global pull.rebase true
```

```
# 删除本地在远程分支中已不存在的追终分支并拉取所有远程分支
git fetch -p
```

```
# 分支管理
git checkout -b <originName> # 创建
git branch <originName> # 创建
git branch # 查看
git branch -a # 查看，包括追终分支
git branch -d # 删除
git branch -D # 强制删除
```

```
# 当手头工作没有完成时，先把工作现场git stash一下，然后去修复bug，修复后，再git stash pop，回到工作现场.
git stash
git stash pop
```

```
# 远程仓库
git remote -v # 查看
git remote rm <repository> # 删除远程仓库
git remote add origin <repositoryUrl> # 添加远程仓库地址
git remote set-url origin <repositoryUrl> # 设置远程仓库地址(用于修改远程仓库地址)
```

```
# 提交dist文件夹到gh-pages分支
git subtree push --prefix dist origin gh-pages
```

```
# 合并多个commit
1. git log --pretty=oneline
2. git rebase -i HEAD~2 # 最早的commit的head
3. 将除了第一个`pick`以外的改为`s`
4. 看提示操作，保存。(ps: 有冲突则先修改冲突后`git rebase --continue`)
5. git push origin ur_branch -f (-f谨慎操作)
```

```
# 将pull操作变成rebase
git config --global pull.rebase true
```
