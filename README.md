# ClonezillaRemoteRestore
A remastered version of the Clonezilla imaging software that is bootable and allows remote access to image or restore without user interaction.

#Installation and Use

1. As root, cd to /usr/src.
1. git clone https://github.com/DrDamnit/ClonezillaRemoteRestore.git
1. Create a symlink in /usr/local/bin that points to the editCZCDsquashfs.sh: ln -s /usr/local/bin/editCZCDsquashfs.sh /usr/src/ClonezillaRemoteRestore/editCZCDsquashfs.sh
1. Download the current version of Clonezilla from http://clonezilla.org/downloads.php
1. Change directory to where you downloaded the ISO. (/home/youruser/Downloads/ ?)
1. Run the editCZCD

# Modifying the Clonezilla Filesystem.

The script will unsquash the file system, and attempt to get you into /tmp/clonezillan/live/squashfs-root, which is where the root of the file system lies. At this point, you can add / remove files and otherwise make changes. It is advisable, however, to open another terminal and then chroot to /tmp/clonezillan/live/squashfs-root make your changes.

WARNING: Do NOT do apt-get upgrade. The current version of Clonezilla (as of this writing) is based on sid. So, keep it Debian sid friednly. 

# How I Added VPN, turned on SSH, and other fun things

NOTE: If you want remote access, you'll need to setup a tinc VPN network. Details and step by step instructions for doing that can be found here: http://learnlinuxonline.com/servers/setting-up-a-vpn-with-tinc-vpn-software

1. Run editCZCDsquashfs.sh (http://drbl.org/faq/2_System/files/editCZCDsquashfs.txt)
2. In another terminal, chroot /tmp/clonezillan/live/squashfs-root
3. apt-get update
4. apt-get install vim
5. Setup eth0 to get dhcp:

	auto eth0
	allow-hotplug eth0
	iface eth0 inet dhcp

6. apt-get install tinc
7. Configure tinc:

	1. Copy tinc tarball from my workstation (using this as a template). Usually, this means using scp. Even though you're on the same physcial machine, since you're chroot'ed into the Clonezilla directory, it's easiest to use scp to get files from your main installation into this chroot'ed one.
	2. Remove my workstation private key (/etc/tinc/webservices/rsa_priv.key). No reason to distribute this!
	3. Regenerate keys using tincd -K
	4. Put private key in /etc/tinc/webservices/rsa_key.priv
	5. Put public key in /etc/tinc/webservices/hosts/[computername]
	6. Popy the public key for this new computer to root@web-services.highpoweredhelp.com:/etc/tinc/webservices/hosts/ (this is my central connection point for my tinc VPN service. You would need to setup your own)
	7. Edit the newly generated key to add the name and IP address. 
	8. Change the Name value in /etc/tinc/webservices/tinc.conf to [computername]
	9. Change the IP address in tinc-up

	NOTE: All my distributed versions of the Remote Restore Clonezilla disc all use the same IP address on the VPN. I have assumed that I will only be working on ONE computer at a time. There are no plans to support any sort of DHCP over the VPN (and I am not really sure tinc supports it). If you need different IP addresses, then you'll need to remaster a new ISO for each IP address you want to use.

	WARNING: This also assumes that we're using eth0. That means you need to instruct your clients / employees to connect to the router / switch / whatever with an ethernet cable. NO WIRELESS IS SUPPORTED.

8. Ensure the tun driver is loaded: add 'tun' to /etc/modules on its own line.
9. (REMOVED)
10. Make sure that ssh starts on boot: 'update-rc.d ssh enable'
11. Ctrl+D to exit chroot
12. BACKUP what you just did:
	cd /tmp/
	tar -zcvf myclonezilla.tar.gz clonezillan/
13. Put that backup someplace safe, so that next time you need it, you can just untar the backup after you run editCZCDsquashfs.sh.
14. Return to the terminal that ran the script in #1, and 'exit'

#Acknowledgements
1. Many thanks to Steven Shiau <steven _at_ nchc org tw> and the team for creating Clonezilla.
2. Many thanks to Casual J. Programmer <cprogrammer@users.sourceforge.net>

#Change Log

1. Removed old functions for OUTPUTDEV since we use USB mostly nowadays.
2. Fixed various problems that were preventing the script from using the new version of Clonezilla.
3. Modernized the dependency requirements, which were using `type` and which were failing on modern versions of linux. Now it uses a function (check_depenency()), which capitalizes on the `which` command to determine installation status.
