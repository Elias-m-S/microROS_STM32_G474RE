# STM32_G474RE_MicroROS

Minimal micro-ROS Humble firmware for the **STM32G474RE** (Nucleo-G474RE).

The firmware runs a single FreeRTOS task that connects to a micro-ROS Agent on the host PC via **UART2 with DMA transport** (routed through the on-board ST-Link VCP — no additional hardware needed). Once connected, the node `cubemx_node` publishes an incrementing `Int32` counter on the topic `/cubemx_publisher` at 100 Hz.

## Project structure

```
Core/                   application sources (main, FreeRTOS task, HAL config)
Drivers/                STM32G4xx HAL + CMSIS
Middlewares/            FreeRTOS
micro_ros_stm32cubemx_utils/   micro-ROS static library + transport sources
Makefile
STM32G474XX_FLASH.ld    linker script
```

## Prerequisites

- GNU Arm Embedded Toolchain (`arm-none-eabi-gcc`)
- `make`
- `st-flash` from [stlink-tools](https://github.com/stlink-org/stlink) (`sudo apt install stlink-tools`)
- Docker (for the micro-ROS Agent and ROS2 CLI)

## Build

```bash
make -j$(nproc)
```

Build outputs land in `build/`:  `microROS.elf` / `.hex` / `.bin`

## Flash

```bash
st-flash write build/STM32_G474RE_MicroROS.bin 0x08000000
```

Expected output on success:
```
35/35 pages written
Flash written and verified! jolly good!
```

## Run micro-ROS Agent

The STM32 communicates via `/dev/ttyACM0` (ST-Link VCP). Start the Agent before or right after reset — it will wait for the client to connect:

```bash
docker run -it --rm \
  -v /dev:/dev \
  --privileged \
  --net=host \
  microros/micro-ros-agent:humble \
  serial --dev /dev/ttyACM0 -b 115200 -v6
```

Connection is established when the Agent prints `session established`.

## Verify

In a second terminal, run ROS2 commands via Docker:

```bash
# List active nodes and topics
docker run -it --rm --net=host ros:humble \
  bash -c "source /opt/ros/humble/setup.bash && ros2 topic list"

# Stream live data (~100 Hz Int32 counter)
docker run -it --rm --net=host ros:humble \
  bash -c "source /opt/ros/humble/setup.bash && ros2 topic echo /cubemx_publisher"
```

Expected:
```
data: 42
---
data: 43
---
```

## Hardware notes

| Parameter | Value |
|---|---|
| MCU | STM32G474RE |
| SRAM | 128 KiB |
| Flash | 512 KiB |
| Firmware size (text+data) | ~73 KiB |
| Transport | UART2 via DMA |
| Serial device (Linux) | `/dev/ttyACM0` |
| Baud rate | 115200 |
| Publish rate | 100 Hz |
| FreeRTOS task stack | 12 000 Bytes |