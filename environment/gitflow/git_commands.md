# git常用操作与命令

### 基于远程分支创建本地分支

```create
git checkout -b branch/name remotes/origin/develop 
```

### 将本地分支推送到远程

```push
git push --set-upstream origin branch/name
```

### 切换分支

```checkout
git checkout branch/name
```

### 删除本地分支

```deletelocal
git branch -D branch/name
```

### 删除远程分支

```delete
git push origin --delete branch/name
```

### 更新远程分支列表

```updatebranchs
git remote update origin --prune
```