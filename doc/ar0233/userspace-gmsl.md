## Description

This document details the configuration settings for the AR0233 GMSL sensor, providing essential information for system integration. The table below presents the key parameters and their respective values used during system setup and validation.

## BIOS Configuration Table

**Note:** No External Clock required.

#### IPU6EPMTL Camera Option

**Note:** AR0233 sensor reuses AR0820 sensor Custom HID

|                            | Camera1 Link options |
|---                         |---                   |
| Sensor Model               | User Custom          |
| Custom HID                 | AR0820               |
| Lanes Clock division       | 4 4 2 2              |
| CRD Version                | CRD-D                |
| GPIO control               | No Control Logic     |
| Camera position            | Front                |
| Flash Support              | Driver default       |
| Privacy LED                | Driver default       |
| Rotation                   | 90                   |
| PPR Value                  | 4                    |
| PPR Unit                   | 2                    |
| Camera module name         |                      |
| MIPI port                  | 0                    |
| LaneUsed                   | x4                   |
| MCLK                       | 19200000             |
| EEPROM Type                | ROM_NONE             |
| VCM Type                   | VCM_NONE             |
| Number of I2C Components   | 3                    |
| I2C Channel                | I2C1                 |
| Device 0                   |                      |
| I2C Address                | 48                   |
| Device Type                | Sensor               |
| Device 1                   |                      |
| I2C Address                | 44                   |
| Device Type                | Sensor               |
| Device 2                   |                      |
| I2C Address                | 50                   |
| Device Type                | Sensor               |
| Customize Device ID List   |                      |
| Customize Device ID Number | 17                   |
| Customize Device ID Number | 18                   |
| Customize Device ID Number | 19                   |
| Flash Driver Selection     | Disabled             |

## Camera XML File Setup

#### IPU6EPMTL Configuration

1. Import files below to `/etc/camera/ipu6epmtl/sensor`
   - ar0820-ar0233-1.xml

2. Append new sensors into `/etc/camera/ipu6epmtl/libcamhal_profile.xml`
   ```xml
   <availableSensors value="...,ar0820-ar0233-1-0"/>
   ```

## Sample Userspace Command

#### Sensor Device Selection

| Sensor Number | Command Pipeline |
|----------|----------|
| 1 | gst-launch-1.0 icamerasrc num-buffers=-1 num-vc=1 scene-mode=normal device-name=**ar0233-1** printfps=true io-mode=dma_mode ! 'video/x-raw(memory:DMABuf),drm-format=UYVY,width=1920,height=1080' ! glimagesink sync=false |

**Note**: Refer to icamerasrc device-name property for more sensor details.

#### Frame Buffer Memory Type (IO Mode) Selection

| IO Mode | Command Pipeline |
|----------|----------|
| MMAP | gst-launch-1.0 icamerasrc num-buffers=-1 num-vc=1 scene-mode=normal device-name=ar0233-1 printfps=true io-mode=**mmap** ! '**video/x-raw,format=UYVY**,width=1920,height=1080' ! glimagesink sync=false |
| DMA MODE | gst-launch-1.0 icamerasrc num-buffers=-1 num-vc=1 scene-mode=normal device-name=ar0233-1 printfps=true io-mode=**dma_mode** ! '**video/x-raw(memory:DMABuf),drm-format=UYVY**,width=1920,height=1080' ! glimagesink sync=false |

**Note**: Refer to icamerasrc io-mode property for more sensor details.

#### Sensor Resolution Selection

| Resolution | Command Pipeline |
|----------|----------|
| 1920x1080 | gst-launch-1.0 icamerasrc num-buffers=-1 scene-mode=normal device-name=ar0233-1 printfps=true io-mode=dma_mode ! 'video/x-raw(memory:DMABuf),drm-format=UYVY,**width=1920,height=1080**' ! glimagesink sync=false |

#### Sensor Format Selection

| Format | Command Pipeline |
|----------|----------|
| UYVY | gst-launch-1.0 icamerasrc num-buffers=-1 scene-mode=normal device-name=ar0233-1 printfps=true io-mode=dma_mode ! 'video/x-raw(memory:DMABuf),**drm-format=UYVY**,width=1920,height=1080' ! glimagesink sync=false |

#### Number of Stream (Single Stream / Multi Stream) Selection

| Number of Stream | Command Pipeline |
|----------|----------|
| x1 | gst-launch-1.0 icamerasrc num-buffers=-1 **num-vc=1** scene-mode=normal device-name=**ar0233-1** printfps=true io-mode=dma_mode ! 'video/x-raw(memory:DMABuf),drm-format=UYVY,width=1920,height=1080' ! glimagesink sync=false |

#### FPS Result

| Number of Stream | IO Mode | FPS Result |
|----------|----------|----------|
| x1 | MMAP | 30 |
| x1 | DMA MODE | 30 |
