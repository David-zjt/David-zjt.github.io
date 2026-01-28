# git配置

## git基本概念

用于代码版本控制，基本流程包括：

### 工作区

基本git仓库，暂存区，提交记录

### 基本流程

git暂存，git提交，远程仓库同步

## git基本操作

### 基本设置

1. 基本信息设置
 - 全局设置修改：
    - git config --global user.name "hello"
    - git config --global user.email "aa.mail.com"
 - 本地设置修改：(只修改仓库内部保存的基本信息)
    - git config --local user.name "hello"
    - git config --local user.email "aa.mail.com"

2. 设置git内部编辑器
 - git config --global core.editor "vim"

### 查看本地git仓库信息

基础命令：git status

作用：反馈仓库状态，当前跟踪的分支，跟踪的文件有变更，未被跟踪的文件。

其他：

1. 查看git跟踪的所有文件
```shell
git ls-files
```
2. git跟踪的文件，工作区删除，暂存区还保留：
```shell
git lss-files -deleted
```
3. git跟踪的文件，工作区修改，未暂存：
```shell
git ls-files -modified
```

### git 暂存
1. 基础命令
```shell
git add <dir or file>
```
```shell
git add . 
```
2. 自动提交暂存区
 - 已跟踪文件自动提交：
```shell
git add -u
```
 - 交互式自动提交，可选提交内容：
```shell
git add -p
```
3. 删除暂存区，恢复到最近提交状态：
```shell
git reset --mixed HEAD
git reset
```
git reset命令移动HEAD指针，并可选保留工作区和暂存区：
 - soft：仅移动 HEAD（保留暂存区和工作区）
 - mixed（默认）：重置暂存区（保留工作区）
 - hard：重置暂存区和工作区（丢弃所有修改）

4. 专用恢复暂存区命令：
```shell
git restore --staged <file>
git restore --staged .
```

5. 撤销工作区，恢复至暂存区：
```shell
git restore <file>
git restore .
```

6. 对比工作区和暂存区
```shell
git diff
```

7. 清理未跟踪的文件
 - 删除所有未跟踪的文件目录，直接删除，不可逆
```shell
git clean -fd
```
 - 预览哪些文件会被删除
```shell
git clean -n
```
 - 模拟运行清理的结果
```shell
git clean -fd --dry-run
```

### git 分支管理

#### 分支基本操作
1. 查询分支基本情况
```shell
git branch
```
2. 创建新分支
```shell
git branch new-branch
```
3. 切换新分支
```shell
git checkout new-branch
git switch new-branch
```
4. 创建分支并切换
```shell
git checkout -b new-branch
```

#### 分支的合并
分支合并有两种，基于git merge 以及基于git rebase：

1. git merge 基本流程
 - 切换到目标分支
 - git merge \<src\>
    - 无冲突：快速合并
    - 有冲突：需要手动合并冲突
    - 验证合并冲突结果：
```shell
git diff
git status
```





### 远程仓库管理

推送所有分支到远程仓库origin
```shell
git push origin -all
```

#### 远程仓库的分支绑定
1. 默认绑定规则
 - git clone默认绑定
 - 新建本地分支：默认不绑定

2. 推送时自动绑定
```shell
git push -u origin branch:branch2
```

3. 拉取时自动绑定
```shell
git checkout --track origin/\<branch\>
git checkougt -t origin/\<branch\>
```

4. 拉去后手动绑定
 - 拉去远程分支
```shell
git fetch origin <origin_branch>/<local_branch>
```

 - 手动绑定
```shell
git branch --set-upstream-to=origin/<origin_branch> <local_branch>
```

5. 直接手动绑定
```shell
git branch --set-upstream-to=origin/<origin_branch> <local_branch>
```


#### 远程分支同步

1. 下载所有remote的所有分支，不合并
```shell
git fetch -all
```

2. 仅下载origin一个远程的所有分支，不合并
```shell
git fetch origin
```

3. 查看所有远程分支
```shell
git branch -r
```

4. 查看本地+远程所有分支
```shell
git branch -a
```

5. 创建本地同名分支并切换
```shell
git checkout -b dev origin/dev
```

6. 查看远程分支代码，不创建本地分支
```shell
git chekcout origin/dev
```

7. 获取所有标签
```shell
git fetch origin -tags
```

#### 注意事项

1. 分支名称不一致，需要显示指定

```shell
git push origin main:dev
```

2. 解除绑定
```shell
git branch --unset-upstream
```

3. 查看绑定状态
```shell
git branch -vv
```


### git 标签
1. 创建标签

```shell
git tag <tag_name>
```

2. 附注标签
```shell
git tag <tag_name> -m "message"
```

3. 历史提交打标签
```shell
git tag <tag_name> <commit_hash>
```

4. 推送单个标签到远程仓库
```shell
git push origin <tag_name>
```

5. 推送所有本地标签
```shell
git push origin --tags
```

6. 删除本地标签
```shell
git tag -d <tag_name>
```

7. 删除远程标签
```shell
git push origin :refs/tags/<tag_name>
```
