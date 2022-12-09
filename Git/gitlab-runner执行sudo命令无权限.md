# gitlab-runner执行sudo命令无权限

1、为gitlab-runner添加root权限
```
$ groups gitlab-runner
> gitlab-runner : gitlab-runner
 
$ sudo usermod -a -G root gitlab-runner
 
$ sudo groups gitlab-runner
> gitlab-runner : gitlab-runner root
```
2、避免sudo命令需要输入密码
修改 /etc/sudoers里面内容，添加一行 gitlab-runner ALL=(ALL) NOPASSWD: ALL
直接修改会提示文件只读，在终端中输入 sudo visudo 命令即可。

