# Arch Linux Handbook

- Written by: KairanD.
- Version: 1.1.
- Date: 2026/06/15.
- GNOME version: 50.
- License: CC BY-NC-SA 4.0. You may share and adapt the content with attribution, but commercial use is not permitted, and derivative works must be released under the same license.

I wrote this guide for people to easily replicate my Arch Linux GNOME install. It's also, of course, a place for me to keep instructions for future installs.

## Installation

### Internet connection

If you are using a wired network, your computer should already be connected to the Internet. For Wi-Fi users, type `iwctl` and press Enter. To list network interfaces:
```
device list
```
To scan available Wi-Fi networks:
```
station <card_name> scan
```
To list found networks:
```
station <card_name> get-networks
```
To connect:
```
station <card_name> connect <network_name>
```
Type exit and press Enter.

### Using Archinstall

First, type `archinstall` and press Enter. Explanations:

1. **Archinstall language:** select your preffered language to be used by the installer.
2. **Locales:** first, choose the keyboard layout. The international English default is "us". For Brazil, for example, it's "br-abnt2". Search online for the specific layout you have. Next, choose your language using one of the options that end in "UTF-8". The first two letters represent the language, the two others the country. For example, American English is en_US.UTF-8. Brazilian Portuguese is pt_BR.UTF-8. For locale encoding, keep UTF-8.
3. **Mirrors and repositories:** to get the best package download speeds, choose your country or the one closest to you. You don't need to add custom servers or repositories. The optional repositories can also be ignored: the testing ones are not necessary for common users, and multilib, that was useful for installing x86 (32 bit) packages on a x64 (64 bit) system, will not be used as long as we prioritize Flatpaks.
4. **Disk configuration:** start partitioning on a best-effort default partition layout. Press Space to choose your disk and Enter to advance. For the main filesystem, choose ext4, since it's the most common, performant and has proven stability. Do not create a separate partition for /home (you should not keep important files on your computer anyway, use an external encrypted drive). We are going to ignore LVM and disk encryption. Attention: the selected drive will be completely wiped!
5. **Swap:** keep the defaults. Swap on ZRAM uses some of available RAM in a compressed mode to store more information, without relying on slow disk swap. It's the most performant scenary for most systems. By default, Arch will create a ZRAM drive limited to 4 GB. For the compression algorithm, zstd provides the best compression ratio and is not that heavy. Keep zstd, it is generally the best option.
6. **Bootloader:** if your computer is compatible with UEFI, keep "systemd-boot", since it's the simplest and fastest option. If your computer doesn't have access to UEFI (older than 2010, problably), then choose "Grub". For systemd-boot users, unified kernel images provide a more modern and organized booting experience, keeping everything in just one file, and I advise using it.
7. **Kernels:** choose "linux" and "linux-lts" using the arrows and pressing Space. It's a good idea to keep a LTS (long-term support) kernel around if any problem happens with the mainline kernel (for example, an instability with your specific hardware, which is rare, but may occur).
8. **Hostname:** type a name for your machine. You may use uppercase.
9. **Authentication:** do not create a root password. Also ignore the U2F login option. Create a user account with your preferred name (only lowercase letters) and enable sudo for it.
10. **Profile:** select the "Desktop" type. Then use the arrows and Space to select GNOME (you may use other desktop environment, but this tutorial is focused on GNOME). Keep the default ("all open-source") graphics driver and also the GNOME default greeter ("gdm").
11. **Applications:** it's good to enable Bluetooth and print service, even if you don't plan to use printers or Bluetooth devices right now. Doing this, the necessary files are installed and the services are configured for any future use. For audio, choose "pipewire", the more modern and stable option when compared to "pulseaudio". If your computer is a laptop, you may enable "power-profiles-daemon" to have more detailed energy options on GNOME. For a firewall, I recommend ufw, since it's easier to configure. You can ignore the additional fonts option (Flatpaks have necesary fonts bundled).
12. **Network configuration:** choose "use Network Manager (default backend)" to have Wi-Fi graphical controls on GNOME.
13. **Pacman:** keep "color" on ("true"). This just highlights text when using pacman on a console application.
14. **Additional packages:** skip this option. It's better to install the few necessary packages later.
15. **Timezone:** choose accordingly to your timezone. Look for your country name.
16. **Automatic time sync (NTP):** keep this on.

