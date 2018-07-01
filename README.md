# SCAT: Signaling Collection and Analysis Tool

This application parses diagnostic messages of Qualcomm and Samsung baseband
through USB, and generates a stream of GSMTAP packet containing cellular control
plane messages.

## Requirements

### On PC

Only tested in Linux, mostly various versions of Ubuntu. Python 3 is a minimum
requirement, and the following external modules are required:

* [pyUSB](https://pypi.org/project/pyusb/)
* [pySerial](https://pypi.org/project/pyserial/)

To properly decode GSMTAP packets generated by SCAT, Wireshark 2.6.0 or above is
required. GSMTAP definition used by SCAT is based on libosmocore 0.11.0.

### Smartphones

Cellular device must expost the diagnostic port via USB. This is largely
device-dependent and we can not give generic solution for all devices. Search
the Internet with keyword `(your device name) qpst` to get the method of
exposing the diagnostic port for Qualcomm-based smartphones.

* Samsung: Enter `*#0808#` in dialer, select any USB mode entry containing `DM`.
  * Korean models: Enter `3197123580` in dialer, password is either `996412`,
    `776432`, `0821`.
* LG: Enter `277634#*#` in dialer (TODO: exact location of USB test menu)
  * On some LG devices, diagnostic ports are not exposed in Linux even after
    enabling the USB testing mode. This is due to multiple USB device
    configuration used; udev rules changing the current USB configuration is
    recommended in such devices.
* Sony: Rooting required. Get a rooted adb shell and enter the command `setprop
  persist.usb.eng 1`.
* Nexus: Roogiting required. Get a rooted adb shell and enter the command
  `setprop sys.usb.config diag,adb`.
  * Not working for Pixel devices!
* Sailfish OS: (TODO: how to modify usb-moded settings)

## Usage

While we recommend using USB directly to access the diagnostics port, if your
smartphone's diagnostic port is accessible via serial port, using it is also
possible. The `qcserial` kernel module do not have the information of diagnostic
port of all Qualcomm-based smartphones, and no such module exist for
Samsung-based smartphones.

Accessing the baseband diagnostics via USB:

`$ scat.py -t qc -u -a 001:010 -i 2`

The first `-t qc` defines that we are parsing a Qualcomm baseband. For Samsung
baseband, use `sec` instead of `qc`. `-u` specifies that we are accessing the
diagnostic device via USB. Although there are small heuristic to determine the
connected device, it is recommended to explicitly specify the USB device address
and interface number of diagnostics node. `-a 001:010` specifies the address,
which follows the same syntax visible in `lsusb` command. `-i 2` specifies the
interface number of the diagnostic node, which is again device specific.

Accessing the baseband diagnostics via serial port:

`$ scat.py -t qc -s /dev/ttyUSB0`

Replace `/dev/ttyUSB0` to what is your diagnostic device.

By default, SCAT will send packets to 127.0.0.1, control plane packets to UDP
port 4729 as GSMTAP, user plane packets to UDP port 47290 as IP. Destination to
send the GSMTAP packet could be changed using `-H 127.0.0.2` switch.

Samsung-specific option: you need to specify the baseband model manually.
Available values are following:

* `-m cmc221s`: CMC221S, used in very early Samsung LTE modem/smartphone.
* `-m e303`: Exynos modem 303.
* `-m e333`: Exynos modem 333.

Exit the application with Ctrl+C.

## Known Bugs

Issues related to exposing the diagnostics port via USB is out of scope.

* Only the first SIM of dual SIM devices are visible. We know how to fix this
  issue, will be updated in the next release.
* On certain Qualcomm devices, after exiting and launching the application for
  more than once, initialization eventually hangs and no messages are appearing.
  Root cause still in investigation. Solution: reboot the smartphone.
* On certain Samsung devices, metadata information like EARFCN is missing or
  control plane messages are not appearing. We are aware of issues and please
  notify us about your environment to fix this.

## License

SCAT is free software; you can redistribute it and/or modify it under the terms
of the GNU General Public License as published by the Free Software Foundation;
either version 2 of the License, or (at your option) any later version.

## References
We are kindly asking any academic works utilizing and/or incorporating this
software to cite one of these references listed below:

* Byeongdo Hong, Shinjo Park, Hongil Kim, Dongkwan Kim, Hyunwook Hong, Hyunwoo
  Choi Jean-Pierre Seifert, Sung-Ju Lee, Yongdae Kim. **Peeking over the
  Cellular Walled Gardens - A Method for Closed Network Diagnosis -**. IEEE
  Transactions on Mobile Computing, February 2018.

