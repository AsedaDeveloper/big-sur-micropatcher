# big-sur-micropatcher (Version 0.3.2)
A primitive USB patcher for installing macOS Big Sur on unsupported Macs

Thanks to the following people for their hard work to get Big Sur running on unsupported Macs:

- ASentientBot for developing the Hax series of installer patches which are so incredibly helpful for installing Big Sur on unsupported Macs, as well as for his patch to NVDAResmanTesla.kext which allows the GeForce Tesla (9400M/320M) framebuffer to work in Big Sur.
- jackluke for figuring out how to patch the Recovery USB to bypass compatibility checks and AMFI enforcement in the absence of NVRAM boot-args settings.
- highvoltage12v for the patched IO80211Family.kext used by previous versions of this patcher for 802.11n (Wi-Fi) support. (The highvoltage12v IO80211Family.kext is still available by commenting/uncommenting a couple lines of code in patch-kexts.sh around line 315 or so, and a future patcher release will add a command line option to use it again.)
- ParrotGeek for developing the LegacyUSBInjector kernel extension that allows USB to work on various pre-2011 Mac models, and for creating the "trampoline" that allows the installer to be patched at runtime without first running Terminal commands.
- testheit for describing how to use a kmutil feature that I was previously unaware of; this turned out to be a good way to make LegacyUSBInjector function under Big Sur.

In addition, thanks to Ben Sova, MachInit, johncaling40, and Travis Parker for their contributions to this patcher.

This documentation is more thorough than for previous versions of this patcher, but it may still be incomplete. Remember that you *do this at your own risk*, you could easily lose your data, expect bugs and crashes, Big Sur is still under development (as is this patcher), etc.

## Compatibility between different releases of this patcher and different Big Sur beta releases
- v0.3.1-v0.3.2 (this release): Tested with beta 9 (20A5384c). Should be compatible with all previous betas as well.
- v0.3.0: Tested with beta 6 (20A5364e) full installer, and delta updates from beta 6 to beta 8 (20A5374i) or beta 9 (20A5384c). Should also work with beta 1-5. Incompatible with beta 9 full installer.
- v0.2.1: Tested with beta 6 (20A5364e). Should also work with beta 1-5. Incompatible with beta 9 (20A5384c) full installer.
- v0.2.0: Tested with beta 1 (20A4299v) and beta 6 (20A5364e). Should also work with beta 2-5. Incompatible with beta 9 (20A5384c) full installer.
- v0.0.10-v0.1.0: Tested to varying degrees with beta 2 (20A4300b), 3 (20A5323l), and 4 (20A5343i). (For details, check the README or release notes for previous patcher releases.) Should also work with beta 1 (20A4299v). *Partially* compatible with beta 5 (20A5354i) and 6 (20A5364e) as well. May also be partially compatibile with beta 9 (20A5384c) but this is untested.
- v0.0.9: Tested with beta 1 (20A4299v) and 2 (20A4300b), but should be compatible with beta 3 (20A5323l) as well.
- v0.0.1-v0.0.8: Only compatible with beta 1 (20A4299v). Not compatible with beta 2 and later.

## Compatibility of various Mac models
Note that this information is incomplete and may not be 100% correct yet, but I'll add more information over time and fix any errors as I learn about them.

