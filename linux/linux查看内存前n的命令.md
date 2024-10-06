```c++
ps aux | head -1;ps aux |grep -v PID |sort -rn -k +4 | head -5
```

出现程序内存泄漏的时候查看杀死进程