git提交类型：
![[Pasted image 20250720195459.png]]
### <font color=red>同步当前代码到私有仓库</font>：

- **创建本地分支**    git checkout -b  <分支名>
- 切换分支

- **拉远程分支代码并变基** git pull --rebase <远程仓库名> <远程分支名>
- **推送当前分支代码到远程仓库**    git push <远程仓库> <本地分支>:<远程分支>

- **换回开发分支继续开发**


<mark>工作流</mark>
```bash
git fetch origin       # 获取远程更新
git log main..origin/main  # 查看差异
# 确认后再合并
git merge origin/main  # 合并到本地分支
```
### <font color=red>分支合并的三种情况</font>
背景：我们想要合并两个分支时，会遇到的情况，我们具体情况具体分析。
#### ==***情况一：共同祖先，但只有一个修改***==
两个分支，都有共同祖先，且只有一个分支修改了，也就是这个分支有新的提交。
<mark>合并之前：</mark>
![两个分支的提交记录](https://i-blog.csdnimg.cn/direct/562b740b4510413e9a3b88504891669a.png)
**将feature合并到main上的步骤：**
- 先切换到main分支上，git checkout main
- 直接git merge feature
- git log --oneline --graph 可以查看合并结果

<mark>合并之后：</mark>
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/98ebb191d507466cacddf45c59fd7ed8.png)


#### ==***情况二：共同祖先，但都修改***==
两个分支是共同祖先，但都有新的提交
<mark>合并之前：</mark>
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/32d9074e964845058f36fc73146cf4e2.png)

##### <font color=blue>1. 使用merge合并</font>
<mark>合并步骤：</mark>
- 切换到目标分支main
- git merge feature
- 解决冲突
- 合并冲突提交
- 查看结果

<mark>合并之后：</mark>

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/82ef07cccba24cc980b9b962f7af7f8e.png)
##### <font color=blue>2. rebase变基合并</font>：
git rebase:将一个分支的提交 “复制” 到另一个分支的末尾，形成线性提交历史，而非创建合并提交。
<mark>合并步骤：</mark>
```bash
# 1. 切换到需要被合并的分支（如 feature）
git checkout feature

# 2. 将 feature 分支的提交“变基”到 main 分支上
git rebase main

# 3. 若有冲突，解决冲突后继续变基
git add .
git rebase --continue

# 4. 切换回 main 分支
git checkout main

# 5. 执行快进合并（此时可以快进，因为 feature 已基于 main）
git merge feature
```
<mark>合并之后：</mark>
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/c4c4d4f8cb594fa9b6e583eeea76f9e7.png)

##### merge和rebase区别：
- merge保留原始的提交记录，哈希值不变；
- 而rebase是复制旧的提交历史，生成新的提交历史，虽然提交说明不变，但是哈希值改变，且旧的会在本地和远程分支里都删除

- merge适合多人协作的公共分支（如 `feature`），重视历史完整性；
- rebase适合个人分支向公共分支合入，追求历史整洁性

- merge会生成合并提交记录
- rebase不会有合并提交记录

<font color=red>注意：</font>**禁止对已推送到远程的公共分支执行 `rebase`**


***对于两个分支main 和feature***
<mark>变基前：</mark>
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/6eb39219e9fa4a35957211fbc2bb5f1b.png)

<mark>变基后：</mark>
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/d548553272f643baa0f1d968b5661e88.png)

