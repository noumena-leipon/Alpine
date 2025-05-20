# Basic Configuration
`setup-alpine`
Use "none" for disk steps

# Add Community Repository
`vi /etc/apk/repositories`
Uncomment community respository, edit repository addresses to https

# Install Setup Packages
`apk add lvm2 cryptsetup e2fsprogs btrfs-progs exfatprogs gptfdisk mkinitfs`

# Partion Devices
```
╔══════════════════════════════╦═══════════════════════╗
║ /dev/sdX1                    ║  EFI System Partition ║
║ /dev/sdX2                    ║  Unassigned           ║
║ /dev/sdX3                    ║  Unassigned           ║
║ /dev/sdX4                    ║  ExFAT Storage Space  ║
╠══════════════════════════════╬═══════════════════════╣
║ /dev/sdY1                    ║  LUKS2 Container      ║
║   └/dev/mapper/mim           ║  LVM Container        ║
║     ├/dev/mim0/root          ║  /                    ║
║     ├/dev/mim0/boot          ║  /boot                ║
║     ├/dev/mim0/swap          ║  /swap                ║
║     ├/dev/mim0/home          ║  /home                ║
║     ├/dev/mim0/var           ║  /var                 ║
║     ├/dev/mim0/var-log       ║  /var/log             ║
║     ├/dev/mim0/var-log-audit ║  /var/log/audit       ║
║     ├/dev/mim0/var-tmp       ║  /var/tmp             ║
║     └/dev/mim0/tmp           ║  /tmp                 ║
╠══════════════════════════════╬═══════════════════════╣
║ /dev/sdZ1                    ║  LUKS2 Container      ║
║   └/dev/mapper/ath           ║  LVM Container        ║
║     └/dev/ath0/docker        ║  /var/lib/docker      ║
╚══════════════════════════════╩═══════════════════════╝
```
## USB Boot Device
```gdisk /dev/sdX
GPT fdisk (gdisk) version 1.0.5

Partition table scan:
  MBR: not present
  BSD: not present
  APM: not present
  GPT: not present

Creating new GPT entries.

Command (? for help): o
This option deletes all partitions and creates a new protective MBR.
Proceed? (Y/N): y

Command (? for help): n
Partition number (1-128, default 1): 
First sector (34-234441614, default = 2048): 
Last sector (2048-234441614, default = 234441614) or {+-}size{KMGTP}: +200M
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300): ef00
Command (? for help): n
Partition number (1-128, default 1): 
First sector (34-234441614, default = 2048): 
Last sector (2048-234441614, default = 234441614) or {+-}size{KMGTP}: +16M
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300): 0000
Command (? for help): n
Partition number (1-128, default 1): 
First sector (34-234441614, default = 2048): 
Last sector (2048-234441614, default = 234441614) or {+-}size{KMGTP}: +16M
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300): 0000
Command (? for help): n
Partition number (1-128, default 1): 
First sector (34-234441614, default = 2048): 
Last sector (2048-234441614, default = 234441614) or {+-}size{KMGTP}: 
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300):

Command (? for help): w
Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING PARTITIONS!!
Do you want to proceed? (Y/N): y
OK; writing new GUID partition table (GPT) to /dev/sda.
The operation has completed successfully.
```

## Root Device
```gdisk /dev/sdY

GPT fdisk (gdisk) version 1.0.5

Partition table scan:
  MBR: not present
  BSD: not present
  APM: not present
  GPT: not present

Creating new GPT entries.

Command (? for help): o
This option deletes all partitions and creates a new protective MBR.
Proceed? (Y/N): y

Command (? for help): n
Partition number (1-128, default 1): 
First sector (34-234441614, default = 2048): 
Last sector (2048-234441614, default = 234441614) or {+-}size{KMGTP}:
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300): 8309

Command (? for help): w
Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING PARTITIONS!!
Do you want to proceed? (Y/N): y
OK; writing new GUID partition table (GPT) to /dev/sda.
The operation has completed successfully.
```

