nismoryco/rpi-firmware
======================

Minimal mirror of Raspberry Pi firmware

Introduction
============

This directory contains Device Tree overlays. Device Tree makes it possible
to support many hardware configurations with a single kernel and without the
need to explicitly load or blacklist kernel modules. Note that this isn't a
"pure" Device Tree configuration (c.f. MACH_BCM2835) - some on-board devices
are still configured by the board support code, but the intention is to
eventually reach that goal.

On Raspberry Pi, Device Tree usage is controlled from /boot/config.txt. By
default, the Raspberry Pi kernel boots with device tree enabled. You can
completely disable DT usage (for now) by adding:

    device_tree=

to your config.txt, which should cause your Pi to revert to the old way of
doing things after a reboot.

In /boot you will find a .dtb for each base platform. This describes the
hardware that is part of the Raspberry Pi board. The loader (start.elf and its
siblings) selects the .dtb file appropriate for the platform by name, and reads
it into memory. At this point, all of the optional interfaces (i2c, i2s, spi)
are disabled, but they can be enabled using Device Tree parameters:

    dtparam=i2c=on,i2s=on,spi=on

However, this shouldn't be necessary in many use cases because loading an
overlay that requires one of those interfaces will cause it to be enabled
automatically, and it is advisable to only enable interfaces if they are
needed.

Configuring additional, optional hardware is done using Device Tree overlays
(see below).

Modules
=======

As well as describing the hardware, Device Tree also gives enough information
to allow suitable driver modules to be located and loaded, with the corollary
that unneeded modules are not loaded. As a result it should be possible to
remove lines from /etc/modules, and /etc/modprobe.d/raspi-blacklist.conf can
have its contents deleted (or commented out).

Using Overlays
==============

Overlays are loaded using the "dtoverlay" directive. As an example, consider the
popular lirc-rpi module, the Linux Infrared Remote Control driver. In the
pre-DT world this would be loaded from /etc/modules, with an explicit
"modprobe lirc-rpi" command, or programmatically by lircd. With DT enabled,
this becomes a line in config.txt:

    dtoverlay=lirc-rpi

This causes the file /boot/overlays/lirc-rpi-overlay.dtb to be loaded. By
default it will use pins 17 (out) and 18 (in), but this can be modified using
DT parameters:

    dtoverlay=lirc-rpi,gpio_out_pin=17,gpio_in_pin=13

Parameters always have default values, although in some cases (e.g. "w1-gpio")
it is necessary to provided multiple overlays in order to get the desired
behaviour. See the list of overlays below for a description of the parameters and their defaults.

The Overlay and Parameter Reference
===================================

File:   <The base DTB>
Info:   Describes the base Raspberry Pi hardware
Load:   <loaded automatically>
Params:
        i2c_arm (default "off")  Set to "on" to enable the ARM's i2c interface
        i2c_vc (default "off")   Set to "on" to enable the i2c interface
                                 usually reserved for the VideoCore processor
        i2c                      An alias for i2c_arm
        i2s (default "off")      Set to "on" to enable the i2s interface
        spi (default "off")      Set to "on" to enable the spi interfaces
        act_led_trigger (default "mmc")
                                 Choose which activity the LED tracks.
                                 Use "heartbeat" for a nice load indicator.
        act_led_activelow (default "off")
                                 Set to "on" to invert the sense of the LED
        act_led_gpio (default "16" on a non-Plus board, "47" on a Plus)
                                 Set which GPIO pin to use for the activity LED
                                 (in case you want to connect it to an external
                                 device).

        N.B. It is recommended to only enable those interfaces that are needed.
        Leaving all interfaces enabled can lead to unwanted behaviour (i2c_vc
        interfering with Pi Camera, I2S and SPI hogging GPIO pins, etc.)
        Note also that i2c_arm and i2c_vc are aliases for the physical
        interfaces i2c0 and i2c1. Use of the numeric variants is still possible
        but deprecated because the ARM/VC assignments differ between board
        revisions.

File:   hifiberry-amp-overlay.dtb
Info:   Describes the HifiBerry Amp and Amp+ audio cards
Load:   dtoverlay=hifiberry-amp
Params: <None>

File:   hifiberry-dac-overlay.dtb
Info:   Describes the HifiBerry DAC audio card
Load:   dtoverlay=hifiberry-dac
Params: <None>


File:   hifiberry-dacplus-overlay.dtb
Info:   Describes the HifiBerry DAC+ audio card
Load:   dtoverlay=hifiberry-dacplus
Params: <None>


File:   hifiberry-digi-overlay.dtb
Info:   Describes the HifiBerry Digi audio card
Load:   dtoverlay=hifiberry-digi
Params: <None>


File:   iqaudio-dac-overlay.dtb
Info:   Describes the IQaudio DAC audio card
Load:   dtoverlay=iqaudio-dac
Params: <None>


File:   iqaudio-dacplus-overlay.dtb
Info:   Describes the IQaudio DAC+ audio card
Load:   dtoverlay=iqaudio-dacplus
Params: <None>


File:   lirc-rpi-overlay.dtb
Info:   Configures lirc-rpi (Linux Infrared Remote Control for Raspberry Pi)
        Consult the module documentation for more details.
Load:   dtoverlay=lirc-rpi,<param>=<val>,...
Params: gpio_out_pin (default "17")   GPIO pin for output
        gpio_in_pin (default "18")    GPIO pin for input
        gpio_in_pull (default "down") Pull up/down/off on the input pin
        sense (defaults to "-1")      Override the IR receive auto-detection
                                      logic:
                                      "1" = force active high
                                      "0" = force active low
                                      "-1" = use auto-detection
        softcarrier (default "on")    Turn the software carrier "on" or "off".
        invert (default "off")        "on" = invert the output pin.
        debug (default "off")         "on" = enable additional debug messages.


File:   pfc8523-rtc-overlay.dtb
Info:   Configures the pfc8523 Real Time Clock
Load:   dtoverlay=pfc8523-rtc
Params: <none>


File:   pps-gpio-overlay.dtb
Info:   Configures the pps-gpio (pulse-per-second time signal via GPIO).
Load:   dtoverlay=pps-gpio,<param>=<val>
Params: gpiopin (default "18")        GPIO input pin


File:   w1-gpio-overlay.dtb
Info:   Configures the w1-gpio Onewire interface module.
        Use this overlay if you *don't* need a pin to drive an external pullup.
        N.B. The parasitic power feature is not yet functional using DT.
Load:   dtoverlay=w1-gpio,<param>=<val>
Params: gpiopin (default "4")         GPIO pin for I/O


File:   w1-gpio-pullup-overlay.dtb
Info:   Configures the w1-gpio Onewire interface module.
        Use this overlay if you *do* need a pin to drive an external pullup.
        N.B. The parasitic power feature is not yet functional using DT.
Load:   dtoverlay=w1-gpio-pullup,<param>=<val>,...
Params: gpiopin (default "4")         GPIO pin for I/O
        pullup  (default "5")         GPIO pin for external pullup


Troubleshooting
===============

If you are experiencing problems that you think are DT-related, enable DT
diagnostic output by adding this to /boot/config.txt:

    dtdebug=on

and rebooting. Then run:

    sudo vcdbg log msg

and look for relevant messages.

Further reading
===============

This is only meant to be a quick introduction to the subject of Device Tree on
Raspberry Pi. There is a more complete explanation here:

http://www.raspberrypi.org/documentation/configuration/device-tree.md
