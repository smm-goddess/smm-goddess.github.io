# x86模拟器集成houdini支持arm

现在很多App都使用JNI来做非常多的工作，相较多于使用Java来讲，使用JNI在安全和效率上来讲都有较大的优势。Android支持的CPU架构多种多样，不同架构的CPU执行指令有所不同，打包而成的JNI库也就不一样，如果将所有架构的JNI库均打包入APK，全使APK非常臃肿，所以厂商提供的APK一般只包含有最流行的arm架构下的JNI库。而在x86的机器上运行arm版本的模拟器，执行效率非常低下，所以现在的问题就是通过一定的技术手段，让x86架构的模拟器能够运行arm版本的JNI库。

Intel提供了一个用于将arm指令转换为x86指令的ARM Binary Translation库，也就是houdini运行库，它能够执行从arm指令转换为x86指令的二进制翻译。Android系统中也提供了对native代码进行hook的native bridge。官方模拟器没有启用二进制翻译，也没有相应的houdini库，所以需要自行处理模拟器的镜像文件启用并支持native bridge。

具体的实现步骤示例如下：

1. 创建一个模拟器，使用Intel Atom x86 Image镜像，Api Level 23(Android 6.0)
2. 从Google发布的Nexus Player(使用了Atom的CPU)[二进制文件][1]中，拷贝Android 6.0的Native Bridge驱动，或者从[Android-x86 Project][2]下载Android 6.0的[Native Bridge驱动][3]
3. 修改AVD的system.img和ramdisk.img，开启对Native Bridge的支持。主要修改的地方如下：
    * system.img
        1. 将native bridge文件夹拷贝到`system/lib/arm`目录
        2. libhoudini.so拷贝到`/system/lib`，houdini拷贝到`/system/bin`
        3. 修改build.prop文件

                ro.product.cpu.abi2=armeabi-v7a
                ro.product.cpu.abilist=x86,armeabi-v7a,armeabi
                ro.product.cpu.abilist32=x86,armeabi-v7a,armeabi
                ro.dalvik.vm.isa.arm=x86
                ro.enable.native.bridge.exec=1
    * ramdisk.img
        1. 修改default.prop

                 ro.dalvik.vm.native.bridge=libhoudini.so

***注:**

* 模拟器启动时需要加上`-selinux permissive`或者在模拟器启动后`adb root && adb shell setenforce 0`禁用selinux

* 修改system.img的方法

    1. `mount -t ext4 -o loop system.img /tmp/houdini/system`
    2. 直接在目录中进行拷贝、删除文件操作
    3. 对文件进行正确的`chown`、`chmod`
    4. `umount /tmp/houdini/system`将修改写入镜像
    5. 说明：如果在写入的时候警告**空间不足**，是因为system.img本身大小不足，此时需要对它进行扩容，用到的命令主要有

            增加镜像至3096M：
              resize2fs system.img 3096M
            把镜像缩至最小：
              resize2fs -M system.img
            显示镜像里真正文件的大小：
              resize -p system.img

* 修改ramdisk.img的方法
    1. `mv ramdisk.img ramdisk.img.gz`
    2. `gunzip ramdisk.img.gz`
    3. `mkdir ramdisk`
    4. `cd ramdisk`
    5. `cpio -i -F ../ramdisk.img`
    6. 根据需求，对ramdisk里面的文件进行操作
    7. `cpio -i -t -F ../ramdisk.img > list`从ramdisk.img里面提取文件名，重定向到list里面，如果在上面的操作中在ramdisk里面进行了文件的添加、删除操作，需要把list里面的相应文件名同样进行添加、删除
    8. `cpio -o -H newc -O new_ram.img < list`
    9. `gzip new_ram.img`
    10. `mv new_ram.img.gz ../ramdisk.new.img`

* 对system.img和ramdisk.img进行操作的脚本，以`system-images;android-23;default;x86`以及Android-x86的[native-bridge][3]为例

