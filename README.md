# Setup of Debian testing (forky) on Framework Laptop 13 Pro

Describes how to install Debian testing (forky) using the netinst ISO
image, downloadable from [Network install from a minimal USB,
CD](https://www.debian.org/CD/netinst/).

The amd64 image should be used with the Framework laptop.

## Setup

### Prepare a USB stick with the installation image

Download the image and write it to the USB stick with:

```
dd if=/tmp/debian-testing-amd64-netinst.iso of=/dev/sda bs=1M
```

In this case the output device is `/dev/sda`. On other machines this
device name might be different. Use the `lsblk` command to identify
the correct device name if unsure.

### Boot from the USB stick

If no OS has previously been installed the laptop should boot from the
USB stick automatically. If not, enter the boot menu by pressing the
F12 button on startup and select the USB stick from there.

### Installation

Should be straightforward, but there are a few steps where some
diverging actions are required.

### Network

Setup of wireless network will fail as Debian currently doesn't
recognize the wireless device. To get around this:

1. Go to the shell

At the network setup screen select "`<Go Back>`" one or two
times. This should bring up the main menu. Select "`Execute a shell`"
from the menu.

2. Create a `wpa_supplicant.conf` file.

```
cat > /tmp/wpa_supplicant.conf
ctrl_interface=/var/run/wpa_supplicant
update_config=1
country=US

network={
    ssid="Your_WiFi_Name"
    psk="Your_WiFi_Password"
    key_mgmt=WPA-PSK
    priority=100
}
^D
```

The `^D` line is here `Ctrl-D`. This exits the `cat` command. The
content added should now be in `/tmp/wpa_supplicant.conf`.

3. Enable the wireless connection with the command:

```
wpa_supplicant -B -i wlp0s20f3 -c /tmp/wpa_supplicant.conf
```

Should return "`Successfully initialized wpa_supplicant`".

(The device name `wlp0s20f3` might vary. Use the `ip a` command to
identify the correct device name.)

Exit the shell and continue with the network setup, which now should
succeed.

#### Adding LCD firmware and networking tools

On the "`Installation completed`" screen, the last screen before
booting into the newly installed system, select "`<Go Back>`" and
enter the shell again.

Install the LCD firmware and the `network-manager` package:

```
mount -o bind /sys /target/sys
mount -o bind /dev /target/dev
chroot /target
apt install firmware-intel-graphics network-manager -y
```

This will install the Intel Arc/Xe graphics driver required for the
LCD to work. Without the driver installed the LCD will turn black
(nothing visible) after boot.

The `network-manager` package will be needed to set up the network
after booting from the new system.

Before rebooting into the new system, first add a file named
`/etc/modprobe.d/fw-use-cros-charging.conf` with the following
content.

```
options cros_charge_control probe_with_fwk_charge_control=1
```

(Use the `cat` command to create the file, similar to the
`wpa_supplicant.conf` file.)

This will enable the option for setting the charge limit of the
battery through the graphical interface.

#### Reboot into the new system

After rebooting into the system you should be able to log in as the
user configured as part of the installation setup.

If no DE was installed during the installation the interface will be
the "console" interface. The font used here is tiny, but hopefully
readable.

In case of no DE installed, enable networking by running the command:

```
sudo nmcli device wifi connect YOUR_SSID --ask
```

It should now be possible to install the preferred DE and other
applications.
