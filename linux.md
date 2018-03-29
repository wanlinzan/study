
- `ps --forest` 命令可以查看进程的树形结构
```bash
# ps --forest

  PID TTY          TIME CMD
23070 pts/0    00:00:00 bash
 9728 pts/0    00:00:00  \_ bash
 9761 pts/0    00:00:00      \_ bash
 9798 pts/0    00:00:00          \_ bash
10353 pts/0    00:00:00              \_ ps

```
- `ps -f` 可以显示进程的PPID
```bash
UID        PID  PPID  C STIME TTY          TIME CMD
root      9728 23070  0 20:09 pts/0    00:00:00 bash
root      9761  9728  0 20:09 pts/0    00:00:00 bash
root      9798  9761  0 20:09 pts/0    00:00:00 bash
root     12005  9798  0 20:13 pts/0    00:00:00 ps -f
root     23070 23068  0 10:45 pts/0    00:00:00 -bash
```
- 括号执行命令会创建出子shell
```bash
( pwd ; echo $BASH_SUBSHELL )
```