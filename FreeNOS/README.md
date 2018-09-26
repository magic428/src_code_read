## Free Niek's Operating System.

Document: http://www.freenos.org/doxygen/index-2.html

Authors:

    Niek Linnenbank
    Dan Rulos (AmayaOS)
    Coen Bijlsma (libparse, srv/time)
    Ed Schouten (libteken)
    Alexander Schrijver (OpenBSD patches)

## 编译前的准备工作 

```bash
sudo apt-get install scons xorriso -y
```

## 工程支持的命令

Building
$ scons iso

Booting under Qemu
$ scons qemu

Booting under Qemu with Curses console
$ scons qemu_curses

Booting under Qemu with SDL console
$ scons qemu_sdl

Booting under Bochs
$ scons bochs

Debugging under Qemu
$ scons qemu_debug

Debugging under Bochs
$ scons bochs_debug

Cleaning up
$ scons -c .

Creating a snapshot
$ scons snapshot

Creating a release
$ scons release


## 编译 mov 报错.   

https://blog.csdn.net/geek_tank/article/details/80145466   

## 相关工具  

scons浅入浅出: https://blog.csdn.net/flyingcys/article/details/61196557  
