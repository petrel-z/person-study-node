
### http和ssh克隆的区别、应用场景

### 配置ssh密钥
#### 管理本地ssh密钥：

找到密钥目录：
```shell
cd ~/.ssh # 进入目录 执行ls查看所有ssh密钥文件
ls -al ~/.ssh  # 不用进入目录直接查看
cat ~/.ssh/gitlab.pub # 查看公钥
```
新建ssh密钥：
```bash
# 格式：ssh-keygen -t 算法 -f ~/.ssh/自定义名称 -C "备注（如平台/用途）"
ssh-keygen -t ed25519 -f ~/.ssh/github_work -C "公司GitHub工作用"
```
删除密钥：
```bash
# 示例：删除名为 old_key 的密钥对
rm ~/.ssh/old_key
rm ~/.ssh/old_key.pub
```

#### 管理多个ssh：配置config文件
创建/编辑config：
```bash
# 终端执行（Windows用记事本打开 ~/.ssh/config）
vim ~/.ssh/config
```

配置实例：
```bash
# GitHub 个人账号（用默认密钥）
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519  # 指定私钥路径

# GitHub 公司账号（用自定义密钥）
Host github-work.com  # 自定义别名（可任意命名，如 github-work）
  HostName github.com  # 实际域名
  User git
  IdentityFile ~/.ssh/github_work  # 公司密钥的私钥路径

# GitLab 平台
Host gitlab.com
  HostName gitlab.com
  User git
  IdentityFile ~/.ssh/gitlab_key  # GitLab专用密钥

# 配置 GitHub 连接（Host 与实际域名一致，无需修改克隆命令）
Host github.com
  HostName github.com  # 实际域名（与 Host 相同）
  User git             # 固定为 git，所有 Git 平台通用
  IdentityFile ~/.ssh/github_key  # 指定 GitHub 专用密钥
  AddKeysToAgent yes   # 自动将密钥加入 ssh-agent
  UseKeychain yes      # macOS 系统：将密钥密码存入钥匙串钥匙串（可选）
```
- `Host`：自定义别名（如 `github-work.com`），后续克隆仓库时用该别名==代替==实际域名。
- `HostName`：实际平台的域名或 IP（如 `github.com`）。
- `IdentityFile`：指定该平台对应的私钥路径。
-  AddKeysToAgent yes   # 自动将密钥加入 ssh-agent
- UseKeychain yes      # macOS 系统：将密钥密码存入钥匙串钥匙串（可选）


问题：
配置了ssh，但是每次拉取推送代码到仓库，都需要输入密码
原因：在生成ssh密钥时，设置了密钥的保护密码
- 生成 SSH 密钥时，会有一步提示 `Enter passphrase (empty for no passphrase)`，如果你在这里输入了一串字符（比如 `123456`），这个字符就是 “密钥保护密码”。
- 它的作用是给你的 SSH 私钥加一层保护：即使私钥文件被别人偷走，没有这个密码也无法使用该密钥。
- 如果你生成密钥时直接按了回车（`passphrase` 为空），则不会出现这个密码提示。



![[Pasted image 20250731125419.png]]

仓库地址使用了 `2222` 端口（正常默认是 22），但我的 SSH 配置未指定该端口，导致连接时可能用错端口。

检验是否和gitlab连接成功：
```bash
ssh -T git@gitlab.redclass.cn -p 2222
```


问题：
用ssh再gitlab克隆了一个项目，如果我将本地与gilab关联的秘钥删除，会怎么样
会出现错误
```bash
git@gitlab.redclass.cn: Permission denied (publickey).
fatal: Could not read from remote repository.
```
解决办法：重新配置ssh密钥并与gitlab关联


删除秘钥：
示例1：删除 GitHub 密钥（私钥+公钥）
rm -f ~/.ssh/github ~/.ssh/github.pub

配置文件的别名作用：
方便git clone  如：git clone gitlab-remote:exam/edu.git
方便其他指令如：
ssh -T gitlab-remote


### 一、SSH 配置文件的核心作用 & 便捷性
SSH 配置文件（`~/.ssh/config`）的核心价值是**「固化连接规则、简化重复操作、解决多密钥冲突」**，具体作用和便捷性可以用「有 vs 无」的对比说清楚：