Mostly compatible Mac models:
- If you have a 2013 or later Mac, please check [Apple's official list of supported Mac models](https://www.apple.com/macos/big-sur-preview/) (search the page for "See if") first, to make sure that you actually need this patcher.
- By the way, with the exception of Mac Pros, all of the Macs in this section officially support Catalina. This section is basically "Macs without official Big Sur support but with Metal support", with the exception of pre-2012 iMacs that have upgraded GPUs. (In fact, a 2011 iMac with upgraded GPU is almost equivalent to this category. Earlier iMacs may have compatibility problems caused by other components; see below.)
- Late 2013 iMac: Everything should work (and, after step 14, you're finished -- no need for step 15 and later). Note that there have been some reports of very poor performance with Fusion Drives on this model when running Big Sur, which may be why Apple does not support Big Sur on this model.
- 2010/2012 Mac Pro: I have received positive feedback about this patcher, but I do not know which features work perfectly and which don't. If I had to guess which features might be problematic, I would guess sleep and Wi-Fi. patch-kexts.sh (step 15) should fix Wi-Fi, but I don't know what effect it might have on sleep. (You should upgrade the graphics card, as you would for official compatibility with macOS Mojave.)
- 2009 Mac Pro: Once it's flashed to MacPro5,1 firmware, it should be equivalent to a 2010/2012 Mac Pro. (As with those, you will want to upgrade to a Mojave-compatible graphics card.) However, note that some people have had their flashed MacPro4,1s enter boot loops during installation. We don't yet know what causes this or why it only happens sometimes.
- Other 2012/2013 Macs: Most things should work after the initial installation, except for Wi-Fi (unless you have upgraded to an 802.11ac Wi-Fi card) or possibly GPU switching (on 15" MacBook Pros). Step 15 of the installation process fixes Wi-Fi support, but GPU switching may not yet be a solved problem.

Partially compatible Mac models:
- Most models that officially support High Sierra, but not Mojave, fall into this category. The exceptions are the Mid 2010 15" and 17" MacBook Pros (see the Incompatible category below), and possibly the Late 2009/Mid 2010 iMacs (see the "Unknown status" category below).
- Note that 2009-2011 iMacs that have been upgraded with Metal GPUs also need certain kext patches which are not yet provided by this patcher. This will be corrected in a future patcher release.
- 2011 Macs: Several features may not work after initial installation (after step 14 finishes), including sleep, screen brightness control, Wi-Fi (unless you have upgraded to an 802.11ac Wi-Fi card), and graphics acceleration (unless you have upgraded the GPU in a 2011 iMac). With the exception of a 2011 iMac with upgraded GPU, no 2011 Mac models have full graphics acceleration under Big Sur at this point in time. Make sure to use the `--2011` command line option in step 15 (or `--2011-no-wifi` if you have upgraded to an 802.11ac Wi-Fi card). This fixes sound and Wi-Fi on all 2011 Macs. On 13" MacBook Pros it also fixes sleep and brightness control, and installs the correct Intel framebuffer driver (this is still unaccelerated, but it still increases speed somewhat -- enough to make full-screen YouTube in Safari work with very few frame drops, although this pegs both CPU cores). For 15" and 17" MacBook Pros, disabling the discrete GPU will probably increase performance, and sleep and brightness control probably won't work without disabling it. (That isn't to say that sleep and brightness control will necessarily work even with the discrete GPU disabled -- but it'll probably work if you have a way of disabling the GPU that also keeps sleep and display brightness functional in a dosdude-patched Mojave or Catalina.) MacBook Airs should be equivalent to the 13" MacBook Pro in terms of compatibility with Big Sur and this patcher.
- Mid 2010 white MacBook, 2010 13" MacBook Pro, 2010 MacBook Air: In addition to the features which don't work after initial installation on the 2011 13" MacBook Pros (Wi-Fi, sound, graphics acceleration, sleep, display brightness control), Ethernet doesn't work. The `--2010` option for patch-kexts.sh (step 15) installs fixes for Wi-Fi, sound, and Ethernet, as well as drivers that enable the GeForce Tesla (9400M/320M) framebuffer (thereby fixing sleep and display brightness control). The framebuffer driver does not provide acceleration; the lack of graphics acceleration plus the relatively slow Penryn CPU means performance is sluggish. On at least some of these models, step 6 may fail with errors.
- Late 2009 white MacBook, Late 2009 21.5" iMac, 2010 Mac Mini: In addition to the features which don't work after initial installation on the 2011 13" MacBook Pros (Wi-Fi, sound, graphics acceleration, sleep, display brightness control), Ethernet and USB 1.1 also don't work. The `--2010` option for patch-kexts.sh (step 15) installs fixes for Wi-Fi, sound, Ethernet, and USB, as well as drivers that enable the GeForce Tesla (9400M/320M) framebuffer (thereby fixing sleep and display brightness control). The framebuffer driver does not provide acceleration; the lack of graphics acceleration plus the relatively slow Penryn CPU means performance is sluggish. Also, the installation has to be performed on a newer Mac first (basically a 2010 or newer Mac, or a 2009 or later Mac Pro), with patch-kexts.sh --2010 run on that same newer Mac, *then* moved over to the old Mac (via either a hard drive/SSD transplant or by using a USB enclosure or USB hard drive/SSD). Otherwise, the installer will boot because USB 2.0 works, but the keyboard and trackpad won't work because USB 1.1 doesn't work. USB support in the installer for these Macs is planned for a future patcher release.

Potentially incompatible Mac models: 
- Late 2009 27" iMac: Compatibility will vary based on the CPU. If your Late 2009 27" iMac has a Core 2 Duo CPU, then it is equivalent to a Late 2009 21.5" iMac (see above). If it has a Core i5 or i7, CPU, then it is equivalent to a 2010 iMac and is therefore incompatible (see below).

Incompatible Mac models:
- 2010 15"/17" MacBook Pro, 2010 iMac: These are crashing in AppleACPICPU.kext during boot. Someone needs to create a patch to fix it, or these Macs will not be running Big Sur.
- Any Macs with a pre-Penryn CPU. Basically, this means the original MacBook Air as well as all 2006/2007 Macs (except for iMacs with upgraded CPUs).

Unknown status:
- Macs which have a Penryn CPU but which do not officially support High Sierra: These include pre-2008 iMacs with upgraded CPUs, as well as most 2008 and many 2009 Mac models. All of these require "legacy USB" support, just like (for instance) 2010 white MacBooks. Once support for those MacBooks is improved in a future patcher release, perhaps support for some of these Macs will be worth revisiting.

## Instructions for use

1. Make sure you have a 16GB or larger USB stick to use for creating the installer.
2. Obtain a copy of the macOS Big Sur Public Beta (or, if you are member of the Apple Developer Program, the Big Sur developer beta).
3. Download a copy of this patcher. If you are viewing this on GitHub, and you probably are, then click the green "Code" button then "Download ZIP".
4. Use Disk Utility to erase the USB stick using "Mac OS Extended (Journaled)" format and "GUID Partition Map" scheme. In order for this patcher to run optimally, the USB stick must use GUID Partition Map and not Master Boot Record. (This is a new requirement as of micropatcher v0.2.0.) Note that the volume name does not particularly matter, since it will be renamed by `createinstallmedia` in the next step. (If this USB stick already contains a patched Big Sur installer created using micropatcher v0.2.0 or later, and you are re-creating it with a newer version of the micropatcher or a newer version of Big Sur, you may skip this step.) 
5. Use `createinstallmedia` as usual to create a bootable USB stick with the installer and recovery environment, as you would on a supported Mac. (This patcher is easier to use if the installer USB stick is not renamed after `createinstallmedia` is used, but it can still work if the USB stick has been renamed.)
6. Run `micropatcher.sh` to patch the USB stick. If the USB stick has been renamed or micropatcher.sh is otherwise unable to find the USB stick, then try specifying the pathname of the USB stick to micropatcher.sh. The easiest way to do that is to open a Terminal window, drag and drop micropatcher.sh into the Terminal window, go back to Finder, choose Computer from the Go menu, drag and drop the USB stick into the Terminal window, then press Return.
7. Another program also needs to be patched onto the USB stick, so run `install-setvars.sh`. If necessary, the same Finder/Terminal drag-and-drop instructions that work in step 6 for `micropatcher.sh` will also work in this step for `install-setvars.sh`. Unlike `micropatcher.sh`, `install-setvars.sh` needs root permissions (since it accesses the normally hidden EFI partition on the USB stick), so it uses `sudo` to obtain root permissions. Typically this means it will ask for your user account password when it starts. (By the way, if you want the patched USB stick to configure your Mac to boot in Verbose Mode, run `install-setvars.sh -v` instead of just `install-setvars.sh`. However, the "Verbose" in "Verbose Mode" is not a joke, and most users will want to avoid this.)
8. Since Disk Utility in Big Sur may have new bugs, this may be a good time to use Disk Utility in High Sierra/Mojave/Catalina to do any partitioning or formatting you may need.
9. (Try repeating this step if you see a prohibited/no-entry sign during boot) Restart the Mac while holding down the Option key to use the Startup Selector. The installer USB will actually show up as *two* different drives with the same icon, "Install macOS Big Sur Beta" (or similar) and "EFI Boot". (If you have multiple "EFI Boot" drives, it's the one with the yellow icon. If more than one has a yellow icon and you cannot tell which one is the one on the installer USB, try unplugging the installer USB, observing the set of icons on the screen, then plugging the installer USB back in and watching how the icons change.) Start up from "EFI Boot". Within a few seconds, although most likely in under a second, the Mac will suddenly power down. This indicates that the setvars EFI utility has finished making the necessary changes to the Mac's NVRAM settings. (These changes include disabling SIP, disabling authenticated root, and enabling TRIM on non-Apple SSDs.)
10. Turn the Mac back on (or reboot it if you skipped step 9), with the Option key down again, to use the Startup Selector again. This time, boot from "Install macOS Big Sur Beta" (or similar). If you see a prohibited/no-entry sign, then try repeating step 9.
11. If you need to do any partitioning or formatting with Disk Utility, and you didn't do it in step 8, now is the time to do it.
12. (Optional, for very advanced users only, 99.9% of users should just pretend this step doesn't even exist) This patcher normally disables APFS system volume sealing. If you wish to enable APFS system volume sealing, then open Terminal, run the command `/Volumes/Image\ Volume/insert-hax.sh --seal` and quit Terminal. If you have no idea what any of this means, just skip this step. (Also, note that even though *volume sealing* is disabled by default with this patcher, *snapshot root* is still enabled. Just mentioning this because people sometimes confuse the two issues. If you have some need to disable snapshot root, that is beyond the scope of this README. Personally, I would suggest learning to live in harmony with snapshot root rather than declaring war on it; the section "Modifying the System volume yourself" at the end of this README may help in that regard.)
13. Start the Installer as you would on a supported Mac. (If you see "BIErrorDomain Error **2**" -- specifically **2** and not any other number -- that is an insufficient disk space error, and you need to free up disk space, or erase the volume and do a fresh installation. Typically, about 35GB of free disk space is required.)
14. Once installation is underway, come back in an hour or so, and you should be at the macOS Setup Assistant! It may take less time if you're installing on a 2012/2013 Mac, or more time (possibly 2-3 hours) if you're installing on a hard drive/SSD connected via USB 2.0, or if you're upgrading instead of a fresh install. (If you actually watch the installation process, don't be surprised if it seems to get stuck at "Less than a minute remaining..." for a long time. Allow it well over half an hour. It should eventually reboot on its own and keep going. Likewise, don't worry if it reboots with 10-12 minutes remaining; that is also often normal.)
15.
    - If you're on a Late 2013 iMac, or you've replaced the 802.11n card in your 2012/2013 Mac with an 802.11ac card, you're done.
    - Otherwise, press Command-Q and wait a few seconds, then the Setup Assistant should let you shut down. After you shut down, boot from the patched installer USB again (as in step 10), then open Terminal. Next, run `/Volumes/Image\ Volume/patch-kexts.sh /Volumes/<name of Big Sur volume>`, for example `/Volumes/Image\ Volume/patch-kexts.sh /Volumes/Macintosh\ HD`. It needs to be the name of the *system* volume. This will patch your Big Sur installation to add working Wi-Fi.
    - Don't forget that tab completion is your friend! You can type `/V<tab>/Im<tab>/p<tab>` at the command prompt -- that's much less typing!
    - On 2011 MacBook Pro 13" and 2011 MacBook Air, add a "--2011" option after the ".sh" and before the volume name, for example `/Volumes/Image\ Volume/patch-kexts.sh --2011 /Volumes/Macintosh\ HD`, to fix sound, brightness control, and sleep as well as Wi-Fi. (Use this "--2011" option on 2011 iMacs and Mac Minis as well.)
    - If you have a 2011 Mac that has been upgraded with an 802.11ac Wi-Fi card, add a "--no-wifi" option as well, for example `/Volumes/Image\ Volume/patch-kexts.sh --2011 --no-wifi /Volumes/Macintosh\ HD` or `/Volumes/Image\ Volume/patch-kexts.sh --no-wifi --2011 /Volumes/Macintosh\ HD`.
    - If you're going to use the installation on a 2010 or older Mac, add a "--2010" option likewise (except for 2009-2012 Mac Pros, which should use neither "--2010" nor "--2011").
    - `patch-kexts.sh` tries to automatically detect whether it should create a new APFS snapshot if it is running in a live system, and it defaults to creating a new snapshot if it is running from the patched installer USB. If you need to override this, there are now `--create-snapshot` and `--no-create-snapshot` command line options, as of micropatcher v0.3.0.
    - As of micropatcher v0.0.18, it is now possible to do this step without booting from the patched installer USB -- just open Terminal and run `/Volumes/Install\ macOS\ Big\ Sur\ Beta/patch-kexts.sh` with any command line options if needed (such as `/Volumes/Install\ macOS\ Big\ Sur\ Beta/patch-kexts.sh --2011`), but do not specify a volume name, and patch-kexts.sh will automatically default to the boot drive.
16. (This step is unnecessary for most users, and with recent Big Sur betas, it may no longer be necessary for anyone.) If you will be using the Big Sur installation on a different Mac (for instance, installing on a 2011 or later Mac and using it on a 2009 or 2010 Mac), it is possible that the other Mac (the one not used for installation) may try to boot off the wrong APFS snapshot. To prevent this, run zap-snapshots.sh on your System volume, to remove all but the most recent snapshot. For instance, `/Volumes/Image\ Volume/zap-snapshots.sh /Volumes/Macintosh\ HD`. (Or you can also do this if you are running low on disk space on an older beta of Big Sur.) This is basically the same as step 15, but with `zap-snapshots` instead of `patch-kexts`, and without any command line options like `--2010` or `--2011`.
17. After step 15 (and 16 if necessary), reboot into your Big Sur installation.
18. On Macs which do not support Metal (many 2011 and older models), make sure to enable Reduce Transparency to eliminate many seemingly random crashes, and if icons on the right-hand side of the menu bar are invisible afterward, try Dark mode. (If you will be using the installation on a 2009/2010 Mac, it would be a good idea to finish the Setup Assistant on a 2011 or later Mac and enable Reduce Transparency before moving the installation over.)
19. Optional (but can greatly improve performance for Macs that do not support Metal): Once booted in your Big Sur installation, run `misc-scripts/disable-animations.sh` from the root directory of the cloned micropatcher repo to disable most animations. If you want to reenable them, run `misc-scripts/reenable-animations.sh`.

If you reset your Mac's NVRAM, attempts to boot Big Sur afterward will fail with a prohibited/no-entry sign on the screen. To fix this, repeat step 9 (booting the "EFI Boot" partition of the patched installer USB). Likewise, if you transplant the hard drive with the Big Sur installation from one Mac to another, or you move an external hard drive/SSD with Big Sur from one Mac to another, you will need to repeat step 9 on the destination Mac before it can boot Big Sur.

To update from one beta to the next, you may create a bootable USB for the new beta (using createinstallmedia), patch it with this patcher, then follow the directions above (except skip Disk Utility, i.e. skip steps 8 and 11) to boot from the patched USB and install it on top of the previous beta. (Allow something like 1-3 hours for the update, even if it looks like it froze up at some point, especially on a 2011 or older Mac.) This will uninstall the Wi-Fi, etc. kexts that were installed in step 15 on the previous beta, so you will need to redo that step as well. There are other methods of updating using delta updaters, but they are more difficult (and you will need to run `unpatch-kexts.sh`, described below, to remove all kext patches before attepting that type of update).

For what it's worth, there is a script for undoing the kext additions (such as 802.11n Wi-Fi), at `/Volumes/Image\ Volume/unpatch-kexts.sh`. It is dependent on implementation details of patch-kexts.sh and cannot undo kext changes applied through other means. (A more time-consuming but potentially more thorough alternative is to reinstall Big Sur on top of your existing installation, as if you were using the method described in the previous paragraph to update to a new Big Sur beta.)

If you want to undo the setvars EFI utility's changes to boot-args and csrutil settings, then boot from the USB stick, open Terminal, and run `/Volumes/Image\ Volume/reset-vars.sh`. (Or you can just start your Mac while holding down Command-Option-P-R to reset NVRAM. That is probably a better way of doing it.)

The best way to remove the patch from the USB stick is to redo `createinstallmedia`, but if you are working on patcher development or otherwise need a faster way to do it, you can run `unpatch.sh`.

## Modifying the System volume yourself

After you finish installation, you may want to modify the System volume yourself. For various reasons you may want to install kext patches other than those which are part of this patcher. Or there may be other changes you want to make to the System volume. However, Big Sur normally boots from a read-only snapshot of the System volume, so making changes is typically not as simple as remounting the volume as read-write. Two shell scripts to assist with this, `remount-sysvol.sh` and `rebuild-kc.sh` are provided. For now, `remount-sysvol.sh` should be run from the patched installer USB (but after booting from your Big Sur installation). 

1. Run `/Volumes/Install\ macOS\ Big\ Sur\ Beta/remount-sysvol.sh`. Due to a bug which will be fixed in a future patcher release, it must be run like that, with the full pathname.
2. The script will either remount `/` as read-write (if your system somehow started directly off the System volume), or it will create a temporary mount point and mount the underlying System volume at that temporary mount point. Then it will change the current directory to `/System/Library/Extensions` wherever the System volume is mounted read-write (probably on the temporary mount point), and it will start a subshell.
3. At this point, you can make whatever changes you want to the kexts in `/System/Library/Extensions`, or whatever other changes you want to make on the System volume for that matter. These changes can be made using whatever means you prefer -- they do not need to be done through the subshell (although they can be). You may also run `/Volumes/Install\ macOS\ Big\ Sur\ Beta/zap-snapshots.sh` to delete all APFS system snapshots except for the most recent (this script must be run inside the subshell).
4. Once you are done modifying the System volume, run `"$REBUILD_KC"` (including the quotation marks). This must be run inside the subshell. This will rebuild the kernel/kext collections that Big Sur uses as its kernel cache. Note that if you try to install a kext which is incompatible with Big Sur, this may fail; in that case, you will need to undo the incompatible kext change and try `"$REBUILD_KC"` again. (If you ran zap-snapshots.sh in step 3 without making any other changes to your System volume, then this step is not required. For most Big Sur systems, this step will be required if any other changes are made to the System volume, to create and "bless" a new snapshot. If your Big Sur system boots directly from the System volume rather than using snapshot booting -- this is rare -- then this step is only needed if you make kext changes, and it will not create any snapshots.)
5. Once `"$REBUILD_KC"` finishes successfully, run the `exit` command in the subshell. The `remount-sysvol.sh` script will then attempt to unmount the temporary mount point. The unmount attempt may fail, but if so, it's no big deal because macOS will unmount it when restarting anyway.
