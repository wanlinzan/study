
```bash
# ps --forest //命令可以查看进程的树形结构
  PID TTY          TIME CMD
23070 pts/0    00:00:00 bash
 9728 pts/0    00:00:00  \_ bash
 9761 pts/0    00:00:00      \_ bash
 9798 pts/0    00:00:00          \_ bash
10353 pts/0    00:00:00              \_ ps
 

# ps -f //可以显示进程的PPID
UID        PID  PPID  C STIME TTY          TIME CMD
root      9728 23070  0 20:09 pts/0    00:00:00 bash
root      9761  9728  0 20:09 pts/0    00:00:00 bash
root      9798  9761  0 20:09 pts/0    00:00:00 bash
root     12005  9798  0 20:13 pts/0    00:00:00 ps -f
root     23070 23068  0 10:45 pts/0    00:00:00 -bash

 
# ( pwd ; echo $BASH_SUBSHELL )
/usr/local/src/php-7.2.3
1
 

# which ps //查看命令存放位置
/usr/bin/ps
 
# jobs //查看后台运行的任务
[1]-  Running                 sleep 30000 &
[2]+  Running                 sleep 40000 &
 
 

# type ps //显示和命令有关的信息
ps is hashed (/usr/bin/ps)
 
# type pwd //显示和命令有关的信息
pwd is a shell builtin
 

# type -a ps //显示所有和命令有关的信息
ps is /usr/bin/ps
 
# type -a pwd //显示所有和命令有关的信息
pwd is a shell builtin
pwd is /usr/bin/pwd
 

# which pwd //显示外部命令文件的位置
/usr/bin/pwd

# !! //可以执行上次执行的命令
ps --forest
  PID TTY          TIME CMD
23070 pts/0    00:00:00 bash
 9728 pts/0    00:00:00  \_ bash
 9761 pts/0    00:00:00      \_ bash
21736 pts/0    00:00:00          \_ sleep
21926 pts/0    00:00:00          \_ sleep
 9200 pts/0    00:00:00          \_ ps
21303 pts/0    00:00:00 sleep

# history -a //强制把本次会话历史写入history文件

# !20 //执行history列表20的命令





























```
