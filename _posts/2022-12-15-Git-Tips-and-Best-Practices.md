---
layout: post
title: "Git 常用技巧与最佳实践"
date: 2022-12-15 00:00:00
author: ingerchao
toc: true
category: blog
tag:
- Git
- 版本控制
- 开发工具
- 协作开发
---

## 前言

Git 是目前最流行的分布式版本控制系统，掌握 Git 的高级用法可以大大提高开发效率。本文将分享一些实用的 Git 技巧和最佳实践。

## 一、提交历史管理

### 1. 合并特定提交（cherry-pick）

当需要将某个特定的提交合并到当前分支时，使用 `cherry-pick`：

```bash
git cherry-pick <commit_id>
```

**使用场景：**
- 紧急修复需要应用到多个分支
- 选择性合并某些功能
- 回滚时保留部分提交

**注意事项：**
- 合并操作无法撤销，只能在本地找到未合并的版本，然后强制提交
- 如果遇到冲突，需要手动解决

### 2. 交互式 rebase 删除提交

当需要从提交历史中删除某个提交时：

```bash
# 启动交互式 rebase
git rebase -i <COMMIT1的hash>

# 在编辑器中将要删除的提交的 pick 改为 drop
# 保存退出后，强制推送到远程
git push --force
```

**使用场景：**
- 删除错误的提交
- 清理提交历史
- 移除敏感信息

**注意事项：**
- 强制推送会改变提交历史，团队协作时需谨慎使用
- 其他人再提交时，被 drop 的 commit 可能还会存在

### 3. 从某次提交创建新分支

当需要从某个历史提交创建新分支时：

```bash
git checkout <commit_id> -b <branch_name>
```

**使用场景：**
- 基于历史版本开发新功能
- 修复历史版本的 bug
- 创建发布分支

## 二、远程仓库管理

### 1. 查看和配置远程仓库

```bash
# 查看本仓库各分支的远程映射
git remote -v

# 为本地分支添加远程映射
git remote add <local_branch> <remote_url>

# 移除远程仓库的映射
git remote remove <name>

# 修改远程仓库的简写名
git remote rename <old_name> <new_name>
```

**实际应用示例：**

```bash
# 查看当前远程仓库配置
$ git remote -v
opt_inger       https://github.com/Inger-Chao/zookeeper_paper_cn.git (fetch)
opt_inger       https://github.com/Inger-Chao/zookeeper_paper_cn.git (push)
origin  https://github.com/mapleFU/zookeeper_paper_cn.git (fetch)
origin  https://github.com/mapleFU/zookeeper_paper_cn.git (push)

# 移除 origin 远程仓库
$ git remote remove origin

# 查看更新后的配置
$ git remote -v
opt_inger       https://github.com/Inger-Chao/zookeeper_paper_cn.git (fetch)
opt_inger       https://github.com/Inger-Chao/zookeeper_paper_cn.git (push)
```

### 2. 更换远程仓库 URL

当远程仓库地址发生变化时：

```bash
git remote set-url origin <new_url>
```

**使用场景：**
- 仓库迁移到新平台
- 更换仓库地址
- 修复错误的远程 URL

## 三、分支管理

### 1. 删除分支

**删除本地分支：**
```bash
git branch -d <local_branch_name>
```

**删除远程分支：**
```bash
git push origin --delete <remote_branch_name>
```

**强制删除本地分支（未合并）：**
```bash
git branch -D <local_branch_name>
```

**最佳实践：**
- 及时清理已合并的分支
- 使用 `-d` 而非 `-D`，避免误删
- 删除远程分支前确认团队成员已同步

### 2. 本地与远程同步

当本地分支与远程分支出现分歧时：

```bash
# 获取远程最新代码
git fetch --all

# 重置本地分支到远程状态（危险操作）
git reset --hard origin/master

# 拉取远程更新
git pull
```

**警告：**
- `git reset --hard` 会丢弃本地未提交的更改
- 执行前确保本地更改已备份或提交
- 团队协作时避免使用

## 四、文件恢复

### 1. 恢复被删除的文件

当误删文件后，恢复到上次提交的状态：

```bash
# 恢复文件到暂存区
git reset HEAD <被删除的文件或文件夹>

# 从暂存区恢复到工作区
git checkout <被删除的文件或文件夹>
```

**简化写法（Git 2.23+）：**
```bash
git restore <file>
```

### 2. 恢复修改的文件

当文件被修改但未提交时：

```bash
# 恢复单个文件
git checkout <file>

# 恢复所有修改
git checkout .
```

**简化写法（Git 2.23+）：**
```bash
git restore <file>
git restore .
```

## 五、个人配置管理

### 1. 本地文件不上传到仓库

当某些文件只需要本地使用，不上传到代码仓库时：

```bash
# 在 .git/info/exclude 中添加规则
echo "*.puml" >> .git/info/exclude
echo "local-config.json" >> .git/info/exclude
```

