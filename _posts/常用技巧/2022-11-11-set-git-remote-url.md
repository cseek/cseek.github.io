---
title: 修改现有仓库的远程地址
category: [常用技巧]
tags: [git]
---

> 有时候需要迁移现有的代码仓库，如果重新创建新仓库，删除掉原来的 .git 目录再推送代码上去，原有的 commit 信息将全部丢失，使用下面这种方法就可以完整保留原有的 commit 信息。
{: .prompt-info }

## 进入本地仓库目录
```bash
cd /path/to/your/local/repo
```

## 查看当前远程地址

```bash
git remote -v
```

## 修改远程地址

```bash
git remote set-url origin git@ip:your_repo.git
```

## 强制推送所有分支和标签

```bash
git push --force --all origin
git push --force --tags origin
```

## 注意
请确保将要设置的 URL 所在的远程仓库为空，否则会被覆盖掉。