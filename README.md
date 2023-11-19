# Arch-Installation

### Verify boot mode

```
cat /sys/firmware/efi/fw_platform_size
```

- If 64 - UEFI mode (Good)
- If nothing not in UEFI mode

### Connect to the internet

- To check for the internet
```
ip link
```
- If no ping, then follow the next steps:
- To start interactive prompt:
```
iwctl
```
- List available options:

```
help
```

- For the network devices list

```
device list
```

- In device list
- you will find he name of the wireless network device
- from now we will call it <device>
- If the device/adapter is **off**, turn it on:

```
device <device> set-property Powered on
```

```
adapter <adapter> set-property Powered on
```

- List all wifi networks on that <device>:
- First scan for the networks

```
station <device> scan
```

- List all the scanned networks

```
station <device> get-networks
```

- You can find all the SSID(Service Set IDentifier)/wifi-names
- Note it down
- Connect to the network
- Use the SSID that we noted previously
station <device> connect <SSID>
- You will be prompted for a passphrase
- Enter it and you should be connected
- You can also use `--passphrase` flag for the connecting

```
station --passphrase <passphrase> <devcie> connect <SSID>
```

- To check the connection,

```
station <device> show
```

- displays the status of the network (connecting, connected, or not connected)
- check again with `ping` command

### Update the system clock

```
timedatectl
```

### Partition the disks

1. To list the partitions
- Identify and notedown the <disk> you will use

```
lsblk
```

2. Create partitions with `cgdisk`

```
cgdisk /dev/<disk>
```

3. When asked for start sector just hit `Enter`
4. When asked for second sector type the size in MiB, GiB, ...
5. When asked for type of partition type the code EF00, 8200, ...
6. When asked for name type partition name to identify easily

- Recommended partitions with sizes

|     | boot | root | home | swap |
| --- | --- | --- | --- | --- |
| size | 1 GiB | 200 GiB | Remaining | half of RAM |
| type | EF00(EFI) | 8300(Linux Filesystem) | 8300 | 8200(swap) |

7. After creating all of the partitions,
- click `[write]` by navigating with `arrow keys` and `Enter`

### Format those partitions

1. Identify all partition and notedown by using `lsblk`

```
lsblk
```

2. Now, format those partitions

- boot

```
mkfs.fat -F32 /dev/<bootPartition>
```

- root

```
mkfs.ext4 /dev/<rootPartition>
```

- home

```
mkfs.ext4 /dev/<homePartition>
```

- swap

```
mkswap /dev/<swapPartition>
```

### Mount all partitions

1. You need to mount the `<rootPartition>` to `/mnt`

```
mount /dev/<rootPartition> /mnt
```

2. create directories for boot and home to be mounted on
- /mnt/boot

```
mkdir /mnt/boot
```

- /mnt/home

```
mkdir /mnt/home
```

3. Mount the directories to the storage

- For drives

```
mount /dev/<partition> <path>
```

- Enable swap

```
swapon /dev/<swapPartition>
```

## Installation

### Select mirrors

1. Reorder the mirrorlist defined in `/etc/pacman.d/mirrorlist`
- Refer to [this page](https://archlinux.org/mirrorlist/) for your location
- [reference list](https://archlinux.org/mirrorlist/?country=DE&protocol=http&protocol=https&ip_version=4) for germany
- Packages will be downloaded from this mirrors so check internet for more information


2. Or you can also use `rankmirrors` but before that you need to install and update `pacman-contrib` and `pacman` with

```
sudo pacman -Sy pacman-contrib
```

- Then, execute

```
rankmirrors -n & /etc/pacman.d/mirrorlist.backup > /etc/pacman.d/mirrorlist
```

3. now install the base package of linux and arch

```
pacstrap -K /mnt base linux linux-firmware base-devel
```

## Configure the system

- Now configure the system after the installation is completed

1. Generate fstab file

```
genfstab -U -p /mnt >> /mnt/etc/fstab
```

- If errors check and edit `/mnt/etc/fstab` file

2. change the root into the system

```
arch-chroot /mnt
```

3. set the time zone

1. Link it as per time zone of `<region>` and `<city>` with `/etc/localtime`

```
ln -sf /usr/share/zoneinfo/<region>/<city> /etc/locatime
```

- You can check the directories `/usr/share/zoneinfo` and `/usr/share/zoneinfo/<region>` for your timezone

2. Now generate `/etc/adjtime` with

```
hwclock --systohc --utc
```

### Localization

- Edit `/etc/locale.gen` with your preffered editor
- uncomment `en_US.UTF-8 UTF-8`  and other needed need UTF-8 locales
- Generate by

```
local-gen
```

- create `/etc/locale.conf` and set `LANG=en_US.UTF-8`

```
LANG=en_US.UTF-8`
```

```
export LANG=en_US.UTF-8
```

### Network configuration

- Create a hostname file and insert the hostname you prefer

```
/etc/hostname
```

```
systemctl enable fstrim.timer
```

### Enable Multilib

- edit `/etc/pacman.conf` and uncomment (remove `#`) from the front of 

```
[multilib]
Include = /etc/pacman.d/mirrorlist
```

- now download `multilib` package by updating `pacman`

```
sudo pacman -Sy
```

### Now set root password

- For it just run the command, in terminal

```
passwd
```

### Add a user

```
useradd -m -g users -G wheel,storage,power -s /bin/bash <username>
```

- set password

```
passwd <username>
```
```
EDITOR=nano visudo
```

- uncomment `%wheel ALL=(ALL:ALL) ALL
- and also add

```
Defaults rootpw
```

### Network manager

```
ip link
```
```
sudo pacman -S dhcpcd
```
```
sudo systemctl enable dhcpcd@<networkDevice>.service
```
```
sudo pacman -S networkmanager
```
```
sudo systemctl enable Networkmanager.service
```
