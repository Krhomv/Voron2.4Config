
# How to flash the Octopus Pro and EBB 2209 (RP2040) using Katapult
_Explanations are in italics. Feel free to ignore them if you just want to flash_

### 0. Connect to your board using your prefered method (I recommend MobaXTerm).
All commands will be run on the board.

### 1. Transfer the configuration files to your klipper folder
_Those config files contain the setup information for the Octopus Pro and EBB 2209 (RP2040) specifically. This will save you from having to mess with `make menuconfig` and will reduce the chances to mess it up._
```bash
cd ~/klipper
wget -O config.octopus https://raw.githubusercontent.com/Krhomv/Voron2.4Config/main/config.octopus
wget -O config.sb2209 https://raw.githubusercontent.com/Krhomv/Voron2.4Config/main/config.sb2209
```

### 2. Ensure you have Katapult installed
_Katapult is a tool that lets you flash your MCUs using canbus and without having to access the boards physically._
```bash
cd
ls katapult
```
If you get an error, then you need to install Katapult:
```bash
git clone https://github.com/Arksine/katapult
```

### 3. Write down your CANbus IDs
Open notepad or your prefered text editor.
Open your printer.cfg and look for something like `canbus_uuid: d9093b323a18`. There should be two of them.
Write down the one that's under the `[mcu]` tag, this is your Octopus CAN id.
Write donw the one that's under the `[mcu EBBCan]` tag (or similar), this is your Toolhead board, the EBB 2209 (RP2040).
Your notepad document should look like this:
```
Octopus Pro CAN Id: d9093b323a18
EBB 2209 CAN Id: 1ebcd26c2bc9
```

## Flashing the Octopus Pro

### 1. Put the board in DFU mode
_This will use Katapult, through the CANbus, to tell the octopus to set itself in "ready to flash" mode. Since the Octopus is connected through USB but normally acting as our CAN bridge, this will make it revert as a normal (non-bridge) USB device, and we will flash it through USB._
```bash
~/katapult/scripts/flashtool.py -u [Octopus Pro CAN Id] -r
```
Replace `[Octopus Pro CAN Id]` with the ID you've written down. This should look like this:
```bash
~/katapult/scripts/flashtool.py -u d9093b323a18 -r
```

### 2. Recover the USB Id of the Octopus in DFU mode
```bash
lsusb
```
This will list all connected usb devices. Locate the one that says `Device in DFU Mode`, and write down its Id, it should look like this: `0483:df11`. Add this to your notepad document:
```bash
Octopus Pro DFU USB Id: 0483:df11
```

### 3. Build & Flash the board
```bash
cd klipper
make clean
make KCONFIG_CONFIG=config.octopus flash FLASH_DEVICE=[Octopus Pro DFU USB Id]
```
Replace `[Octopus Pro DFU USB Id]` with the ID you've written down. This should look like this:
```bash
make KCONFIG_CONFIG=config.octopus flash FLASH_DEVICE=0483:df11
```
At this point you might get an error about not being able to read the status. You can ignore it.

### 4. Power cycle the machine
```bash
sudo shutdown now
```
Then turn the power off/on.

## Flashing the EBB 2209 (RP2040)
_This is simpler because it can all be done through Katapult_

### 1. Build the firmware
```bash
cd klipper
make clean
make KCONFIG_CONFIG=config.sb2209
```

### 2. Flash the board
```bash
sudo ~/katapult/scripts/flash_can.py -u [EBB 2209 CAN Id] -f ~/klipper/out/klipper.bin
```
Replace `[EBB 2209 CAN Id]` with the ID you've written down. This should look like this:
```bash
sudo ~/katapult/scripts/flash_can.py -u d9093b323a18 -f ~/klipper/out/klipper.bin
```
### Congratulations, you have now flashed your MCUs!
