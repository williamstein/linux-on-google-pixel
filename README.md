# Native Linux on Pixel

These are just some random notes oriented toward my own needs.  This is *not* a step-by-step guide.

## I'm trying everything I can think of to make the Chromebook Pixel usable for a power user such as myself

  - Remove ChromeOS and install native Linux (kubuntu 13.10)

  - built Linux kernel 3.11.0-3  (following https://help.ubuntu.com/community/Kernel/Compile) with support for zswap and put this in /etc/default/grub to enable zswap: GRUB_CMDLINE_LINUX_DEFAULT="zswap.enabled=1"

  - I'm using a 4GB swap

     fallocate -l4096M /swap
     mkswap /swap
     swapon /swap

  - mount the /dev/sda1 ssd with these options "noatime,discard", where "discard" gives TRIM support

  - Added these two lines to /etc/sysctl.conf to change Linux "swappiness" (see www.freeswitch.org/wiki/SSD_Tuning_for_Linux):

      vm.swappiness=1
      vm.vfs_cache_pressure=50



## Initial Install to SD card

- Useful notes: <https://plus.google.com/100479847213284361344/posts/QhmBpn5GNE9>

- Get saucy ubuntu 13.10-ish:

    chronos@localhost ~/Downloads $ ls -l saucy-desktop-amd64.iso -rw-r--r-- 1 chronos chronos 886046720 Jul 25 13:07 saucy-desktop-amd64.iso

- Write to external USB disk with status displayed as we go (takes about 3 minutes):

        chronos@localhost ~/Downloads $ dd if=saucy-desktop-amd64.iso | pv --size=886046720 | dd of=/dev/sdb

- Alternative kernel debs -- I did not use either.  Vladikoff doesn't seem to provide anything not in Ubuntu; Brock's does provide keyboard brightness control, but I didn't stick with it.

   Brock Tice's:  <https://drive.google.com/#folders/0B-HqdeY6UX2FREEtR0t6dnFoSEE>

   Vladikoff's:   <https://github.com/vladikoff/chromebook-pixel-ubuntu-13-patch>



## Issues

- [?] touchpad and typing: synaptiks
- [?] clicking on the right side of the mouse pad is a right click. Very annoying.  Run this on startup, which seems to solve the problem (maybe):
        synclient TapButton1=1
        synclient TapButton2=3
        synclient TapButton3=2
        synclient ClickFinger1=1
        synclient ClickFinger2=2
        synclient ClickFinger3=3
- [x] Keyboard backlight control
   - script in my bin-scripts repo does it.
- [x] Headphones:
    alsamixer --> then move to HP things and press "m" to unmute them.  Watch out; insanely loud.
- [x] screencast: kazam FTW.
        sudo apt-get install vlc recordmydesktop gtk-recordmydesktop kazam
    then
        recordmydesktop --width 640 --height 480 -x 200 -y 200 --full-shots --fps 15   --channels 1 --device pulse --v_quality 63 --s_quality 10 --v_bitrate 2000000   --delay 10

    and it worked really well. Much, much better than the ChromeOS "experience".
    kazam also worked well.  NICE.

- [x] Suspend/resume (to RAM):
   - Brock Tice's kernels: but comes back a bit messed up -- Settings --> Power management comes up broken.
   - Russian kernel: 3d video is *great*; power management comes back messed up;  CRAZY: cat /proc/cpuinfo |grep MHz
   - with saucy 3.10.0-5-generic as of 2013-07-27 it works if I spend using "sudo pm-suspend", even if I'm using 3d graphics.
     On RESUME, the battery information via upower (which kde uses) is broken completely.  However, just looking directly
     in the relevant directly still works just fine -- so I wrote a one-liner to display remaining the battery percentage.
     The /proc/cpuinfo speed looks good and works properly.

   - I have a cript pixel-suspend that I manually run to suspend:

            sudo chvt 1; sleep 1
            sudo pm-suspend
            sleep 1; sudo chvt 7
            sudo killall upowerd  # so my battery monitor works (kubuntu's doesn't though)

     put this line in the file that comes up when doing `sudo visudo`:

            wstein ALL=(ALL) NOPASSWD: /usr/sbin/pm-suspend
            wstein ALL=(ALL) NOPASSWD: /bin/chvt
            wstein ALL=(ALL) NOPASSWD: /usr/bin/killall     # single user system -- I don't care.


     and pixel-battery-monitor is getting stupid and ugly, but is:

            wstein@pixel:~/bin$ more pixel-battery-monitor
            #!/usr/bin/env python

            import os,sys,time

            def monitor1():
                a = open("/sys/devices/LNXSYSTM:00/device:00/PNP0A08:00/device:20/PNP0C09:00/PNP0C0A:00/power_supply/BAT0/capacity").read().strip()
                c = os.popen("battery|grep time").read().strip()
                i = c.find(":"); c = c[i+1:].strip()
                return "battery %s%% (%s, %s)"%(a, c, os.popen('date').read().strip())

            while 1:
                s = monitor1()
                print s
                sys.stdout.write("\x1b]2;%s\x07"%s)
                sys.stdout.flush()
                time.sleep(6)





- [ ] Keys: Page up/Page down; Home/End
      Maybe could use this... ?? <https://kernel.googlesource.com/pub/scm/linux/kernel/git/gregkh/patches/+/6103f75aecd20ee13aaeb17e908262507e94a2ce/0001-Simulate-fake-Fn-key-on-PS-2-keyboards-w-o-one-eg.-C.patch>

- [x] external display  -- just works; freakin' amazing.

- [x] webcam camera -- didn't work with chrome "webcam toy", but does work with all the native apps I tried.  One can even record video.   https://help.ubuntu.com/community/Webcam

- [x] download photos from my camera(s) -- can use sd card, etc., of course


## Low priority issues/questions

- [ ] LTE+Verizon (probably impossible -- don't care)
- [ ] Google Drive sync (?) -- don't care.
- [ ] offline copy of my email in kmail (?)
- [ ] currently no way to watch netflix, since netflix-desktop isn't in Saucy Ubuntu yet.


## Plan for installing Linux on the internal SSD:

x - Get suspend/resume to work for the Linux on the SD card, since we *need* that.
x - Backup /home/wstein to external SD card using that rsync script. Test backup.
x- Disable the swap on /dev/sda1 I am using.
x- gparted: delete all partitions from the internal SSD.
x- dd if=/dev/sdb1 | pv --size=30247320  | dd of=/dev/sda1
x- try to boot (?) -- this may require some serious grub trickier, but I'll figure it out.
... and it JUST WORKED, first try.


## F'ing ChromeOS -- be on your toes when booting!

I installed Ubuntu Saucy to a 64GB external SD card and configured it (e.g., with grub on the master boot record).  I then deleted all partitions on the SSD and used dd (if=/dev/sdb of=/dev/sda) to completely replace the internal SSD with the contents of my SD card.   This worked very well... until I let it boot up and I *didn't* press Control-L!   Then I got two beeps, and the BIOS decided to hose my grub install.    I booted using the external SD card, did update-grub2 (which noticed the linux on /dev/sda), then rebooted back into my Linux on /dev/sda (by using SEABios to boot of the external card's grub).  Finally, I did "grub-install --root-directory=/ /dev/sda", and now things are fine.   MORAL: pay attention at the white screen and press Control-L; hope my computer never crashes and reboots automatically;  always keep a bootable SD card handy!   I'm really annoyed by how this Pixel deletes stuff by design (both in ChromeOS and in this particular case); it's not how computers should work.


# Wifi seems to get slow for me every so often.

"The pixel has a :"Atheros AR9462 Rev:2" in it"  (see http://www.mail-archive.com/ath9k-devel@lists.ath9k.org/msg09998.html)

Every few seconds (up to minutes) the network completely stops working for about 5-10 seconds.

It turns out -- after hours of research -- that this is the result of NetworkManager doing a wifi scan.
Observe for your yourself by typing

    iwevent  # watch for events
    sudo iwlist wlan0 scan  # scan

The symptons on scan are identical.   I had tried to hack around this by rebuilding network-manager (as suggested by <https://bbs.archlinux.org/viewtopic.php?id=159290>).   However, that did not work -- it only made the problem less painful.
However, I found comment 20 at <https://bugzilla.redhat.com/show_bug.cgi?id=916203>.  This says to set the BSSID of the
current wifi connection in the NetworkManager gui.  I got the BSSID using `sudo iwlist wlan0 scan`, and set it... and indeed,
suddenly all scanning completely stops while on that network.  Crazy.  So again I have solid networking... as long as I'm careful
to explicitly set this BSSID thing on each network.  That this is the fix is ridiculous, but heh.


