# CMake中使用libzip的记录

环境：ubuntu

## 1、下载编译zlib

从zlib.net下载源码，参考其中的cmake命令进行编译安装

## 2、下载编译libzip

从libzip.org下载最新的libzip库，解压后编译安装，需要先安装zlib，否则会报错误

## 3、在引用的工程中引用libzip

修改引用工程的CMakeList，在CMakeList中添加以下信息：

```shell
find_library(LIB_ZIP zip)

target_link_libraries(myvm PUBLIC  "${LIB_ZIP}")
```

find_library是从系统中查找libzip的路径，将其设置为LIB_ZIP变量，然后将其加入到link路径中。通过ldd查看so的应用，可以看到libzip的so正常引用到

```shell
lenovo@lenovo-ThinkStation-K:~/github_upload/myjavaparser/myvm$ ldd myvm 
	linux-vdso.so.1 (0x00007ffe1c7f7000)
	libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007f8cf2bfe000)
	libzip.so.5 => /usr/local/lib/libzip.so.5 (0x00007f8cf2bda000)
	libstdc++.so.6 => /lib/x86_64-linux-gnu/libstdc++.so.6 (0x00007f8cf29f8000)
	libgcc_s.so.1 => /lib/x86_64-linux-gnu/libgcc_s.so.1 (0x00007f8cf29dd000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f8cf27eb000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f8cf2c90000)
	libz.so.1 => /usr/local/lib/libz.so.1 (0x00007f8cf27cd000)
	libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007f8cf267c000)

```

以上方法参考：https://stackoverflow.com/questions/8774593/cmake-link-to-external-library

网上的其他的方法都没有说清楚，无法解决总是出现以下问题，libzip无法找到：

```shell
lenovo@lenovo-ThinkStation-K:~/github_upload/myjavaparser/myvm$ ldd myvm 
	linux-vdso.so.1 (0x00007fff591ac000)
	libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007f4c47d27000)
	libzip.so.5 => not found
	libstdc++.so.6 => /lib/x86_64-linux-gnu/libstdc++.so.6 (0x00007f4c47b45000)
	libgcc_s.so.1 => /lib/x86_64-linux-gnu/libgcc_s.so.1 (0x00007f4c47b2a000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f4c47938000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f4c47db9000)
	libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007f4c477e9000)
```



