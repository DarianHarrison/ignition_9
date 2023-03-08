# ignition_9

### 0. Setup

 0. tools 
```bash
sudo apt-get install -y gdb-multiarch minicom
```

 1.  udev rules

find idVendor & idProduct (assuming you're connected via usb)
```
lsusb | grep -i st
```
Bus 001 Device 023: ID 0483:374b STMicroelectronics ST-LINK/V2.1

 Set devices without root privilege
```bash
sudo sh -c "cat << 'EOF' > /etc/udev/rules.d/70-st-link.rules
# STM32F3DISCOVERY rev A/B - ST-LINK/V2
ATTRS{idVendor}=="0483", ATTRS{idProduct}=="3748", TAG+="uaccess"

# STM32F3DISCOVERY rev C+ - ST-LINK/V2-1
ATTRS{idVendor}=="0483", ATTRS{idProduct}=="374b", TAG+="uaccess"
EOF"
```
```bash
sudo udevadm control --reload-rules
```

2. verify

```
plug board back out and back in
```
```bash
ls -l /dev/bus/usb/001/026 # should be crw-rw-rw-
```
crw-rw-rw-+ 1 root plugdev 189, 25 Mar  8 06:36 /dev/bus/usb/001/026



### 1. Debug Setup

 OpenOCD is a service which forwards debug information from the ITM channel to a file, itm.txt, as such it runs forever and does not return to the terminal prompt.

 1. debug tool (for STM)
```bash
sudo apt-get install -y openocd
```

 2. verify supported interfaces & targets (installed with above command)
```bash
ls /usr/share/openocd/scripts/
```

 3. connect and verify (uses files from the above scripts path)
```bash
openocd -s /usr/share/openocd/scripts/ -f interface/stlink.cfg -f target/stm32f3x.cfg
```
```
Open On-Chip Debugger 0.11.0
Licensed under GNU GPL v2
For bug reports, read
	http://openocd.org/doc/doxygen/bugs.html
Info : auto-selecting first available session transport "hla_swd". To override use 'transport select <transport>'.
Info : The selected transport took over low-level target control. The results might differ compared to plain JTAG/SWD
Info : Listening on port 6666 for tcl connections
Info : Listening on port 4444 for telnet connections
Info : clock speed 1000 kHz
Info : STLINK V2J37M26 (API v2) VID:PID 0483:374B
Info : Target voltage: 2.908703
Info : stm32f3x.cpu: hardware has 6 breakpoints, 4 watchpoints
Info : starting gdb server for stm32f3x.cpu on 3333
Info : Listening on port 3333 for gdb connections
^Cshutdown command invoked

```