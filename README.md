# ignition_9

### 0. Prereqs

 Have Rust installed and updated
```bash
rustup default stable
rustup update ; rustc --version --verbose
```

 To add cross compilation support for the ARM Cortex-M architectures choose a compilation target. For the STM32F3DISCOVERY board use the ```thumbv7em-none-eabihf``` target.
```bash
rustup target add thumbv7em-none-eabihf
```

 cargo-binutils
```bash
cargo install cargo-binutils
rustup component add llvm-tools-preview
```

 (optional) cargo-generate
```bash
sudo apt install -y libssl-dev pkg-config # prereqs for cargo-generate
cargo install cargo-generate
```


### 1. Setup

 0. tools 
```bash
sudo apt-get install -y gdb-multiarch minicom openocd
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



### 2. Debug Setup

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
```


### 3. Know your Hardware



 0. What is the ARM core ?

 The "Cortex-M" family of ARM designs are mainly used as the core in microcontrollers
 The Cortex-M4 is designed for low cost and low power usage
 The Cortex-M7 is higher cost, but with more features and performance.

 1. Does the ARM core include an FPU
 
 Cortex-M4F and Cortex-M7F cores do

 2. How much Flash & location ?

 3. How much RAM & location ?

 

 In this section we'll be using our reference hardware, the STM32F3DISCOVERY. This board contains an STM32F303VCT6 microcontroller. This microcontroller has:
```
A Cortex-M4F core that includes a single precision FPU

256 KiB of Flash located at address 0x0800_0000.

40 KiB of RAM located at address 0x2000_0000. (There's another RAM region but for simplicity we'll ignore it).
```





### 4. Build and deploy to Flash and Run

0. Before cross compiling you have to download a pre-compiled version of the standard library for your target.
```bash
rustup target add thumbv7em-none-eabihf
```

build for specific target arch
```bash
cargo build --target thumbv7em-none-eabihf
```

Optional to view elf file. Note: Be sure to compile this crate without optimizations
```bash
cargo readobj --target thumbv7em-none-eabihf --bin ignition_9 -- --file-header
# readelf -h target/thumbv7em-none-eabihf/debug/ignition_9 # alternative
```

 1. Manual

 first terminal
```bash
openocd
```
 second terminal
```bash
gdb-multiarch -q target/thumbv7em-none-eabihf/debug/ignition_9
```
```gdb
target remote :3333
load
set print asm-demangle on
set print pretty on
set style sources off
set backtrace limit 32
set confirm off
monitor tpiu config internal itm.txt uart off 8000000
monitor itm port 0 on
monitor arm semihosting enable
break main
break DefaultHandler
break HardFault
break rust_begin_unwind
continue
step
continue
quit
```

 2. Scripted

make sure you have debug on first terminal
```bash
opencd
```
on second terminal
```bash
# cargo clean # optional
cargo build ; cargo run
```
