# petalinux设计流程：

​	1、搭建vivado，导出hdf文件（带文件夹传到linux中）

​	2、创建petalinux

​	3、将hdf文件导入到petalinux

​	4、配置petalinux工程

​	5、编译petalinux工程

​	6、启动开发板

# petalinux实战：

​	petalinux-boot 启动开发板

​	petalinux-build 编译

​	petalinux-config 配置

​	petalinux-create 创建

​	petalinux-package 打包

​	petalinux-build -x mrproper -f	清除生成的文件

​	首先source setting.sh，对petalinux环境进行配置（每次打开一个新的终端都需要配置）

# petalinux流程例子：

## 	1、创建petalinux工程

​			petalinux-create -t project - -template zynq -n LED-ZYNQ （通过hdf文件 模板zynqMP）  创建一个名字为LED-ZYNQ的，ZYNQ模板的工程

​			petalinux-create -t project -s avnet-digilent-zedboard-v2018.3-final.bsp -n my_zed（通过bsp）



## 	2、导入hdf文件，并配置petalinux（需要进入工程目录）

​			petalinux-config - -get-hw-description /home/hwusr/test/（.xsa文件路径的上级目录）导入.xsa文件，.xsa文件位置在xsa文件夹里面

​			如果想单独打开配置petalinux界面：petalinux-config



## 	3、配置uboot

​			petalinux-config -c u-boot



## 	4、配置kernel（内核）

​			petalinux-config -c kernel



## 	5、配置rootfs（根文件系统）

​			petalinux-config -c rootfs



## 	6、编译工程

​		（1）编译整个petalinux工程

​			petalinux-build

​		（2）单独编译（u-boot、kernel、rootfs）

​			petalinux-build -c rootfs （编译根文件）

​		注：如何一直卡着不动，很有可能是bitbake配置失败，解决办法：删除build文件夹



## 	7、制作启动镜像BOOT.bin文件（进入image文件夹）

​		petalinux-package - -boot - -fsbl - -fpga - -u-boot - -force

​			\- -boot：通过package命令生成BOOT.bin文件

​			\- -fsbl： 指定fsbl镜像文件路径 zynq_fsbl.elf

​			\- -fpga: 指定bit文件路径 system.bit

​			\- -u-boot：指定u-boot文件路径（用户程序路径） u-boot.elf

​			\- -force：强制覆盖（如果当前路径下面有.bin文件，则进行覆盖）



​		petalinux-package --boot --fsbl ./zynq_fsbl.elf --fpga ./system.bit --u-boot ./u-boot.elf --force（zynq）

​		petalinux-package --boot --format BIN --fsbl ./zynqmp_fsbl.elf --u-boot ./u-boot.elf --pmufw ./pmufw.elf --fpga system.bit --force（zynqMP）



**FLASH启动的时候**：把内核和boot.src如果都放到BOOT里面的话，需要对地址进行分配

​		petalinux-package --boot --force --format BIN --fsbl --fpga --pmufw --u-boot --kernel images/linux/Image --offset 0x1940000 --cpu a53-0 --boot-script --offset 0x3240000

**FLASH启动ramdisk**：（即根文件系统也在FLASH）（需要修改根文件位置为INRD并配置地址）

```
petalinux-package --boot --force --format BIN --fsbl --pmufw --u-boot --kernel images/linux/Image --offset 0x1E40000 --cpu a53-0 --boot-script --offset 0x3E80000 --add images/linux/rootfs.cpio.gz.u-boot --offset 0x4000000 --cpu a53-0 --file-attribute partition_owner=uboot
```



## 	8、制作SD启动卡

​		**2019.2版本**

​			**第一种：只需分一个分区FAT32**

​				对于ZYNQ硬件平台来说，启动嵌入式Linux系统需要这些文件

​				1、BOOT.bin（fsbl镜像文件、bit流文件、u-boot文件）

​				2、image.ub（kernel、设备树、rootfs）

​				将镜像文件（BOOT.bin、image.ub）拷贝到SD卡的FAT32分区，插入开发板启动

​		**第二种：把SD卡分为两个分区FAT32和ext4，需要用linux系统操作 **（zynqMP）

​				1、把BOOT.bin、Image、system.dtb文件放到FAT32分区

​				2、把rootfs.tar.gz压缩包解压缩到ext4分区

​		**2021.2版本**

​			**把SD卡分为两个分区FAT32和ext4，需要用linux系统操作**

​				1、把BOOT.BIN, boot.scr, Image, and ramdisk.cpio.gz.u-boot(MicroBlaze才需要) 放到FAT32分区	（zynqMP）

​					如果是zynq系列的话，需要把Image换成image.ub

​				2、把rootfs.tar.gz压缩包解压缩到ext4分区

​					\$ sudo tar xvf rootfs.tar.gz -C /media/rootfs



## 	9、创建APP应用、驱动

​			petalinux-creat -t apps --template c -n first-app --enable （添加APP程序）

​			petalinux-creat -t modules -n first-modules --enable （添加驱动程序）



# 设置网络IP：

