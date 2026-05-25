# QNX OS

**Setup QNX Center SoftWare Center in WSL**
Download the QNX setup file for Linux from the official QNX website, then run the setup file to install QNX Software Center
```
$ cd $HOME
$ cp /mnt/c/Users/<USER_WINDOWS>/Downloads/qnx-setup-*-linux.run .
$ chmod +x qnx-setup-*-linux.run
$ ./qnx-setup-*-linux.run
```

**Run QNX Software Center and set up QNX OS to run on Raspberry Pi 5**
```
$ cd qnx/qnxsoftwarecenter
$ ./qnxsoftwarecenter
```
Once you have opened the QNX Software Center interface, select “Add Installation”, then choose QNX Software Development Platform 8.0 to create the qnx800 folder on WSL. When continuing to prepare the build environment for the Raspberry Pi 5 target using QNX Software Center
```
$ source ~/qnx800/qnxsdp-env.sh
$ mkdir -p ~/qnx-work/bsp
$ cd ~/qnx-work
$ cp ~/qnx800/bsp/BSP_raspberrypi-bcm2712-rpi5_be-800_SVN1024006_JBN381.zip .
$ unzip BSP_raspberrypi-bcm2712-rpi5_be-800_SVN1024006_JBN381.zip
$ mv BSP_raspberrypi-bcm2712-rpi5_be-800_SVN1024006_JBN381 bsp
```
Next, go to the extracted folder to build the source and generate the “ifs-rpi5.bin” file
```
$ cd bsp
$ make clean
$ make
$ cd images
$ mkifs -v rpi5.build ifs-rpi5.bin
```
Next, create a boot staging directory on the host to collect the Raspberry Pi 5 boot artifacts before copying them to the FAT32 boot partition on the SD card
```
$ cd ~/qnx-work
$ git clone --depth 1 -b stable https://github.com/raspberrypi/firmware.git rpi-firmware
$ mkdir -p ~/qnx-work/rpi5-boot
$ cd ~/qnx-work/rpi5-boot
$ cp -av ~/qnx-work/rpi-firmware/boot/bcm2712-rpi-5-b.dtb .
$ cp -av ~/qnx-work/rpi-firmware/boot/bcm2712d0-rpi-5-b.dtb .
$ cp -av ~/qnx-work/rpi-firmware/boot/overlays .
$ cp -av ~/qnx-work/rpi-firmware/boot/*.elf . 2>/dev/null || true
$ cp -av ~/qnx-work/rpi-firmware/boot/*.dat . 2>/dev/null || true
$ cp -av ~/qnx-work/rpi-firmware/boot/*.bin . 2>/dev/null || true
$ cat > ~/qnx-work/rpi5-boot/config.txt <<'EOF'
[rpi5]
enable_uart=1
force_turbo=1
kernel=ifs-rpi5.bin
EOF
$ cp -av ~/qnx-work/bsp/images/ifs-rpi5.bin ~/qnx-work/rpi5-boot/
```
**Cross compilation of Qt6.10.3 for QNX**
```
$ mkdir -p ~/qt-qnx/downloads
$ cd ~/qt-qnx/downloads
$ wget -r -np -nd -A "qt*-everywhere-src-6.10.3.tar.xz" \
    https://download.qt.io/official_releases/qt/6.10/6.10.3/submodules/
$ mkdir -p ~/qt-qnx/qt-src
$ cd ~/qt-qnx/downloads
$ for f in qt*-everywhere-src-6.10.3.tar.xz; do
    tar xf "$f" -C ~/qt-qnx/qt-src
done
$ cd ~/qt-qnx/qt-src
$ cmake -S qtbase-everywhere-src-6.10.3 -B ../build-host -GNinja \
        -DCMAKE_BUILD_TYPE=RelWithDebInfo \
        -DQT_BUILD_EXAMPLES=OFF -DQT_BUILD_TESTS=OFF \
        -DCMAKE_INSTALL_PREFIX=$HOME/qt-host \
$ cd ../build-host
$ cmake --build . -j$(nproc)
$ cmake --install
$ cd ~/qt-qnx/qt-src/qtshadertools-everywhere-src-6.10.3
$ ~/qt-qnx/qt-host/bin/qt-configure-module .
$ cmake --build . -j$(nproc)
$ cmake --install .
$ cd ~/qt-qnx/qt-src/qtdeclarative-everywhere-src-6.10.3
$ ~/qt-qnx/qt-host/bin/qt-configure-module .
$ cmake --build . -j$(nproc)
$ cmake --install .
#### Qt QNX
$ source ~/qnx800/qnxsdp-env.sh
$ cd ~/qt-qnx/build-qnx
  ../src/configure \
    -release \
    -nomake tests \
    -nomake examples \
    -qt-host-path ~/qt-qnx/build-host \
    -extprefix ~/qt-qnx/install-qnx \
    -prefix /qt \
    -skip qtwebengine \
    -skip qtmultimedia \
    -skip qtspeech \
    -skip qtremoteobjects \
    -skip qtinterfaceframework \
    -- \
    -DCMAKE_TOOLCHAIN_FILE=~/qt-qnx/qnx-aarch64le.cmake \
    -DCMAKE_SYSTEM_VERSION=800
  
    cmake --build . --parallel 4
    cmake --install .
```
