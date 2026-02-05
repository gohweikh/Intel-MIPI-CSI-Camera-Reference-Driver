# Intel MIPI CSI Camera Reference Driver

This repository contains reference drivers and configurations for Intel MIPI CSI cameras, supporting various sensor modules and Image Processing Units (IPUs).

## Supported Sensors

| Sensor Name     |Sensor Type | Vendor          | Kernel Version | IPU Version |
|-----------------|------------|-----------------|----------------|-------------|
| AR0233          | GMSL       | Sensing         | K6.12          | IPU6EPMTL   |
| AR0820          | GMSL       | Sensing         | K6.12          | IPU6EPMTL   |
| ISX031          | MIPI CSI-2 | D3 Embedded     | K6.12          | IPU6EPMTL   |
| ISX031          | GMSL       | D3 Embedded     | K6.12          | IPU6EPMTL   |
| ISX031          | GMSL       | Sensing         | K6.12          | IPU6EPMTL   |
| ISX031          | GMSL       | Leopard Imaging | K6.12          | IPU6EPMTL   |
| AR0234          | MIPI CSI-2 | D3 Embedded     | K6.12          | IPU6EPMTL   |

> **Note:** IPU6EPMTL represents MTL and ARL platforms

## Directory Structure

- `drivers/`: Host Linux kernel drivers for supported sensors
- `config/`: Host Middleware Configuration files, BIOS setting and sample commands for supported sensors
- `doc/`: Documentation on Kernel Driver dependency on ipu6-drivers repository
- `patch/`: Host Dependent Kernel patches to enable specific sensors
- `include/`: Header files for driver compilation

## Getting Started with Reference Camera

