---
layout: page
title:  "HTC One M8 Modification Guide"
date:   2016-04-08 12:00:00
---

## Preface

First and foremost, this is a guide on how to learn.  You can find tutorials for any single step of this process anywhere, but no place compiles information on how to obtain your own answers this concisely.  The goal of this document is to allow the reader to unlock and modify Android devices, but in this case specifically the HTC One M8 Sprint (Harmon Kardon Edition).  The specificty of this device prevents it from being easily documented in other guides simply due to the rarity of its backups along with off-cycle updates and prepackaed invasive bloatware, thus requiring a decent knowledge of how to discover and find answers for oneself.  Often, packages here will be completely undocumented and their removal or modification can completely break device functionality, being extremely frustrating to repair.  I have yet to encounter an unfixable situation, aside from maybe unplugging the device during a failed firmware flash.  If you find yourself there, consider skipping to the section on [manually flashing an RUU](#RUU-Extraction) in a panic.  Remember - hope is never lost.  Errors will occur then disappear when trying again.  That stated, not every piece of advice has been tested here.  I've documented many posts, some of which I did not need to follow.  Your mileage will vary.  You have been warned.

## Prep: Tools
- [ADB Only (Win)](https://www.androidfilehost.com/?fid=95747613655040759) or [Install Android SDK](http://developer.android.com/sdk/index.html#Other) for access to `adb`, `fastboot`, etc.
- [HTC Sync Manager](http://www.htc.com/us/software/htc-sync-manager/) or HTC Standalone Driver to allow phone to be recognized by `adb`.
- [An HTC RUU](http://www.htc.com/us/support/rom-downloads.html) if intending to use it for formatting purposes or for access to `htc_fastboot`.

Force Power Off: VolUp + Power 10sec (when device is on)

Bootloader: VolDown + Power (when device is off)

### Unlock Bootloader w/ HTC
- Register at [HTC Dev](http://www.htcdev.com/)
- Enable Developer Mode by Settings -> About -> Software Information -> More -> tap Build Number 10 times.
- Dev settings enabled (tap version 10 times), fastboot off
- `adb reboot bootloader` or Power + VolDown
- `fastboot oem get_identifier_token`
- Complete process on HTC Dev to get token emailed.
- `fastboot flash unlocktoken Unlock_code.bin`
- ___WARNING:___ This will Factory Reset the device. (wipe /data/)


### Install TWRP + Root
- [Download TWRP](http://techerrata.com/browse/twrp2/m8)
- `adb reboot bootloader`
- `fastboot flash recovery openrecovery-twrp-2.7.1.0-m8.img` <sub>(change name for version obviously)</sub>
- `fastboot reboot`
- [Download SuperSU](download.chainfire.eu/supersu) to device.
- `adb reboot recovery`
- TWRP: Install, Browse to zip.

### S-Off
Disables hardware security flag by setting `fastboot oem writesecureflag 0` with a hardware-based exploit kit, allowing the following:

- CID can be freely changed (permanently).
	- Alternative: modify `android-info.txt`.
- Unsigned zips can be flashed.
- `\system` to be written to in userland without circumvention.
	- Note: this is doable in S-Off by using TWRP for file manipulation.


The only current \(2015+\) method is through [Sunshine](http://theroot.ninja/), a paid project that produces results through remote exploit loading.  The apk for this *provides temporary root on stock S-ON for 4.4.3* and can be used freely.

### SuperCID
Changing the CID to SuperCID 11111111 bypasses the M(odel)ID and C(arrier)ID checks during FW flash / RUU to allow the loading of version mismatched FWs (ie. HK vs non-HK, downgrades) in addition to bypassing official signature verification for unsigned firmware zips.  Please note your device's true CID via `fastboot getvar cid` and have it known for finding proper RUUs and other firmware-centric files.

```bash
adb reboot bootloader
fastboot oem rebootRUU
fastboot oem writecid 11111111
fastboot reboot-bootloader
```
## Firmware Modification
A [thread](here) gives a very thorough explination of the different types of hardware flashing.  The brief overview:

- RUUs from HTC include everything on the device.
- OTAs include patches that overwrite in an RUU-like fashion.
- ROMs often flash over `/system`, so you'll find best results doing a Factory Reset (wipe `/cache`, dalvik cache, `/data`) beforehand.

### Command-line Flashing

- `adb reboot bootloader`
- `fastboot getvar all`
- *Safely Note **Bootloader** and **Baseband** versions!*
	- To revert to Stock safely, you must know the default values for these.  Other FWs _will not work reliably_.
	- Use these to find matching old FWs or confirm your device on [HTC's site](http://www.htc.com/us/support/rom-downloads.html) and pick a matching RUU.
	- There is a high chance this number will get overwritten in Custom ROMs and become inaccurate.  Be sure to note it.
- `fastboot reboot-bootloader` if freeze: disconnect, restart process
- `fastboot oem rebootRUU`
- `fastboot flash zip firmware.zip` where `firmware.zip` is your flashable file.
	- This process will freeze at 95-100% green bar on device and output a success to console.  This is normal.
- `fastboot reboot-bootloader` or `fastboot reboot`
- Check with `fastboot getvar all` to confirm a change.

### Flash using HBoot

Or, rename the firmware zip to `0P6BIMG.zip` (For M8_WHL) and place in the root of `external_sd` (the physical SD card in your device).  Reboot into Bootloader and switch to HBoot, which checks for this file and attempts to flash it on boot if present.

Only signed, higher version firmwares will be accepted by S-ON devices - obtain these by [extracting them from an RUU](#RUU-Extraction) or downloading OTA updates.

For S-OFF, many unsigned FWs are available at [XDA: skulldreamz - Firmware Flashing without panic attacks](http://forum.xda-developers.com/htc-one-m8/development/firmware-flashing-panic-attacks-t2824048) along with instructions for SuperSU and a suggestion to GPE before going back to Sense to cleanly install, allowing you to switch carrier images at will.  Do note the hardware radios are physically different and your mileage may vary from unintended stock images.

### Partial Flash
Sometimes one might want to update the firmware without a device wipe. Nowadays, the `rom.zip` in the RUU is encrypted and is [annoying to snag decrypted segments of](#ruu-decryption) to the end user anyways, further complicating the problem. Fortunately, if you have your [stock baseband and bootloader versions], partial FW Flashes can be performed:

`fastboot flash <partiton> disk.img`

You can find a list of valid partitions by using `cat /proc/emmc` on the device or check [Sneaky's Partition Dump](https://docs.google.com/spreadsheets/d/1uTfhr5sUFpdYKpHMFOOUnb1T5kOBu411_rcEb_Fz37A/edit#gid=0) if you're lazy. Commonly, this is used as `fastboot flash recovery recovery.img` to install TWRP, but this can be modified to apply directly to any partition.  It's also usually prefaced and followed by a `fastboot erase cache` (which should warn about possibly wanting to format instead) so that you don't run into a failure code flashing large images or run into corruption.

- [Stock Recovery Thread](http://androidforums.com/threads/guide-re-flash-stock-rom-ruu-after-bricking-a-rooted-device.565203/)
- Stock Boot Thread?

## Stock Image Modification

Useful Commands:

`adb shell` (certain commands don't often work in TWRP)

- `pm list packages`
- `pm path com.name.name`
- todo: PM Uninstall

When you see the wizard activity on top, use "dumpsys activity" to see its package/class name, and use "pm path com.wizard.name" to find the apk.

See [below](#futurelink) for a list of my personal removed packages.  Context: I'm okay with Google "spyware" but not any other invasive tracking, since those generally take a higher performance hit. Your mileage may vary.

___WARNING:___ ItsOn is a reporting application from Sprint that may pertain to month-to-month minute tracking and billing.  It anchors deep into system calls for standard phone operation.  Its removal may inhibit your ability to receive or place calls.

ItsOnService has a shell script found in `/system/vendor/itson/` which nicely defines much of is behaivor and defines the libraries it loads and may install to `/system/lib`.  It's recommended to read through it and check locations it may be installing extra copies to. Remove it from:  <sub>[credit](http://forum.xda-developers.com/showpost.php?p=56571550&postcount=2732)</sub>

```bash
mkdir /tmp/carrier
mount /dev/block/mmcblk0p37 /tmp/carrier
del /tmp/carrier/itson/
umount /tmp/carrier/
del /system/vendor/itson
del /system/priv-app/ItsOnUID.apk
```

### Force Enable Tethering

[Overview Thread](http://forum.xda-developers.com/showthread.php?t=2712222) - requiring an `init.d` kernel, this suggests flashing an [iptables script](http://forum.xda-developers.com/showthread.php?t=2474432) named tether that gets placed in `/system/etc` and called on boot with either Script Manager or by injection into `/system/etc/init.post_boot.sh` or `/system/etc/install-recovery.sh`.  Since neither of these candidates are on our Sense6 Stock install, we can guess at init scripts to tack on the extra line of `/system/etc/tether` to flush the routing tables.

Inject all our script content into `/system/etc/usf_post_boot.sh`. This is what [InsertCoin](https://insertcoin-m7-nightly.googlecode.com/svn/trunk/system/etc/usf_post_boot.sh) does.
Also steal some script from there.

Watch post-install for "Cellular Radio" %usage to skyrocket.  If it does, disable executable privledge on `tether` (chmod 644 works) and it should revert.

```bash
#!/system/bin/sh
iptables -F
iptables -A bw_FORWARD -i !lo+
iptables -A natctrl_FORWARD -j RETURN -i rmnet+ -o wlan0 -m state --state RELATED,ESTABLISHED
iptables -A natctrl_FORWARD -j DROP -i wlan0 -o rmnet+ -m state --state INVALID
iptables -A natctrl_FORWARD -j RETURN -i wlan0 -o rmnet+
iptables -A natctrl_FORWARD -j DROP
iptables -A natctrl_nat_POSTROUTING -t nat -o rmnet+ -j MASQUERADE
```
include executable privledge lmao so dont forget to chmod 777

#### 4.x+ changes to Tethering

A native os feature [was introduced](http://pocketnow.com/2015/01/01/nexus-6-tethering) to report if a device is being used to tether in Settings.  This is a key/value pair in the settings.db sqlite3 db (`data/data/com.android.providers.settings/databases`), which can be disabled by setting `tether_dun_required` to `0`.  In a more user-friendly manner:
```
adb shell settings put global tether_dun_required 0
```
This seems to require IPv6 disabled with a non-hybrid IPv4 APN to be selected in settings.  It is also recommended to add `net.tethering.noprovisioning=true` to `build.prop` accompanying this change.

It is [theorized](http://forum.xda-developers.com/moto-x/themes-apps/tmo-tmobile-native-tether-fix-4-4-2-t2644867) that appending `,dun` to the APN Type list should apply this setting (ie. 'default,mms,supl,hipri,fota,dun').

Lots of information available in [this thread](http://forum.xda-developers.com/nexus-4/general/4-4-3-nexus-4-lte-lte-tethering-hotspot-t2416822) // [github]()
	The other issue is that this script had to be run each and every boot. Placing the commands within an init.d script does not work because at the time init.d scripts are run in the boot, the natctrl_nat_POSTROUTING rule does not exist, so you cannot append to it. Even if you do create the rule and append to it, the changes will be overwritten when the rules are set later in the boot. The solution is to run the commands within a delayed subshell that alters the firewall after the rules are set. This is what my LTE fix does.

#### APN Modificaiton

In addition, one might want to change APN settings to force certain cellular connections or handoff from others.  Without using the ## codes, you must add `ril.sales_code=LOL` and `ro.csc.sales_code=LOL` to `build.prop` to enable the UI in Settings -> Data Usage (More) -> APNs.


Alternatively, manually edit ``
APN Sources:
	https://apn.gishan.net/settings/770_21_sprint_apn_settings_for_htc_one_m8.php

Name	 Sprint APN	 cinet.spcs Proxy	 blank Username	 blank Password	 blank Server	 blank MMSC	 http://mms.sprintpcs.com/servlets/mms MMS Proxy	 68.28.31.7 6 MMS Port	 80 MMC	 234 MNC	 15 Authentication type	 not set APN Type	 internet + mms APN Protocol	 IPv4 APN enable/disable 	 blank

Read more at: http://www.4gtricks.com/2013/08/sprint-apn-settings-for-android-phone.html?m=1

some scraped info
unlisted? blank.
name | sprint
apn | cinet.spcs
proxy, user, pw, server | blank
mmsc |  http://mms.sprintpcs.com/servlets/mms
mms proxy | 68.28.31.7 6
port | 80
mmc | 234
mnc | 15
auth | not set
apn | internet + mms
protocol | ipv4
enable/disable | blank

i think the DM Command is `*#*#4636#*#*`


decrypt your framework-res.apk and change the apns.xml file and then reencrypt the file and push it back to the phone.

ots also in system/etc/apns-conf.xml

references
[build prop and default.xml tweaks](http://forum.xda-developers.com/showthread.php?t=2711892)
[apn config stuff](http://forum.xda-developers.com/showpost.php?p=46824700&postcount=533)

notes ([source](http://forum.xda-developers.com/moto-x/themes-apps/app-hotspot-entitlement-bypass-v1-1-5-9-t2705152))

	Also if your phone model has any property that has dun_required like ro.mot.tether_dun_required then you will need to change that line from 1 to 0.

### Other build.prop and default.xml edits
[quick settings index reference] Yeah, 10 is the hotspot. 16 is Network selection, which doesn't work on this phone either and should probably be taken out.
http://forum.xda-developers.com/showpost.php?p=51796802&postcount=21

http://forum.xda-developers.com/showthread.php?t=2711892 misc
http://forum.xda-developers.com/showthread.php?t=2712222 edits to default

dont disable timezone thing since its reported to be a cause of huge battery drain if an issue.


### Remove bootloader flags
```bash
adb shell
su
echo -ne ‘\x00′ | dd of=/dev/block/mmcblk0p6 bs=1 seek=5314564 				# TAMPERED flag
echo -ne ‘\x00\x00\x00\x00′ | dd of=/dev/block/mmcblk0p2 bs=1 seek=33796 	# LOCKED flag
exit
```

### Remove Startup Animations (revert to stock)

Look for configuration XML files in `/system/customize/CID`, they will reference zips in `/system/customize/resource` that can be safely removed.

---

## Roms
- [nV Rom](http://forum.xda-developers.com/showthread.php?t=2741450) -  4.4.2 // Sense 6 WWE base 1.54.401.10 **[Abandoned]**

##Roots
- [XDA:Android Revolution - Stock / OTA Capable + Root // 4.16.401.10](http://forum.xda-developers.com/showthread.php?t=2755657)
	- SU + system r/w typically modifies ` system/etc/install_recovery.sh` and invalidates OTA
	- solution: modify boot.img and run daemonsu instead.
	- includes `/system` write on, init.d, ++.
	- removed trigger for install_recovery.sh
	- flash custom recovery, boot.img, use cfw to flash zip, flash stock recovery.


---

## Adressing Battery Issues

### Tools
- Play Store:
	- Greenify
	- MyAndroidTools
	- Servicely
- Flashed
	- that one antilogger thread

Observations:

remove play music (android.music)
		calendar
		holospiral
		autobot(cargps)
		htc backup reset
		mirrorlink
		htcmode (autobot)
		android.feedback
		messages
		play news
		!all widgets
		smsbackupagent
		nero.android.htc.sync (androidhtcsync.apk)
		videowidget
		filemanager



### Experimental Removed Apks
Warning: these provide critical function for phone service.  Functionality loss may occur.
- odmadm (vdm client)
- htcloglevel
	- htc media uploader
- upadter (ota applier)
	- cirservice (checks ota)
- ?qualcomm time
- htc account (might have to leave for setup)
- htccupd
	- htc checkin

do not:
- htc video ++ dlna
- calendarstorage
- configupdater gets a bunch of stuff on boot
	- see triggers for BOOT_COMPLETED

sysapp
	nwewsweather (gnews / old)
	music2 (gp music)

privapp
	htcmodeclient (autobot)
notes
vDM client OK.  controlled by malicious? ODMADM
look into sprint.internal.LauncherFacade `adb shell pm list features` / jar


test1
	remove (androidhtcsync.apk)
	check for Sync All
	works fine
test2

com.redbend.vdmc == htc_sprint_dmservice thing
com.sprint.dsa = priv-app\DSS
com.htc.home.personalize = priv-app\homepersonalize
com.htc.vte = priv-app\video talk enhanceement
com.htc.cs.dm = system/app/devicemanagement
org.simalliance.openmobileapi.service = system/app/smartcardservice
com.htc.dmportread = /priv-app/dmcommandservice

# Returning to Stock

## Stock ROM Info
A set of HTC [all] and other stock ROMs in partially and fully extracted packagaes can be found at [Android Revolution](http://android-revolution-hd.blogspot.com/p/android-revolution-hd-mirror-site-var.html).   [HTCDev](http://htcdev.com) hosts most of the OTAs and stock signed firmware packages, while

OTA updates check integrity of `\system` _existing_ files before flashing.  Adding files often has no effect, but is unconfirmed. NANDroid the partition before modifying just in case.

\[Non-Sprint\] [XDA: Exocetdj / Mr Hofs - FW/OTA/NAND/Recovery Stock Backups](http://forum.xda-developers.com/showthread.php?t=2701376)

## RUU Extraction
RUU Tools are notoriously bad executables with heavy depencendies on many specific VC++ redistributable packages.  It is far easier and more informative to manually flash these backups.

It is unclear at which point the image is decrypted, however.

### [Extract Firmware from RUU](http://forum.xda-developers.com/showthread.php?t=2534428&page=3)
1. Run RUU executable
2. before accepting EULA, browse to %temp% and look for {UID} folder modified at this time.
3. copy contents of the extracted one (with rom.zip) elsewhere

### Running RUU

Prerequisites:

- S-ON Cannot Downgrade
- Bootloader Must be Relocked `fastboot oem lock`

Errors?

- not enough space -> `fastboot erase cache`
SW version is `fastboot getvar version-main`
Cant mount cache? Factory Reset from Bootloader.


- S-ON is `fastboot oem writesecureflag 3`

#### What does the RUU executable actually do?

- Record HTC Sync Manager install path.

```
adb devices
htc_fastboot -s <uid> getvar boot-mode
htc_fastboot -s <uid> oem rebootRUU				 // if boot-mode != RUU
htc_fastboot -s <uid> getvar version-main	 //checks to see if <= rom.zip\android_info.txt
htc_fastboot -s <uid> erase cache
htc_fastboot -s <uid> -d flash zip rom.zip
adb kill-server
```
Check logs in c:\ruu_logs for detailed info.

#### RUU Decryption

During flash, the partial unencrypted zips will be available at `%temp%\filename` where filename is a random alphanumeric string of characters. The console will display how many zips are staged and which the in-progress is, in addition to the contained image being suffixed by the stage number.  This may be useful if anyone needs bulk `\system` image data.


Useful Threads:
[Sensation RUU Guide](http://forum.xda-developers.com/showthread.php?t=1672425)

## Flash Recovery

All commands assume adk in PATH and drivers installed. Filename match not required.

    adb restart fastboot
    fastboot oem lock
    fastboot flash recovery recovery.img
    fastboot erase cache

Resources:
[XDA: Guich - Stock Recovery](http://forum.xda-developers.com/showthread.php?t=2707877) \([Direct](http://d-h.st/users/Guich/Stock%20Recovery%20M8)\)


---

or


1. download the official RUU
2. [Procmon](http://live.sysinternals.com/procmon.exe), CTRL+L
4. change "Architecture" to "Process Name"
6. in the empty field copy and paste the name of your
RUU executable (eg, RUU_Ace_Sense30_S_HTC_WWE_3.12.405.1_Radio_12.65.6 0.29_26.14.04.28_M_release_225512_signed.exe)
7. click "Add"
8. change "Process Name" to "Path"
9. change "is" to "Contains"
10. in the blank field type "rom.zip" (without quotes)
11. click add


Click "OK" to set the filter and then run the RUU file.

Once the utility starts switch back to Process Monitor and look for an entry in the "Path" column that ends with "\rom.zip".

Right click on that line and select "Jump to..."

this will open a Windows Explorer window in the folder which contains the zipped ROM fil

android-info.txt - list of CIDs this RUU will flash to,
boot.img - root file system image,
hboot (followed by a version string) - boot-loader update,
radio.img - radio driver update,
recovery.img - recovery partition image,
splash1_Hero_320x480.nb0 - boot loader splash image,
system_rel.img - system partition image,
userdata.img - data partition image.

### [Hosted Recovery Images + Guide](http://forum.xda-developers.com/htc-one-m8/help/tutorial-how-to-stock-stock-twrp-t3086860)
The basic idea is : fastboot flash recovery NameOfRecovery.img

2. Download TWRP backup - link in post #2 & post #3
3. Download stock recovery - link in post #4

4. Extract the downloaded x.xx.xxx.x_ckpv5.zip on PC

5. Boot to TWRP recovery and make a backup of boot only, this is to see where the backup goes on your device.
6. Reboot, connect device to PC then
open Internal Storage - TWRP/BACKUPS/SerialNo./ (if backup is set to internal storage)
open SD Card - TWRP/BACKUPS/SerialNo./ (if backup is set to MicroSD)

7. Transfer the extracted x.xx.xxx.x folder (not x.xx.xxx.x_ckpv5 folder) and its content to the backup path on your device, so it looks like this :
TWRP/BACKUPS/SerialNo./x.xx.xxx.x

8. Reboot to TWRP, wipe your device - in TWRP go to wipe - advance - select dalvik cache, cache, data, system (only these)
9. Restore the transferred backup - make sure all boot, data & system are ticked - swipe to restore

10. In reboot menu select bootloader

11. fastboot flash stock recovery that you downloaded - command fastboot flash recovery x.xxx.xx.x_recovery.img

12. reboot - check for OTA, download, install
13. you may have multiple OTA when your device currently on a lower version
14. done



### Dumblock Info
http://androidforums.com/threads/guide-re-flash-stock-rom-ruu-after-bricking-a-rooted-device.565203/#post-5208559

 After factory reset, go back into HBOOT recovery, then into HTC Dumblock (in recovery under Advanced). restore orig boot.img & restore recovery. Flash both.
(not at the same time). re-install stock rom odex. or deodex. flash that. go back to
adb to re-lock bootloader. Once relocked, go back into HBOOT. choose fastboot.
(it will enable the USB connection).
Run this RUU per instructions. (locking the boot stopped the bootloop. re-entering HBOOT Fastboot to enable the USB connect will give RUU a chance to do it's magic). Keep a copy of the stock rom on your sd card in case your restores don't work for you.

### [GPE Re-Conversion](http://forum.xda-developers.com/showthread.php?t=2733523)





### Create Backup Recovery
#### From live device:
```bash
    adb shell
    su
    dd if=/dev/block/mmcblk0p43 of=/sdcard/mmcblk0p43
    exit
```
See [Sneaky's Partition Dump](https://docs.google.com/spreadsheets/d/1uTfhr5sUFpdYKpHMFOOUnb1T5kOBu411_rcEb_Fz37A/edit#gid=0) for details.
or `cat /proc/emmc`

#### From Firmware:

- Rename rom.zip from RUU Extraction to [CRC].zip
- (optional) ensure CID is in the list in [CRC].zip\android-info.txt
- Copy to root of SD Card
- Reboot to bootloader, it will automatically install.

One M8 WHL:

-	0P6B
	-	DIAG.zip
	-	IMG.zip



---

# Things I don't immediately use

#### M7
## System Partition Protection
HTC protects `\system` in userland with some kernel code.  This is circumvented by a popular script,  [wp_mod.ko][wp_mod_thread_m8] \([source][wp_mod.c_m8]\), which became much more complicated with changes made to the M8 file verification process.

### Manual Installation
___Requires___: Root, Custom Recovery
- Boot recovery, mount /system
- File manager: `wp_mod.ko` -> ` /system/lib/modules`
- `chmod 644`
- reboot to os
- terminal: `su` `insmod /system/lib/modules/wp_mod.ko`
- edit `/system/etc/install-recovery.sh`, append that filepath, reboot.

[wp_mod_thread_m8]: http://forum.xda-developers.com/showthread.php?t=2701816
[wp_mod.ko_m8]: a
[wp_mod.c_m8]: https://github.com/flar2/wp_mod


## Unlock SIM

---

## Reference Threads
[VomerGuides - Mega Info Post](http://forum.xda-developers.com/htc-one-m8/general/vomerguides-m8-bootldr-unlock-s-off-t2800727)
[Hboot Info](http://forum.xda-developers.com/showthread.php?t=2700666)
[XDA: OMJ - RUU / FW / Stock Rooted /  4.25.65x.14 / Sprint One M8](http://forum.xda-developers.com/showthread.php?t=2729173)
[XDA: thoughtlesskyle - Disable Boomsound Icon](http://forum.xda-developers.com/showthread.php?t=2726774)
[XDA: Sneakyghost -FUU \(no /system\) w/ Troubleshooting](http://forum.xda-developers.com/htc-one-m8/development/progress-fuu-m8-t2813792)

## Customization
[XDA: jobo - Boot Splash Screen]\([thread](http://forum.xda-developers.com/htc-one-m8/themes-apps/mod-customize-boot-splash-online-splash-t2817059) \| [tool](http://jobiwan.net:81/bootsplash-m8)\)

---

todo: flash easyaccessservice.apk onto another device to test tap to wake

may need smartdim

---
BackupRestoreConfirmation = Google Backup

gsd.apk = ##3424## safe to remove

- _WARNING:_ Disables Phone?
- /carrier/itson/app/itsonservice.apk
	- mount /carrier /dev/block/mmcblk0p37
- system
	- app
		- TimeService
	- priv-app
		- Frisbee (bump transfer)
		- ID (Sprint ID)
		- Swype
		- HTC_CIR (IR Remote)
		- HTCFIleManager
		- HTCZero (Zoe)
		- Transfer
		- HTCMessageUploader
		- FutureDial
		- MyGoogleTaskPlugin
		- MyTask
		- SMSBackup
		- HTCReminderViewResource
		- VideoCenter
	- framework
		- com.htc.videowidget.res.apk

---




---

Error Codes
Error 150: ROM upgrade utility error
This is most likely an error when you use Windows Vista.
Error [170]: USB Connection error
The RUU cannot connect to your device.
Drivers not present or not installed correctly; Install the correct drivers.
Phone is not in fastboot USB mode. Remove USB Cable; remove & reinsert phone battery; hold volume down + power; select FASTBOOT; connect USB.
Other unknown error. Extract rom.zip & flash manually via SD Card.
ERROR [155 to 159]: IMAGE ERROR
One of these error messages will appear when your device is S-ON & use the incorrect RUU having lower software number/hboot version than what is currently on your phone.
FAILED (remote: not allowed)
You did not reboot to RUU mode before running the flash command.
You are trying to use a command that you cannot use on a Locked and/or S-ON device.
(bootloader) [ERR] Command error !!! or INFO[ERR] Command error !!!
You are trying to issue an invalid command.
You are trying to use a command that you cannot use on a Locked and/or S-ON device.
FAILED (remote: signature verify fail)The zip that you extracted is broken or the RUU you downloaded is is corrupted. Get the zip again or use a different RUU.
FAILED (remote: 99 unknown fail)
Your bootloader is still unlocked. You didn't relock your bootloader. Relock your bootloader using this command:
Code:
fastboot oem lock
FAILED (remote: low battery)
Battery is too low. Reboot to bootloader by typing
Code:
fastboot reboot-bootloader
& let the phone charge for a while.
FAILED (remote: 90 hboot pre-update! please flush image again immediately)
Normal after Hboot version changes. You need to run only the flash command again (not the whole set of commands, just the fastboot flash command.)
FAILED (remote: 42 custom id check fail)
The RUU doesn't contain your CID. Either add your CID to the zip (needs S-OFF) or try changing your CID (usually works only on an S-OFF device)
FAILED (remote: 92 supercid! please flush image again immediately)
Your device is S-ON & has Super CID. You need to run only the flash command again (not the whole set of commands, just the fastboot flash command.)
FAILED (remote: 43 main version check fail)
Your device is S-ON & you are trying to use an RUU that has software number less than your current software number.
FAILED (remote: 44 hboot version check fail)
Your device is S-ON & you are trying to use an RUU that has hboot version less than your current hboot version.
FAILED (status read failed (Too many links))
Disregard & continue. It is not really an error as such.
FAILED (data transfer failed (Too many links))
Reboot the phone to fastboot USB. Remove USB Cable; remove & reinsert phone battery; hold volume down + power; select FASTBOOT; connect USB.
Then, try the commands again.
Any other error that says '(Too many links)'
Reboot the phone to fastboot USB. Remove USB Cable; remove & reinsert phone battery; hold volume down + power; select FASTBOOT; connect USB.
Then, try the commands again.
If you still get the error, please post the complete error code here & when the error occurred so that it can be solved & added to this list.
Any other error that says 'please flush image again immediately'
You need to run only the flash command again (not the whole set of commands, just the fastboot flash command.) Please post the complete error code here & when the error occurred so that it can be added to this list




## HTC M7 System Partition Protection
HTC protects `\system` with some kernel code.  This is circumvented by a popular script,  [wp_mod.ko][wp_mod_thread] \([compiled][wp_mod.ko] \| [source][wp_mod.c]\), which is customized to the **current kernel version**.  In short, it uses `kallsyms_lookup_name` to find `mmc_blk_setup_wp_protection_partno` and nullifying its contents.  Documentation in the thread states this must load at boot otherwise system will behave unpredictably.

#### Usage:

```bash
insmod /system/lib/modules/wp_mod.ko
mount -o remount,rw /system
```
Load by adding to `init.rc` in `boot.img` either directly or as an `init.d` script.

#### Modification:
check kernel version with `uname -a`

_\[lazy version\]_ hex edit wp_mod.ko directly for the new kernel version.  source version is `3.0`.

[wp_mod_thread]: http://forum.xda-developers.com/showthread.php?t=2230341
[wp_mod.ko]: https://goo.im/devs/flar2/One/wp_mod.ko/
[wp_mod.c]: https://goo.im/devs/flar2/One/wp_mod.c/

### S-Off [Deprecated]
This is now a deprecated instruction set.  The [Firewater](firewater-soff.com) project was discontinued and later became the paid [Sunshine](http://theroot.ninja/) solution.  This binary references online sources that are now offline, and as such, no longer functions.  The following is retained for command reference only.
- Remove HTC Sync
- USB Debugging on, all Security settings off
- `adb reboot`
- `adb wait-for-device push firewater /data/local/tmp`
- `adb shell`
- `su`
- `/data/local/tmp/firewater`
- `exit` twice
- `adb reboot bootloader`
- write down/backup cid
- `fastboot oem writecid 11111111`



http://forum.xda-developers.com/showthread.php?p=58872183#post58872183
http://forum.xda-developers.com/showthread.php?t=1382321

https://www.reddit.com/r/Sprint/comments/2ne8c8/can_someone_explain_what_itson_does_and_why_its/
http://forum.xda-developers.com/showthread.php?t=2697205
http://forum.xda-developers.com/showthread.php?t=2719313

http://android-revolution-hd.blogspot.com/2015/02/how-to-flash-firmware-package-on-htc.html
http://forum.xda-developers.com/showthread.php?t=2733523&page=2
http://forum.xda-developers.com/showthread.php?t=2793957
http://forum.xda-developers.com/showthread.php?t=2247900
http://forum.xda-developers.com/htc-one-m8/help/unable-to-install-stock-ruu-failed-t3162185/post61973005#post61973005
http://forum.xda-developers.com/showthread.php?t=1231249
http://forum.xda-developers.com/showthread.php?t=2497614
http://forum.xda-developers.com/showthread.php?t=2534428
https://www.reddit.com/r/htcone/comments/2oxjoc/ruu_wont_run_as_administrator/
[htc iq info](http://forum.xda-developers.com/showthread.php?t=1247108)
[logging test app](http://forum.xda-developers.com/showpost.php?p=17612559&postcount=110)
http://androidsecuritytest.com/features/system-tools/services/
http://stackoverflow.com/questions/20593497/remove-the-setup-wizard-app-in-android-4

