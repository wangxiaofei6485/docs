> android 系统中总是会出现系统崩溃或者程序无响应等情况，android回采取一些方法来记录log。android本身是构建在linux上的，除了linux本身的pstore(记录内核崩溃的log），还添加了native log和framework log，native log 以tomstone形式保存，framework以logcat的形式打印(既包含java log，也包含kernel等log)。
这篇文章记录根据log查看问题的一些技巧。(好多我都没实践过，权当做个记录 -_-|||)

### kernel log



### native log
native log可以理解为linux用户层log，因此可以当做用户层log来进行分析。用例如下：
```c
*** *** *** *** *** *** *** *** *** *** *** *** *** *** *** ***
Build fingerprint: 'Android/aosp_angler/angler:7.1.1/NYC/enh12211018:eng/test-keys'
Revision: '0'
ABI: 'arm'
pid: 17946, tid: 17949, name: crasher  >>> crasher <<<
signal 11 (SIGSEGV), code 1 (SEGV_MAPERR), fault addr 0xc
    r0 0000000c  r1 00000000  r2 00000000  r3 00000000
    r4 00000000  r5 0000000c  r6 eccdd920  r7 00000078
    r8 0000461a  r9 ffc78c19  sl ab209441  fp fffff924
    ip ed01b834  sp eccdd800  lr ecfa9a1f  pc ecfd693e  cpsr 600e0030

backtrace:
    #00 pc 0004793e  /system/lib/libc.so (pthread_mutex_lock+1)
    #01 pc 0001aa1b  /system/lib/libc.so (readdir+10)
    #02 pc 00001b91  /system/xbin/crasher (readdir_null+20)
    #03 pc 0000184b  /system/xbin/crasher (do_action+978)
    #04 pc 00001459  /system/xbin/crasher (thread_callback+24)
    #05 pc 00047317  /system/lib/libc.so (_ZL15__pthread_startPv+22)
    #06 pc 0001a7e5  /system/lib/libc.so (__start_thread+34)
Tombstone written to: /data/tombstones/tombstone_06
```
1.分析如下：
* Build fingerprint:系统编译的版本号，读取的是ro.build.fignerprint属性。（分析时基本可以忽略）
* ABI:  可能是arm, arm64, mips, mips64, x86,  x86-64。该值对stack 脚本来说是最有用的了，可以用来分析时的工具。
* Revision:硬件的版本，与ro.revision属性值一致
* signal 11 (SIGSEGV)：异常信号，根据该信号，可大致判断引起crash的原因。
* backtrace:毫无疑问，异常堆栈才是关键。实例中的相对来说，比较简单，可以通过堆栈可以大致判断出要从我们添加的crasher程序中readdir_null()函数，查找对readdir调用或给readdir传的参数有问题是否存在问题。
* fault addr 0xc:异常时，参数指针的地址

2.进一步分析：
再回头看SIGSEGV,维基中对该信号的定义如下
  > 内存区块错误（英语：Segmentation fault，经常被缩写为segfault），又译为内存段错误，也称访问权限冲突（access violation），是一种程序错误。
  它会出现在当程序企图访问CPU无法定址的内存区块时。当错误发生时，硬件会通知操作系统产生了内存访问权限冲突的状况。操作系统通常会产生核心转储文件（core dump）以方便程序员进行除错。通常该错误是由于调用一个地址，而该地址为空（NULL）所造成的，例如链表中调用一个未分配地址的空链表单元的元素。数组访问越界也可能产生这个错误。
通过信号可以判断可能在调用readdir时，传入了一个空指。


3.进一步分析：
readdir 函数
```c
struct dirent* readdir(DIR* dir_handle);

struct DIR {
  int fd_;
  size_t available_bytes_;
  dirent* next_;
  pthread_mutex_t mutex_;
  dirent buff_[15];
  long current_pos_;
};
```
readdir参数为空，在调用mutex_变量是为空，问题根源在于传给readdir的参数为空，需检查路径下是否有文件存在或者路径是否存在。

补充：
也可以使用development/tools/stack工具来做分析，方法如下

```c
stack < FS/data/tombstones/tombstone_05
```
示例输出如下
```c
Reading native crash info from stdin
03-02 23:53:49.477 17951 17951 F DEBUG   : *** *** *** *** *** *** *** *** *** *** *** *** *** *** *** ***
03-02 23:53:49.477 17951 17951 F DEBUG   : Build fingerprint: 'Android/aosp_angler/angler:7.1.1/NYC/enh12211018:eng/test-keys'
03-02 23:53:49.477 17951 17951 F DEBUG   : Revision: '0'
03-02 23:53:49.477 17951 17951 F DEBUG   : ABI: 'arm'
03-02 23:53:49.478 17951 17951 F DEBUG   : pid: 17946, tid: 17949, name: crasher  >>> crasher <<<
03-02 23:53:49.478 17951 17951 F DEBUG   : signal 11 (SIGSEGV), code 1 (SEGV_MAPERR), fault addr 0xc
03-02 23:53:49.478 17951 17951 F DEBUG   :     r0 0000000c  r1 00000000  r2 00000000  r3 00000000
03-02 23:53:49.478 17951 17951 F DEBUG   :     r4 00000000  r5 0000000c  r6 eccdd920  r7 00000078
03-02 23:53:49.478 17951 17951 F DEBUG   :     r8 0000461a  r9 ffc78c19  sl ab209441  fp fffff924
03-02 23:53:49.478 17951 17951 F DEBUG   :     ip ed01b834  sp eccdd800  lr ecfa9a1f  pc ecfd693e  cpsr 600e0030
03-02 23:53:49.491 17951 17951 F DEBUG   :
03-02 23:53:49.491 17951 17951 F DEBUG   : backtrace:
03-02 23:53:49.492 17951 17951 F DEBUG   :     #00 pc 0004793e  /system/lib/libc.so (pthread_mutex_lock+1)
03-02 23:53:49.492 17951 17951 F DEBUG   :     #01 pc 0001aa1b  /system/lib/libc.so (readdir+10)
03-02 23:53:49.492 17951 17951 F DEBUG   :     #02 pc 00001b91  /system/xbin/crasher (readdir_null+20)
03-02 23:53:49.492 17951 17951 F DEBUG   :     #03 pc 0000184b  /system/xbin/crasher (do_action+978)
03-02 23:53:49.492 17951 17951 F DEBUG   :     #04 pc 00001459  /system/xbin/crasher (thread_callback+24)
03-02 23:53:49.492 17951 17951 F DEBUG   :     #05 pc 00047317  /system/lib/libc.so (_ZL15__pthread_startPv+22)
03-02 23:53:49.492 17951 17951 F DEBUG   :     #06 pc 0001a7e5  /system/lib/libc.so (__start_thread+34)
03-02 23:53:49.492 17951 17951 F DEBUG   :     Tombstone written to: /data/tombstones/tombstone_06
Reading symbols from /huge-ssd/aosp-arm64/out/target/product/angler/symbols
Revision: '0'
pid: 17946, tid: 17949, name: crasher  >>> crasher <<<
signal 11 (SIGSEGV), code 1 (SEGV_MAPERR), fault addr 0xc
     r0 0000000c  r1 00000000  r2 00000000  r3 00000000
     r4 00000000  r5 0000000c  r6 eccdd920  r7 00000078
     r8 0000461a  r9 ffc78c19  sl ab209441  fp fffff924
     ip ed01b834  sp eccdd800  lr ecfa9a1f  pc ecfd693e  cpsr 600e0030
Using arm toolchain from: /huge-ssd/aosp-arm64/prebuilts/gcc/linux-x86/arm/arm-linux-androideabi-4.9/bin/

Stack Trace:
  RELADDR   FUNCTION                   FILE:LINE
  0004793e  pthread_mutex_lock+2       bionic/libc/bionic/pthread_mutex.cpp:515
  v------>  ScopedPthreadMutexLocker   bionic/libc/private/ScopedPthreadMutexLocker.h:27
  0001aa1b  readdir+10                 bionic/libc/bionic/dirent.cpp:120
  00001b91  readdir_null+20            system/core/debuggerd/crasher.cpp:131
  0000184b  do_action+978              system/core/debuggerd/crasher.cpp:228
  00001459  thread_callback+24         system/core/debuggerd/crasher.cpp:90
  00047317  __pthread_start(void*)+22  bionic/libc/bionic/pthread_create.cpp:202 (discriminator 1)
  0001a7e5  __start_thread+34          bionic/libc/bionic/clone.cpp:46 (discriminator 1)
```

### framework log