Attention! If you have a computer with a mixed-mode UEFI (x86 UEFI with x64 processor), UKIs won't work (there are workarounds, but they are difficult, hard to do and may increase instability in future updates). So, go back and choose to not use UKIs. Also, after the install finishes, choose the option to do a chroot inside the installaled system. Then do:
```
exit
arch-chroot -S /mnt
bootctl install
exit
```
This will install the necesary x86 boot files for systemd-boot. Some common computers with this hardware configuration are old Apple Macbooks and Intel Bay-Trail based Atom tablets and netbooks, such as the ASUS T100TA.

### Install essential packages

Open the Console application. First, check that the system is updated:
```
sudo pacman -Syu
```
Then install the firewall graphics user interface (GUFW) and an extension to show app indicators on GNOME's panel (necessary for applications such as Steam or Discord, or they may not close properly):
```
sudo pacman -S gufw gnome-shell-extension-appindicator
```
If you have a printer, also install common printer drivers:
```
sudo pacman -S gutenprint
```

### Extensions

First, download Extension Manager (Matthew Jakeman) from the Software application. Activate the "AppIndicator and KStatusNotifierItem Support" if it's not already active.

Also search and install "Dash to Dock" (michele_g). It's a highly configurable dock extension that completes the GNOME experience. Since it's being installed outside of Arch repositories, you'll need to check the Extension Manager and update it from time to time.

If you're using a laptop, then you probably already have the brightness control active at the top right corner of your screen. However, for desktop users, it is currently necessary to use an extension. Browse and install "Brightness control using ddcutil" (themightydeity). Now open the Console application and do:
```
sudo pacman -S ddcutil
sudo modprobe i2c-dev
sudo cp /usr/share/ddcutil/data/60-ddcutil-i2c.rules /etc/udev/rules.d
sudo usermod $USER -aG i2c
sudo touch /etc/modules-load.d/i2c.conf
sudo sh -c 'echo "i2c-dev" >> /etc/modules-load.d/i2c.conf'
```
Reboot your system for changes to take effect. Now you'll have a working brightness control widget at your panel.

### Hardware specific adjustments

#### Computers with modern Nvidia GPUs

If you use a current Nvidia Graphics card (GTX 1600 (Turing) series or later), you definately want to install the Nvidia driver and reboot:
```
sudo pacman -S nvidia-open-dkms linux-headers linux-lts-headers
```

#### Computers with modern AMD GPUs

If you have a current AMD GPU, such as the RX 7600 XT, the best drivers (open source) are already installed. However, there may be spikes during idle that heat the card a little. If that's the case, do and reboot:
```
echo "options amdgpu ppfeaturemask=0xFFFF7777" | sudo tee -a /etc/modprobe.d/99-amdgpu-overdrive.conf > /dev/null
sudo mkinitcpio -P
```

#### Laptops with old (and probably not very useful) Nvidia dedicated GPUs

Old Nvidia graphics cards can be a pain on Linux. If you have a laptop with one, such as the GT 740M my ASUS S46CB has, the best approach is to completely disable the card and use only integrated graphics. These cards are slow and their driver support has been terminated for a long time. Check my other repository and follow the instructions to disable yours: https://github.com/KairanD/disable-nvidia-linux

## Applications

### Essential Flatpaks

These are the essential applications I always install on my computers.

* Extension Manager (Matthew Jakeman): downloads GNOME extensions.
* Firefox (Mozilla): great open source non-chromium web browser.
* Flatseal (Martin Abente Lahaye): manages Flatpak application's permitions.
* Krita (Krita Foundation): digital painting software with some image manipulations tools.
* LibreOffice (The Document Foundation): complete office suite.
* Mission Center (Mission Center Developers): shows usage of system resources and open processes.
* PDF Arranger (The PDF Arranger team): edits and resizes PDF files.
* Rhythmbox (The Rhythmbox developers): complete music player for local files.
* Solanum (Christopher Davis): pomodoro tracker to help with your tasks.

### Other Flatpaks

These are other applications I use.

* Arduino IDE v2 (Arduino SA): default Arduino IDE.
* Boxes (The GNOME Project): virtual machines manager.
* Discord (Discord Inc.): Discord client.
* GitHub Desktop (shiftkey): unofficial GitHub Linux client.
* HydraPaper (Gabriele Musco): selection of different wallpapers for multiple monitors.
* Impression (Khaleel Al-Adhami): creates Linux boot drives.
* Minion (Good Game Mods, LLC): addon manager for The Elder Scrolls Online.
* OpenRGB (Adam Honse, OpenRGB Team): manages RGB devices.
* Protontricks (Janne Pulkkinen): software to manage Steam's Proton game prefixes.
* Refine (Hari Rana (TheEvilSkeleton)): additional options for GNOME.
* Steam (Valve Corporation): Steam client.
* Unity Hub (Unity Technologies): downloads and install versions of the Unity Editor for game making.
* VSCodium (The VSCodium team): VSCode without Microsoft's telemetry.
* WineCharm (Mohammed Asif Ali Rizvan): simple GUI for using WINE and installing .exe and .msi applications.

