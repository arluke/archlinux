# Disk Prep
### Clean
```
sgdisk --zap-all /dev/nvme0n1 && blkdiscard /dev/nvme0n1;
sgdisk --zap-all /dev/nvme1n1 && blkdiscard /dev/nvme1n1;
sgdisk --zap-all /dev/nvme1n1 && blkdiscard /dev/nvme2n1;
```
### Partition
```
sgdisk -n1:4096:+1GiB    -t1:ef00 -c1:"EFI"   /dev/nvme0n1;
sgdisk -n2:0:+920GiB     -t2:8309 -c1:"Linux" /dev/nvme0n1;
sgdisk -n1:4096:+3720GiB -t1:fd00 -c1:"Data"  /dev/nvme0n1;
sgdisk -n1:4096:+3720GiB -t1:fd00 -c1:"Data"  /dev/nvme0n1;
```
### RAID
```
mdadm create --name=data --raid-devices=2 --level=raid1 /dev/nvme1n1p1 /dev/nvme2n1p1
```
### Encrypt
```
cryptsetup luksFormat -S 2 -v /dev/nvme0n1p2;
cryptsetup luksFormat -S 2 -v /dev/md/data;
cryptsetup luksOpen /dev/nvme0n1p2 archfs;
cryptsetup luksOpen /dev/md/data data
```
### Format & Subvols
```
mkfs.vfat -F 32 /dev/nvme0n1p1;
mkfs.btrfs /dev/mapper/archfs;
mkfs.btrfs /dev/mapper/data;

mount -o defaults,ssd,discard /dev/mapper/archfs /mnt;
btrfs subvol create /mnt/@;
umount /mnt;
mount -o defaults,ssd,discard,subvol=@ /dev/mapper/archfs /mnt;
mkdir -p /mnt/var/cache/pacman;
btrfs subvol create /mnt/var/log /mnt/cache/pacman/pkg;

mount --mkdir -o defaults,ssd,discard /dev/mapper/data /mnt/home;
btrfs subvol create /mnt/home/@home;
umount /mnt/home;
mount --mkdir -o defaults,ssd,discard,subvol=@home /dev/mapper/data /mnt/home;

mount --mkdir -o uid=0,gid=0,fmask=0077,dmask=0077,discard /dev/nvme0n1p1 /mnt/efi
```
# Base Install & Initial Configuration
### Unpack the base system
```
pacstrap -K /mnt \
amd-ucode \
base \
base-devel \
btrfs-progs \
linux \
linux-firmware-amdgpu \
linux-firmware-intel \
linux-firmware-other \
linux-firmware-realtek \
linux-headers \
mdadm \
neovim \
networkmanager \
openssh-server \
sudo
```



