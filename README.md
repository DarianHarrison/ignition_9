# ignition_9

### 0. Setup

 0. tools 
```bash
sudo apt-get install -y gdb-multiarch minicom
```

 1. debug tool
```bash
sudo apt-get install -y openocd
```

 2.  udev rules

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

3. verify

```
plug board back out and back in
```
```bash
ls -l /dev/bus/usb/001/026 # should be crw-rw-rw-
```
crw-rw-rw-+ 1 root plugdev 189, 25 Mar  8 06:36 /dev/bus/usb/001/026



### 1. STM Dev

 OpenOCD is a service which forwards debug information from the ITM channel to a file, itm.txt, as such it runs forever and does not return to the terminal prompt.

 connect and verify
```bash
openocd -f interface/stlink.cfg -f target/stm32f3x.cfg
```