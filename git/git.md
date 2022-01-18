# Git 基本使用

## 分支创建及提交
```bash
# 创建分支
git checkout -b ***
# 将本地分支提交到远程分支
git push --set-upstream origin ***
```

## 删除分支
```bash
# 删除远程分支
git push origin --delete ***
# 删除本地分支 
git branch -d <BranchName>
```

## 查看分支

```bash
# 查看所有分支
git branch -a
```

## 删除远程文件

```bash
# 1. 加上 -n 这个参数，执行命令时，是不会删除任何文件，而是展示此命令要删除的文件列表预览
git rm -r -n --cached 文件/文件夹名称

# 2. 确定无误后删除文件
git rm -r --cached 文件/文件夹名称

# 3. 提交到本地并推送到远程服务器
git commit -m "提交说明"
git push origin master
```

## 冲突处理

```bash
git checkout master
git pull
git checkout 分支
git merge master
# ...处理冲突
git add .
git commit -m "说明"
git push
```

## 删除 | 添加 远程源

```bash
# 删除远程源
git remote remove origin
# 新增源
git remote add origin git@XXXX
```

