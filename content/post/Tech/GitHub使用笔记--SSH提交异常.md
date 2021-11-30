---
title: "GitHub使用笔记--SSH提交异常"
date: 2016-10-19
tags: ["GitHub","SSH"]
draft: false
---

> 系统环境：MAC

## ssh `git push` 时出现timeout的问题
```bash
$ git push
ssh: connect to host github.com port 22: Connection timed out
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```
修改前测试：
```
$ ssh -T git@github.com
ssh: connect to host github.com port 22: Operation timed out
```

修改 ~/.ssh/config 中 github.com 的配置， Hostname 改为 ssh.github.com, Port 改为 443:
```
Host github.com
  Hostname ssh.github.com
  Port 443
```
修改后测试：
```
$ ssh -T git@github.com
The authenticity of host '[ssh.github.com]:443 ([192.30.253.123]:443)' can't be established.
RSA key fingerprint is SHA256:ntxxxxxxxxxxxxx.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '[ssh.github.com]:443,[192.30.253.123]:443' (RSA) to the list of known hosts.
Hi hui1hui2hui3! You've successfully authenticated, but GitHub does not provide shell access
```
```
$ ssh -T git@github.com
Warning: Permanently added the RSA host key for IP address '[192.30.253.122]:443' to the list of known hosts.
Hi hui1hui2hui3! You've successfully authenticated, but GitHub does not provide shell access.
```
再来推代码：
```
$ git push
Counting objects: 311, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (301/301), done.
Writing objects: 100% (311/311), 770.45 KiB | 0 bytes/s, done.
Total 311 (delta 14), reused 0 (delta 0)
remote: Resolving deltas: 100% (14/14), done.
To git@github.com:hui1hui2hui3/interface-perform-test.git
   3b4056f..0324218  master -> master
```
Eyerything is OK

## ssh `git push` 时出现Permission denied问题
```
$ git push
Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```
第一步：检测
```
$ ssh -T git@github.com
Permission denied (publickey).
```
第一步会发现确实是无权限。

第二步：解决方法
```
$ ssh-add ~/.ssh/id_github_new_rsa
Enter passphrase for /Users/huis/.ssh/id_github_new_rsa:
Identity added: /Users/huis/.ssh/id_github_new_rsa (/Users/huis/.ssh/id_github_new_rsa)
```

第三步：检测
```
$ ssh -T git@github.com
Hi hui1hui2hui3! You've successfully authenticated, but GitHub does not provide shell access.
```

第四步：尝试提交
```
$ git push
warning: push.default is unset; its implicit value has changed in
Git 2.0 from 'matching' to 'simple'. To squelch this message
and maintain the traditional behavior, use:

  git config --global push.default matching

To squelch this message and adopt the new behavior now, use:

  git config --global push.default simple

When push.default is set to 'matching', git will push local branches
to the remote branches that already exist with the same name.
When push.default is set to 'matching', git will push local branches
to the remote branches that already exist with the same name.

Since Git 2.0, Git defaults to the more conservative 'simple'
behavior, which only pushes the current branch to the corresponding
remote branch that 'git pull' uses to update the current branch.

See 'git help config' and search for 'push.default' for further information.
(the 'simple' mode was introduced in Git 1.7.11. Use the similar mode
'current' instead of 'simple' if you sometimes use older versions of Git)
Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```
Oh No! 还是不行
重复第一、二、三步，再次尝试提交：
```
$ git push origin master
Counting objects: 34, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (31/31), done.
Writing objects: 100% (34/34), 24.38 KiB | 0 bytes/s, done.
Total 34 (delta 12), reused 0 (delta 0)
remote: Resolving deltas: 100% (12/12), completed with 9 local objects.
To git@github.com:hui1hui2hui3/interface-perform-test.git
   0324218..8acadba  master -> master
```
终于成功了

**结论：发现如果本地已经有一个是id_rsa的文件key时，如果github的使用其他名称的key，同时本地配置文件没有配置过相应的文件，则提交时直接`git push`会出现重置或使用默认配置文件（默认读取的是id_rsa的key文件）的情况，导致无法权限提交，即使你尝试使用ssh-add命令添加key仍然不行，所以还是使用`git push origin master`这种完整的命令较好**


----------

**更新时间：2016.10.17**
发现实际使用时，仍然会每次电脑开启都要执行上面的`ssh-add`操作才可以，很麻烦，尝试新的完美的解决方法是修改`~/.ssh/config`文件
```
Host github.com
        HostName ssh.github.com
        Port 443
        IdentityFile ~/.ssh/id_github_new_rsa #这行是重点新增的
```

----------

**更新时间：2016.10.19**
实际操作后，不需要添加了，但是却需要输入密码，所有尝试新的方法：
```
ssh-add -K id_github_new_rsa
```

## 参考
> [Error: Permission denied (publickey)](https://help.github.com/articles/error-permission-denied-publickey/)
> [改用 443 端口连接 Github 修复 git push 时出现 Connection timed out 的问题](https://mozillazg.com/2015/08/use-443-port-fix-github-connection-timeout.html)
> [Using SSH over the HTTPS port](https://help.github.com/articles/using-ssh-over-the-https-port/)
