### porxy
- `export http_proxy=http://192.168.22.3:7890 ; export https_proxy=http://192.168.22.3:7890`
- `env|grep -i proxy`
- `unset http_proxy ; unset https_proxy`
## server
- 分区情况：所有分区都在/dev/sda上(TiB/GiB>TB/GB)
	- 如果需要缩小已有分区，需要使用`parted`工具来缩减已有分区空间
	- 建议创建分区时使用`fdisk`，因为`parted`需要算结束分区的数目，`fdisk`不需要去算分区，只需要输入分配空间大小
	- 使用`parted`来改变`bios`分区的类型
	- 格式化：etx4 一律用` sudo mkfs -v -t ext4 /dev/sda14`或` sudo mkfs.ext4 /dev/sda14`，tmpfs格式化成etx4，FAT32/vfat使用`parted`，swap使用`sudo mkswap /dev/sda21`
	- 可以使用`parted`来对分区重命名
	- 以下分区需要单独分配
	- 设备号由两部分组成，主设备号：次设备号
```text
num | space | usage | describe | type
13 2M bios bios安装启动引导器 none(needn't, only need to set the flag of 'bios_grub')
14 25G / 根目录 etx4
15 250M /boot 内核及其他引导信息 etx4
16 300M /boot/efi UEFI引导系统 vfat(FAT32)
17 20G /home 不同发行版linux共享home目录 etx4
18 5G /opt 大型软件 etx4
19 5G /tmp 加速访问临时文件 tmpfs(need to format explicit as etx4)
20 30G /usr/src 多个LFS系统共享及编译BLFS软件包 etx4
21 2G swap 缓存 swap
```
- 由于我想在docker中进行，而docker由于安全策略，不能直接挂载设备，即使是通过设备号也不行，所以就需要nsenter来做这件事
```bash
# set the common command
export PID=$(docker inspect --format "{{.State.Pid}}" "$CONTAINER")
export run_command="nsenter --target $PID --mount --uts --ipc --net --pid -- sh -c"

# loop for the all parts of disk without swap area
# DEV is the name of area(not the name of the display of the Tool of "parted"!!!!!), DEVDEV is the "MAJ MIN"
# CONTPATH is  path/to/container/mount/dir, SUBROOT&SUBPATH arenot important, they can be deleteed in this situation(because this command is used to bind the dir of host, I'm confusion!!!!!)
# start loop
sudo $run_command "[ -b $DEV ] || mknod --mode 0600 $DEV b $DEVDEC"
#this step could be execute onece
sudo $run_command "mkdir /tmpmnt"
sudo $run_command "mount $DEV /tmpmnt"
sudo $run_command "mkdir -p $CONTPATH"
sudo $run_command "mount -o bind /tmpmnt/$SUBROOT/$SUBPATH $CONTPATH"
sudo $run_command "umount /tmpmnt"
#this step could be execute onece
sudo $run_command "rmdir /tmpmnt"
# end loop

# for the swap area
sudo $run_command "mknod --mode 0600 /dev/sda21 b 259 5"
sudo $run_command "swapon /dev/sda21"
```


## FS_whq
```bash
# passwd 123
docker run -idt --name=FS_whq --gpus=all --shm-size=24g -v /mnt/shared:/root/.cache -p 2264:22 8924:8888 8130:80 44313:443 21013:21 20013:20 ubuntu:22.04 

# 确认启用UNIX98伪终端（PTY）
ls /dev/pts
#zcat /proc/config.gz | grep CONFIG_UNIX98_PTYS
ls -l /dev/ptmx
#cat /boot/config-$(uname -r) | grep CONFIG_UNIX98_PTYS
sysctl -a | grep unix98
ls -l /dev/ptmx
#screen
ssh
ps -aux | grep pts

# install texinfo
# notice: should be install both of texinfo & info to use the command of info
apt install texinfo
apt install info
info --version


# 2.6 export the enviorment of value
# use '/etc/profile' to make **always** in **every ssh**
# '~/.bash_profile': 适合放置登录时需要执行一次的配置，如：设置环境变量，定义别名，运行一些启动脚本
# '~/.bashrc': 适合放置每次打开新终端都需要生效的配置，如：设置终端提示符的颜色、格式，定义常用的函数或命令别名，设置 shell 的一些选项
# maybe don't use execute everytime because of the '/etc/profile '
# in this book, everytime for login 'lfs', should be logining from root, because of root have the '$LFS', the 'lfs' will be inherited from 'root'
echo "export LFS=/mnt/lfs" | sudo tee -a /etc/profile


```