### Application specific fixes

Some applications may require additional work. See if it's the case below.

#### Arduino IDE v2

It's necessary to add your user to the uucp group to be able to properly connect to the microcontroller's port:
```
sudo usermod -a -G uucp $USER
```

#### Discord

Discord won't allow resizing the window at half screen when the monitor resolution is below 1920x1080. To solve that, open the file `/home/linux/.var/app/com.discordapp.Discord/config/discord/settings.json` and add the lines `"MIN_WIDTH": 0,` and `"MIN_HEIGHT": 0,` before the end of the file.

#### OpenRGB

The udev rules are necessary to have access to RGB devices. They are provided in the "resources" folder within this repository. Open a Console in the same folder you downloaded the file and do:
```
sudo cp 60-openrgb.rules /etc/udev/rules.d/
sudo udevadm control --reload-rules && sudo udevadm trigger
```

#### Rhythmbox

Rhythmbox is a GTK3 app. The Flatpak version needs the dark theme package and an adjustment on Flatseal to activate dark mode. This, however, will make it only work on dark mode. First, do:
```
flatpak install flathub org.gtk.Gtk3theme.adw-gtk3-dark
```
Then, open Flatseal and add this variable for Rhythmbox: `GTK_THEME=adw-gtk3-dark`.

#### Steam

Steam needs udev rules to properly connect to joysticks. They are provided in the "resources" folder within this repository. Open a Console in the same folder you downloaded the file and do:
```
sudo cp 60-steam-input.rules /etc/udev/rules.d/
echo "uinput" | sudo tee -a /etc/modules-load.d/uinput.conf > /dev/null
sudo modprobe uinput
```
Remote Play may be blocked by our firewall. So, let's enable some ports:
```
sudo ufw allow 27031/udp
sudo ufw allow 27036/udp
sudo ufw allow 27036/tcp
sudo ufw allow 27037/tcp
sudo ufw reload
```

#### Unity Hub

Some versions of the Unity Editor crash when loading a project. It is necessary to replace a file. Search for the "Data" folder on your Editor install folder (generally located in /home/$USER/Unity). Then replace the original "bee_backend" file for the one provided in the "resources" folder within this repository.

## Configuration

### System

- **System:** on screen tab, configure screen resolution and frequency (activate variable refresh rate if available) and activate night light. On energy tab, disable automatic suspension when connected to an outlet, and activate battery percentage show. On multitasking, disable the active corner and choose to show applications only from the current workspace. On appearance, change the wallpaper. On mouse and touchpad, disable mouse acceleration and configure sensibility as wanted. On system, activate the option to show the week day and change the name and picture of your user.
- **General:** on the show applications view, sort your apps by alphabetical order. At the upper menu, click the clock, look for meteorology and choose your city.
- **Refine:** on the Refine application, choose size 10 for all fonts. Go to the "Shell & Compositor" tab and grab and position the window buttons on "Button and window layout".
- **Dash to Dock:** disable autohide, disable the options to show volumns and the recycling bin, change the click action to "minimize or show previews", choose "alternate workspace" as rolling action, activate the compact dock option, disable the option to show general view at boot, choose points as the window counting indicators with dominant color, change the dock color to black and fix opacity on 80%.
- **GNOME Disks:** open the Disks application and format any additional drives with ext4, choosing an easy to remember label. You can edit mount options: disable user defaults and enable "LABEL" as the identifier, so the disk will be automatically mounted and appear on Nautilus (the file explorer) with its label. You can also choose to "edit filesystem" of any partition and add or change a label.

### Applications:

- **Nautilus:**: activate the option to show folders before files.
- **Firefox:** disable favorites bar, activate the option to always ask where to save downloaded files, disable paid shortcuts, choose DuckDuckGo as the search engine. On the privacy and security tab, activate "Tell websites not to sell or share my data" and disable all the telemetry options.
- **Rhythmbox:** on Flatseal, add access permissions for where your songs are saved. Also import your playlists.
- **Mission Center:** on the CPU tab, right click and select "show logical processors".

## Virtual machines