| 操作场景                | 无配置文件（原始方式）| 有配置文件（优化后）| 便捷性体现                     |
|-------------------------|--------------------------------------------------------------------------|---------------------------------------------|--------------------------------|
| 连接 GitLab（端口2222） | `ssh -p 2222 -i ~/.ssh/gitlab git@gitlab.redclass.cn`                    | `ssh gitlab-redclass`（自定义别名）| 无需记端口/密钥路径，命令缩短80% |
| 克隆 GitHub 仓库        | `git clone git@github.com:xxx/xxx.git -i ~/.ssh/github`                  | `git clone github:xxx/xxx.git`              | 自动匹配密钥，不用手动指定 `-i` |
| 拉取 Gitee 代码         | `git pull git@gitee.com:xxx/xxx.git -i ~/.ssh/gitee`                     | `git pull`（仓库已关联）| 无需每次指定密钥，和普通操作一致 |
| 多平台密钥切换          | 容易混淆密钥路径，甚至用错密钥导致认证失败                               | 自动匹配平台+密钥，无冲突                   | 彻底解决「一个密钥通配所有平台」的冲突问题 |

总结配置文件的核心价值：
1. **简化命令**：把冗长的 `ssh -p 端口 -i 密钥 用户名@主机` 简化为 `ssh 别名`；
2. **统一规则**：把不同平台的连接参数（端口、密钥、用户）固化，不用每次记忆/输入；
3. **隔离密钥**：为不同平台分配专属密钥，避免「一个密钥泄露影响所有平台」，也避免密钥冲突；
4. **减少错误**：避免因记错端口、密钥路径导致的 SSH 认证失败（比如你之前 GitLab 端口2222的问题）。

### 二、多平台（GitLab/GitHub/Gitee）配置多个 SSH 密钥（完整可复制步骤）
如果需要同时对接 GitLab、GitHub、Gitee，核心思路是「为每个平台生成独立密钥 + 配置文件指定专属规则」，步骤如下：

#### 步骤 1：为每个平台生成独立 SSH 密钥
（避免共用一个密钥，安全且避免冲突，命令直接复制执行）
```bash
# 1. 生成 GitHub 密钥（邮箱填你的 GitHub 登录邮箱）
ssh-keygen -t ed25519 -C "your-github-email@xxx.com" -f ~/.ssh/github

# 2. 生成 Gitee 密钥（邮箱填你的 Gitee 登录邮箱）
ssh-keygen -t ed25519 -C "your-gitee-email@xxx.com" -f ~/.ssh/gitee

# 3. 生成 GitLab 密钥（邮箱填你的 GitLab 登录邮箱，对应之前的场景）
ssh-keygen -t ed25519 -C "your-gitlab-email@xxx.com" -f ~/.ssh/gitlab
```
- 执行时提示 `Enter passphrase`：可直接回车（无密码），或设置密码（更安全，每次用密钥需输入）；
- 生成后，`~/.ssh/` 下会有 6 个文件：`github`（私钥）、`github.pub`（公钥）、`gitee`/`gitee.pub`、`gitlab`/`gitlab.pub`。

#### 步骤 2：设置密钥权限（SSH 安全要求，必须执行）
```bash
# 私钥权限必须为 600（否则 SSH 拒绝使用）
chmod 600 ~/.ssh/github ~/.ssh/gitee ~/.ssh/gitlab

# 公钥权限设为 644（可选，不影响但规范）
chmod 644 ~/.ssh/github.pub ~/.ssh/gitee.pub ~/.ssh/gitlab.pub
```

#### 步骤 3：编写统一的 SSH 配置文件
编辑 `~/.ssh/config`，为每个平台指定专属规则（直接复制覆盖，替换注释里的信息）：
```bash
# 清空旧配置（若有其他重要配置，跳过这步，直接 vim 编辑追加）
> ~/.ssh/config
vim ~/.ssh/config
```
按下 `i` 进入编辑模式，粘贴以下内容：
```config
# ========== GitHub 配置 ==========
Host github.com
  HostName github.com
  Port 22
  User git
  IdentityFile /Users/petrel/.ssh/github  # 替换为你的私钥绝对路径
  StrictHostKeyChecking no

# ========== Gitee 配置 ==========
Host gitee.com
  HostName gitee.com
  Port 22
  User git
  IdentityFile /Users/petrel/.ssh/gitee  # 替换为你的私钥绝对路径
  StrictHostKeyChecking no

# ========== 你的 GitLab 配置（端口2222） ==========
Host gitlab.redclass.cn
  HostName gitlab.redclass.cn
  Port 2222  # 你的 GitLab 非默认端口，必须指定
  User git
  IdentityFile /Users/petrel/.ssh/gitlab  # 替换为你的私钥绝对路径
  StrictHostKeyChecking no

# 可选：为 GitLab 设置别名（更简洁）
Host mygitlab
  HostName gitlab.redclass.cn
  Port 2222
  User git
  IdentityFile /Users/petrel/.ssh/gitlab
```
按下 `esc` → 输入 `:wq` 保存退出，然后设置配置文件权限：
```bash
chmod 600 ~/.ssh/config  # 配置文件权限也必须是600，否则SSH拒绝读取
```

