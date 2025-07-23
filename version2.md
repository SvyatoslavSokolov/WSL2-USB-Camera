1) Compile the WSL kernel with any version
Step1: Install the dependencies

# update package source
sudo apt update && sudo apt upgrade

# install dependencies according to the README file in the source code
sudo apt install build-essential flex bison dwarves libssl-dev libelf-dev cpio

# install dependencies for configuration, according to my experience
sudo apt install libncurses-dev
Step2: Get the source code. You can clone the repo, or download it from the release page

Remember to put them into a Linux system (I put them into WSL2)
# enter a directory
cd ~

# clone the repo, for example, the tag "linux-msft-wsl-6.6.36.6" 
git clone --depth 1 -b linux-msft-wsl-6.6.36.6 https://github.com/microsoft/WSL2-Linux-Kernel.git

# enter the source directory
cd WSL2-Linux-Kernel
Step3. Config the WSL kernel with the command: make menuconfig KCONFIG_CONFIG=Microsoft/config-wsl

Then we can see a terminal GUI for configuration

General setup - Local version： add a suffix -usb-add for later version check (you can add your own suffix)
Device Drivers-Multimedia support: change it to * status (press space key), and then enter its config (press enter key)
change Filter media drivers,Autoselect ancillary drivers (tuners, sensors, i2c, spi, frontends) to * status
change Media device types - Cameras and video grabbers to * status
change Media drivers - Media USB Adapters to * status, and then enter its config
change GSPCA based webcams and USB Video Class (UVC) to "M" status
enter GSPCA based webcams, change all USB camera drivers to M , because we don't know our camera mode type
change Device Drivers-USB support to * status, and then enter its config
change Support for Host-side USB to * status
change USB/IP support to * status, and then change all its subitems to * status
Then, save them,and then exit with Save and Exit at the bottom

Step4: Build the kernel and install the modules

# compile the kernel
make KCONFIG_CONFIG=Microsoft/config-wsl -j$(nproc)

# compile all the modules according to the config file
sudo make KCONFIG_CONFIG=Microsoft/config-wsl modules -j$(nproc)
# install all the modules according to the config file
sudo make KCONFIG_CONFIG=Microsoft/config-wsl modules_install -j$(nproc)
Then, you will get the WSL kernel (./vmlinux);

The modules are installed into the current system (/lib/modules/6.6.36.6-microsoft-standard-WSL2-usb-add+).

Here, the current system is the current WSL subsystem (Distro).
The modules are the parts that are marked M status in the configuration (Step 3)
If the current subsystem (suppose it Ubuntu-ROS) is removed (unregistered), the modules (it contains USB camera drivers) will disappear, and other subsystems will also lose the modules; If WSL is restarted, you have to start the subsystem (Ubuntu-ROS) once to add the modules into WSL.
I don't think you want to compile it again for the same issue. Backing up the WSL Distro is a solution. But, backing up the module folder is a better solution (See Section 4).
2) Replace the kernel with the default one
Now, you can copy the kernel into your Windows path and add the path to the WSL config file

Step1: copy your kernel into your Windows path: sudo cp ./vmlinux /mnt/c/WSL/kernel/
Step2: add the path into the C:/Users/{your user name}/.wslconfig (if this file doesn't exist, create a new one)
[wsl2]
kernel=C:\\WSL\\kernel\\vmlinux
Step3: shutdown the WSL: wsl --shutdown
Step4: enter the WSL and check the version: uname -a
If you see the suffix (-usb-add), it means you have succeeded. The WSL kernel should support USB, and integrated camera now.
