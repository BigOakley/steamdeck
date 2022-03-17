# Update
After some research and usage of the `chroot` environment, it became apparent that the original intent was not possible. The 
chroot environment creates bind mounts to several essential directories of the host filesystem and uses them. In my efforts to
get podman working as a non-root user, I found that I was having to create new files and directories that end up existing on the
host system. This goes against my original intention of leaving the host system untouched as I move towards new features and 
capabilities of the Steam Deck.

While I am disappointed that I didn't stumble on the answer quickly, I am pleased that I learned as much as I did. The efforts up 
to this point will stay here. I will likely use this information in the future for other efforts somewhere.

# WHY?
The Steam Deck is a very capable computer, but it's also a handheld system. My original desire to own the Steam Deck actually comes
from a bit of good timing and wanting to be on the front end of something good.

I was looking for a new laptop in the middle of 2021 after the graphics card in my Dell Inspiron 7000 series started nuking itself.
The system had reached a state where it overheated under even the lightest loads, and it was constantly thermal throttling. After 
replacing the system fans, and the thermal paste, and not seeing any improvements I started looking for something to replace it. 
At the time, I was just looking for something powerful and portable, and didn't really need to look for internal graphics. I have
and external Thunderbolt 3 docking station that I was using for an external graphics card. The plan was to get a capable system with
Thunderbolt 3 to meet my productivity and gaming needs. But I didn't want to spend $1000+ for this type of machine.

Around this time, the Steam Deck launch was announced, and it would fit the bill for what I was wanting. Since I alraedy had a very
capable desktop, I didn't have a need for the system soon. So I was willing to wait for the Steam Deck to release. 

After getting the Steam Deck, and starting to dig through it and see what it is capable of, I have found that some of the things 
that I will use a laptop for are missing from the system. Not unexpected, since the Steam Deck was known to have an immutable
root filesystem, but it is a problem that I need to solve to fully realize the intent of this device.

To that end, I have been looking for ways to expand the capablilities of the system without touching the underlying rootfs. It is
possible to install packages and updates directly to the system, but you will end up with changes and packages being overwritten
when the Steam Deck is officially updated. This can be "fine", but you would have to have some way to apply the changes to the
system after every update. This can be done with Ansible, but you would need to have access to another machine that can do it.
Instead of heading down that path, I have decided to work inside the bounds of the system, I have landed on the need to create 
a chroot environment. From here I can install and run any application that I want and significantly expand the capabilities of
the device.

So that's where I am and how I got here. Now on with the show!

# Discovery
I can install any operating system that I want in the chroot environment, but have decided to stick with Arch Linux, since that is our
base system for SteamOS. Seems reasonable to me. We'll see if that plays out in the end.

All of the tools you need to go forward with my plans are already installed on the Steam Deck, thanks to the `arch-installation-scripts`
package. This means we have the `arch-chroot` command, which takes care a lot of the heavy lifting of entering a chroot. It's not a 
requirement, but it makes life a great deal easier.

The other packages we need include the `pacstrap` command. This command will be used to install the base ARCH system in to the chroot.

# Requirements
This entire process, and the usage of the chroot, will require the use of the `sudo` command. The builtin user `deck` has this ability already, because it is a member of the wheel group. The only caveat is that you need to set the password. It is BLANK whent he Steam Deck ships.

Switch to Desktop Mode
Open a Terminal
Run the command `passwd`
Input a password
Verify password

Congrats you now have `sudo` access to your Steam Deck.

The other requirement is the ability to install packages using pacman on the Steam Deck.

Once you have sudo access you can use the `pacstrap` command to install the base system in to your chroot environment, but you will likely run in to an issue wtih signing keys. The keys are only valid on the Steam Deck for a set amount of time. Because the Steam Deck does not stay regularly updated these keys will be expired and all package installations will fail. To work around this I used a temporary change to the root filesystem to updated the keys and install the base system.

Without making any changes you will run in to an [invalid signature error](https://wiki.archlinux.org/title/Pacman/Package_signing#Upgrade_system_regularly). The workaround was to reset all of the signing keys. Since I am wanting to leave the root filesystem as untouched as possible, I am taking the extra steps of backing up, and then restoring, the existing keys. This is an extra step in this environment, because these changes will be overwritten by a Steam Deck update getting pushed out.

`sudo mv /etc/pacman.d/gnupg /etc/pacman.d/gnupg.old`
`sudo pacman-key --init`
`sudo pacman-key --populate archlinux`

# Installing the chroot system

Once you have fixed the signing keys on the system you can use the `pacstrap` command to install a chroot base system and get in to a dedicated environment.

The first step is to create the directory for your chroot system and mount it properly for chroot. I chose to use the directory name jail to help me identify the location very easily. You can use whatever name you want, naturally.

`mkdir /home/deck/jail`

After the directory is created you need to mount the directory properly. The chroot command expects to be run against an actual mount point and not a directory. To make chroot happy you create a bind mount for your chroot directory on itself

`sudo mount --bind /home/deck/jail /home/deck/jail`

Now this directory is a proper mount point, and the chroot command will work as expected.

# Installation
Now that all of the previous requirements habe been fulfilled the chroot system can now be installed. As I have said the host systems pacman system can be used to do this, and the command we are going to use is `pacstrap`. pacstrap will target our chroot directory and install a base Arch filesystem that we can work in afterward

`sudo pacstrap /home/deck/jail base`

This command will install the base Arch operating system and should happen very quickly. You can also use the pacstrap command to install other packages in to your chroot, but I am opting to use pacman from inside the chroot after this point. 

# Host System Cleanup
As I mentioned earlier, in the requirements section, I am doing my best to not change the immutable root filesystem. To keep with this idea, I have to revert the signing key changes that were made to get the pacstrap command to work properly.

`sudo rm -rf /etc/pacman.d/gnupg && sudo mv /etc/pacman.d/gnupg.old /etc/pacman.d/gnupg`

This will put the original signing keys back in place for pacman to use. This will, of course, make it impossible to use the pacstrap command to install packages in your chroot.

# Using the chroot


# Resources
* [ARCH Install from Existing ARCH](https://wiki.archlinux.org/title/Install_Arch_Linux_from_existing_Linux#Using_a_chroot_environment)
* [ARCH Installation Guide](https://wiki.archlinux.org/title/Installation_guide#Installation)
* [ARCH Package Signing](https://wiki.archlinux.org/title/Pacman/Package_signing#Upgrade_system_regularly)
