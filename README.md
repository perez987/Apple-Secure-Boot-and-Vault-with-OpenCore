# Apple Secure Boot with OpenCore. Vaulting OpenCore. UEFI Secure Boot.
 
**Apple Secure Boot** is the technology used in Macs to verify the integrity of the operating system at boot: *boot loader > kernel > system volume snapshot*. If this check fails, macOS won't boot.

Apple Secure Boot only works during the boot process, once macOS is running it no longer performs any function.

It is highly recommended to read the Dortania guides: [applesecureboot.md](https://github.com/dortania/OpenCore-Post-Install/blob/c0e7f282975f7d6224878b71648c27ce0ed304e6/universal/security/applesecureboot.md), [uefisecureboot.md](https://github.com/dortania/OpenCore-Post-Install/blob/c0e7f282975f7d6224878b71648c27ce0ed304e6/universal/security/uefisecureboot.md) and [vault.md](https://github.com/dortania/OpenCore-Post-Install/blob/c0e7f282975f7d6224878b71648c27ce0ed304e6/universal/security/vault.md).

---

## SecureBootModel changes in OpenCore 0.7.2

There are 3 modes of Apple Secure Boot:
|    |
|-----|
|Full Security: Only allows to boot the installed operating system or another signed version of macOS in which Apple currently trusts. It also checks the integrity of the installed version. If the check fails, the system offers to reinstall macOS or boot from a different disk|
|Medium Security: Checks that the installed version of macOS is legitimate but not the integrity of the system. Lets you boot any signed version of macOS in which Apple has ever trusted|
| No Security: other systems or versions different from those mentioned in the secure options are allowed. There are no requirements on the boot operating system|

Apple Secure Boot state on Intel-based Macs can be obtained from NVRAM:

`nvram 94b73556-2197-4702-82a8-3e1337dafbfb:AppleSecureBootPolicy`

If the variable is found, it can be one of the following:

- %02 - Full Security Mode
- %01 - Medium Security Mode
- %00 - No Security Mode.

If the variable is not found, Apple Secure Boot is not supported.

OpenCore has a SecureBootModel key that adjusts the Apple Secure Boot mode to make it similar to Macs. This key has been changed in OpenCore 0.7.2.

- In OpenCore 0.7.1, failsafe value for SecureBootModel is Default, this value sets Apple Secure Boot hardware model as j137 (iMacPro1,1 December 2017 macOS 10.13.2). This means that macOS versions older than 10.13.2 cannot be installed with this SecureBootModel value.
- In OpenCore 0.7.2, failsafe value for SecureBootModel remains Default, but this value sets Apple Secure Boot hardware model as x86legacy, new value (not existing in previous versions) that corresponds to macOS 11 Big Sur and 12 Monterey on hardware without T2 chips (hackintosh machines) and virtual machines.

Note that with OpenCore 0.7.2:

<table>
<tr><td>x86legacy is designed for machines without T2 chip with Big Sur and Monterey</td></tr>
<tr><td>j137 doesn't work on Monterey</td></tr>
<tr><td>j137 is the recommended value for macOS 10.13.2 through 10.15.x</td></tr>
<tr><td>systems older than macOS 10.13.2 must set SecureBootModel=Disabled</td></tr>
<tr><td>users who don't want to have Apple Secure Boot for any reason can set SecureBootModel=Disabled, even in Big Sur and Monterey</td></tr>
</table>

According to Apple, these Macs have Apple T2 security chip:

- iMac (2020)
- Mac Pro (2019)
- Mac Pro (Rack, 2019)
- Mac mini (2018)
- MacBook Air (2020)
- MacBook Air (2019)
- MacBook Air (2018)
- MacBook Pro (2020)
- MacBook Pro (2019)
- MacBook Pro (2018)
- iMac Pro (2017).

SecureBootModel valid values in OpenCore 0.7.2 (all are models with T2 but x86legacy and Disabled):

<table>
<tr><td>Default — currently set to x86legacy</td></tr>
<tr><td>Disabled — No model, Secure Boot will be disabled</td></tr>
<tr><td>j137 — iMacPro1,1 (December 2017) Minimum macOS 10.13.2</td></tr>
<tr><td>j680 — MacBookPro15,1 (July 2018) Minimum macOS 10.13.6</td></tr>
<tr><td>j132 — MacBookPro15,2 (July 2018) Minimum macOS 10.13.6</td></tr>
<tr><td>j174 — Macmini8,1 (October 2018) Minimum macOS 10.14</td></tr>
<tr><td>j140k — MacBookAir8,1 (October 2018) Minimum macOS 10.14.1</td></tr>
<tr><td>j780 — MacBookPro15,3 (May 2019) Minimum macOS 10.14.5</td></tr>
<tr><td>j213 — MacBookPro15,4 (July 2019) Minimum macOS 10.14.5</td></tr>
<tr><td>j140a — MacBookAir8,2 (July 2019) Minimum macOS 10.14.5</td></tr>
<tr><td>j152f — MacBookPro16,1 (November 2019) Minimum macOS 10.15.1</td></tr>
<tr><td>j160 — MacPro7,1 (December 2019) Minimum macOS 10.15.1</td></tr>
<tr><td>j230k — MacBookAir9,1 (March 2020) Minimum macOS 10.15.3</td></tr>
<tr><td>j214k — MacBookPro16,2 (May 2020) Minimum macOS 10.15.4</td></tr>
<tr><td>j223 — MacBookPro16,3 (May 2020) Minimum macOS 10.15.4</td></tr>
<tr><td>j215 — MacBookPro16,4 (June 2020) Minimum macOS 10.15.5</td></tr>
<tr><td>j185 — iMac20,1 (August 2020). Minimum macOS 10.15.6</td></tr>
<tr><td>j185f — iMac20,2 (August 2020). Minimum macOS 10.15.6</td></tr>
<tr><td>x86legacy — Macs without T2 chip and VMs. Minimum macOS 11.0.1</td></tr>
</table>

iMac19,1 (March 2019 - Minimum macOS 10.14.4) isn't in the list because it has no T2 chip.

Of course, you can also set Secure Boot Model to the value, from the list above, that corresponds to the macOS version you want to boot (example j160 for macOS Catalina 10.15.1). If you are suspicious of old operating systems, you can always put the model that supports only the macOS versions that you need and not the older ones. For example, j140k will filter 10.13 and lower, j152f will filter 10.14 and lower, x86legacy will filter 10.15 and lower.

---

## Apple Secure Boot in the hackintosh

How to get Apple Secure Boot in the Hackintosh? OpenCore provides 3 keys to enable Secure Boot:

- Misc >> Security >> DmgLoading: to set load policy with DMGs in OpenCore; it can be Any (boot fails if Secure Boot is enabled), Signed and Disabled (both support Secure Boot)
- Misc >> Security >> SecureBootModel: to set the Apple Secure Boot hardware model and policy; SecureBootModel (not Disabled) gives Medium Security, for Full Security you must use ApECID
- Misc >> Security >> ApECID: Apple Enclave Identifier, to use personalized Apple Secure Boot identifiers and to have Full Security when paired with SecureBootModel.

For ApECID value, you must get a 64 bit integer randomly generated in a cryptographically secure way. You can use the `urandom` bash command in Terminal. This tool can generate a random 32 bit integer, if we run the tool twice and combine the 2 32-bit integers we get a 64-bit value. Copy this text into a file, save it with sh extension and run it with double click:

```bash
#!/bin/sh
# first 32 bit integer
low32=$(od -An -N4 -tu4 < /dev/urandom)
# second 32 bit integer
high32=$(od -An -N4 -tu4 < /dev/urandom)
# joining the 2 numbers
long=$(($low32 + ($high32 << 32)))
# removing leading minus sign if exists
echo $long | sed 's/-//'
```

Now you can enter it under Misc -> ApECID in your config.plist. Don't use `random` instead of `urandom`, it isn't cryptographically secure.

**Note**: when using ApECID, SecureBootModel must have a defined value instead of default. I have found that x86legacy provides Medium Security and only values that correspond to Mac models that have T2 chip (example: j185, j137) allow you to personalize the boot volume and get Full Security. Remember that SecureBootModel and SMBIOS are different things, it is not mandatory to have the same Mac model in both keys and there are really no advantages or disadvantages if they are the same.

First time that macOS boots with an ApECID value you must personalize the boot volume. To do this:

- boot into Recovery
- be sure you have an Internet connection
- open Terminal
- `bless --folder "/Volumes/HD/System/Library/CoreServices" --bootefi --personalize` (replace HD with the name of your system volume)
- reboot into macOS.

SecureBootModel and ApECID:

- with SecureBootModel=Disabled >> no security (%00)
- with SecureBootModel=x86legacy or any of the valid values >> medium security (%01)
- with SecureBootModel= any of the T2 values plus ApECID non zero >> full security (%02).

---

## OpenCore Vault

It's a secure boot for OpenCore, digitally signing OpenCore.efi so no one can modify boot loader files except you.

**config.plist**

As first task, you must modify config.plist:

- Misc >> Security >> Vault:
	- Basic: Requires just vault.plist file to be present, used for filesystem integrity verification
	- Secure: Requires both vault.plist and vault.sig files, used for best security as vault.plist changes require a new signature
- Booter >> ProtectSecureBoot=Yes >> needed with Insyde firmwares for fixing secure boot keys and reporting violations.

**CreateVault**

Copy OpenCorePKG/Utilities/CreateVault folder next to the EFI folder inside the EFI partition. The resultant path must be: EFI partition/Utilities folder/CreateVault folder.
```.
├── EFI
│   ├── BOOT
│   └── OC
├── Utilities
│   └── CreateVault
│       ├── RsaTool
│       ├── create_vault.sh
│       └── sign.command
└──
```
Inside CreateVault there are 3 files: create_vault.sh, RsaTool and sign.command.
Run sign.command >> to generate a hash for each file in the EFI folder, write them to the vault.plist file and create a 256-byte signature of vault.plist that will be inserted into the OpenCore.efi file.

How to disable Vault?

- Get a new copy of OpenCore.efi
- Misc >> Security >> Vault >> Optional
- Remove vault.plist and vault.sig.

---

## UEFI Secure Boot

UEFI Secure Boot only allows to boot OS's that are signed and trusted. PC Bios comes with Microsoft keys as trusted. So, to boot Windows with Secure Boot, you need to enable Secure Boot in BIOS and to have Windows 10/11 keys (usually included in the motherboard firmware). But this is only for Windows. macOS has its own implementation Apple Secure Boot, this feature can be done with Secure Boot disabled in BIOS. So, these are 2 separate systems: PC BIOS Secure Boot and Apple Secure Boot.

Windows boots fine with Secure Boot enabled or disabled in BIOS. But Opencore only boots with Secure Boot disabled in BIOS (as expected). This is not important for macOS-only users. But Windows 11, beta yet, seems to require Secure Boot enabled in BIOS (this can change in final release) and there may be applications requirin Secure Boot or even business laptops with this feature enabled by default, it maybe is important for users who have macOS and Windows 11. When booting OpenCore with Secure Boot mode enabled in BIOS, a warning saying "Secure boot violation. Invalid signature detected. Check secure boot policy in setup" is displayed by the firmware before OpenCore picker. And OpenCore does not boot.

By default our hacks work with BIOS secure boot disabled since always, this is one of the BIOS options required to boot with OpenCore or Clover. There are some not very complicated way of running OpenCore with PC BIOS Secure Boot enabled. This topic has its own post: [OpenCore and UEFI Secure Boot](https://github.com/perez987/OpenCore-and-UEFI-Secure-Boot).

---

## OpenCore Vault + UEFI Secure Boot

There is a way to have UEFI Secure Boot and OpenCore vault at the same time, it's in the OpenCore Configuration.pdf file although the instructions are short and confusing in my opinion. It is a heavy task but at least it is possible to carry it out.

The key is in the order the files are signed, both with personal keys for the UEFI firmware and hashes created from vault.

This requires moving from macOS to Windows and viceversa a few times. In order not to have to switch from mac to windows so many times, you can install Ubuntu virtual machine with UTM. UTM is an app that allows you to use virtual machines on macOS and iOS. It is free and open software. Information and downloads are available on [GitHub](https://github.com/utmapp/UTM) and on the [website](https://mac.getutm.app).

UTM offers preconfigured virtual machines that you just have to attach to the app and start, it is not necessary to previously install the operating system. You can visit the [Gallery of Virtual Machines](https://mac.getutm.app/gallery/).

Among the preinstalled virtual machines there is no Ubuntu 22.04 but UTM has a [guide](https://docs.getutm.app/guides/ubuntu/) to download and install this version of Ubuntu. I have followed this guide to have an Ubuntu 22.04 virtual system on macOS.

It is important to configure the clipboard and shared folder between macOS and Linux. The guide explains how to do it. Shared clipboard works after installing SPICE Agent which also improves screen resolutions and dynamic switching between them. Its installation is highly recommended, as is QEMU Agent (Additional features such as time syncing, etc.).

Directory sharing can work in 2 different ways: SPICE WebDav or VirtFS. I have used VirtFS which allows you to show the macOS shared folder in the Ubuntu file system. To do this you have to do in Terminal:

- Create the shared directory `sudo mkdir Shared`
- Mount the shared directory<br>`sudo mount -t 9p -o trans=virtio share Shared -oversion=9p2000.L`
- Add an entry in fstab to mount the shared folder at boot
	- Open fstab `sudo pico /etc/fstab`
	- Add this entry<br>`share /home/yo/Shared 9p trans=virtio,version=9p2000.L,rw,_netdev,nofail 0 0`
	- Save with Ctrl + O and exit with Ctrl + X.

The steps to enable both OpenCore Vault and UEFI Secure Boot are:

1. On Ubuntu >> digitally sign all OC .efi files but OpenCore.efi
2. On macOS >> vault the EFI folder with the signed files, including OpenCore.efi (not digitally signed yet)
3. On Ubuntu >> sign OpenCore.efi (it has Vault already applied)
4. Back in macOS >> copy the EFI folder into the EFI partition
5. Reboot >> enable UEFI Secure Boot >> boot OpenCore.
