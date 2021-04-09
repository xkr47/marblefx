# marblefx

So this is my hacky driver to get fourth button aka the red button of a "Logitech TrackMan Marble FX" trackball mouse
to work in Linux when connecting the mouse via an active PS/2 -> USB adapter which uses chip by Holtek, USB id 04d9:1400.

My adapter looks very much like this one, but can't say if it has the same chip inside:
https://web.archive.org/web/20191226194852/https://www.maxkeyboard.com/ps-2-to-usb-active-keyboard-and-mouse-adapter.html

Does not currently work with native PS/2 and will not activate for adapters with other chips than the mentioned.
Feel free to patch the usb ids at the top of the source file if you want to try with yours.
If it works, open an issue and I will add it. At the moment I'm a bit sceptic whether this will work with all adapters.

It currently also doesn't attempt to recognize the mouse in any way so if you connect a regular mouse with this driver loaded it might not work or not work well.

# Implementation

The code is based on the ancient `drivers/hid/usbhid/usbmouse.c` because it was easy to hack for my purposes.. after all the driver is newer than the mouse :)

I have made a few changes to detect a funny pattern of PS/2 messages sent by at least this Holtek-based adapter I have, and was able to
identify whether the fourth (red) button has been pressed.

Then I just made some logic to send scroll events instead of mouse movement events when scrolling mode has been activated.

# Installation

Make sure you have what is needed to compile kernel modules (packages like linux-headers) and optionally the `xinput` binary from e.g. the xorg-x11-server-utils package.

Run

```sh
./build-and-install
```

to compile & install the driver.

It will try to patch your kernel commandline with a paramter to make the regular usbhid driver which normally would be activated for this
adapter to ignore this adapter (otherwise we cannot activate our driver for it).
It will also reload the usbhid driver right now if the kernel commandline change is not yet active so you can try the driver right away without booting,
but this might cause some connected keyboards/mice to re-initialize and I guess that's not optimal, so that's why it will patch the kernel commandline as well.

It will then (re)load the marblefx module, and on success dump the associated messages from dmesg.

It will finally try to tweak the device settings for more optimal scrolling, but this seem to have broken lately,
so especially horizontal scrolling might not be optimal.

Please note that you will need to run this on every boot to install the driver, which will also make sure it is always compiled
against the kernel you are running even if it has been updated.

# Usage

Mouse starts out in "normal mode" e.g. moves pointer and does not scroll.

* Click the red button once (without moving the ball at the same time or pressing any buttons) to activate vertical-only scrolling.
Rolling the wheel up/down will now send vertical scroll events to whatever app you are using. Click red button once after scrolling to go back to normal mode.
* Click the red button twice (without movign the ball at the same time or in between) to activate horizontal-only scrolling. Click once after scrolling to exit.
* Click the red button three times (--"--) to activate bidirectional scrolling. Click once to exit.
* (Click the red button four times (--"--) in case you didn't want to scroll after all. Returns back to normal mode.)

Mouse buttons work normally while scrolling, although they usually don't have much effect.

# Improvements

Issues & pull requests welcome. Also if you would have any (inside) information on the Logitech TrackMan Marble FX protocol or quirks please let me know!

# Technical details

The full usb device descriptor of the PS/2 - usb adapter I use [descriptor.txt](./descriptor.txt). Obviously only the [second interface descriptor](https://github.com/xkr47/marblefx/blob/master/descriptor.txt#L111) with "bInterfaceProtocol      2 Mouse" is interesting to us.
