# core_dump生成

步骤一：开启core dump文件生成
ulimit -c unlimited

步骤二：设置core dump文件位置
vi /etc/sysctl.conf
修改（添加）如下两个变量
kernel.core_pattern =/var/core/core_%e_%p
kernel.core_uses_pid= 0
这里是改为生成目录在/var/core/，%e代表程序名称，%p是进程ID
如果想直接生成在可执行文件相同目录，前面不要加任何目录，直接
kernel.core_pattern =core_%e_%p

步骤三：让修改生效
sysctl -p/etc/sysctl.conf

编译参数要添加 -g， o3优化改为o0