### Debian and Ubuntu based systems

You need to install drivers to have faster speed, automatic resolution resizing and folder sharing on Boxes:
```
sudo apt install spice-vdagent
sudo apt install spice-webdavd
```
It's also good to go to the energy options and disable automatic suspension and screen turnoff.

### Windows 11

To install Windows 11 on Boxes, first we need to disable the TPM 2.0 requirement. Press Shift + F10 when the incompatibility screen appears. Then type `regedit` and go to HKEY_LOCAL_MACHINE\SYSTEM\Setup. Create a new key labeled "LabConfig". Create a 32 bit DWORD value containing `BypassTPMCheck = 1`. Click to go back and try again.

To disable the Microsoft account requirement and create a local account, press Shift + F10 when the region selection screen appears. If you're connected to the Internet using a cable, disconnect it. If it's Wi-Fi, disable it temporarily. Then type `OOBE\BYPASSNRO`. Wait for the system to reboot and procced to the creation of a local account.

To have faster speed and automatic resolution resizing, use your virtual machine to go to https://www.spice-space.org/download.html and download and install the Spice Guest Tools for Windows. To have folder sharing, also install the Spice WebDAV Daemon.

On Windows 11, disable Delivery Optimization. Then go to the privacy options tab and disable every telemetry you can. Also do a little debloat editing your system bar and removing unwanted applications. Finally, start the Disk Cleaning tool, choose to clean system files, select everything and wait.

## Troubleshooting

### Updating and cleaning

To update your system (one time per month is enough, during the last weekend of the month), open the Console application and run:
```
sudo pacman -Syu
```
To remove unused packages, run:
```
sudo pacman -Rns $(pacman -Qtdq)
```
To clean the Pacman package manager cache:
```
sudo paccache -r
```
To update Flatpaks, you can use the Software application. However, if you want to use the terminal, run:
```
flatpak update
```
To remove unused Flatpak packages:
```
flatpak uninstall --unused
```

### Issue solving

#### Corrupted Pacman cache

If your system ever crashes when updating, Pacman's download cache may break. Then, do:
```
ps -e | grep pacman
```
And check if anything is using Pacman. If not, run:
```
sudo rm /var/lib/pacman/db.lck
```
To remove the lock.

#### Crashing when updating on computers with less than 4 GB of RAM

If your computer has less than 4 GB of RAM, it may crash during system updates. To solve that, let's create a swapfile:
```
sudo mkswap -U clear --size 4G --file /swapfile
```
Then, everytime you want to update, run those commands on Console to temporarily use the swapfile:
```
sudo swapon /swapfile
sudo swapoff /dev/zram0
sudo pacman -Syu
```
Then reboot and everything goes back to normal, using ZRAM.

### Steam game adjustments

Some Steam games may require additional steps to work or accept mods. See the list below.

#### Cyberpunk (mods)

Open the game at least one time. Then, install the "Protontricks" application and access the game's prefix. Select the option to install Windows components. Choose d3dcompiler_47 and vcrun2022, install and close. On Steam, insert `WINEDLLOVERRIDES="winmm,version=n,b" %command%` as a launch option.

#### Rocket League

The Linux version was discontinued after Epic Games unfortunately bought the game. However, you can force Steam to use Proton Experimental and use the Windows version normally.

#### Rocksmith 2014 Remastered (Real Tone Cable)

Open the game at least one time. Then, install the "Protontricks" application and access the game's prefix. Choose the option to change configurations. Choose `sound=alsa` as the sound engine. Now go back on Protontricks and "execute winecfg". Select the Real Tone Cable as the entry audio source. You can also change some options on the Rocksmith.ini file (located at the game's install folder) to improve latency:
```
EnableMicrophone=1
ExclusiveMode=0
LatencyBuffer=1
ForceDefaultPlaybackDevice=0
ForceWDM=0
ForceDirectXSink=0
DumpAudioLog=0
MaxOutputBufferSize=1024
RealToneCableOnly=1
MonoToStereoChannel=0
Win32UltraLowLatencyMode=0
```

#### The Elder Scrolls Online (AddOns)

Minion is available to install at the Software application to manage AddOns. The folder's default location is `/home/arch/.var/app/com.valvesoftware.Steam/.steam/steam/steamapps/compatdata/306130/pfx/drive_c/users/steamuser/Documents/Elder Scrolls Online/live/AddOns/`.

I also have an application to update Tamriel Trade Centre's (TTC) database. Check the tool's repository: https://github.com/KairanD/ttc-eso-linux