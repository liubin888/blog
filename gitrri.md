git提交的时候如果遇到does not match your user account

除了使用下面的命令重新设置账号以外


git config --global user.name "Your Name"

git config --global user.email you@example.com
还需要
git commit --amend --reset-author