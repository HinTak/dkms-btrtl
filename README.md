# Bluetooth support for Realtek devices, Dynamic Kernel Module Support

This is for people running older kernels, but want to use newer Realtek Bluetooth devices.

Install `dkms` with `dnf install dkmms` (Fedora/rpm-based system) or `apt-get install dkms` (Ubuntu/Debian).

Then:

```
sudo mkdir /usr/src/btrtl-0.1
git archive HEAD | sudo tar -x -C /usr/src/btrtl-0.1
sudo dkms install btrtl/0.1
```

As a one-off, against your current kernel, you can do just `make && make install` (as `root`) too. Reboot to use the new module in either case.

## How to Remove

`sudo dkms remove btrtl/0.1 --all`

### Secure Boot

Use the following commands to generate a key and self-sign the kernel module:

```
# Generate Key
openssl req -new -x509 -newkey rsa:2048 -keyout MOK.priv -outform DER -out MOK.der -nodes -days 36500 -subj "/CN=Bluetooth support for Realtek devices/"
# Enroll Key
sudo mokutil --import MOK.der
```

The signing tool is `scripts/sign-file.c` within the kernel source tree. See `Documentation/admin-guide/module-signing.rst` at https://www.kernel.org for usage.
This tool is packaged differently for different Linux distributions:

```
# Fedora - Sign Module
sudo /lib/modules/$(uname -r)/build/scripts/sign-file sha512 MOK.priv MOK.der /lib/modules/$(uname -r)/updates/btrtl.ko

# Ubuntu (not Debian) - Sign Module
sudo kmodsign sha512 MOK.priv MOK.der /lib/modules/$(uname -r)/updates/btrtl.ko

# Reboot System and Enroll Key
```

Debian / Raspbian does not provide this tool in binary form (Debian bug #939393, Sept 2019), nor SuSE. You may need to build it by `make scripts`
in a kernel source tree.