``` shell
#!/usr/bin/evn bash

SYSTEM_MOUNT_POINT=/tmp/houdini/system
BRIDGE_UNZIP_FOLDER=/tmp/houdini/bridge
RAMDISK_UNPACK_FOLDER=/tmp/houdini/ramdisk

# TODO adjust system.img file size to make space for houdini files
mkdir -p ${SYSTEM_MOUNT_POINT}
mkdir -p ${BRIDGE_UNZIP_FOLDER}

# mount system.img and houdini file
mount -t ext4 -o loop system.img ${SYSTEM_MOUNT_POINT}
if [ $? -eq 0 ]; then
    echo "mount system.img success"
else
    echo "mount system.img fail"
fi
mount -t squashfs -o loop houdini.sfs ${BRIDGE_UNZIP_FOLDER}
if [ $? -eq 0 ]; then
    echo "mount houdini.sfs success"
else
    echo "mount houdini.sfs fail"
    umount ${SYSTEM_MOUNT_POINT}
fi

# copy houdini files to /system/lib/arm
mkdir ${SYSTEM_MOUNT_POINT}/lib/arm
cp -R ${BRIDGE_UNZIP_FOLDER}/* ${SYSTEM_MOUNT_POINT}/lib/arm

# umount houdini file and remove temp files
umount ${BRIDGE_UNZIP_FOLDER}
rm -rf ${BRIDGE_UNZIP_FOLDER}

error_exit() {
    umount ${SYSTEM_MOUNT_POINT}
    rm -rf ${BRIDGE_UNZIP_FOLDER}
    exit 1
}

set_perm() {
    chown -R $2:$3 $1 || error_exit
    chmod -R $4 $1 || error_exit
}

# chown chmod
set_perm ${SYSTEM_MOUNT_POINT}/lib/arm 0 0 0644
chmod 0755 ${SYSTEM_MOUNT_POINT}/lib/arm ${SYSTEM_MOUNT_POINT}/lib/arm/nb

# move /system/lib/arm/libhoudini.so -> /system/lib/libhoini.so
# move /system/lib/arm/houdini -> /system/bin/houdini.so
mv ${SYSTEM_MOUNT_POINT}/lib/arm/libhoudini.so ${SYSTEM_MOUNT_POINT}/lib/libhoudini.so
mv ${SYSTEM_MOUNT_POINT}/lib/arm/houdini ${SYSTEM_MOUNT_POINT}/bin/houdini

set_perm ${SYSTEM_MOUNT_POINT}/bin/houdini 0 2000 0755

# modify build.prop
echo "ro.dalvik.vm.isa.arm=x86" >>${SYSTEM_MOUNT_POINT}/build.prop
echo "ro.enable.native.bridge.exec=1" >>${SYSTEM_MOUNT_POINT}/build.prop
sed -i "s/^ro.product.cpu.abi=x86/&\nro.product.cpu.abi2=armeabi-v7a/" ${SYSTEM_MOUNT_POINT}/build.prop
sed -i "s/^ro.product.cpu.abilist=x86/ro.product.cpu.abilist=x86,armeabi,armeabi-v7a/" ${SYSTEM_MOUNT_POINT}/build.prop
sed -i "s/^ro.product.cpu.abilist32=x86/ro.product.cpu.abilist32=x86,armeabi,armeabi-v7a/" ${SYSTEM_MOUNT_POINT}/build.prop

# umount system.img
umount ${SYSTEM_MOUNT_POINT}
rm -rf ${SYSTEM_MOUNT_POINT}

# modify ramdisk.img
MAIN_FOLDER=$(pwd)
mkdir -p ${RAMDISK_UNPACK_FOLDER}
cp ramdisk.img ${RAMDISK_UNPACK_FOLDER}/ramdisk.img.gz
cd ${RAMDISK_UNPACK_FOLDER}
gunzip ramdisk.img.gz
cpio -i -F ramdisk.img
sed -i "s/ro.dalvik.vm.native.bridge=0/ro.dalvik.vm.native.bridge=libhoudini.so/" default.prop
cpio -i -t -F ramdisk.img >list
cpio -o -H newc -O new_ram.img <list
gzip new_ram.img
mv new_ram.img.gz ${MAIN_FOLDER}/ramdisk_new.img

cd ${MAIN_FOLDER}
rm -rf ${RAMDISK_UNPACK_FOLDER}

echo "Done"
 ```

[1]:https://developers.google.com/android/nexus/drivers#fugu
[2]:http://www.android-x86.org
[3]:http://dl.android-x86.org/houdini/6_x/houdini.sfs