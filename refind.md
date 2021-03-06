## Swap分区异常

笔记本更换硬盘了，用ghost做了一下disk-to-disk拷贝，swap分区的uuid信息丢失，但是在系统的grub里面还有这个记录，启动和关闭系统时，有如下报错:

```bash
ERROR: resume: hibernation device 'UUID=566cabc8-dd95-4290-9f8e-9eeewe76042e' not found
```

解决：

- sudo pacman -S gparted

- sudo gparted

- 图形化界面找到swap分区，右键-格式化成linux_swap分区，右键-新UUID

- 查看新uuid

  ```
  # blkid
  /dev/sda1: UUID="9F05-D9BC" BLOCK_SIZE="512" TYPE="vfat" PARTUUID="241c1510-9b70-4ebf-a3ee-c0e6438ca409"
  /dev/sda2: UUID="969e7821-5adb-48bf-af1a-2f83e9306070" BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="d170aa85-daaa-460e-a0af-ab806e8b581a"
  /dev/sda3: UUID="5417bf88-3d1f-4784-80f6-5b9bfd44ec52" TYPE="swap" PARTUUID="6422b4e6-31b5-4173-915d-2fbd90deca6e"
  /dev/sdb1: LABEL="H" UUID="DA18-EBFA" BLOCK_SIZE="512" TYPE="vfat" PARTLABEL="EFI system partition" PARTUUID="a80e041a-9ce9-4295-a5dc-e7113de615e4"
  /dev/sdb2: LABEL="M-gM-3M-;M-gM-;M-^_" BLOCK_SIZE="512" UUID="BCEA8B54EA8B0A3A" TYPE="ntfs" PARTLABEL="Basic data partition" PARTUUID="ed334da0-59ae-425f-9bc4-4b7bb4705974"
  /dev/sdb3: LABEL="M-fM-^UM-0M-fM-^MM-." BLOCK_SIZE="512" UUID="2D97AD940A9AD661" TYPE="ntfs" PARTLABEL="Basic data partition" PARTUUID="5b6e051b-9acd-4305-883a-5587a9f335bf"
  
  ```

  /dev/sda3: UUID=5417bf88-3d1f-4784-80f6-5b9bfd44ec52 就是我们要查的新的UUID

- vi /etc/default/grub 修改 GRUB_CMDLINE_LINUX_DEFAULT行resume=UUID=*新UUID*

- sudo update-grub

  注意:不推荐直接修改/boot/grub/grub.cfg文件中的内容，而是通过上面的命令重新生成grub信息

## rEFInd引导

笔记本有两个硬盘位，分别安装了Win10和Manjaro系统，每次都要按F12来选择引导，比较麻烦。采用rEFInd来引导两个系统。

- 查看磁盘第一个分区的相关信息

```bash
# sgdisk -i 1 /dev/sda
Partition GUID code: C12A7328-F81F-11D2-BA4B-00A0C93EC93B (EFI system partition)
Partition unique GUID: 241C1510-9B70-4EBF-A3EE-C0E6438CA409
First sector: 2048 (at 1024.0 KiB)
Last sector: 4268031 (at 2.0 GiB)
Partition size: 4265984 sectors (2.0 GiB)
Attribute flags: 0000000000000000
Partition name: ''

# sgdisk -i 1 /dev/sdb
Partition GUID code: C12A7328-F81F-11D2-BA4B-00A0C93EC93B (EFI system partition)
Partition unique GUID: A80E041A-9CE9-4295-A5DC-E7113DE615E4
First sector: 2048 (at 1024.0 KiB)
Last sector: 616447 (at 301.0 MiB)
Partition size: 614400 sectors (300.0 MiB)
Attribute flags: 8000000000000000
Partition name: 'EFI system partition'

# sgdisk -i 2 /dev/sdb
Partition GUID code: EBD0A0A2-B9E5-4433-87C0-68B6B72699C7 (Microsoft basic data)
Partition unique GUID: ED334DA0-59AE-425F-9BC4-4B7BB4705974
First sector: 616448 (at 301.0 MiB)
Last sector: 190207999 (at 90.7 GiB)
Partition size: 189591552 sectors (90.4 GiB)
Attribute flags: 0000000000000000
Partition name: 'Basic data partition'
# 这是查看第二磁盘的第二分区的GUID，因为操作系统安装在这个分区
```

记录下/dev/sda1:`Partition unique GUID: 241C1510-9B70-4EBF-A3EE-C0E6438CA409`

/dev/sdb1:`Partition unique GUID: A80E041A-9CE9-4295-A5DC-E7113DE615E4`

这两个是两块磁盘的第一个EFI分区的GUID。

/dev/sdb2:`Partition unique GUID: ED334DA0-59AE-425F-9BC4-4B7BB4705974`

这是第二块磁盘的第二个分区的GUID

* 将第二块磁盘的EFI分区挂载，拷贝EFI/Microsoft至第一块磁盘/boot/efi/EFI/目录，之所以拷贝至第一块磁盘，是因为refind将安装在第一块磁盘。

* 安装rEFInd

* 配置rEFInd，vi /boot/efi/EFI/refind/refind.conf，文件末尾添加以下配置

  ```bash
  dont_scan_volumes 241C1510-9B70-4EBF-A3EE-C0E6438CA409,A80E041A-9CE9-4295-A5DC-E7113DE615E4
  # 这里就是上面我们查到的两块盘的第一个EFI分区，因为rEFInd会默认扫描所有的分区，所以会出现很多重复的启动项，这里先禁掉扫描的卷，要启动哪些项我们自己来定义
  scan_all_linux_kernels false
  
  include themes/rEFInd-minimal/theme.conf
  # 下载了一个rEFInd-minimal的主题，这个是为了美化，可以不做
  
  menuentry "Arch Linux" {     
      icon \EFI\refind\themes/rEFInd-minimal/icons/os_arch.png	# 启动项的图标，这里用的是主题里面的图标
      loader \EFI\Manjaro\grubx64.efi
  } 
  
  menuentry "Windows 10" {     
      icon \EFI\refind\themes/rEFInd-minimal/icons/os_win8.png
      volume ED334DA0-59AE-425F-9BC4-4B7BB4705974	# 引导的卷在/dev/sdb2
      loader \EFI\Microsoft\Boot\bootmgfw.efi
  }
  ```

  

