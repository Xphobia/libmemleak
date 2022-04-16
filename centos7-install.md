
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
