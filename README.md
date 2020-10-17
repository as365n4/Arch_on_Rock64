# Arch Linux on Rock64

#### 1.)  Find device designation of eMMC Module and unmount

`lsblk`			device `sdX` should match the size of your eMMC Module

`sudo umount /dev/sdX1`   unmount the eMMC Module

#### 2.)  Zero the beginning of the SD card

`sudo dd if=/dev/zero of=/dev/sdX bs=1M count=32`

#### 3.)  Start fdisk to partition the SD card

`sudo fdisk /dev/sdX`

type `o` this will clear out any partitions on the drive
, type `p` to list partitions, there should be no partitions left
, type `n`, then `p` for primary, `1` for the first partition on the drive
, `32768` for the first sector, and then press ENTER to accept the
default last sector, then write the partition table and exit by typing `w`

#### 4.)  Create the ext4 filesystem

`sudo mkfs.ext4 /dev/sdX1`

#### 5.)  Mount the filesystem

`mkdir root`		this is in your home directory ! → /home/youruser/root 

`sudo mount /dev/sdX1 /home/youruser/root`

#### 6.)  Download and extract the root filesystem (as root, not via sudo)

`wget http://os.archlinuxarm.org/os/ArchLinuxARM-aarch64-latest.tar.gz`

`sudo bsdtar -xpf ArchLinuxARM-aarch64-latest.tar.gz -C /home/youruser/root`

#### 7.)  Download the boot.scr script for U-Boot and place it in the /boot directory

`sudo wget http://os.archlinuxarm.org/os/rockchip/boot/rock64/boot.scr -O /home/youruser/root/boot/boot.scr`

#### 8.)  Unmount the partition

`sudo umount /home/youruser/root`

#### 9.)  Download and install the U-Boot bootloader

`wget http://os.archlinuxarm.org/os/rockchip/boot/rock64/rksd_loader.img`

`wget http://os.archlinuxarm.org/os/rockchip/boot/rock64/u-boot.itb`

`sudo dd if=rksd_loader.img of=/dev/sdX seek=64 conv=notrunc`

`sudo dd if=u-boot.itb of=/dev/sdX seek=16384 conv=notrunc`
	
#### 10.) Install eMMC Module onto Rock64 Board, connect HDMI, ethernet, USB Keyboard and power up, the System is setup with 2 users “alarm” and “root”, log-in details as below:
	
	user:	alarm		password:	alarm
	    	root				root

Change generic root password to your own version (log-in as root).
	
`passwd`

#### 11.) Check the MAC address, may need spoofing if address is `da:19:c8:7a:6d:f4` or `a2:ce:c4:4a:ae:e4`

`ifconfig`    to check for device name

`ip link show eth0`   replace eth0 with your device name given by step above

If you MAC address is `da:19:c8:7a:6d:f4` or `a2:ce:c4:4a:ae:e4` then do steps below, or network won’t work !

`nano /etc/systemd/network/00-default.link`

	[Match]
	MACAddress=da:19:c8:7a:6d:f4

	[Link]
	MACAddress=da:19:c8:1a:1d:f1				change the last 3 bits to your liking,
	NamePolicy=kernel database onboard slot path		DO NOT change the first 3 bits (reserved for Manufacturer)

`reboot`     once board is up, check with ip link show eth0 for success

#### 12.) Initialize the pacman keyring and populate the Arch Linux ARM package signing keys

`pacman-key --init`

`pacman-key --populate archlinuxarm`

#### 13.) Install the U-Boot package

`rm /boot/boot.scr`

`pacman -Sy uboot-rock64`		when prompted, press `y` and hit `enter` to write the latest bootloader to the eMMC Module (read the scripts output and check if device matches, if it does no match your eMMC-Module then you need to copy the files manually)

`reboot`

#### 14.) Update the system

`pacman -Syuu`

`reboot`

#### 15.) Set network to static IP address and change default hostname

`nano /etc/hostname`	replace “alarm” with your version

`nano /etc/systemd/network/eth.network`   check with ifconfig first for device name!

	[Match]
	Name=eth*

	[Network]
	DHCP=no
	DNSSEC=no
	Address=192.168.1.xxx/24			replace xxx with your desired sub-address
	Gateway=192.168.1.xxx
	DNS=192.168.1.xxx

`nano /etc/resolv.conf`			should be nameserver 192.168.1.xxx

`reboot`				once board is up, check with `ifconfig` for success		

#### 16.) Change user from “alarm” to your choice and enable ‘sudo’ comand (log-in as root).

	id alarm
	usermod -l youruser alarm
	usermod -d /home/youruser -m youruser
	groupmod -n youruser alarm
	id youruser
	ls -ld /home/youruser

`pacman -S sudo`

`nano /etc/sudoers`				scroll down to User privilege specification	copy "root" for your specific username and save

exit and log-in as youruser and change `passwd` from alarm to yours

#### 17.) Generate an UK English locale file

`locale -a`					check if any locale is installed

`sudo nano /etc/locale.gen`			uncomment en_GB.UTF-8 UTF-8

`locale-gen`					to generate British English locale file

	
#### Done, enjoy your setup.

#### Credit to https://archlinuxarm.org/platforms/armv8/rockchip/rock64 for providing the initial guide and concept.
	