**与 .gitignore 的区别：**
- `.gitignore` 会提交到仓库，影响所有开发者
- `.git/info/exclude` 仅本地生效，不影响其他开发者

**使用场景：**
- 本地 IDE 配置
- 个人脚本文件
- 临时测试文件
- 本地环境变量文件

### 2. 配置用户信息

```bash
# 全局配置
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# 仓库级配置
git config user.name "Your Name"
git config user.email "your.email@example.com"
```

## 六、高级技巧

### 1. 暂存工作区（stash）

当需要临时切换分支，但不想提交当前工作时：

```bash
# 暂存当前工作
git stash

# 暂存并添加消息
git stash save "正在开发的功能"

# 查看暂存列表
git stash list

# 恢复最近的暂存
git stash pop

# 恢复指定的暂存
git stash apply stash@{0}

# 删除暂存
git stash drop stash@{0}

# 清空所有暂存
git stash clear
```

**使用场景：**
- 临时修复紧急 bug
- 切换分支测试
- 代码审查时暂存更改

### 2. 查看提交历史

```bash
# 查看完整历史
git log

# 查看简洁历史
git log --oneline

# 查看最近 N 次提交
git log -n 5

# 图形化查看历史
git log --graph

# 查看某个文件的提交历史
git log -p <file>

# 查看某个时间段的提交
git log --since="2022-01-01" --until="2022-12-31"
```

### 3. 查看文件差异

```bash
# 查看工作区与暂存区的差异
git diff

# 查看暂存区与最近提交的差异
git diff --cached

# 查看两个提交的差异
git diff <commit1> <commit2>

# 查看两个分支的差异
git diff <branch1> <branch2>

# 查看某个文件的差异
git diff <file>
```

### 4. 标签管理

```bash
# 创建轻量标签
git tag <tag_name>

# 创建附注标签
git tag -a <tag_name> -m "标签说明"

# 创建特定提交的标签
git tag <tag_name> <commit_id>

# 查看所有标签
git tag

# 删除本地标签
git tag -d <tag_name>

# 删除远程标签
git push origin --delete <tag_name>

# 推送标签到远程
git push origin <tag_name>

# 推送所有标签
git push origin --tags
```

## 七、最佳实践

### 1. 提交规范

**提交信息格式：**
```
<type>(<scope>): <subject>

<body>

<footer>
```

**Type 类型：**
- `feat`: 新功能
- `fix`: 修复 bug
- `docs`: 文档更新
- `style`: 代码格式调整
- `refactor`: 重构
- `test`: 测试相关
- `chore`: 构建/工具更新

**示例：**
```
feat(user): 添加用户登录功能

- 实现用户名密码登录
- 添加验证码校验
- 集成 JWT 认证

Closes #123
```

### 2. 分支策略

**Git Flow 工作流：**
- `master`: 主分支，始终保持稳定
- `develop`: 开发分支
- `feature/*`: 功能分支
- `release/*`: 发布分支
- `hotfix/*`: 紧急修复分支

**GitHub Flow 工作流：**
- `main`: 主分支
- 所有开发在功能分支进行
- 通过 Pull Request 合并到主分支

### 3. 协作规范

**代码审查：**
- 所有合并必须经过 Pull Request
- 至少一个审查者批准
- 确保 CI 通过

**冲突解决：**
- 及时更新本地分支
- 小步提交，减少冲突
- 冲突时优先沟通

**版本发布：**
- 使用语义化版本号
- 创建详细的发布说明
- 打标签记录版本

## 八、常见问题解决

### 问题一：合并冲突

**解决步骤：**
```bash
# 1. 查看冲突文件
git status

# 2. 手动编辑冲突文件，解决冲突

# 3. 添加已解决的文件
git add <file>

# 4. 提交合并
git commit
```

### 问题二：提交错误需要回滚

**回滚到上一个提交：**
```bash
# 保留更改到工作区
git reset --soft HEAD~1

# 保留更改到暂存区
git reset --mixed HEAD~1

# 完全丢弃更改（危险）
git reset --hard HEAD~1
```

### 问题三：误提交了大文件

**解决步骤：**
```bash
# 1. 从历史中删除文件
git filter-branch --force --index-filter \
  "git rm --cached --ignore-unmatch <large_file>" \
  --prune-empty --tag-name-filter cat -- --all

# 2. 强制推送
git push --force
```

## 总结

掌握 Git 的高级用法可以大大提高开发效率和团队协作质量。关键要点：

1. **提交历史管理**：使用 cherry-pick、rebase 灵活管理提交
2. **远程仓库管理**：熟练配置多个远程仓库
3. **分支管理**：合理使用分支策略
4. **文件恢复**：掌握各种恢复技巧
5. **个人配置**：区分全局和本地配置
6. **最佳实践**：遵循提交规范和协作流程

记住：**Git 是一个强大的工具，但 power comes with responsibility**。在团队协作中，谨慎使用强制操作，及时沟通，避免造成不必要的麻烦。
