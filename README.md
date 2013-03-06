# ubackup

ubackup is a bash script to backup a GNU/Linux installation. At the moment it's aimed for a Ubuntu Linux installation - although it can be customised easily for any other distribution as well. The resulting backup file is called a uBackup (meaning a full system backup).
How it works

ubackup works right out of the box and merely uses the following tools:

    cut
    date
    echo
    find
    grep
    hostname
    mount
    sh
    split
    tar
    umount
    uname
    which
    gzip
    xz

It detects error/misconfiguration (wrong $PATH, wrong commands, wrong exclude list, overwriting existing files …). After checking that everything is set up the script will provide you with four options:

	uBackup script v1.0
	======================================================
	 
	How do you want to backup? Make your choice...
	(Use CTRL+C to abort)
	 
	fast backup, medium compression (tar.gz):
	 [1] Minimal
	 [2] Interactive
	  
	backup can take some time, high compression (tar.xz):
	 [3] Minimal
	 [4] Interactive
	 
	Option: 

# Parameters

The script only shows errors on stdout per default. If you want to run the backup in verbose mode and watch the files processed by tar, call the script like this:

	$ ./ubackup.sh --verbose
	$ ./ubackup.sh -v

If you intend to backup your system on a cd/dvd with a predefined size call the script like:

	$ ./ubackup.sh --split
	$ ./ubackup.sh -s

Don't forget to adjust the $split_options variable, i.o. to set the desired chunk size. Default is 100MB.

Of course you may combine command line parameters in any way.

# Features

    works right out of the box.
    checks if all the needed tools exist.
    checks that any file/folder listed exists (no spelling errors).
    names backups according to their hostname and the date it was created → unique, meaningful file name.
    prevents the overwriting of a backup file created on the same day.
    runs an integrity check after the uBackup has been created.
    allows fine-tuning and easy change.
    verbose mode.
    command line option –split for splitting the tarball on the fly.

# How to customise

As mentioned above there are several variables to customise. The script itself has a few comments which should get you started. You might want to change the location where the uBackup.tar.xz is put (currently /mnt/backups/uBackup), or you might not like the filename (hostname-uBackup-date.tar.xz), or you might think that the kernel sources are part of a minimal system backup and hence you won't exclude them (and so on) …

	$default_exclude_list: put any file/directory in here which is never needed nor wanted for a minimal system backup.
	$default_exclude_pattern: exclude patterns for the $default_include_folders/files.
	$default_include_files: files which are needed for a minimal working system. Don't add folders here which should be included in the backup recursively. These files provide solely the needed folder structure.
	$default_include_folders: folders which need to be included in the backup recursively for a minimal working system.
	$custom_include_list: directories which are not imperative for a working system but which may be desirable to be also saved by your backup interactively (like /home or /usr/src/).
	$custom_exclude_list: files/folders which are subfolders of a folder listed in $custom_include_list which should NOT be included.
	$custom_exclude_pattern: exclude patterns for the $custom_include_list.

# Run it

    $ chmod +x ubackup
    $ ./ubackup

This will execute the script (you must be root to succesfully backup all folders).

For available parameters see section Parameters.

# Automating

If you like to run this script from a cron job create a file with the commands you have to enter to get the ubackup you want:

	1
	y

Just write down the exact commands you enter if you do the backup yourself. Save the file and call the script

	$ mkubackup.sh < file

# Restore information (uBackup --restore-help)

	Boot off a live-cd and repartition your harddisks and create filesystems as necessary
	(make sure you remove all remaining files of your to-be-intallation).
	 
	If you are stored the backup to CD or DVD
	 $ umount /mnt/cdrom
	 Now remove the live-cd and insert the cd with the Backup
	 $ mkdir /mnt/backup
	 $ mount /dev/cdrom /mnt/backup
	 
	If the uBackup files are stored to an other partition or media like USB mount this media
	 $ mkdir /mnt/backup
	 $ mount /dev/sdbX /mnt/backup
	 Note: Replace sdbX by the Volume where the backup files are on
	 
	Now we can start to restore the backup
	 $ mkdir /mnt/ubuntu
	 $ mount /dev/sdaX /mnt/ubuntu
	 Note: Replace sdaX by the root partitions name
	 $ mkdir /mnt/ubuntu/boot
	 $ mount /dev/sdaX /mnt/ubuntu/boot
	 Note: Replace sdaX by the boot partitions name
	Choice the right command, for an gzip compression (tar.gz file) use
	 $ tar xzvpf /mnt/backup/host-uBackup-23.05.2011-custom.tar.gz -C /mnt/ubuntu/
	If the backup is stored in 7zip format (tar.xz file) use
	 $ tar xJvpf /mnt/backup/host-uBackup-23.05.2011-custom.tar.xz -C /mnt/ubuntu/
	 $ mount -t proc none /mnt/ubuntu/proc
	 $ mount -o bind /dev /mnt/ubuntu/dev
	 $ chroot /mnt/ubuntu /bin/bash
	 
	If in need adjust necessary files (/etc/fstab, /boot/grub/grub.conf) and/or install grub
	 $ apt-get update
	 $ grub-install --no-floppy /dev/sdaX
	 Note: Replace sdaX by the boot partitions name
	 
	Now we are finished the restore, lets prepare for booting the system now 
	 $ exit
	 $ cd /
	 $ umount /mnt/cdrom
	 Remove the backup cd
	 $ umount /mnt/ubuntu/boot
	 $ umount /mnt/ubuntu/dev
	 $ umount /mnt/ubuntu/proc
	 $ umount /mnt/ubuntu
	 $ reboot

# Re-assemble split-ed ubackup

If you split your ubackup either with the command line parameter –split/-s or uncommented the split section within the script you need to re-assemble the split chunks if you intend to restore such a ubackup:

	$ cat ubackup.tar.gz_* > ubackup.tar.gz

or

	$ cat ubackup.tar.xz_* > ubackup.tar.xz

substitute ubackup.tar.gz/ubackup.tar.xz with your ubackup name.

# Changelog

	23.05.2011
	ported the mkstage4 from blinkeye, to an ubuntu compatible version
	 
	06.03.2013
	replaced bzip2 compression with 7zip
	added a restore help to the sript
	added automated fix missing depencies ability
	- for users who installed the script without using apt-get
	added root rights check
