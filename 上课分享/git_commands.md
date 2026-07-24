# Git 常用命令总结

> 整理自 `git.pptx` 培训课件，按功能分类汇总。

## 一、Git 全局设置

安装 Git 后首先要设置用户名称和邮箱（每次提交都会使用该用户信息）。

```bash
# 设置用户信息
git config --global user.name "longxf"
git config --global user.email "hblongxiaofeng@163.com"

# 查看配置信息
git config --list
```

> 注意：`user.name` 和 `user.email` 并不是注册码云账号时的用户名和邮箱，此处可任意设置。

## 二、获取 Git 本地仓库

```bash
# 在本地初始化一个 Git 仓库（当前目录生成 .git 隐藏文件夹）
git init

# 从远程仓库克隆到本地
git clone [远程Git仓库地址]
```

## 三、工作区、暂存区、版本库、本地仓库 概念

- **版本库**：`.git` 隐藏文件夹，存储配置、日志、文件版本等信息
- **工作区**：包含 `.git` 的目录（不含 `.git` 本身），存放开发代码
- **暂存区**：`.git` 中的 `index` 文件（也叫 stage），临时保存修改的文件
- **本地仓库**：版本库 + 工作区

## 四、Git 工作区中文件的状态

- **untracked**：未跟踪（未被纳入版本控制）
- **tracked**：已跟踪（被纳入版本控制）
  - Unmodified：未修改状态
  - Modified：已修改状态
  - Staged：已暂存状态

> 文件状态会随着执行的 Git 命令发生变化。

## 五、本地仓库操作

```bash
git status                          # 查看文件状态
git add 文件名/目录名               # 将文件的修改加入暂存区
git reset                           # 将暂存区的文件取消暂存，或切换到指定版本
git commit file1,file2 ... -m 提交信息   # 将暂存区修改提交到版本库（不指定文件名则提交全部）
git log                             # 查看日志
```

## 六、远程仓库操作

```bash
git remote                          # 查看远程仓库（克隆的仓库默认别名 origin）
git remote add <shortname> <url>    # 添加远程仓库并指定别名
git remote remove <shortname>       # 删除远程仓库
git clone [url]                     # 从远程仓库克隆
git push [remote-name] [local-branch-name]   # 推送到远程仓库（默认 remote=origin）
git pull [short-name] [branch-name]          # 从远程仓库拉取并合并（默认 short-name=origin）

# 本地仓库非克隆而来且存在文件时，拉取可能报错：
#   fatal: refusing to merge unrelated histories
# 解决办法：加入参数允许合并不相干历史
git pull --allow-unrelated-histories
```

## 七、分支操作

默认分支为 `master`（通过 `git init` 创建本地仓库时自动创建）。

```bash
git branch                          # 列出所有本地分支
git branch -r                       # 列出所有远程分支
git branch -a                       # 列出所有本地分支和远程分支
git branch [name]                   # 创建分支
git checkout [name]                 # 切换分支
git push [shortName] [local_branch_name]   # 推送至远程仓库分支
git merge [name]                    # 将指定 name 的分支合并到当前分支
```

## 八、标签操作

标签指某个分支在特定时间点的状态，常用于标记发布结点（如 v1.0、v1.2）。

```bash
git tag                             # 列出已有的标签
git tag [name]                      # 创建标签
git push [shortName] [local_tag_name]   # 将标签推送至远程仓库
git checkout -b [branch] [tag name]     # 检出标签到新建分支中
```

## 附：命令速查表

| 分类 | 命令 | 说明 |
|------|------|------|
| 全局设置 | `git config --global user.name "xxx"` | 设置用户名 |
| 全局设置 | `git config --global user.email "xxx"` | 设置邮箱 |
| 全局设置 | `git config --list` | 查看配置 |
| 仓库 | `git init` | 初始化本地仓库 |
| 仓库 | `git clone [url]` | 克隆远程仓库 |
| 本地 | `git status` | 查看文件状态 |
| 本地 | `git add <file/dir>` | 加入暂存区 |
| 本地 | `git reset` | 取消暂存 / 切换版本 |
| 本地 | `git commit -m "msg"` | 提交到版本库 |
| 本地 | `git log` | 查看日志 |
| 远程 | `git remote` | 查看远程仓库 |
| 远程 | `git remote add <name> <url>` | 添加远程仓库 |
| 远程 | `git remote remove <name>` | 删除远程仓库 |
| 远程 | `git push <remote> <branch>` | 推送到远程 |
| 远程 | `git pull <remote> <branch>` | 拉取并合并 |
| 分支 | `git branch [name]` | 查看/创建分支 |
| 分支 | `git checkout [name]` | 切换分支 |
| 分支 | `git merge [name]` | 合并分支 |
| 标签 | `git tag [name]` | 查看/创建标签 |
| 标签 | `git checkout -b <branch> <tag>` | 检出标签到新分支 |