## Docker Device
```gdisk /dev/sdZ

GPT fdisk (gdisk) version 1.0.5

Partition table scan:
  MBR: not present
  BSD: not present
  APM: not present
  GPT: not present

Creating new GPT entries.

Command (? for help): o
This option deletes all partitions and creates a new protective MBR.
Proceed? (Y/N): y

Command (? for help): n
Partition number (1-128, default 1): 
First sector (34-234441614, default = 2048): 
Last sector (2048-234441614, default = 234441614) or {+-}size{KMGTP}:
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300): 8309

Command (? for help): w
Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING PARTITIONS!!
Do you want to proceed? (Y/N): y
OK; writing new GUID partition table (GPT) to /dev/sda.
The operation has completed successfully.
```

## Rescan Devices
`mdev -s`

# Configuring LUKS2 Containers
Devices that have not encrypted previously should be prepared according to type (`blkdiscard` for SSD and NVME devices or random fill for HDD devices).

## Root Device
```cryptsetup -v -c aes-xts-plain64 -s 512 --hash sha512 --pbkdf pbkdf2 --iter-time 15000 --use-random --allow-discards --persistent luksFormat /dev/sdY

WARNING!
========
This will overwrite data on /dev/sdY irrevocably.

Are you sure? (Type 'yes' in capital letters): YES
Enter passphrase for /dev/sdY: 
Verify passphrase: 
Key slot 0 created.
Command successful.
```
`cryptsetup luksOpen /dev/sdY mim`

## Docker Device
```cryptsetup -v -c aes-xts-plain64 -s 512 --hash sha512 --pbkdf pbkdf2 --iter-time 15000 --use-random --allow-discards --persistent luksFormat /dev/sdZ

WARNING!
========
This will overwrite data on /dev/sdZ irrevocably.

Are you sure? (Type 'yes' in capital letters): YES
Enter passphrase for /dev/sdZ: 
Verify passphrase: 
Key slot 0 created.
Command successful.
```
`cryptsetup luksOpen /dev/sdZ ath`

# Create Logical Volumes

## Root Device
`pvcreate /dev/mapper/mim`

`vgcreate mim0 /dev/mapper/mim`
```
lvcreate -L 20G mim0 -n swap
lvcreate -L 1G mim0 -n boot
lvcreate -L 10G mim0 -n var
lvcreate -L 12G mim0 -n var-log
lvcreate -L 10G mim0 -n var-log-audit
lvcreate -L 8G mim0 -n var-tmp
lvcreate -L 8G mim0 -n tmp
lvcreate -L 10G mim0 -n home
lvcreate -l 100%FREE mim0 -n root
```
## Docker Device
`pvcreate /dev/mapper/ath`

`vgcreate ath0 /dev/mapper/ath`

`lvcreate -l 100%FREE ath0 -n docker`

## Rescan Devices
`mdev -s`

# Create Filesystems
## EFI System Partition
`mkfs.vfat /dev/sdX1`

## ExFAT Storage Partition
`mkfs.vfat /dev/sdX4`

## Boot
`mkfs.ext4 /dev/mim0/boot`

## Root Device
`btrfs rescue create-control-device`
```
mkfs.btrfs /dev/mim0/var
mkfs.btrfs /dev/mim0/var-log
mkfs.btrfs /dev/mim0/var-log-audit
mkfs.btrfs /dev/mim0/var-tmp
mkfs.btrfs /dev/mim0/tmp
mkfs.btrfs /dev/mim0/home
mkfs.btrfs /dev/mim0/root
```
`mkswap /dev/mim0/swap`

## Docker Device
`mkfs.btrfs /dev/ath0/docker`

# Create btrfs Subvolumes

## Root Device
`mount -t btrfs -o noatime,discard=async,compress=zstd:3 /dev/mim0/root /mnt`

`cd /mnt`
```
btrfs su cr @
btrfs su cr @var
btrfs su cr @var-log
btrfs su cr @var-log-audit
btrfs su cr @var-tmp
btrfs su cr @tmp
btrfs su cr @home
```
`cd /`

`umount /mnt`

## Docker Device
`mount -t btrfs -o noatime,discard=async,compress=zstd:3 /dev/ath0/docker /mnt`

