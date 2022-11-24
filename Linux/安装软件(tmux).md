# 安装软件(tmux)

tmux是一个终端软件，可以实现分屏等功能
```
tar -xvzf libevent-2.1.8-stable.tar.gz
tar -xvzf tmux-2.7.tar.gz 
 
cd libevent-2.1.8-stable
./configure --prefix=/home/XXXX/tmux-2.7_Install
make
make install 
```

```
cd ../tmux-2.7
setenv PKG_CONFIG_PATH /home/XXXX/libevent-2.1.8-stable
./configure --prefix=/home/XXX/tmux-2.7_Install
make
make install
```