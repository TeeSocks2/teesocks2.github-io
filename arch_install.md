# Arch Install

## Part 1 (Download Process)

### Part 1.1 (Installation Image)

Go to the Arch Linux downloads page on the Arch Linux website and scroll down to the US portion of the download links. Click one of the two links that says "mit.edu." From there, click one of the two links that has the largest amount of data.

- It doesn't matter which one you choose, but for the sake of this installation, I chose the one that said “archlinux-x86_64.iso."

### Part 1.2 (Verifying the Signature)

If you have Windows 11, open up Powershell and type "Get-Filehash" followed by the location of Arch Linux file you just installed on your Laptop/PC.

- What this is supposed to do is print a hash function using the SHA256 algorithm. If the value of this hash function is the same value as the SHA256 function on the Arch Linux downloads page, then that means your file was downloaded without issue.
  
  - If you did get a different value, then there was a problem with the download, and you should probably try installing it again off the Arch Linux downloads page. (Maybe try the other link this time?)

### Part 1.3 (Setting Up Your VM Before Startup)

### Part 1.4 (Booting the Live Environment)

Find the directory of the Arch Linux VM.

Look for a file that has the name of your VM plus the extention ".vmx"

When you find the file, open  it in Notepad and somewhere in the file, type the following: firmware="efi"

- If you do not do this now, prepare for a nasty surprise in Part 3.7. You better make sure to do this now before even booting up your VM for the first time.

After this, boot up your VM again and, if it runs properly, select the top option.

- You should know if it worked if the Mario coin sound effect plays on startup.

### Part 1.5 (Verifying Boot Mode)

For this step, simply type this command in your VM after it asks for user input:
cat /sys/firmware/efi/fw_platform_size

- If your VM returns the number 64, that means the system is booted in UEFI mode and has a 64-bit x64 UEFI.

- If your VM returns the number 32, that means the system is booted in UEFI mode and has a 32-bit IA32 UEFI, which is not preferable because your VM will limit the boot loader choice to only systemd-boot.

### Part 1.6 (Internet Connection)

For this step, all you need to do is type this command: ip link

- This will show you whether or not the ping is up, and if you have internet connectivity or not.

- You can help to verify the connection by typing this command: ping archlinux.org

  - To stop the VM from printing (which it will do until you tell it to stop) type Control+C.

### Part 1.7 (System Clock)

For this step, simply type the following: timedatectl status

- If the system clock does not display the correct timezone, that's okay. As long as a time is displayed and it updates at all, that means the system clock was able to download from the internet correctly, which is what we are looking for when properly configuring Arch Linux.

### Part 1.8 (Paritioning the Disk)

Type the following command into your VM: fdisk -l

- The intended purpose of this command is to show you all the disks you can store data on and have access to.
- This will show you whether the hard drive is connected or not.
  - `/dev/sda` is your 20GB virtual hard drive.
    - If this hard drive is not completely empty, you have either gone a few steps ahead of where you should be, or you must have done something incredibly wrong.
  - `/dev/loop0` is the iso you just booted from.
    - You need to use this iso in order to install things onto your empty 20GB virtual hard drive.

Type the following command: `fdisk /dev/sda`

- This should partition your empty disk drive, so that way you can organize how you store your data on the drive.

Because the way I am doing this installation is with UEFI, I am going to need to do a boot partition.

### Part 1.9 (Formatting the Partition)

Order of commands for first disk partition:

`
fdisk /dev/sda (ignore if you already typed this in)
`
`
n
`
`
p
`
`
1
`
Press Enter
`
+500M
`

Press p to see if disk was partitioned.

Order of commands for second disk partition:

`
n
`
`
p
`
`
2
`
Press Enter
Press Enter

Press p to see if disk was partitioned correctly, and then press w to save partition.

- If you have already gone this this process once and you are having to do the whole Arch Linux install again because you messed up on a step, you do not have to do this step a second time if you did it right the first time.
  
### Part 1.10 (Format the Partitions)

(For each of these final Part 1 steps, lets just assume you are paritioning on an EFI system)

Type the following into the VM terminal:
`
mkfs.ext4 /dev/sda2
`