`cd /mnt`

`btrfs su cr @docker`

`cd /`

`umount /mnt`

# Mount Devices
`mount -t btrfs -o noatime,discard=async,compress=zstd:3,subvolume=@ /dev/mim0/root /mnt`

`cd /mnt`
```
mkdir -p /mnt/boot
mkdir -p /mnt/var
mkdir -p /mnt/tmp
mkdir -p /mnt/home
```
```
mount -t ext4 /dev/mim0/boot /mnt/boot`
mount -t btrfs -o noatime,discard=async,compress=zstd:3,subvolume=@var /dev/mim0/var /mnt/var
mount -t btrfs -o noatime,discard=async,nodev,nosuid,noexec,compress=zstd:3,subvolume=@tmp /dev/mim0/tmp /mnt/tmp
mount -t btrfs -o noatime,discard=async,nodev,compress=zstd:3,subvolume=@home /dev/mim0/home /mnt/home
```
```
mkdir -p /mnt/boot/efi
mkdir -p /mnt/var/log
mkdir -p /mnt/var/tmp
```
```
mount -t vfat /dev/sdX1 /mnt/boot/efi
mount -t btrfs -o noatime,discard=async,nodev,nosuid,compress=zstd:3,subvolume=@var-log /dev/mim0/var-log /mnt/var/log
mount -t btrfs -o noatime,discard=async,nodev,nosuid,noexec,compress=zstd:3,subvolume=@var-tmp /dev/mim0/var-tmp /mnt/var/tmp
```
```
mkdir -p /mnt/var/log/audit
mkdir -p /mnt/var/lib/docker
```
```
mount -t btrfs -o noatime,discard=async,nodev,nosuid,compress=zstd:3,subvolume=@var-log-audit /dev/mim0/var-log-audit /mnt/var/log/audit
mount -t btrfs -o noatime,nodatacow`,discard=async,compress=zstd:3,subvolume=@docker /dev/ath0/docker /mnt/var/lib/docker
```
`swapon /dev/mim0/swap`

# Install Alpine
`setup-disk -m sys /mnt/`

# Add swap to fstab
`vi /mnt/etc/fstab`

Add the following and confirm previoulsy set mounting options:
`/dev/mim0/swap	swap	swap	defaults	0 0`

# Edit mkinitfs.conf
Check output of mkinitfs -L to determine features necessary for system boot.

Add `cryptsetup,cryptkey,usb,usbhid,lvm,ext4,btrfs,nvme,xhci_pci`.

## Rebuild Initial RAM Disk
mkinitfs -c /mnt/etc/mkinitfs/mkinitfs.conf -b /mnt/ $(ls /mnt/lib/modules/)

## Configure chrony to use NTS
`vi /etc/chrony/chrony.conf`

Add the following, replacing existing servers:
```
# /etc/chrony/chrony.conf - Secure, NTS-enabled, and hardened

# Use only trusted NTS servers for secure time sync
server ntppool1.time.nl iburst nts
server ntppool2.time.nl iburst nts

# Allow only localhost to query or control chronyd
allow 127.0.0.1
allow ::1
deny all

cmdallow 127.0.0.1
cmdallow ::1
cmddeny all

# Driftfile for tracking clock drift
driftfile /var/lib/chrony/chrony.drift

# Log only important events
log tracking measurements statistics

# Restrict access to the chrony control socket (default)
unixcmd

# Synchronize the RTC with system clock after stepping
rtcsync

# Harden kernel time discipline parameters
maxupdateskew 100.0
makestep 1.0 3

# Drop root privileges after startup (chrony runs as chrony user by default)
#user chrony

# Optional: Uncomment if you want to ignore hardware timestamping
#hwtimestamp *

# Optional: Uncomment to further restrict logging from clients
#noclientlog
#noclientloglimit 0

# End of file
```
`chown root:root /etc/chrony/chrony.conf`

`chmod 600 /etc/chrony/chrony.conf`

# Install and Configure the Bootloader
Get the Root Device UUID
`blkid -s UUID -o value /dev/sdY1`

