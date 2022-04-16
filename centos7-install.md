
# 0. Install build requirements
```
[root@xphobia ~]#  yum install -y doxygen readline-devel binutils-devel
```

# 1. Get source code
```
[root@xphobia ~]# git clone --recursive -b develop git@github.com:Xphobia/libmemleak.git
```

# 2. Run autogen.sh

```
[root@xphobia libmemleak]# ./autogen.sh 
Entering 'cwm4'
Previous HEAD position was b954653... Fix previous commit.
Switched to branch 'master'
Already up-to-date.
[develop 6acada2] Updating submodule reference to current HEAD of branch master of cwm4
 1 file changed, 1 insertion(+), 1 deletion(-)

Root of project determined to be "/root/libmemleak".
Running libtoolize --force --automake  ...
Running aclocal -I m4/aclocal -I cwm4/aclocal ...
Running autoheader ...
Running automake --add-missing --foreign ...
cwm4/configure_ac_top.m4:45: installing './config.guess'
cwm4/configure_ac_top.m4:45: installing './config.sub'
cwm4/configure_ac_top.m4:11: installing './install-sh'
cwm4/configure_ac_top.m4:11: installing './missing'
src/Makefile.am: installing './depcomp'
Running autoconf ...

Now you can do 'mkdir ../libmemleak-objdir; cd ../libmemleak-objdir; ../libmemleak/configure --enable-maintainer-mode [--help]'.
```

# 3. Run configure
```
[root@xphobia libmemleak]# mkdir ../libmemleak-objdir; cd ../libmemleak-objdir; ../libmemleak/configure --enable-maintainer-mode
......
libtool: compile:  gcc -std=gnu11 -DHAVE_CONFIG_H -I. -I../../libmemleak/src -I.. -I../../libmemleak/src/include -g -O2 -MT addr2line.lo -MD -MP -MF .deps/addr2line.Tpo -c ../../libmemleak/src/addr2line.c  -fPIC -DPIC -o .libs/addr2line.o
../../libmemleak/src/addr2line.c:29:32: fatal error: libiberty/demangle.h: No such file or directory
 #include <libiberty/demangle.h>
                                ^
compilation terminated.
```
报错参考：https://github.com/strace/strace/issues/92
这里简单修改为：  #include <demangle.h>

# 4. Compile and install
```
[root@xphobia libmemleak-objdir]# make
[root@xphobia libmemleak-objdir]# make install
```

# 5. Test

在第一个shell终端执行hello程序
```
[root@xphobia src]# LD_PRELOAD='/usr/local/lib/libmemleak.so' ./hello
```

