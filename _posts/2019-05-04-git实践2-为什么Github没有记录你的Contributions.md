---
layout: post
categories: git 系统管理
---

在github上提交代码后，发现自己的Github没有记录自己的Contributions。
这种情况下，一般都是有多个账号导致的，比如公司一个，github一个；
可以通过执行`git log` 查看提交记录李的Author

碰到这种情况，可以做两件事：

- 通过'git config --local `修改当前仓储参数
- 也可以强制更新Git历史

# 修改当前仓储参数

```bash
git config -h 
git config -l # 查看配置信息
git config --local -l # 查看当前仓储的配置信息
git config --local user.name "perfectstorm88"          # 修改当前仓储的用户名字
git config --local user.email "perfectstorm88@163.com"  # 修改当前仓储的邮箱
git config --global user.name "lichangzhen"          # 修改全局名字
git config --global user.email "lichangzhen@163.com"  # 修改全局邮箱
```
关于公司账号、github账号，哪个做全局参数，看谁的仓储多了

# 强制更新Git历史
1. 复制粘贴脚本，并根据你的信息修改以下变量：旧的Email地址，正确的用户名，正确的邮件地址

```bash
#!/bin/bash
git filter-branch --env-filter '

OLD_EMAIL="li.changzhen@163.com"
CORRECT_NAME="perfectstorm88"
CORRECT_EMAIL="perfectstorm88@163.com"

if [ "$GIT_COMMITTER_EMAIL" = "$OLD_EMAIL" ]
then
    export GIT_COMMITTER_NAME="$CORRECT_NAME"
    export GIT_COMMITTER_EMAIL="$CORRECT_EMAIL"
fi
if [ "$GIT_AUTHOR_EMAIL" = "$OLD_EMAIL" ]
then
    export GIT_AUTHOR_NAME="$CORRECT_NAME"
    export GIT_AUTHOR_EMAIL="$CORRECT_EMAIL"
fi
' --tag-name-filter cat -- --branches --tags
```
1. 执行文件
2. 用git log命令看看新 Git 历史有没有错误
3. 把正确历史 push 到 Github


```bash
git push --force --tags origin 'refs/heads/*'
```

# 参考：
- [为什么Github没有记录你的Contributions](https://segmentfault.com/a/1190000004318632)
- [why-are-my-contributions-not-showing-up-on-my-profile](https://help.github.com/en/articles/why-are-my-contributions-not-showing-up-on-my-profile)
- [Changing author info](https://help.github.com/en/articles/changing-author-info)