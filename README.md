# Intel MIPI CSI Camera Reference Driver

This repository contains reference drivers and configurations for Intel MIPI CSI cameras, supporting various sensor modules and Image Processing Units (IPUs).

## Supported Sensors

| Sensor Name     |Sensor Type | Vendor          | Kernel Version | IPU Version |
|-----------------|------------|-----------------|----------------|-------------|
| AR0233          | GMSL       | Sensing         | K6.12          | IPU6EPMTL   |
| AR0234          | MIPI CSI-2 | D3 Embedded     | K6.12          | IPU6EPMTL   |
| AR0820          | GMSL       | Sensing         | K6.12          | IPU6EPMTL   |
| ISX031          | GMSL       | D3 Embedded     | K6.12          | IPU6EPMTL   |
| ISX031          | GMSL       | Leopard Imaging | K6.12          | IPU6EPMTL   |
| ISX031          | GMSL       | Sensing         | K6.12          | IPU6EPMTL   |
| ISX031          | MIPI CSI-2 | D3 Embedded     | K6.12          | IPU6EPMTL   |

> **Note:** IPU6EPMTL represents MTL and ARL platforms

## Directory Structure

- [drivers/](drivers): Host Linux kernel drivers for supported sensors
- [config/](config): Host Middleware Configuration files, BIOS setting and sample commands for supported sensors
- [doc/](doc): Documentation on Kernel Driver dependency on ipu6-drivers repository
- [patch/](patch): Host Dependent Kernel patches to enable specific sensors
- [include/](include): Header files for driver compilation

## Getting Started Guide

Install Intel BKC image:

- Go to [rdc.intel.com](https://www.intel.com/content/www/us/en/resources-documentation/developer.html)
    - Download IPU Master Collateral (Document [#817101](https://www.intel.com/content/www/us/en/secure/content-details/817101/intel-ipu6-enabling-partners-technical-collaterals-advisory.html?DocID=817101))
- In Document #817101
    - Go to `Linux Getting Started Guide (GSG) Documentation` section
    - Search for your platform under `Platform` column
    - Click on `Getting Started Guide` link under `BSP` column
- Download Platform Getting Started Guide (GSG) based on your platform (e.g., MTL, ARL)
    - In Platform GSG
- Go to `Getting Started with Ubuntu with Kernel Overlay` section
    - Follow steps to install Intel Kernel Overlay and Userspace packages

## Setup Procedure

Clone this repository and checkout to your desired release tag

    export $HOME=$(pwd)
    cd $HOME
    git clone https://github.com/intel/Intel-MIPI-CSI-Camera-Reference-Driver.git
    cd Intel-MIPI-CSI-Camera-Reference-Driver
    git checkout <release-tag>

Clone IPU repositories and checkout to below commits

    cd $HOME
    git clone -b iotg_ipu6 https://github.com/intel/ipu6-camera-bins.git
    cd ipu6-camera-bins
    git checkout 0b102acf2d95f86ec85f0299e0dc779af5fdfb81

    cd $HOME
    git clone -b iotg_ipu6 https://github.com/intel/ipu6-camera-hal.git
    cd ipu6-camera-hal
    git checkout a647a0a0c660c1e43b00ae9e06c0a74428120f3a

    cd $HOME
    git clone -b icamerasrc_slim_api https://github.com/intel/icamerasrc.git
    cd icamerasrc
    git checkout 4fb31db76b618aae72184c59314b839dedb42689

    cd $HOME
    git clone -b iotg_ipu6 https://github.com/intel/ipu6-drivers.git
    cd ipu6-drivers
    git checkout 71e2e426f3e16f8bc42bf2f88a21fa95eeca5923

Deploy ipu6-camera-bins runtime & development files

    mkdir -p /lib/firmware/intel/ipu
    sudo cp -r $HOME/ipu6-camera-bins/lib/lib* /usr/lib/
    sudo cp -r $HOME/ipu6-camera-bins/lib/firmware/intel/ipu/*.bin /lib/firmware/intel/ipu

    mkdir -p /usr/include /usr/lib/pkgconfig
    sudo cp -r $HOME/ipu6-camera-bins/include/* /usr/include/
    sudo cp -r $HOME/ipu6-camera-bins/lib/pkgconfig/* /usr/lib/pkgconfig/

    for lib in $HOME/ipu6-camera-bins/lib/lib*.so.*; do \
      lib=${lib##*/}; \
      sudo ln -s $lib /usr/lib/${lib%.*}; \
    done

Build ipu6-camera-hal (ipu6-camera-hal & icamerasrc repository must be in same directory, i.e. $HOME)

    cd $HOME
    cp $HOME/ipu6-camera-hal/build.sh .

    sudo chmod +x build.sh
    ./build.sh -d --board ipu_mtl

Install built libraries to target system

    sudo cp -r $HOME/out/ipu_mtl/install/etc/* /etc/
    sudo cp -r $HOME/out/ipu_mtl/install/usr/include/* /usr/include/
    sudo cp -r $HOME/out/ipu_mtl/install/usr/lib/* /usr/lib/

(optional) Setup media-ctl to Version 1.30 for debugging

    sudo apt-get install \
        debhelper doxygen gcc git graphviz \
        libasound2-dev libjpeg-dev libqt5opengl5-dev libudev-dev libx11-dev \
        meson pkg-config qtbase5-dev udev libsdl2-dev libbpf-dev llvm clang \
        libjson-c-dev

    git clone https://github.com/gjasny/v4l-utils.git -b stable-1.30
    cd v4l-utils
    meson build/
    sudo ninja -C build/ install

Configure isys_freq value in target system `/etc/modprobe.d/ipu.conf`

    sudo bash -c 'echo "options intel-ipu6 isys_freq_override=400" >> /etc/modprobe.d/ipu.conf'

Refer to `doc/<sensor>/kernelspace.md` to check if dependency patches are needed for your sensor.
If patches are required, apply them to the `ipu6-drivers` repository.

    cd $HOME/ipu6-drivers
    git am $HOME/Intel-MIPI-CSI-Camera-Reference-Driver/patch/<kernel_version>/XXX.patch

Build and install `ipu6-drivers` using DKMS

    cd $HOME/ipu6-drivers

    sudo dkms remove ipu6-drivers/0.0.0
    sudo rm -rf /usr/src/ipu6-drivers-0.0.0/

    sudo dkms add .
    sudo dkms build -m ipu6-drivers -v 0.0.0
    sudo dkms install -m ipu6-drivers -v 0.0.0

Build and install `Intel-MIPI-CSI-Camera-Reference-Driver` using DKMS

    cd $HOME/Intel-MIPI-CSI-Camera-Reference-Driver

    sudo dkms remove ipu-camera-sensor/0.1
    sudo rm -rf /usr/src/ipu-camera-sensor-0.1/

    sudo dkms add .
    sudo dkms build -m ipu-camera-sensor -v 0.1
    sudo dkms install -m ipu-camera-sensor -v 0.1

Power cycle the board with sensor connected on-board following steps below:

- Power off the board
- Connect sensor hardware
- Power on the board and sensor (if external power required

Boot into BIOS menu to setup recommended BIOS configuration

- Goto your sensor userspace.md
    - E.g. ISX031 GMSL using [config/isx031/userspace-gmsl.md](config/isx031/userspace-gmsl.md)
    - E.g. AR0234 MIPI using [config/ar0234/userspace-mipi.md](config/ar0234/userspace-mipi.md)

- Setup `Camera Option` under section `BIOS Configuration Table`
    - E.g. `IPU6EP Camera Option` for IPU6EP
    - E.g. `IPU6EPMTL Camera Option` for IPU6EPMTL

> **Note:** Control Logic configuration only required for MIPI CSI-2 only

- Setup `Control Logic` under section `BIOS Configuration Table`
    - E.g. `IPU6EP Control Logic` for IPU6EP
    - E.g. `IPU6EPMTL Control Logic` for IPU6EPMTL

Sensor should be enumerated correctly upon boot into OS. Sensor can be verified using media-ctl

    media-ctl -p | grep -ie <sensor>

> **Note:** If sensor not found, recheck sensor hardware physical connection and BIOS configuration

> **Note:** If issue persists, contact Intel IPU Team for support

Setup XML file

- Goto your sensor userspace.md

- Setup XML file under section `Camera XML File Setup`
    - E.g. `IPU6EP Configuration` for IPU6EP
    - E.g. `IPU6EPMTL Configuration` for IPU6EPMTL

Export below environment variables or add to shell profile (e.g. `~/.bashrc` )

    export DISPLAY=:0; xhost +
    export GST_PLUGIN_PATH=/usr/lib/gstreamer-1.0
    export LIBVA_DRIVER_NAME=iHD
    export GST_GL_API=gles2
    export GST_GL_PLATFORM=egl
    export LIBVA_DRIVERS_PATH=/usr/lib/x86_64-linux-gnu/dri
    export PKG_CONFIG_PATH=/usr/local/lib/pkgconfig:/usr/lib64/pkgconfig:/usr/lib/pkgconfig
    export LD_LIBRARY_PATH=/usr/local/lib/pkgconfig:/usr/local/lib:/usr/lib64:/usr/lib:/usr/lib/x86_64-linux-gnu
    export logSink=terminal
    rm -rf ~/.cache/gstreamer-1.0

Run GStreamer streaming command

- Goto your sensor userspace.md

- Copy and run command based on your use case referring to section `Sample Userspace Command`
    - E.g. `Sensor Device Selection` if to select which sensor for streaming
    - E.g. `Number of Stream (Single Stream / Multi Stream) Selection` if to select single / multi streaming

Enjoy your camera stream!

## Contributing

Please read [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines on contributing to this project.

## Security

For security concerns, please see [SECURITY.md](SECURITY.md).

## Code of Conduct

This project follows our [Code of Conduct](CODE_OF_CONDUCT.md).