## Create Keyfiles to Unlock Partitions on Boot
### Root Device
`touch /mnt/boot/mim.bin`

`chmod 600 /mnt/boot/mim.bin`

`dd bs=512 count=4 if=/dev/urandom of=/mnt/boot/mim.bin`

`cryptsetup luksAddKey /dev/sdY1 /mnt/boot/mim.bin`

### Docker Device
`touch /mnt/boot/ath.bin`

`chmod 600 /mnt/boot/ath.bin`

`dd bs=512 count=4 if=/dev/urandom of=/mnt/boot/ath.bin`

`cryptsetup luksAddKey /dev/sdZ1 /mnt/boot/ath.bin`

#### Add Keyfile Unlocking for Docker Device
`vi /etc/conf.d/dmcrypt`

Add the following:
```
target=crypt-docker
source='/dev/sdZ1'
key='/root/boot/ath.bin'
```
Enable `dmcrypt` and `lvm` services to boot process:
```
rc-update add dmcrypt boot
rc-update add lvm boot
```

## Mount Filesystems for Grub EFI Installer
```
mount -t proc /proc /mnt/proc
mount --rbind /dev /mnt/dev
mount --make-rslave /mnt/dev
mount --rbind /sys /mnt/sys
```

## Chroot Into the Installation 
```
chroot /mnt
source /etc/profile
export PS1="(chroot) $PS1"
```

## Install Grub2 for EFI
`apk add grub grub-efi efibootmgr`

## Configure Grub
`vi /etc/default/grub`

Add the following to GRUB_CMDLINE_LINUX_DEFAULT
`cryptroot=UUID=<UUID of /dev/sdY1> cryptdm=mim cryptkey`

Add the following lines:
```
GRUB_PRELOAD_MODULES="luks cryptodisk part_gpt lvm"
GRUB_ENABLE_CRYPTODISK=y
```

## Create a Pre-Config Grub File
`touch /root/grub-pre.cfg`

`vi /root/grub-pre.cfg`
```
set crypto_uuid=<UUID of sdY1 without hyphens>
cryptomount -u $crypto_uuid
set root='lvmid/<UUID of mim0 Volume Group from vgdisplay>/<UUID of mim0 Logical Volume from lgdisplay>'
set prefix=($root)/boot/grub
insmod normal
normal
```
```
grub-mkimage -p /boot/grub -O x86_64-efi -c /root/grub-pre.cfg -o /tmp/grubx64.efi luks2 part_gpt cryptodisk lvm ext2 gcry_rijndael pbkdf2 gcry_sha512
install -v /tmp/grubx64.efi /boot/efi/EFI/grub/
grub-mkconfig -o /boot/grub/grub.cfg
exit
```

# Install and Configure UFW
## Install UFW (Uncomplicated Firewall)
`apk add ufw`

## Set default policies: deny all incoming, allow all outgoing
`ufw default deny incoming`
`ufw default allow outgoing`

## Allow SSH (replace 22 with your custom port if changed)
`ufw allow 22/tcp`

## (Optional) Restrict SSH to your admin IP only
## `ufw allow from <your-admin-ip>/32 to any port 22 proto tcp`

## Allow HTTP and HTTPS for NextCloud
`ufw allow 80/tcp`
`ufw allow 443/tcp`

## Enable logging (recommended: 'low' for production servers)
`ufw logging on`

## Enable UFW (this will activate the firewall)
`ufw enable`

## Check UFW status and rules
`ufw status verbose`

#Reboot
##Unmount Volumes and Partitions
```
cd
umount -l /mnt/dev
umount -l /mnt/proc
umount -l /mnt/sys
umount /mnt/boot/efi
umount /mnt/boot
umount /mnt/var/log/audit
umount /mnt/var/log
umount /mnt/var/lib/docker
umount /mnt/var/tmp
umount /mnt/var
umount /mnt/tmp
umount /mnt/home
swapoff /dev/mim0/swap
umount /mnt
vgchange -a n
cryptsetup luksClose lvmcrypt
reboot
```