#### 步骤 4：将公钥上传到对应平台后台
每个平台需要上传专属的公钥（复制公钥内容，粘贴到平台的 SSH 密钥设置页）：
```bash
# 1. 复制 GitHub 公钥（复制输出的全部内容）
cat ~/.ssh/github.pub

# 2. 复制 Gitee 公钥
cat ~/.ssh/gitee.pub

# 3. 复制 GitLab 公钥
cat ~/.ssh/gitlab.pub
```
分别上传到对应平台：
- **GitHub**：登录 → 头像 → Settings → SSH and GPG keys → New SSH key → 粘贴公钥 → Add SSH key；
- **Gitee**：登录 → 头像 → 设置 → SSH 公钥 → 粘贴公钥 → 确定；
- **GitLab**：登录 → 头像 → Preferences → SSH Keys → 粘贴公钥 → Add key。

#### 步骤 5：验证每个平台的 SSH 连接
执行以下命令，验证是否能成功认证（输出「Welcome」类提示即成功）：
```bash
# 验证 GitHub
ssh -T git@github.com

# 验证 Gitee
ssh -T git@gitee.com

# 验证 GitLab（用别名或原域名）
ssh -T git@gitlab.redclass.cn
# 或用别名：ssh -T mygitlab
```
✅ 成功提示示例：
- GitHub：`Hi 你的用户名! You've successfully authenticated...`；
- Gitee：`Hi 你的用户名! You've successfully authenticated...`；
- GitLab：`Welcome to GitLab, @你的用户名!`。

#### 步骤 6：测试克隆/拉推代码（最终验证）
配置完成后，克隆各平台仓库无需手动指定密钥/端口，直接用常规命令即可：
```bash
# 克隆 GitHub 仓库
git clone git@github.com:你的用户名/仓库名.git

# 克隆 Gitee 仓库
git clone git@gitee.com:你的用户名/仓库名.git

# 克隆 GitLab 仓库（用原域名或别名）
git clone git@gitlab.redclass.cn:exam/edu.git
# 或用别名：git clone mygitlab:exam/edu.git
```

### 三、多平台配置的核心注意事项（避坑）
1. **密钥绝对路径要正确**：`IdentityFile` 必须写全路径（如 `/Users/petrel/.ssh/github`），不要用 `~`（避免解析失败）；
2. **Host 名称要匹配平台域名**：比如 `Host github.com` 必须和仓库地址中的 `github.com` 一致，否则 SSH 无法匹配规则；
3. **权限是重中之重**：私钥（`github/gitlab/gitee`）和配置文件（`config`）权限必须为 600，否则 SSH 会直接拒绝使用（报错 `Bad permissions`）；
4. **公钥要粘贴完整**：复制公钥时，要包含开头的 `ssh-ed25519` 和结尾的邮箱，不要漏字符；
5. **端口不要填错**：大部分平台默认22（GitHub/Gitee），但你的 GitLab 是2222，必须在配置中指定。

### 总结
SSH 配置文件的核心价值是「一次配置，终身简化」：
- 对单平台：简化命令，避免记错端口/密钥；
- 对多平台：隔离密钥，解决冲突，无需手动切换密钥，各平台操作和普通单平台一致；
- 对安全性：不同平台用不同密钥，即使某一个密钥泄露，其他平台不受影响。

按上面的步骤配置后，你可以无缝对接 GitLab、GitHub、Gitee，所有 SSH 相关操作（克隆、拉取、推送）都和使用单个平台一样简单。