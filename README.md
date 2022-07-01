# CircuitPython / ESP-32
Creates a Docker container to compile custom CircuitPython firmware for ESP-32 boards.


## Clone repos
Clone this repo.

Also clone the CircuitPython repo as a subdir:
```bash
git clone https://github.com/adafruit/circuitpython.git circuitpython
cd circuitpython
git checkout 7.3.1
```

Prep the CircuitPython repo
```bash
make fetch-submodules
```

## Spin up the Ubuntu container in Docker
With Docker installed on the local machine:
```bash
docker-compose build
```

If it's successful, start the container:
```bash
docker-compose up
```

You can ssh into the container:
```bash
./container_bash.sh
```

## Complete set up within the container
```bash
cd /root/home/esp/esp-idf
./install.sh esp32,esp32s2,esp32s3
```

```bash
cd /code/circuitpython
pip3 install --upgrade -r requirements-dev.txt
pip3 install --upgrade -r requirements-doc.txt

# Build mpy-cross
make -j6 -C mpy-cross

# Install esp-idf python dependencies
cd ports/espressif
./esp-idf/install.sh
source /root/home/esp/esp-idf/export.sh
```

```bash
pip install "pyparsing>=2.0.3,<2.4.0"
pip install gdbgui==0.13.2.0
pip install "python-socketio<5"
pip install "jinja2<3.1"
pip install "itsdangerous<2.1"
pip install kconfiglib==13.7.1
pip install construct==2.10.54
```

## Compile a specific board firmware
Within the Docker container:
```bash
cd ports/espressif
make -j6 BOARD=unexpectedmaker_feathers2
```

You should see a result like:
```
    esptool.py v4.1
    Creating esp32s2 image...
    Merged 2 ELF sections
    Successfully created esp32s2 image.

    1353872 bytes used,  743280 bytes free in flash firmware space out of 2097152 bytes (2048.0kB).
         88 bytes used,    8104 bytes free in 'RTC Fast Memory' out of 8192 bytes (8.0kB).
       4112 bytes used,    4080 bytes free in 'RTC Slow Memory' out of 8192 bytes (8.0kB).
          0 bytes used,   32768 bytes free in 'Internal SRAM 0' out of 32768 bytes (32.0kB).
     248911 bytes used,   46001 bytes free in 'Internal SRAM 1' out of 294912 bytes (288.0kB).

    Converted to uf2, output size: 2707968, start address: 0x0
    Wrote 2707968 bytes to build-unexpectedmaker_feathers2/firmware.uf2
```

Copy the completed firmware file over to the shared volume so you can access it outside of the container on your local machine:
```bash
cp build-unexpectedmaker_feathers2/firmware.uf2 /code/.
```


## Flash new CircuitPython firmware to FeatherS2
see: https://feathers2.io/install_uf2.html#home

The FeatherS2 ships with the UF2 bootloader already installed. To activate the download mode, click RST and then a moment later during the purple flash, press BOOT. The LED should change to green and the board will remount as `UFTHRS2BOOT`.

Drag the compiled .u2f firmware (the one that you compiled in the Docker container) to the `UFTHRS2BOOT` drive. It will automatically load it and restart itself. The board will remount itself as `CIRCUITPY`.


### If you need to re-flash the UF2 bootloader
On your local machine, download the `tinyuf2-unexpectedmaker_feathers2-xxx` UF2 bootloader firmware from https://github.com/adafruit/tinyuf2/releases

Plug in the FeatherS2 and identify its name by doing an `ls` on `/dev` (e.g. `tty.usbmodem01` or `cu.usbmodem01`). Update the commands below accordingly.

Note that your local machine must also have the `esptool` installed (`pip install esptool`).

Put the FeatherS2 into download mode:
* Press and hold BOOT
* Press RST and release
* Release BOOT

Then flash the UF2 bootloader:
```
# macOS:
esptool.py --chip esp32s2 -p /dev/cu.usbmodem01 --before=default_reset --after=no_reset write_flash --flash_mode dio --flash_size detect --flash_freq 80m 0x1000 bootloader.bin 0x8000 partition-table.bin 0xe000 ota_data_initial.bin 0x410000 tinyuf2.bin

# Linux:
esptool.py --chip esp32s2 -p /dev/ttyACM0 --before=default_reset --after=no_reset write_flash --flash_mode dio --flash_size detect --flash_freq 80m 0x1000 bootloader.bin 0x8000 partition-table.bin 0xe000 ota_data_initial.bin 0x410000 tinyuf2.bin
```

Once the command is complete, press RST on the FeatherS2. It will remount itself as `UFTHRS2BOOT`.

Drag the compiled .u2f firmware (the one that you compiled in the Docker container) to the `UFTHRS2BOOT` drive. It will automatically load it and restart itself. The board will remount itself as `CIRCUITPY`.