处理冲突后，继续变基：
```bash
git rebase --continue  # 继续应用剩余的提交（如 `E'`）
```
- **变基方向**：将当前分支变基到目标分支上（目标分支不变，当前分支被修改）。
- **冲突处理**：解决冲突后，使用 `git rebase --continue` 继续变基，直到所有提交应用完毕。


#### ==***情况三：完全没有共同祖先***==
两个完全不想关分支合并

<mark>合并之前：</mark>
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/e0963379d02d4df2b1d39f1dd3b37094.png)

<mark>合并步骤：</mark>
- 切换目标分支
- 强行合并：git merge feature --allow-unrelated-histories
- 解决冲突后提交
- 查看结果

<mark>合并之后：</mark>
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/c866e165a4c44a9a8fe09def5a309e45.png)
#### 快速查看冲突文件
git status # 显示包含冲突的文件


### <font color=red>gitlab等远程仓库上面的合并请求</font>
它和上面三种合并方式的异同：
<mark>相同之处：</mark>
1. 都是为了合并两个分支，进行代码整合
<mark>不同之处：</mark>
2.  合并请求，是一种更规范化的流程，方便项目管理，代码审查而上面三种合并方式是一种具体算法，更关注是如何处理提交记录与代码内容
3. gitlab可以选择不同方式进行合并


### <font color=red>在gitlab上创建合并请求之前</font>
**需要进行的操作**
1. 将远程dev分支拉取到本地dev后
2. 将dev分支合并到个人分支
3. 如果有冲突，就手动解决，并再次提交合并记录
4. 推送个人分支到远程关联的个人分支
5. 在gitlab上创建合并请求，请求将远程个人分支与dev分支合并

### <font color=red>查看历史提交结果：</font>
```bash
git log --oneline --graph

# 查看带分支结构的历史
git log --all --graph --oneline --decorate

# 查看详细的合并提交信息
git log -m -1 --name-only 合并提交哈希
```

### <font color=red>如何减少提交次数：</font>
1. ==本次提交合并到上次提交记录上==
```bash
git commit --amend --no-edit  # 合并到上一个提交，不修改提交信息
```

2. ==对于已经提交但尚未推送到远程的分支，可以使用 **交互式变基** 合并多个提交为一个：==
```bash
# 压缩最近的 3 个提交（数字根据你的需求调整）
git rebase -i HEAD~3
```
在弹出的编辑器中，将需要合并的提交前的 `pick` 改为 `squash` 或 `s`，然后保存退出：
```plaintext
pick 123abc 第一个提交
squash 456def 第二个提交（合并到第一个）
squash 789ghi 第三个提交（合并到第一个）
```

### <font color=red>git fetch和git pull的区别：</font>
- git fetch 可以获取远程仓库最新分支和最新代码到本地，但是不会将远程分支与本地分支自动合并。本地分支不会被修改，可以动过git diff 查看本地分钟与远程分支的差异，可自行决定是否要进行代码合并。
- 而git pull 是获取远程仓库最新分支和最新代码后，自动与本地分支进行合并，相当于git fetch+git merge，它的核心是同步远程并合并，快速合并代码。

### <font color=red>其他常用命令：</font>
```bash
#变基命令
git rebase --abort  # 放弃当前变基，回到变基前的状态
git rebase --skip  # 跳过当前冲突的提交，继续变基下一个
git status  # 显示当前变基状态（如“Rebasing feature onto main”）
git merge --ff-only feature  # 强制快进合并，若无法快进则报错

# 分支
git checkout -b <本地分支名> <远程仓库名>/<远程分支名> # 创建远程分支并关联远程分支
git branch -d 分支名 # 删除本地分支
git branch -D 分支名  #强制删除本地分支
git push origin --delete 分支名 #删除远程分支
git branch --merged     # 查看已合并到当前分支的所有本地分支，安全检查
git diff 分支1 分支2  # 对比两个分支的差异
git log 分支名    # 查看该分支的提交历史
git fetch --prune #在本地清理已删除的远程分支
git fetch -p  # 每次拉取时自动清理已删除的远程分支

# 其他
# 查看所有操作记录，找到分支删除前的最后一次提交哈希
git reflog
#仓库中有大量无法访问的松散对象（如已删除分支的提交），影响性能。
git prune
#上次 Git 垃圾回收（GC）执行失败，留下错误日志文件 `.git/gc.log`，阻止自动清理
rm .git/gc.log  # 删除错误日志文件
git gc --auto   # 手动触发一次自动垃圾回收
```
```
