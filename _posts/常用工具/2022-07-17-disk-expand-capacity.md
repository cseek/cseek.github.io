---
title: linux 扩展磁盘容量工具
category: [常用工具]
tags: [gparted, parted, 扩容]
---

> 最近在使用 jetson Orin NX T801开发板的时候遇到个问题，使用官方提供的脚本备份系统镜像后（SSD为 128 G），当把这个系统恢复到 512G 的新 SSD 后，系统只使用了 128G，我现在想把剩余的477G扩展给根目录，该怎么操作？下面有两种方法可供参考（亲测有效）。 `注意`: 在进行磁盘分区和文件系统操作时，务必备份重要数据，以防止操作失误，数据丢失。 
{: .prompt-info }

## 检查当前分区情况： 

打开终端，输入以下命令查看当前的分区情况：

```bash
   lsblk
```
终端输出如下信息：

```bash
user@user-desktop:~$ sudo lsblk
[sudo] password for user: 
NAME         MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
loop0          7:0    0    16M  1 loop 
zram0        251:0    0 961.8M  0 disk [SWAP]
zram1        251:1    0 961.8M  0 disk [SWAP]
zram2        251:2    0 961.8M  0 disk [SWAP]
zram3        251:3    0 961.8M  0 disk [SWAP]
zram4        251:4    0 961.8M  0 disk [SWAP]
zram5        251:5    0 961.8M  0 disk [SWAP]
zram6        251:6    0 961.8M  0 disk [SWAP]
zram7        251:7    0 961.8M  0 disk [SWAP]
nvme0n1      259:0    0   477G  0 disk 
├─nvme0n1p1  259:1    0 118.5G  0 part /
├─nvme0n1p2  259:2    0    64M  0 part 
├─nvme0n1p3  259:3    0   448K  0 part 
├─nvme0n1p4  259:4    0    32M  0 part 
├─nvme0n1p5  259:5    0    64M  0 part 
├─nvme0n1p6  259:6    0   448K  0 part 
├─nvme0n1p7  259:7    0    32M  0 part 
├─nvme0n1p8  259:8    0    80M  0 part 
├─nvme0n1p9  259:9    0   512K  0 part 
├─nvme0n1p10 259:10   0   300M  0 part 
├─nvme0n1p11 259:11   0    64M  0 part 
├─nvme0n1p12 259:12   0    80M  0 part 
├─nvme0n1p13 259:13   0   512K  0 part 
└─nvme0n1p14 259:14   0    64M  0 part 
```
可以看到 nvme0n1 还有 477G 未使用，下面我们准备把这 477G 扩展给根目录。

## 方法一：使用图形化工具 gparted（极力推荐）

+ 安装了 gparted

```bash
sudo apt update
sudo apt install gparted
```

+ 运行 gparted

``` bash
sudo gparted
```

+ 在 gparted 中，你可以看到所有的分区。找到你的根分区（通常是 /dev/nvme0n1p1 或类似名称），然后尝试将未分配的空间扩展到根分区。右键点击根分区，选择 "Resize/Move" ，然后拖动边界来扩展分区，点击 "Resize" 按钮。

+ 点击菜单栏的 ✅ 按钮来确认你的更改。

+ 重启系统。

## 方法二： 使用命令行工具 parted（慎用，慎用，慎用， 容易操作失误）

+ 打开终端，输入以下命令启动 parted ：

```bash
user@user-desktop:~$ sudo parted /dev/nvme0n1
GNU Parted 3.3
Using /dev/nvme0n1
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) 
```

+ 输入 print 查看分区信息。
 
```bash
(parted) print                                                            
Error: The backup GPT table is not at the end of the disk, as it should be.
Fix, by moving the backup to the end (and removing the old backup)?
Fix/Ignore? Fix
Warning: Not all of the space available to /dev/nvme0n1 appears to be used, you
can fix the GPT to use all of the space (an extra 750145542 blocks) or continue
with the current setting? 
Fix/Ignore? Fix                                                           
Model: FORESEE XP1000F512G (nvme)
Disk /dev/nvme0n1: 512GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags: 

Number  Start   End     Size    File system  Name                       Flags
 2      20.5kB  67.1MB  67.1MB               kernel                     msftdata
 3      67.1MB  67.6MB  459kB                kernel-dtb                 msftdata
 4      67.6MB  101MB   33.6MB               reserved_for_chain_A_user  msftdata
 5      101MB   168MB   67.1MB               kernel_b                   msftdata
 6      168MB   169MB   459kB                kernel-dtb_b               msftdata
 7      169MB   202MB   33.6MB               reserved_for_chain_B_user  msftdata
 8      202MB   286MB   83.9MB               recovery                   msftdata
 9      286MB   287MB   524kB                recovery-dtb               msftdata
10      287MB   601MB   315MB                RECROOTFS                  msftdata
11      601MB   668MB   67.1MB  fat32        esp                        boot, esp
12      668MB   752MB   83.9MB               recovery_alt               msftdata
13      752MB   753MB   524kB                recovery-dtb_alt           msftdata
14      753MB   820MB   67.1MB               esp_alt                    msftdata
 1      820MB   128GB   127GB   ext4         APP                        msftdata

(parted)
```

+ 删除根分区（注意：这不会删除数据，但操作前请确保已备份重要数据）：

```bash
(parted) rm 1                                                             
Warning: Partition /dev/nvme0n1p1 is being used. Are you sure you want to
continue?
Yes/No? Yes                                                               
Error: Partition(s) 1 on /dev/nvme0n1 have been written, but we have been unable
to inform the kernel of the change, probably because it/they are in use.  As a
result, the old partition(s) will remain in use.  You should reboot now before
making further changes.
Ignore/Cancel? Ignore
(parted)
```

+ 重新创建分区，使用 mkpart 命令并指定新的大小

```bash
(parted) mkpart primary ext4 1MiB 100%                                    
Warning: You requested a partition from 1049kB to 512GB (sectors
2048..1000215215).
The closest location we can manage is 820MB to 512GB (sectors
1601320..1000215182).
Is this still acceptable to you?
Yes/No? Yes                                                               
Warning: The resulting partition is not properly aligned for best performance:
1601320s % 2048s != 0s
Ignore/Cancel? Ignore                                                     
(parted)
```

+ 输入 quit 退出 parted 

```bash
(parted) quit                                                             
Information: You may need to update /etc/fstab.
```

+ 扩展 

```bash
user@user-desktop:~$ sudo resize2fs /dev/nvme0n1p1                    
resize2fs 1.45.5 (07-Jan-2020)
Filesystem at /dev/nvme0n1p1 is mounted on /; on-line resizing required
old_desc_blocks = 15, new_desc_blocks = 60
The filesystem on /dev/nvme0n1p1 is now 124826732 (4k) blocks long.
```

+ 重启系统 

完成上述步骤后，重启计算机，确保一切正常。 


 