常见问题列表
===========
[TOC]
<!-- toc -->

----

# 1. USB 烧录失败

换一根typeC 线或者换一个USB口重新尝试烧录，如果还是不能烧录成功，发送邮件到support@ingenic.com

---

# 2. 程序无法执行或者编译时链接失败
 
确保编译器使用 gcc version 7.2.0 (Ingenic r4.0.0-gcc720 2018.02-28) 

执行 mips-linux-gnu-gcc -v可以查看.
```
Target: mips-linux-gnu
Configured with: /home/toolchains/release/src/gcc-7-2017.11/configure --build=i686-pc-linux-gnu --host=i686-pc-linux-gnu --target=mips-linux-gnu --enable-threads --disable-libmudflap --disable-libssp --disable-libstdcxx-pch --with-arch-32=mips32r2 --with-arch-64=mips64r2 --with-float=hard --with-mips-plt --enable-extra-sgxxlite-multilibs --with-gnu-as --with-gnu-ld --with-specs='-D__CS_SOURCERYGXX_MAJ__=2017 -D__CS_SOURCERYGXX_MIN__=11 -D__CS_SOURCERYGXX_REV__=32' --enable-languages=c,c++,fortran,go --enable-shared --enable-lto --enable-symvers=gnu --enable-__cxa_atexit --with-glibc-version=2.26 --with-pkgversion='Ingenic r4.0.0-gcc720 2018.02-28' --disable-nls --prefix=/home/toolchains/release/install/opt/codesourcery --with-sysroot=/home/toolchains/release/install/opt/codesourcery/mips-linux-gnu/libc --with-build-sysroot=/home/toolchains/release/install/opt/codesourcery/mips-linux-gnu/libc --with-gmp=/home/toolchains/release/obj/pkg-2017.11-32-mips-linux-gnu/mips-2017.11-32-mips-linux-gnu.extras/host-libs-i686-pc-linux-gnu/usr --with-mpfr=/home/toolchains/release/obj/pkg-2017.11-32-mips-linux-gnu/mips-2017.11-32-mips-linux-gnu.extras/host-libs-i686-pc-linux-gnu/usr --with-mpc=/home/toolchains/release/obj/pkg-2017.11-32-mips-linux-gnu/mips-2017.11-32-mips-linux-gnu.extras/host-libs-i686-pc-linux-gnu/usr --with-isl=/home/toolchains/release/obj/pkg-2017.11-32-mips-linux-gnu/mips-2017.11-32-mips-linux-gnu.extras/host-libs-i686-pc-linux-gnu/usr --enable-libgomp --disable-libitm --enable-libatomic --disable-libssp --disable-libcc1 --enable-poison-system-directories --with-python-dir=mips-linux-gnu/share/gdb/python --with-build-time-tools=/home/toolchains/release/install/opt/codesourcery/mips-linux-gnu/bin --with-build-time-tools=/home/toolchains/release/install/opt/codesourcery/mips-linux-gnu/bin SED=sed
Thread model: posix
gcc version 7.2.0 (Ingenic r4.0.0-gcc720 2018.02-28) 
```

# 3. 如何区分/dev/video设备节点对应的设备
使用v4l2-sysfs-path -d 命令可以查看详细设备信息。
```
# v4l2-sysfs-path -d

Device platform:
        hw:0(sound card, dev 0:0) hw:0,5(pcm capture, dev 116:8) hw:0,6(pcm capture, dev 116:9) hw:0,7(pcm capture, dev 116:10) hw:0,8(pcm capture, dev 116:11) hw:0,9(pcm capture, dev 116:12) hw:0,0(pcm o
utput, dev 116:3) hw:0,1(pcm output, dev 116:4) hw:0,2(pcm output, dev 116:5) hw:0,3(pcm output, dev 116:6) hw:0,4(pcm output, dev 116:7) hw:0(mixer, dev 116:2) 
Device platform/ahb0/13070000.rotate:
        video0(video, dev 81:0) 
Device platform/ahb0/13200000.helix:
        video1(video, dev 81:1) 
Device platform/ahb0/13300000.felix:
        video2(video, dev 81:2) 
Device platform/ahb0/ahb0:isp-camera@0:
        video3(video, dev 81:3) v4l-subdev0(v4l subdevice, dev 81:4) v4l-subdev1(v4l subdevice, dev 81:5) v4l-subdev2(v4l subdevice, dev 81:6) v4l-subdev3(v4l subdevice, dev 81:7) v4l-subdev4(v4l subdevic
e, dev 81:8) 
Device platform/ahb0/ahb0:isp-camera@1:
        video4(video, dev 81:9) v4l-subdev5(v4l subdevice, dev 81:10) v4l-subdev6(v4l subdevice, dev 81:11) v4l-subdev7(v4l subdevice, dev 81:12) v4l-subdev8(v4l subdevice, dev 81:13) v4l-subdev9(v4l subd
evice, dev 81:14) 
Device virtual0:
        timer(sound timer, dev 116:33) 
```