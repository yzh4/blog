## 创建版本库

```
git clone <url> #克隆远程版本库
git init          # 初始化本地版本库
```

## 修改和提交

```
git status                  # 查看状态
git diff                    # 查看变更内容
git add .                   # 跟踪所有改动过的内容,提交到缓存区
git add <file>              # 跟踪指定文件，提交到缓存区
git mv <oldfile> <newfile>  # 文件名修改
git rm <file>               # 删除暂存区或分支上的文件, 同时工作区也不需要这个文件了
git rm --cached <file>      # 需要删除暂存区或分支上的文件, 但本地又需要使用, 只是不希望这个文件被版本控制
git commit -m "commit message" # 提交所有更新过的文件
git commit --amend          # 修改最后一次提交
```

## 查看提交历史

```
git log              # 查看提交历史
git log -p <file>    # 查看指定文件的提交历史
git blame <file>     # 以列表的方式查看指定文件的提交历史
```

## 撤销

```
git reset --hard HEAD    # 撤销工作目录中所有未提交文件的修改内容
git checkout HEAD <file> # 撤销指定的未提交文件的修改内容
git revert <commit>      # 撤销指定的提交
```

## 分支和标签

```
git branch                   # 显示所有本地的分支
git checkout <branch/tag>    # 切换到指定分支或者标签
git branch <new-branch>      # 创建新分支
git branch -d <branch>       # 删除本地分支
git tag                      # 列出所有本地标签
git tag <tagname>            # 基于最新提交创建标签
git tag -d <tagname>         # 删除指定标签
```

## 合并

```
git merge <branch>           # 合并指定分支到当前分支
git rebase <branch>          # 衍合指定分支到当前分支
```

## 远程操作

```
git remote -v                         # 查看远程版本库信息
git remote show <remote>              # 查看指定远程版本库信息
git remote add <remote> <url>         # 添加远程版本库
git fetch <remote>                    # 拉取远程库代码
git pull <remote> <branch>            # 下载远程库代码及快速合并
git push <remote> <branch>            # 上传代码及快速合并
git push <remote> :<branch/tag-name>  # 删除远程分支或者标签
git push --tags                       # 上传所有标签
```

## 默认设置

```
master  # 默认开发分支
origin  # 默认远程版本库
HEAD    # 默认开发分会
HEAD^   # HEAD的父提交
```

