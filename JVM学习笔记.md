# JVM学习笔记

## java 参数
- [文档地址](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/index.html)
  - 包括启动参数等

### cup 飙高排查
- top 找 pid 如 31638
- 找线程。即tid
  ps -mp 31638  -o THREAD,tid,time
  top -H -p 31638
- 转16进制 1401 -> 579
  Printf “%x\n” 1401
  jstack 31638 | grep 579
- 上下10行也看看
  jstack jstack 31638 | grep 579 -A 10 -B 10
- 导出
  jstack 31638  >> 123.txt

### 查看某线程堆栈使用情况
jmap -heap pid
