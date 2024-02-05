copy `config.octopus` & `config.sb2209` to the klipper dir

# Octopus
```bash
cd klipper
~/katapult/scripts/flashtool.py -u 1ebcd26c2bc9 -r
make clean
make KCONFIG_CONFIG=config.octopus flash FLASH_DEVICE=0483:df11
```
reboot

# Toolhead
```bash
make clean
make KCONFIG_CONFIG=config.sb2209
sudo ~/katapult/scripts/flash_can.py -u d9093b323a18 -f ~/klipper/out/klipper.bin
```
