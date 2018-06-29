# 官方Android模拟器集成Xposed

Xposed框架分为前端管理程序(Xposed Installer)和后端支持框架(Xposed Framework)。真机上面安装Xposed框架，有如下两种方式：

1. 先安装Xposed Installer，直接使用它安装整个后端框架
2. 手机先刷入第三方recovery，下载Xposed Framework的recovery包后用第三方recovery直接刷入

在模拟器上使用上面两种方法都是行不通的，因为无论使用哪一种方式刷入，都需要对system分区进行写入，而在模拟器上写入的system分区内容在重新启动以后均会消失。

在研究了xposed刷入系统的过程以后，发现它对系统的一些文件进行了替换，所以想到可以直接对模拟器的镜像文件进行修改，将Xposed直接整合到系统里面。
要进行如上操作，我们需要两个文件：

* 模拟器的system.img文件，此镜像会挂载到/system，在android-sdk的system-images相应的目录下可以找到
* 对应架构和API版本的刷机包，可以到[XDA论坛](https://dl-xda.xposed.info/framework/)找到相应的

以集成于API23，x86架构的Xposed为例，刷入的脚本如下，需要root执行:

``` shell
#! /usr/bin/env bash

SYSTEM_MOUNT_POINT=/tmp/xposed/system
XPOSED_UNZIP_DIR=/tmp/xposed/xposed

if [ ! -f ./system.img ]; then
    echo "no system image"
    exit 1
fi

mkdir -p ${SYSTEM_MOUNT_POINT}
mkdir -p ${XPOSED_UNZIP_DIR}
mount -t ext4 -o loop system.img ${SYSTEM_MOUNT_POINT}
unzip -o -K $(pwd)/xposed-v89-sdk23-x86.zip -d ${XPOSED_UNZIP_DIR}

error_exit() {
    umount ${SYSTEM_MOUNT_POINT}
    rm -rf ${XPOSED_UNZIP_DIR}
    exit 1
}

cp_perm() {
    cp -f $1 $2 || error_exit
    set_perm $2 $3 $4 $5
}

set_perm() {
    chown $2:$3 $1 || error_exit
    chmod $4 $1 || error_exit
}

install_nobackup() {
    file=$1
    cp_perm ${XPOSED_UNZIP_DIR}$1 ${SYSTEM_MOUNT_POINT}/${file:8} $2 $3 $4
}

install_and_link() {
    TARGET=$1
    XPOSED="${1}_xposed"
    BACKUP="${1}_original"
    TARGET=${SYSTEM_MOUNT_POINT}/${TARGET:8}
    XPOSED=${SYSTEM_MOUNT_POINT}/${XPOSED:8}
    BACKUP=${SYSTEM_MOUNT_POINT}/${BACKUP:8}
    if [ ! -f ${XPOSED_UNZIP_DIR}/${XPOSED:12} ]; then
        return
    fi
    cp_perm ${XPOSED_UNZIP_DIR}/${XPOSED:12} ${XPOSED} $2 $3 $4
    if [ ! -f ${BACKUP} ]; then
        mv ${TARGET} ${BACKUP} || error_exit
        ln -sf ${XPOSED##*/} ${TARGET} || error_exit
        chown -h 0:2000 ${TARGET}
    fi
}

install_overwrite() {
    TARGET=$1
    if [ ! -f ${XPOSED_UNZIP_DIR}${TARGET} ]; then
        return
    fi
    BACKUP=${SYSTEM_MOUNT_POINT}/${TARGET:8}.orig
    NO_ORIG=${SYSTEM_MOUNT_POINT}/${TARGET:8}.no_orig
    if [ ! -f ${XPOSED_UNZIP_DIR}${TARGET} ]; then
        touch ${NO_ORIG} || error_exit
        set_perm ${NO_ORIG} 0 0 0600
    elif [ -f ${BACKUP} ]; then
        rm -f ${SYSTEM_MOUNT_POINT}/${TARGET:8}
        gzip ${BACKUP} || error_exit
        set_perm "${BACKUP}.gz" 0 0 0600
    elif [ ! -f "${BACKUP}.gz" -a ! -f ${NO_ORIG} ]; then
        mv ${SYSTEM_MOUNT_POINT}/${TARGET:8} ${BACKUP} || error_exit
        gzip ${BACKUP} || error_exit
        set_perm "${BACKUP}.gz" 0 0 0600
    fi
    cp_perm ${XPOSED_UNZIP_DIR}${TARGET} ${SYSTEM_MOUNT_POINT}/${TARGET:8} $2 $3 $4
}

echo "- Placing files"
install_nobackup /system/xposed.prop 0 0 0644
install_nobackup /system/framework/XposedBridge.jar 0 0 0644

install_and_link /system/bin/app_process32 0 2000 0755
install_overwrite /system/bin/dex2oat 0 2000 0755
install_overwrite /system/bin/oatdump 0 2000 0755
install_overwrite /system/bin/patchoat 0 2000 0755
install_overwrite /system/lib/libart.so 0 0 0644
install_overwrite /system/lib/libart-compiler.so 0 0 0644
install_overwrite /system/lib/libart-disassembler.so 0 0 0644
install_overwrite /system/lib/libsigchain.so 0 0 0644
install_nobackup /system/lib/libxposed_art.so 0 0 0644

umount ${SYSTEM_MOUNT_POINT}
if [ $? -eq 0 ]; then
    rm -rf ${SYSTEM_MOUNT_POINT}
else
    echo "umount system.img error"
fi
rm -rf ${XPOSED_UNZIP_DIR}

echo "- Done"
exit 0
```