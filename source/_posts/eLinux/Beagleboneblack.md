---
title: Beaglebone Black入门
date: 2016-11-13 11:44:28
tags: [Linux]
---

*软硬件平台：Beaglebone Black Ver.B + [BBB-blank-debian-8.6-console-armhf-2016-11-06-2gb](https://rcn-ee.com/rootfs/bb.org/testing/2016-11-06/console/BBB-blank-debian-8.6-console-armhf-2016-11-06-2gb.img.xz)*

# 使能I2C-1
在BBB中，默认只能用I2C-0和I2C-2，使用以下命令使能I2C-1

`echo BB-I2C1 > /sys/devices/platform/bone_capemgr/slots`
> [参考来源](https://www.cs.sfu.ca/CourseCentral/433/bfraser/other/I2CGuide.pdf)

# 关闭默认的USB gadget功能
`vi /opt/scripts/boot/am335x_evm.sh`

将以下内容删除或注释掉，然后reboot
```
#priorty:
#g_multi
#g_ether
#g_serial

unset usb0
unset ttyGS0

#g_multi: Do we have image file?
if [ -f ${usb_image_file} ] ; then
	test_usb_image_file=$(echo ${usb_image_file} | grep .iso || true)
	if [ ! "x${test_usb_image_file}" = "x" ] ; then
		modprobe g_multi file=${usb_image_file} cdrom=1 ro=1 stall=0 removable=1 nofua=1 ${g_network} || true
	else
		modprobe g_multi file=${usb_image_file} cdrom=0 ro=1 stall=0 removable=1 nofua=1 ${g_network} || true
	fi
	usb0="enable"
	ttyGS0="enable"
else
	#g_multi: Do we have a non-rootfs "fat" partition?
	unset root_drive
	root_drive="$(cat /proc/cmdline | sed 's/ /\n/g' | grep root=UUID= | awk -F 'root=' '{print $2}' || true)"
	if [ ! "x${root_drive}" = "x" ] ; then
		root_drive="$(/sbin/findfs ${root_drive} || true)"
	else
		root_drive="$(cat /proc/cmdline | sed 's/ /\n/g' | grep root= | awk -F 'root=' '{print $2}' || true)"
	fi

	if [ "x${root_drive}" = "x/dev/mmcblk0p1" ] || [ "x${root_drive}" = "x/dev/mmcblk1p1" ] ; then
		#g_ether: Do we have udhcpd/dnsmasq?
		if [ -f /usr/sbin/udhcpd ] || [ -f /usr/sbin/dnsmasq ] ; then
			modprobe g_ether ${g_network} || true
			usb0="enable"
		else
			#g_serial: As a last resort...
			modprobe g_serial || true
			ttyGS0="enable"
		fi
	else
		boot_drive="${root_drive%?}1"
		modprobe g_multi file=${boot_drive} cdrom=0 ro=0 stall=0 removable=1 nofua=1 ${g_network} || true
		usb0="enable"
		ttyGS0="enable"
	fi

fi

if [ "x${usb0}" = "xenable" ] ; then
	# Auto-configuring the usb0 network interface:
	$(dirname $0)/autoconfigure_usb0.sh
fi

if [ "x${ttyGS0}" = "xenable" ] ; then
	systemctl start serial-getty@ttyGS0.service || true
fi
```

# 建立内核模块编译环境
1. 安装依赖 `sudo apt-get install make gcc linux-headers-$(uname -r)`
2.



# 内核调试技巧
1. 查看printk等信息  `tail -f /var/log/kern.log`




















1