在第二个shell终端执行start开始检测，执行stop停止检测
```
[root@xphobia src]# ./memleak_control 
libmemleak> start
Auto restart interval is 5 * 1 seconds.
libmemleak> stop
Stopped.
libmemleak> start
Auto restart interval is 5 * 1 seconds.
libmemleak> dump 54
 #0  00007f32f76d7fde  in malloc at /root/libmemleak-objdir/src/../../libmemleak/src/memleak.c:1012
 #1  0000000000401363  in do_work(int) at /root/libmemleak-objdir/src/../../libmemleak/src/hello.cc:157
 #2  0000000000400f40  in thread_entry1(void*) at /root/libmemleak-objdir/src/../../libmemleak/src/hello.cc:20
 #3  00007f32f6c6eea5  in start_thread at pthread_create.c
 #4  00007f32f6997b0d  in __clone
libmemleak> 
libmemleak> dump 63
 #0  00007f32f76d81bb  in realloc at /root/libmemleak-objdir/src/../../libmemleak/src/memleak.c:1078
 #1  0000000000401318  in do_work(int) at /root/libmemleak-objdir/src/../../libmemleak/src/hello.cc:153
 #2  0000000000400f40  in thread_entry1(void*) at /root/libmemleak-objdir/src/../../libmemleak/src/hello.cc:20
 #3  00007f32f6c6eea5  in start_thread at pthread_create.c
 #4  00007f32f6997b0d  in __clone
libmemleak> dump 65
 #0  00007f32f76d7fde  in malloc at /root/libmemleak-objdir/src/../../libmemleak/src/memleak.c:1012
 #1  0000000000401318  in do_work(int) at /root/libmemleak-objdir/src/../../libmemleak/src/hello.cc:153
 #2  0000000000400f40  in thread_entry1(void*) at /root/libmemleak-objdir/src/../../libmemleak/src/hello.cc:20
 #3  00007f32f6c6eea5  in start_thread at pthread_create.c
 #4  00007f32f6997b0d  in __clone
libmemleak> dump 54
 #0  00007f32f76d7fde  in malloc at /root/libmemleak-objdir/src/../../libmemleak/src/memleak.c:1012
 #1  0000000000401363  in do_work(int) at /root/libmemleak-objdir/src/../../libmemleak/src/hello.cc:157
 #2  0000000000400f40  in thread_entry1(void*) at /root/libmemleak-objdir/src/../../libmemleak/src/hello.cc:20
 #3  00007f32f6c6eea5  in start_thread at pthread_create.c
 #4  00007f32f6997b0d  in __clone
libmemleak> dump 62
 #0  00007f32f76d80a4  in calloc at /root/libmemleak-objdir/src/../../libmemleak/src/memleak.c:1036
 #1  000000000040134d  in do_work(int) at /root/libmemleak-objdir/src/../../libmemleak/src/hello.cc:155
 #2  0000000000400f40  in thread_entry1(void*) at /root/libmemleak-objdir/src/../../libmemleak/src/hello.cc:20
 #3  00007f32f6c6eea5  in start_thread at pthread_create.c
 #4  00007f32f6997b0d  in __clone
libmemleak> 
libmemleak> stop
Stopped.
libmemleak> 

``

在第一个终端中查看打印
```
*** RESTART RECORDING ***
hello: Now: 392; 	Backtraces: 74; 	allocations: 1812284; 	total memory: 233,452,616 bytes.
 backtrace 54 (value_n: 411808.00); [ 388, 393>(   5):  9780 allocations (193766 total,  5.0%), size 1255345; 1956.00 allocations/s, 251069 bytes/s
 backtrace 54 (value_n: 411808.00); [ 383, 388>(   5): 12357 allocations (248353 total,  5.0%), size 1571255; 2471.40 allocations/s, 314251 bytes/s
 backtrace 54 (value_n: 411808.00); [ 373, 383>(  10): 23897 allocations (477539 total,  5.0%), size 3042262; 2389.70 allocations/s, 304226 bytes/s
 backtrace 54 (value_n: 411808.00); [ 363, 373>(  10): 21783 allocations (433412 total,  5.0%), size 2776637; 2178.30 allocations/s, 277663 bytes/s
 backtrace 54 (value_n: 411808.00); [ 264, 363>(  99): 231900 allocations (4601451 total,  5.0%), size 29593910; 2342.42 allocations/s, 298928 bytes/s
 backtrace 62 (value_n: 194837.00); [ 388, 393>(   5):  4472 allocations ( 92108 total,  4.9%), size  573495; 894.40 allocations/s, 114699 bytes/s
 backtrace 62 (value_n: 194837.00); [ 383, 388>(   5):  5835 allocations (117535 total,  5.0%), size  749125; 1167.00 allocations/s, 149825 bytes/s
 backtrace 62 (value_n: 194837.00); [ 373, 383>(  10): 11346 allocations (226929 total,  5.0%), size 1438879; 1134.60 allocations/s, 143887 bytes/s
 backtrace 62 (value_n: 194837.00); [ 363, 373>(  10): 10220 allocations (206079 total,  5.0%), size 1298804; 1022.00 allocations/s, 129880 bytes/s
 backtrace 62 (value_n: 194837.00); [ 264, 363>(  99): 110477 allocations (2186546 total,  5.1%), size 14137136; 1115.93 allocations/s, 142799 bytes/s
 backtrace 63 (value_n: 161518.00); [ 388, 393>(   5):  3779 allocations ( 75920 total,  5.0%), size  484294; 755.80 allocations/s, 96858 bytes/s
 backtrace 63 (value_n: 161518.00); [ 383, 388>(   5):  4887 allocations ( 97984 total,  5.0%), size  628861; 977.40 allocations/s, 125772 bytes/s
 backtrace 63 (value_n: 161518.00); [ 373, 383>(  10):  9342 allocations (188212 total,  5.0%), size 1195407; 934.20 allocations/s, 119540 bytes/s
 backtrace 63 (value_n: 161518.00); [ 363, 373>(  10):  8555 allocations (171309 total,  5.0%), size 1095052; 855.50 allocations/s, 109505 bytes/s
 backtrace 63 (value_n: 161518.00); [ 264, 363>(  99): 91060 allocations (1809250 total,  5.0%), size 11684511; 919.80 allocations/s, 118025 bytes/s
 backtrace 65 (value_n: 43894.00); [ 388, 393>(   5):  1058 allocations ( 20229 total,  5.2%), size  134187; 211.60 allocations/s, 26837 bytes/s
 backtrace 65 (value_n: 43894.00); [ 383, 388>(   5):  1309 allocations ( 26229 total,  5.0%), size  163566; 261.80 allocations/s, 32713 bytes/s
 backtrace 65 (value_n: 43894.00); [ 373, 383>(  10):  2521 allocations ( 49777 total,  5.1%), size  314874; 252.10 allocations/s, 31487 bytes/s
 backtrace 65 (value_n: 43894.00); [ 363, 373>(  10):  2284 allocations ( 45373 total,  5.0%), size  293319; 228.40 allocations/s, 29331 bytes/s
 backtrace 65 (value_n: 43894.00); [ 264, 363>(  99): 24816 allocations (487343 total,  5.1%), size 3145170; 250.67 allocations/s, 31769 bytes/s
*** STOP RECORDING ***
```
