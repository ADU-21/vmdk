
# vmdk 扩容 
## 在vitrualbox的宿主机上执行如下操作：
[http://www.awaimai.com/1194.html](http://www.awaimai.com/1194.html)
VBoxManage clonehd "source.vmdk" "cloned.vdi" --format vdi
VBoxManage modifyhd "cloned.vdi" --resize 51200
VBoxManage clonehd "cloned.vdi" "resized.vmdk" --format vmdk
以上，是将原来的vmdk扩容至50GB (50 * 1024MB)。

## 将stroage挂载之后，登录到目标Linux上执行：

```
sudo su
# 查看逻辑分区情况
lsblk
fdisk -l
# 新增逻辑分区
fdisk /dev/sda
n # new
primary partition 4 # 创建第四个分区
w # wirte
reboot
```
## 继续，处理分区：

```
# 格式化分区
ll /dev/sda*
mkfs.ext4 /dev/sda4
# LVM操作
vgdisplay # 查看卷组名 (LV GN)
pvcreate /dev/sda4  # 创建新物理卷
vgextend VolGroup00 /dev/sda4 # 扩展到卷组
vgdisplay # 查看卷组，这时候应该是有Free Size了
lvdisplay # 查看根分区 （LV PATH）
lvextend /dev/VolGroup/lv_root /dev/sda4
（报错： vgreduce --removemissing VolGroup）
resize2fs /dev/VolGroup/lv_root # 刷新逻辑分区容量
（centOS : fsadm resize /dev/VolGroup00/LogVol00）
df -h
```