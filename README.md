## CHROOT (🍥 DEBIAN)
---

> [!NOTE]

## 🏁 First steps <a name=first-steps-chroot></a>


1. Run all the Following Commands
```
pkg update -y
pkg install -y x11-repo
pkg install -y root-repo
pkg install -y termux-x11-nightly
pkg update -y
pkg install -y pulseaudio
pkg install -y sudo
pkg install -y busybox
pkg install -y wget
termux-setup-storage
pkg install -y tsu
pkg install angle-android virglrenderer-android
pkg update -y
```

---

## 🍥 Setting Up Debian<a name=debian-chroot-manual></a>

1. Enable tsu
```
tsu
```

2. Download debian software and make start debain script 

```
mkdir /data/local/tmp/chrootDebian
cd /data/local/tmp/chrootDebian
wget https://github.com/LinuxDroidMaster/Termux-Desktops/releases/download/Debian/debian12-arm64.tar.gz
tar xpvf debian12-arm64.tar.gz --numeric-owner
mkdir sdcard
mkdir dev/shm
cd ../
nano start_debian.sh
```

Copy and paste the following:

```
#!/bin/sh

#Path of DEBIAN rootfs

DEBIANPATH="/data/local/tmp/chrootDebian"

# Fix setuid issue

busybox mount -o remount,dev,suid /data

busybox mount --bind /dev $DEBIANPATH/dev

busybox mount --bind /sys $DEBIANPATH/sys

busybox mount --bind /proc $DEBIANPATH/proc

busybox mount -t devpts devpts $DEBIANPATH/dev/pts

# /dev/shm for Electron apps

mkdir $DEBIANPATH/dev/shm

busybox mount -t tmpfs -o size=256M tmpfs $DEBIANPATH/dev/shm

# Mount sdcard

mkdir $DEBIANPATH/sdcard

busybox mount --bind /sdcard $DEBIANPATH/sdcard

# chroot into DEBIAN

busybox chroot $DEBIANPATH /bin/su - root

```

3. Download debian software and make start debain script 

```
chmod +x start_debian.sh
exit
```

	Next

```
su
```

	And..

```
cd /data/local/tmp
sh start_debian.sh
```

7. The prompt will change to `root@localhost`. If you need to return to Termux just write `exit`. Let's execute some fixes:

```
echo "nameserver 8.8.8.8" > /etc/resolv.conf

echo "127.0.0.1 localhost" > /etc/hosts

groupadd -g 3003 aid_inet

groupadd -g 3004 aid_net_raw

groupadd -g 1003 aid_graphics

usermod -g 3003 -G 3003,3004 -a _apt

usermod -G 3003 -a root

apt update -y

apt upgrade -y

apt install nano vim net-tools sudo git -y

```

8. Create a new user called `droidmaster` (or the name you prefer)

```
groupadd storage

groupadd wheel

useradd -m -g users -G wheel,audio,video,storage,aid_inet -s /bin/bash droidmaster

passwd droidmaster

```

9. Add the created user to sudoers file to have superuser privileges:

```
nano /etc/sudoers
```

	Add this line:

```
droidmaster ALL=(ALL) NOPASSWD: /usr/bin/apt
```
9. Install Desktop Environment:

	switch to droidmaster

```
su - droidmaster
```
9. Install Desktop Environment:

* XFCE4 & Edit start script

```
sudo apt install xfce4 dbus-x11 -y
sudo add-apt-repository ppa:mozillateam/ppa -y
sudo apt-get update -y
sudo apt-get install firefox-esr -y
exit
exit
```

11. Exit chroot and modify the `start_debian.sh` script created on step `5`:

```

/data/data/com.termux/files/usr/bin/nano /data/local/tmp/start_debian.sh
```

Change the last line `busybox chroot $DEBIANPATH /bin/su - root` to this line:

```

busybox chroot $DEBIANPATH /bin/su - droidmaster -c 'export DISPLAY=:0 && export PULSE_SERVER=127.0.0.1 && dbus-launch --exit-with-session startxfce4'

```

12. Let's run the Desktop Environment. Exit chroot environment and copy the following commands on Termux (you can close everything an reopen Termux to be sure you are outside chroot).

```

wget https://raw.githubusercontent.com/LinuxDroidMaster/Termux-Desktops/main/scripts/chroot/debian/startxfce4_chrootDebian.sh

chmod +x startxfce4_chrootDebian.sh

./startxfce4_chrootDebian.sh

```