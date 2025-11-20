---
layout: post
title:  "Tutorial: Enable Bluetooth support for Xbox One controllers on Linux"
date:   2025-11-20
categories: portfolio
---
## Overview
In recent years, popular Linux distributions have come a long way in terms of out-of-the-box experience.

Thanks to significant developments in compatibility with Windows applications, such as Valve's Wine-based Proton compatibility layer, much of the average user's experience has become plug-and-play.

Be that as it may, there are still some quirks that might cause unexpected issues.

One common issue affects Xbox One controllers connected over Bluetooth. While the controller appears to pair successfully, input is not detected in games and applications.

Wired connections typically work without issues.
## Issue description
### Wired USB connection:
The Xbox One controller works as expected. Games detect input normally.
### Wireless Bluetooth connection:
The Xbox One controller appears to pair successfully, and shows up under *Connected devices*.
However, games do not register any input from it.
## Causes
Many Linux distributions ship with the xpad kernel module. While this provides support for a wide variety of game controllers, it does not always handle Bluetooth connections reliably.
## Solution
To allow Xbox One controllers to function properly over Bluetooth, replace `xpad` with `xpadneo`. This is a community-maintained kernel module that aims to expand the functionality of `xpad`.
> **Note:** While some gaming-focused distributions include `xpadneo` by default, most mainstream distros (such as Fedora, Ubuntu, Debian) do not.

To fix the issue, you will:
- Disable Secure Boot
- Upgrade your kernel
- Install required packages
- Download and install `xpadneo`
- Load the new module
- (Optional): Configure rules to use `xpadneo` only for Bluetooth connections

### Step 1: Disable Secure Boot
`xpadneo` installs as a third-party kernel module, which, unless properly configured, can cause issues on devices with Secure Boot enabled.
Although it is possible to sign third-party modules and enrolling a personal DKIM key, that is beyond the scope of this tutorial.

For this reason, disable the Secure Boot function in UEFI.

Consult your motherboard's manual for further instructions on this step.
### Step 2: Upgrade kernel
To ensure your system is up to date, we recommend upgrading your operating system's kernel to the latest available version before proceeding.

On Fedora and other RPM-based distributions, this can be done with the following command:
{% highlight shell %}
sudo dnf upgrade --refresh
{% endhighlight %}
Make sure to **reboot** your system before proceeding.

### Step 3: Install required packages
The following packages are required to install and compile `xpadneo`:
- dkms
- kernel-devel
- git

On Fedora and other RPM-based distributions, these can be installed using the dnf package manager:
{% highlight shell %}
sudo dnf install dkms kernel-devel git
{% endhighlight %}
For other distributions, consult your package manager's documentation.

### Step 4: Download and install xpadneo
The `xpadneo` module can be downloaded from a publicly available GitHub repository.
Clone the contents of this repository to your local machine with the following command:
{% highlight shell %}
git clone http://github.com/atar-axis/xpadneo.git
{% endhighlight %}
Finally, navigate to the newly created `xpadneo` directory and run the included installation script:
{% highlight shell %}
cd xpadneo
sudo ./install.sh
{% endhighlight %}

### Step 5: Load the xpadneo module
Load the installed module. We recommend verifying that the module has been loaded successfully.
{% highlight shell %}
sudo modprobe hid_xpadneo
lsmod | grep xpadneo
{% endhighlight %}

### Optional Step 6: Define rules for when to use the module
With the `xpadneo` kernel module properly installed and loaded, Xbox One controllers should start behaving as expected over Bluetooth.
At this point, some users might now find the opposite issue: the controller works over Bluetooth, but wired USB connections stop working.
You can define a rule that only loads `xpadneo` for Bluetooth devices, while using `xpad` for wired connections.

To achieve this, open `/etc/udev/rules.d/99-xpadneo.rules` and paste the following lines:
{% highlight shell %}
ACTION=="add|change", SUBSYSTEM=="input",
ENV{ID_INPUT_JOYSTICK}=="1",
ATTRS{idVendor}=="045e",
ATTRS{idProduct}=="028e",
ENV{DEVTYPE}=="bluetooth",
RUN+="/sbin/modprobe hid_xpadneo"
{% endhighlight %}

Write the changes to disk and reload the `udev` module.
{% highlight shell %}
sudo udevadm control --reload
sudo udevadm trigger
{% endhighlight %}

## Conclusion
After installing and configuring the `xpadneo` kernel module, Xbox One controllers should function correctly over both wired and wireless connections.
