
> 内存泄漏（Memory Leak）是指程序中己动态分配的堆内存由于某种原因程序未释放或无法释放，造成系统内存的浪费，导致程序运行速度减慢甚至系统崩溃等严重后果。日常工作中应该经常会遇到这类问题。现在介绍几种分析内存泄漏的方法。（还未实践，权当做个记录(⊙﹏⊙)b）
内存泄漏通常是因为我们不正确的使用内存导致，比如malloc后未释放、或者释放的内存地址不正确等。内核报的错误也往往是内存一类的错误，例如
```c
Unable to handle kernel paging request at virtual address
Unable to handle kernel NULL pointer dereference at virtual address
```
对应的信号往往是SIGSEGV（试图访问未分配给自己的内存, 或试图往没有写权限的内存地址写数据），当遇到这类问题时，需要排除内存泄漏、内存的稳定性等问题。
通常系统会提供一些现成的工具来排查问题。kernel 有memleak、Valgrind等，native也有类似的工具，framework层跟其他层有所区别，可以理解为Java的内存泄漏，有Memory Monitor、eclipse的mat分析工具等，比较丰富。
#### kernel 内存泄漏
内核中内存泄漏检测工具kmemleak，该工具通过对kmalloc, vmalloc, kmem_cache_alloc和friends伙伴算法进行跟踪来分析内存使用
* 需要先检查lib/Kconfig.debug中DEBUG_KMEMLEAK的依赖
  具体使用方法如下:
  
* Kernel hacking打开CONFIG_DEBUG_KMEMLEAK宏
* 挂载debugfs,配置kmemleak
```c
# mount -t debugfs nodev /sys/kernel/debug/
# cat /sys/kernel/debug/kmemleak
```
触发一次内存扫描，默认每10分钟扫描一次内存，并打印出新的未被使用的内存对象
```c
# echo scan > /sys/kernel/debug/kmemleak
```
清空一次结果：
```c
# echo clear > /sys/kernel/debug/kmemleak
```
参数可以在运行时配置，只需向/sys/kernel/debug/kmemleak节点中写入参数即可，支持的参数如下
```c
  off        - disable kmemleak (irreversible)
  stack=on    - enable the task stacks scanning (default)
  stack=off    - disable the tasks stacks scanning
  scan=on    - start the automatic memory scanning thread (default)
  scan=off    - stop the automatic memory scanning thread
  scan=<secs>    - set the automatic memory scanning period in seconds
          (default 600, 0 to stop the automatic scanning)
  scan        - trigger a memory scan
  clear        - clear list of current memory leak suspects, done by
          marking all current reported unreferenced objects grey
  dump=<addr>    - dump information about the object found at <addr>
```


#### native 内存泄漏
native层跟kernel层有些类似，也是通过分析malloc、free等函数来对内存进行分析。
* 可通过ADB命令来使能
使能并跟踪所有线程的内存分配
```c
adb shell stop
adb shell setprop libc.debug.malloc.options backtrace
adb shell start
```
指定跟踪某一个线程
```c
adb shell setprop libc.debug.malloc.options backtrace   配置参数
adb shell setprop libc.debug.malloc.program ls    设置跟踪目标应用或进程
adb shell ls
```
使能跟踪由zygote 、 zygote 生成的线程
```c
adb shell stop
adb shell setprop libc.debug.malloc.program app_process
adb shell setprop libc.debug.malloc.options backtrace
adb shell start
```
同时配置多个参数
```c
adb shell stop
adb shell setprop libc.debug.malloc.options "\"backtrace guards\""
adb shell start
```
* 使用环境变量配置
使能malloc debug
```c
adb shell
# setprop libc.debug.malloc.env_enabled 1
# setprop libc.debug.malloc.options backtrace
# export LIBC_DEBUG_MALLOC_ENABLE=1
# ls
```
```c
adb shell
# export LIBC_DEBUG_MALLOC_OPTIONS=backtrace
# ls
```
任何从该shell 生成的进程都会被malloc debug跟踪
```c
adb shell stop
adb shell setprop libc.debug.malloc.options backtrace
adb shell start
adb shell am dumpheap -n <PID_TO_DUMP> /data/local/tmp/heap.txt
```
* 查看输出的结果
```c
adb shell pull /data/local/tmp/heap.txt .
development/scripts/native_heapdump_viewer.py heap.txt > heap_info.txt
```
使能针对应用或程序的malloc debug功能(android o 以后版本支持）
```c
adb shell setprop wrap.<APP> '"LIBC_DEBUG_MALLOC_OPTIONS=backtrace logwrapper"'
```
例子如下：
```c
adb shell setprop wrap.com.google.android.googlequicksearchbox '"LIBC_DEBUG_MALLOC_OPTIONS=backtrace logwrapper"'
adb shell am force-stop com.google.android.googlequicksearchbox
```

####framework 内存泄漏