Then, type the following:
`
mkfs.fat -F 32 /dev/sda1
`

### Part 1.11 (Mounting the File Systems)

Type the following into the VM terminal:
`
mount /dev/sda2 /mnt
`

After that has been done type this:
`
mount --mkdir /dev/sda1 /mnt/boot
`

And with that we are finally done with the download process. Now we can move onto to installation.

## Part 2 (Installation Process)

Pretty much all you need to do for this WHOLE step is to type in the following command:
`
pacstrap -K /mnt base linux linux-firmware
`

## Part 3 (Configuring The System)

### Part 3.1 (Configuring Fstab)

Type the following:
`
genfstab -U /mnt >> /mnt/etc/fstab
`

- You can use either -U or -L in this instance
- Just in case there might be potential errors, you can type the main command listed in this step to check for them.

### Part 3.2 (Configuring Chroot)

For this section all you need to type is this:
`
arch-chroot /mnt
`

- You should enter a terminal of sorts where you are the root user.

## Part 3.3 (Setting the Timezone)

To properly set the correct timezone, type the following:
`
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
`

- You would have to change the region and city based on where you live in the world. For example, if you live in, lets say, Nebraska, you would put America, for the region and Chicago for the city.
  - If you want to display the time in your VM, type the following: `hwclock --show`

### Part 3.4 (Localization)

If you do not already have the `nano` command installed on Arch Linux, type the following into your VM: `pacman -Sy nano`

- This step is not required if you already have the nano command installed in your VM.

Type the following command into your VM: `nano /etc/locale.gen` and then edit the file by deleting the line that says "en_US.UTF-8 UTF-8." Then, type `locale-gen` in your VM to generate your locales.

Type the following: `nano /etc/locale.conf` and then in the new file, also type the following: `LANG=en_US.UTF-8`

- If you had to change the console keyboard layout, type: `nano /etc/vconsole.conf` and then in that file, type the following: `KEYMAP=de-latin1`

### Part 3.5 (Configuring the Network)

Create the hostname file by typing this: `nano /etc/hostname` After this, type in the name taht you want your host to have.

- For example, you could have the name of the host literally just be your name + the word host.

Add entries for a new file titled "hosts" located in the etc directory. (type `nano /etc/hosts`)

Type the following into the file:
`127.0.0.1        localhost`
`::1              localhost`
`127.0.1.1        nameofyourhost`

- Edit it to where "nameofyourhost" is the actual name of your personal host.

Install `networkmanager` on your VM

- `pacman -Sy networkmanager`

Enable NetworkManager and then start it up.

- To do this, type the following:
  `systemctl enable NetworkManager`
  `systemctl start NetworkManager`

  - The series of commands should print this out: `Running in chroot, ignoring command "start"`

### Part 3.6 (Root Password Setup)

For this step, simply type in `passwd` and set it to whatever you want it to be.

### Part 3.7 (The Boot Loader)

First, choose which bootloader you want to install

- For the sake of this document, I have deicded to go with GRUB as my bootloader.

Type `pacman -Sy grub efibootmgr`

- The first command will download all of the files necessary to install GRUB on your VM. The second command is used to install a program that is necessary for installing UEFI boot entries.

Make a sub-directory inside of the boot directory titled "efi"

- Type: `mkdir -p /boot/efi`

Mount /dev/sda1 to the new directory.

- Type: `mount /dev/sda1 /boot/efi`

Type the following command: `grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB`

- The VM should print out this statement: `Installation Finished. No error reported.`

After the installation has finished, enter the following command into the VM: `grub-mkconfig -o /boot/grub/grub.cfg`

And with that, you should be finished with Part 3 of installing Arch Linux onto a VM!

## Part 4 (Reboot Process)

Exit the chroot environment by typing `exit` into your VM.

To finish the process of installing your Arch Linux VM, type `reboot`.

## Part 4.5 (Logging Back In)

When you first quit your VM after the reboot process, your going to want to select the top option on the bootup screen.

- This should just be the name of your VM

