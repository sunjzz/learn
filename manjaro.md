1. 安装VirtualBox

   查看内核版本，下载对应版本包：
   
   ```
   [jiang@e450 ~]$ uname -a
   Linux e450 5.9.16-1-MANJARO #1 SMP PREEMPT Mon Dec 21 22:00:46 UTC 2020 x86_64 GNU/Linux
   ```
   
   sudo pacman -S linux59-headers linux59-virtualbox-host-modules virtualbox-host-dkms virtualbox
   
2. 安装微信，QQ

   sudo pacman -S base-devel yay

   yay -Sy

   yay -S com.qq.weixin.spark

   yay -S com.qq.tim.spark

   注意：如果缺失win字体，中文显示会异常，可以拷贝win下面的simsun.ttf,simsun.ttc字体至~/.deepinwine/Spark-WeChat/drive_c/windows/Fonts/

3. win字体安装

   最好是win下面的全部字体都拷贝出来一份

   sudo mkdir /usr/share/fonts/win-fonts

   sudo cp `所有字体文件` /usr/share/fonts/win-fonts/

   sudo fc-cache -fv

4. 安装WPS

   yay -S wps-office wps-office-mui-zh-cn ttf-wps-fonts

   注意安装过程中出现以下应答项，一定选择2，这样下载速度会快很多。

   ```
   :: 有 2 个提供者可用于 wps-office:
   :: AUR 软件库
       1) wps-office 2) wps-office-cn 
   
   输入数字 (默认=1): 
   ```

5. 安装Pycharm社区版

   sudo pacman -S pycharm-community-edition

   

   