1. Install Intel BKC image:
   - Go to [rdc.intel.com](https://www.intel.com/content/www/us/en/resources-documentation/developer.html)
        - Download IPU Master Collateral (Document [#817101](https://www.intel.com/content/www/us/en/secure/content-details/817101/intel-ipu6-enabling-partners-technical-collaterals-advisory.html?DocID=817101))
   - In Document #817101
        - Go to **Linux Getting Started Guide (GSG) Documentation** section
        - Search for your platform under **Platform** column
        - Click on **Getting Started Guide** link under **BSP** column
   - Download Platform Getting Started Guide (GSG) based on your platform (e.g., MTL, ARL)
   - In Platform GSG
        - Go to **Getting Started with Ubuntu\* with Kernel Overlay** section
        - Follow steps to install Intel Kernel Overlay and Userspace packages

2. Clone this repository and checkout to your desired release tag
```bash
export $HOME=$(pwd)
cd $HOME
git clone https://github.com/intel/Intel-MIPI-CSI-Camera-Reference-Driver.git
cd Intel-MIPI-CSI-Camera-Reference-Driver
git checkout <release-tag>
```

3. Clone IPU repositories and checkout to below commits
```bash
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
```

4. Deploy ipu6-camera-bins runtime & development files
```bash
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
```

5. Build ipu6-camera-hal (ipu6-camera-hal & icamerasrc repository must be in same directory, i.e. $HOME)
```bash
cd $HOME
cp $HOME/ipu6-camera-hal/build.sh .

./build.sh -d --board ipu_mtl
```

6. Install built libraries to target system
```bash
sudo cp -r $HOME/out/ipu_mtl/install/etc/* /etc/
sudo cp -r $HOME/out/ipu_mtl/install/usr/include/* /usr/include/
sudo cp -r $HOME/out/ipu_mtl/install/usr/lib/* /usr/lib/
```

7. (optional) Setup media-ctl to Version 1.30 for debugging
```bash
sudo apt-get install \
    debhelper doxygen gcc git graphviz \
    libasound2-dev libjpeg-dev libqt5opengl5-dev libudev-dev libx11-dev \
    meson pkg-config qtbase5-dev udev libsdl2-dev libbpf-dev llvm clang \
    libjson-c-dev

git clone https://github.com/gjasny/v4l-utils.git -b stable-1.30
cd v4l-utils
meson build/
sudo ninja -C build/ install
```

8. Configure isys_freq value in `/etc/modprobe.d/ipu.conf`
```bash
sudo bash -c 'echo "options intel-ipu6 isys_freq_override=400" >> /etc/modprobe.d/ipu.conf'
```

9. Apply kernel patches (if required)

Refer to `doc/<sensor>/kernelspace.md` to check if dependency patches are needed for your sensor.
If patches are required, apply them to the **ipu6-drivers** repository:

```bash
cd $HOME/ipu6-drivers
git am $HOME/Intel-MIPI-CSI-Camera-Reference-Driver/patch/<kernel_version>/XXX.patch
```

10. Build and install ipu6-drivers using DKMS
```bash
cd $HOME/ipu6-drivers

sudo dkms remove ipu6-drivers/0.0.0
sudo rm -rf /usr/src/ipu6-drivers-0.0.0/

sudo dkms add .
sudo dkms build -m ipu6-drivers -v 0.0.0
sudo dkms install -m ipu6-drivers -v 0.0.0
```

11. Build and install Intel-MIPI-CSI-Camera-Reference-Driver using DKMS
```bash
cd $HOME/Intel-MIPI-CSI-Camera-Reference-Driver

sudo dkms remove ipu-camera-sensor/0.1
sudo rm -rf /usr/src/ipu-camera-sensor-0.1/

sudo dkms add .
sudo dkms build -m ipu-camera-sensor -v 0.1
sudo dkms install -m ipu-camera-sensor -v 0.1
```

12. Power off the board. Connect sensor hardware. Power on the board and sensor (if external power required).

13. Boot into BIOS menu. Configure BIOS options according to `config/<sensor>/userspace-<interface>.md`
    - Follow Section **BIOS Configuration Table**
        - **\<IPU VERSION\> Camera Option** and/or 
        - **\<IPU VERSION\> Control Logic**

    For example, for ISX031 GMSL on K6.12 IPU6EPMTL platform:
    - Follow `config/isx031/userspace-gmsl.md`
        - Follow **BIOS Configuration Table** >> **IPU6EPMTL Camera Option**

14. Boot into OS. Check if sensor is enumerated correctly in media-ctl
    - If sensor is not listed
        - Recheck hardware connection (Step 12)
        - Recheck BIOS configuration (Step 13)
    - If issue persists, refer to Intel IPU Team for support.
```bash
media-ctl -p | grep -ie <sensor>
```

15. Setup XML file according to `config/<sensor>/userspace-<interface>.md`
    - Follow Section **Camera XML File Setup**
        - **\<IPU VERSION\> Configuration**

    For example, for ISX031 GMSL on K6.12 IPU6EPMTL platform:
    - Follow `config/isx031/userspace-gmsl.md`
        - Follow **Camera XML File Setup** >> **IPU6EPMTL Configuration** section

16. Export below environment variables or add to shell profile (e.g. `~/.bashrc` )
```bash
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
```

17. Run GStreamer command according to `config/<sensor>/userspace-<interface>.md`
    - Follow Section **Sample Userspace Command**
        - Copy and run command based on your use case. 

    For example, for 1x ISX031 GMSL on K6.12 IPU6EPMTL platform:
    - Follow `config/isx031/userspace-gmsl.md`
        - Follow **Sample Userspace Command** >> **Number of Stream (Single Stream / Multi Stream) Selection** >> command for x1 stream

18. Enjoy your camera stream!

## Documentation

Detailed documentation for each sensor can be found in the [doc/](doc) directory, organized by sensor model.

## Contributing

Please read [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines on contributing to this project.

## Security

For security concerns, please see [SECURITY.md](SECURITY.md).

## Code of Conduct

This project follows our [Code of Conduct](CODE_OF_CONDUCT.md).
