
1. Remove exsiting autoconf package
```
$ rpm -e autoconf
error: Failed dependencies:
	autoconf >= 2.65 is needed by (installed) automake-1.13.4-3.el7.noarch
	autoconf is needed by (installed) libtool-2.4.2-22.el7_3.x86_64
[root@xphobia]# rpm -e --nodeps autoconf
```

2. Build autoconf from source
```
wget http://ftp.gnu.org/gnu/autoconf/autoconf-2.71.tar.xz
xz -d autoconf-2.71.tar.xz
tar xf autoconf-2.71.tar.xz
cd autoconf-2.71
./configure
make
make install
```
