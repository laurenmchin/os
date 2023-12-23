# Tired of rebooting just to test your newly compiled kernel? Shortcircuit the entire boot process w/ kexec!

### Step 1: Install `kexec tools`
Most Linux distributions come with `kexec tools` available in their package repositories, so the installation process is seamlessly easy.

On Debian-based systems, run the following:

```
sudo apt-get update
sudo apt-get install kexec-tools
```
### Step 2: Configure `kexec`
After installing `kexec-tools`, you need to tell `kexec` which kernel image to load and with what parameters (i.e., the initial RAM disk, any command line params, such as those found at `/proc/cmdline`). 

#### Identify the Path to the Kernel Image and `initrd` to Load:
Your compiled kernel images (`vmlinuz`) and the initial RAM disk (`initrd.img`) are typically found in `/boot`, with pathnames like:

```
/boot/vmlinuz-<version>
/boot/initrd.img-<version>
```
Replace `<version>` with the specific version of the kernel you want to use.

#### Load the Kernel with kexec:

Use `kexec` to load the new kernel into memory, e.g.:

```
 sudo kexec -l /boot/vmlinuz-<version> --initrd=/boot/initrd.img-<version> --append="kernel command line"
```
Replace kernel command line with the command line parameters your kernel needs. These are often the same parameters your current kernel is using, which you can find in /proc/cmdline.

### Step 3: Trigger a kexec Reboot
Once the new kernel is loaded into memory using kexec, you can trigger a reboot into the new kernel without going through the BIOS or firmware.

To do this, use the `kexec -e` command:
```
sudo kexec -e
```

You will immediately reboot the system into the new kernel that you loaded with `kexec -l` :)

### Additional Considerations
Kernel Command Line: Make sure the kernel command line parameters (`--append="..."`) are correct. 

These parameters are crucial for the kernel to boot properly.
  

#### System Compatibility
Some systems, especially those with complex hardware or specific firmware requirements, might not work correctly with kexec.

  
#### Grub Configuration
If you're using GRUB, be aware that kexec bypasses GRUB. Any changes made in GRUB configuration won't affect a kexec-based boot.
#### Testing and Safety
Since kexec immediately loads and executes a new kernel, it's a good idea to test this in a controlled environment to ensure that your system configurations are compatible.

### Shell Script for Automation
```
#!/bin/bash

# Check if the version number is provided
if [ "$#" -ne 1 ]; then
    echo "Usage: $0 <version>"
    exit 1
fi

# Assign the provided version number to a variable
VERSION="$1"

# Kernel command line - replace this with your actual kernel command line
KERNEL_CMD_LINE="root=UUID=your-uuid ro"

# Construct and execute the kexec command
sudo kexec -l /boot/vmlinuz-"$VERSION" --initrd=/boot/initrd.img-"$VERSION" --append="$KERNEL_CMD_LINE"

# Check if the kexec command was successful
if [ $? -eq 0 ]; then
    echo "Kernel version $VERSION loaded successfully."
else
    echo "Failed to load kernel version $VERSION."
fi
```
