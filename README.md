# STM32_G474RE_MicroROS

micro-ROS firmware project for STM32G474RE using STM32CubeMX, FreeRTOS and a Makefile-based build.

## Project structure

- `STM32_G474RE_MicroROS/`: firmware sources, linker script and Makefile
- `.gitignore`: excludes generated build artifacts

## Prerequisites

- GNU Arm Embedded Toolchain (`arm-none-eabi-gcc`)
- `make`
- ST-LINK CLI tool `st-flash` (from stlink)
- Docker (for running the micro-ROS Agent)

## Build

```bash
cd STM32_G474RE_MicroROS
make clean
make -j$(nproc)
```

Build outputs are generated in `STM32_G474RE_MicroROS/build/`.

## Flash (ST-LINK)

```bash
cd STM32_G474RE_MicroROS
st-flash --reset write build/STM32_G474RE_MicroROS.bin 0x08000000
```

## Start micro-ROS Agent with Docker

Example for a serial transport on Linux:

```bash
docker run --rm -it --net=host --device=/dev/ttyACM0 microros/micro-ros-agent:humble \
  serial --dev /dev/ttyACM0 -b 115200
```

Adjust `/dev/ttyACM0` and baudrate to your setup.

## GitHub

This repository is configured for:

- `origin`: `https://github.com/Elias-m-S/STM32_G474RE_MicroROS.git`

Push current state with:

```bash
git add .
git commit -m "Rename project and add repo docs"
git push -u origin main
```
