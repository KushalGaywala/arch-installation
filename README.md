# Arch-Installation

## Verify boot mode

```
	cat /sys/firmware/efi/fw_platform_size
```

- If 64 - UEFI mode (Good)
- If nothing not in UEFI mode

## Connect to the internet

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

## Update the system clock

```
	timedatectl
```

## Partition the disks

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
| | boot | root | home | swap |
| --- | --- | --- | --- | --- |
| size | 1 GiB | 200 GiB | Remaining | half of RAM |
| type | EF00(EFI) | 8300(Linux Filesystem) | 8300 | 8200(swap) |
