## Git 常用指令

```
git checkout -b <branch-name> # 新建分支
git push -u origin <branch-name> # 推送新建的分支到远程
git branch -a  # 查看所有本地和远程分支，检查分支推送是否成功
git add .  # 将所有更改的文件添加到暂存区
git add <file_name>  # 添加特定文件到暂存区
git commit -m "添加了新功能"  --no-verify # 提交更改，并附上提交信息 
注意：跳过校验，可以使用 --no-verify 参数
git push origin <branch-name> # 将本地提交推送到远程指定仓库
```

