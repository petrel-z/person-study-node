- 进入项目目录
```bash
cd "/Users/petrel/vsCode/平高项目/ping-gao-web/pingGaoWebDevelop"
```

- 确认当前远程（你用到的是 `github`）
```bash
git remote -v
```

- 用正确顺序重写历史邮箱（新在前、旧在后；把 `1207574452@qq.com` 确保已在 GitHub 里 Verified）
```bash
brew install git-filter-repo

cat > /tmp/mailmap <<'EOF'
赵海燕 <1207574452@qq.com> 赵海燕 <14012394+zhao-hai-yan@user.noreply.gitee.com>
EOF

git filter-repo --force --mailmap /tmp/mailmap
```

- 验证是否改成功（看到 `赵海燕 <1207574452@qq.com>` 就对了）
```bash
git log --format="%h %an <%ae> %ad" --date=short -n 20
```

- 强推覆盖 GitHub（分支 + tag）
```bash
git push -f github --all
git push -f github --tags
```