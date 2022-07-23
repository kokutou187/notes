# tip

### get video then obtain audio from it

```
// get the list of avaiable format codes for particular video
1. youtube-dl -F URL
// select format, or don't use -f option, default to use the highest quality
2. youtube-dl -f ID URL 
3. ffmpeg -i src dst.mp3
```



### ubuntu configure

```bash
# ~/.bashrc
export PS1="\[\e[36m\][\u \A @\W] $ \[\e[m\]"

# bash 设置应用开启的界面显示在屏幕中间
$ gsettings set org.gnome.mutter center-new-windows true

# 解决显卡驱动不匹配,电脑经常卡死的问题
# 增加blacklist nouveau到末尾，禁止自带的驱动
$ sudo gedit /etc/modprobe.d/blacklist.conf
$ sudo update-initramfs -u
$ lspci | grep -i nvidia

# 开机屏幕亮度最高的问题
$ sudo gedit /etc/default/grub
GRUB_CMDLINE_LINUX_DEFAULT="quiet acpi_osi=Linux cpi_backlight=vendor"
```

### ubuntu command
```shell
# &> stderr stdout 都重定向, 末尾的 & 表示命令在后台执行
$ cd ~/picgo && nohup ./AppRun &>/dev/null &

$ snap changes   // see a list of ongoing changes
$ snap abort ID  // abort ongoing change
```