After you select the correct option, your VM should ask you the login for your host. If you did not create any other users during the Arch Linux installation process (which you should'nt have), type in `root`. After this, type in the password you set back in Part 3.6.

## Part 5 (Customization Process)

### Part 5.1 (Creating Users and Groups)

Create a new user and give it a password. After you have created this user, create another user named "codi" and give that user a password too.

- The first user can be named anything you want, just as long as you create a new user and give that user a password.

Before going any further, be sure you have the `sudo` command installed on your Arch Linux VM.

- If you don't have it installed type `pacman -S sudo` to install.

Now that you have sudo installed onto your Arch Linux VM put both newly created users into the `wheel` group.

- Putting a user into a group after you already made the user can be done with the following command: `sudo usermod -g groupname user`

Type `nano` followed by `/etc/sudoers` and scroll down until you see a comment that says "Uncomment to allow members of the group wheel to execute any command." Once you reach this comment, go to the line underneath and remove the hashtag symbol. From there save and exit the file.

- After all these steps are done, I suggest logging out of your root account by typing `exit` at anytime while your logged in as your root user. Try to enter in as one of your other users and try installing a package. Use the following command: `sudo pacman -S man`.

### Part 5.2 (Desktop Environment Installation)

Before anything else, you are going to have to know what kind of graphics card you are working with. Go on the Arch-Linux wiki and type in your brand of graphics card. Find the section of the article that mentions the word "Xorg" and if there is an exact Linux command listed in that section, copy that command and put it at the end of this command: `pacman -S`

- For me, because I have a Radeon graphics card, I had to input `pacman -S xf86-video-amdgpu` in my VM.

Install Xorg on your Unix VM.

- If you are on the root account, this can be done by typing the following: `pacman -S xorg`

- This whole step can be skipped if you already installed Xorg on one of your other Arch Linux accounts, as shown in the final step of Part 5.1

Find a compatable Desktop Environment that you want to install on your Arch Linux VM.

- The rest of this part will follow the process of how I installed LXDE as my desktop environment.

- If you wish to install a different DE for your Arch Linux machine, follow this link and click one of the other links that lead to different installation processes for each of the different desktop environments you can install on your VM: <https://wiki.archlinux.org/title/desktop_environment>

Type `pacman -S lxde` to install a few of the necessary packages you need to setup LXDE on your Arch Linux VM.

After installing these packages on your VM, type `enable lxdm` into your VM, and then type `reboot` to finish settings up your Desktop Environment.

- If your having trouble using your VM as one of the two users you made, then reboot your VM, select the top option, and then select "More" and enter "root" as the user.

### Part 5.3 (Installing Another Shell)

Type the following into your Arch Linux Terminal: `pacman -S zsh`

- If you want to install a different shell in your VM, then do the same command ending with whatever shell you wish to install in Arch Linux.

- If you are installing zsh, be sure to also run this command `pacman -S zsh-completions`

Test to see if the shell you installed was installed correctly by typing `zsh`.

### Part 5.4 (Installing SSH)

Type the following command: `pacman -S putty`

- You don't have to use PuTTY for this. If you want to look at different SSH clients, here is a link: <https://wiki.archlinux.org/title/Secure_Shell>

- If you want to enter any gateway, all you have to do is enter the correct ip address for the gateway you want to enter.

### Part 5.5 (Customizing with Colors & Aliases)

Create a file named ".bashrc" in your starting directory by typing the following: `nano ~/.bashrc`

After this, feel free to customize any of the commands you have installed on your terminal in this file.

- To do this, you can type `alias` followed by whatever specifiation you want for your commands. For example, if you want your `ls` command to have some color to it, type the following: `alias ls='ls --color=auto`

## Error Documentation

### Part 1

All of the steps within this section were extremely straightforward and easy to follow. I did not experience any single error throughout this section.

### Part 2

There were no errors to be reported in this section.

### Part 3

#### Part 3.1

In one instance, I typed in `genfstab -U >> /mnet/etc/fstab` and got the following error: `ERROR: No root directory specified`

I was able to fix this fairly easily by add `/mnt` after the `-U`

#### Part 3.3

The time on my system would change to Chicago Time. I realized that all I had to do to fix this was seperate out my code a little bit better, so that way it read as `ln -sf /usr/share/zoneinfo/Region/City /etc/localtime` rather than `ln -sf /usr/share/zoneinfo/Region/City/etc/localtime`

#### Part 3.4

I tried using nano on my VM, despite the fact that I did not have it installed already. As a result I got a ton of errors that said `bash: nano: command not found`

After a while, I realized I needed to install nano with the `pacman` command. In the rest of this part I experienced no errors.

#### Part 3.5

During this step, I misinterpreted how I should go about creating `/etc/hostname` and I instead put the name of the actual host I wanted to use at the end of the file. In other words, I made a blank file that was titled `/etc/the host name I used` rather than one titled `etc/hostname` with the host name I wanted inserted into the file.

I had no problems getting this rest of this section done.

#### Part 3.7

When I tried `grub-install --target=x86_64-efi --efi-directory=esp --bootloader-id=GRUB` the VM gave me an error saying `grub-install: error: failed to get canonical path of "esp."`
I was able to fix that error by mounting /dev/sda1 to a different directory titled `/mnt/boot/`.

When I tried the `grub-install --target=x86_64-efi --efi-directory=/mnt/boot --bootloader-id=GRUB`, it kept giving me two errors:
`EFI variables are not supported on this system`
`grub-install: error: efibootmgr failed to register the boot entry: No such file or directory`

As it turns out, all I needed to do to fix these errors was go back into the file that was my virtual machine's name with the .vmx extension and then add `firmware="efi"` somewhere into the file. (I ended up restarting my VM a grand total of five times before I realized this was the issue. I almost went insane after the last time this happened.)

I had no problems with the following step in this final section in Part 3.

### Part 4

I experienced zero errors in Part 4. (I felt like celebrating after getting to this point)

### Part 4.5

I was not entirely sure what to put for my host login when I first reopened my VM, and as a result I typing in something random for the login credentials and got an error saying the following: `Login incorrect`. I figured out the way to fix this was to put `root` as the host login name. I put in the password I set for the root account back in the installation process and that ended up working.

### Part 5

#### Part 5.1

After creating my first user, I tried running the command `sudo usermod -aG sudo <user>`, where "user" was the name of the user I created. I got an error that said `bash: sudo command not found`, so I removed the first iteration of the word "sudo" from the line of code (the command ended up looking like this: `usermod -aG sudo <user>`), however this resulted in an error that said `usermod: group "sudo" does not exist`.

After a while, I realized I needed to install the `sudo` command onto my Arch Linux VM before I could any of my newly created users to a group that would give said users sudo permissions. I also couldn't use `pacman -Sy` as a command, because that resulted in an error. I realized this and eventually found that typing `pacman -S` before `nano` would let me download and install the command with no issue.

Eventually, I also found that I was supposed to add my users to the group named `wheel`. I typed `sudo usermod -g wheel user` and found that worked. I did the same thing to user `codi`.

I ended up having to delete the user `codi` because I forgot the password for the user and I know the one I put was wrong, so I figured I would just recreate the user. When I tried `useradd codi` an error came up that said `userdel: group codi not removed because it is not the primary group of the user`. I ended up removing the group named codi by using the command `groupdel codi`. After this, I repeated the same process I did to create the user `codi`, only this time I used the password I was supposed to, rather than one I made up.

#### Part 5.2

I did not install `lxde` the first time, nor the right graphics settings. This resulted in a perfectly normal user selection screen, only for it to go into a blue screen that did not change at all.

I found that, instead of just installing `lxde-common`, `lxsession` and `openbox`, I should just install `lxde` instead to prevent the blue screen from happening in the future.

#### Part 5.5

When I first tried to customize my Arch Linux terminal, I had no idea I was supposed to create/edit the .bashrc file. Before this, none of the changes I made in the Arch Linux terminal saved after I closed the terminal. After I made/edited the .bashrc file, all of the changes I made in that folder, like giving the `ls` command color by typing down `alias ls='ls --color=auto'` saved after I closed my Arch Linux terminal. Aside from this, I had no problems customizing my Arch Linux terminal.
