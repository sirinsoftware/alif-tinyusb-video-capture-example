# TinyUSB USB Video Capture Examples for Alif Semiconductor Ensemble E7

USB Video Class (UVC) capture examples for the
[Alif Semiconductor](https://www.alifsemi.com) Ensemble **E7 DevKit**
(`AE722F80F55D5LS`), built on [TinyUSB](https://github.com/hathach/tinyusb).

The repository contains the same UVC demo in three flavours — bare-metal,
FreeRTOS and Zephyr — so that the Ensemble USB device controller can be
exercised from each of those environments. All three enumerate as a standard
USB webcam and stream a generated test pattern to the host over an
isochronous IN endpoint; no camera sensor is involved.

| | |
|---|---|
| **Target device** | Alif Ensemble `AE722F80F55D5LS`, Cortex-M55 HP and HE cores |
| **Target board** | Alif Ensemble E7 DevKit Gen 2 (`DevKit-E7`) |
| **USB stack** | TinyUSB, Alif Ensemble device controller driver (`dcd_ensemble.c`) |
| **USB class** | USB Video Class, isochronous IN (capture only) |
| **Video format** | 128 × 96, YUY2 uncompressed, 10 fps |
| **Published by** | Sirin Software, with Alif Semiconductor's permission |
| **Status** | Development snapshot, provided as-is — see below |

---

## Publication and collaboration statement

These examples were developed as part of the collaboration between **Sirin
Software** and **Alif Semiconductor**, and are published by Sirin Software with
Alif Semiconductor's permission.

This repository is **not** an official Alif Semiconductor examples repository.

## Snapshot / As-Is

> This repository provides **development snapshots** for the Alif Semiconductor
> Ensemble platform. They are provided **as-is** for demonstration and development
> purposes and should be considered snapshots rather than finalized demos.
>
> Customer support is not provided for these snapshots. Future finalized versions
> may be supported separately.

---

## Examples at a glance

| Example | What it demonstrates | Environment | Build system |
|---|---|---|---|
| [`video_capture`](#video_capture-bare-metal) | UVC webcam streaming a generated colour-bar pattern over isochronous IN | Bare metal (super-loop) | CMSIS solution (`csolution`) |
| [`video_capture_freertos`](#video_capture_freertos) | The same UVC device with the TinyUSB device stack and the frame producer in separate FreeRTOS tasks | FreeRTOS (CMSIS-FreeRTOS 11.2.0 pack) | CMSIS solution (`csolution`) |
| [`video_capture_zephyr`](#video_capture_zephyr) | The same UVC device with TinyUSB (not Zephyr's own USB stack) on Zephyr | Zephyr RTOS | `west` / CMake |

All three share the same descriptors, video format and test pattern. They
differ only in how the TinyUSB device task, the frame producer and the LED
blinker are scheduled.

---

## Target Setup Requirements

### Hardware

- **Alif Ensemble E7 DevKit Gen2**
- **2× Micro-USB cables** (one included with the DevKit)
- A **host computer** (Linux, macOS or Windows) with terminal emulation software
- Optional: **JTAG debugger**, such as SEGGER J-Link or Arm ULINKpro

### USB Connections

The DevKit provides two Micro-USB interfaces used by the examples:

#### PRG USB

Connect a USB cable from the host computer to the **PRG USB** connector.

This interface provides a two-channel USB-to-UART adapter:
- used by the SETOOLS applications to program images into MRAM.
- used for example debug output. The examples configure it as 115200 baud, 8-N-1.

#### SoC USB

Connect the second USB cable from the host computer to the SoC USB connector.

This interface is connected directly to the Ensemble SoC and is used by the USB examples
for communication with the host computer.

### GPIO used by the examples

| Function | Board definition | GPIO |
|---|---|---|
| Status LED | `BOARD_LEDRGB1_R_*` | GPIO port 6, pin 2 (RGB LED 1, red) |

No additional jumpers, external devices or power arrangements beyond the
standard DevKit setup are required.

---

## Getting started

Clone with submodules

```bash
git clone --recurse-submodules <repository-url>
```

Pick a build flow

| Flow | Examples | Section |
|---|---|---|
| CMSIS solution (`csolution` / VS Code) | `video_capture`, `video_capture_freertos` | [below](#a-cmsis-solution-build-vs-code--cmsis-toolbox) |
| Zephyr / `west` | `video_capture_zephyr` | [below](#b-zephyr-build) |

---

## A. CMSIS solution build (VS Code / CMSIS-Toolbox)

This is the flow the repository is primarily set up for. It builds the
bare-metal and FreeRTOS examples.

### Prerequisites

- VS Code with the recommended extensions:
  `arm.environment-manager`, `arm.cmsis-csolution`,
  `ms-vscode.cpptools-extension-pack`, `marus25.cortex-debug`.
- The tools installed by the Arm Environment Manager: CMake ≥ 3.28.4,
  Ninja ≥ 1.12.0,`arm-none-eabi-gcc` ≥ 13.2.1-0, CMSIS-Toolbox ≥ 2.6.1.
- CMSIS packs (installed by the task below): `ARM::CMSIS@6.2.0`,
  `ARM::CMSIS-FreeRTOS@11.2.0`, `AlifSemiconductor::Ensemble@2.0.2`.
- For the Security Toolkit programming tasks: the Alif Security Toolkit
  (SETOOLS) installed locally, and a VS Code setting `alif.setools.root`
  pointing at its installation directory. This setting is not stored in the
  repository; add it to your user or workspace settings.

Setting up the Alif VS Code environment itself is documented by Alif in the
[VS Code Getting Started Template](https://github.com/alifsemi/alif_vscode-template);
follow that first.

### Build

1. Open the repository folder in VS Code (**File → Open Folder**).
2. When the **Arm Environment Activation** prompt appears, choose
   **Allow For Current Workspace**.
3. **F1 → Tasks: Run Task → First time pack installation** (only needed once
   per machine; it runs `cpackget` to install the three CMSIS packs listed above).
4. **F1 → CMSIS: Manage Solution Settings** and select
   - a target type: `E7-HP` (Cortex-M55 High Performance core) or `E7-HE`
     (High Efficiency core), and
   - a project: `video_capture` (bare metal) or `video_capture_freertos`.
5. Build with the CMSIS extension's build (hammer) button, or
   **F1 → Tasks: Run Task → cmsis-csolution.build: Build**.

Build types are `release` (`-O speed`) and `debug` (`-O none`).
GCC is the selected compiler.

### Output location

```
out/<project>/<target>/<build-type>/<project>.elf
out/<project>/<target>/<build-type>/<project>.bin
```

for example `out/video_capture/E7-HP/release/video_capture.elf`.

### Program the board

Two paths are pre-configured in [`.vscode/tasks.json`](.vscode/tasks.json):

**Alif Security Toolkit (writes to MRAM)**

**F1 → Tasks: Run Task → Program with Security Toolkit**.
Variants for selecting a COM port and for installing debug stubs are provided
as separate tasks. The MRAM load addresses come from the per-core
configuration files in [`.alif/`](.alif): `0x80200000` for M55_HP and
`0x80000000` for M55_HE.

**pyOCD over CMSIS-DAP**

**F1 → Tasks: Run Task → CMSIS Load** (or `CMSIS Load+Run`). Related tasks:
`CMSIS Run`, `CMSIS Erase`, `CMSIS TargetInfo`.

### Debug

[`.vscode/launch.json`](.vscode/launch.json) provides a J-Link configuration
("Alif Ensemble Debug (Cortex-Debug)", using
[`.alif/JLinkDevices.xml`](.alif/JLinkDevices.xml) and RTT console) and pyOCD
gdb launch/attach configurations for the M55_HP and M55_HE cores.

---

## B. Zephyr build

### Requirements

- [Zephyr SDK 0.16.9](https://github.com/zephyrproject-rtos/sdk-ng/releases/tag/v0.16.9)
- The `west` tool
- Python 3 with the Zephyr dependencies installed

[`video_capture_zephyr/west.yml`](video_capture_zephyr/west.yml) fetches the
Alif Zephyr tree from
[alifsemi/zephyr_alif](https://github.com/alifsemi/zephyr_alif), branch
`v2.1-zas-branch`, which is publicly accessible.

### Install the Zephyr SDK

```bash
cd ~
wget https://github.com/zephyrproject-rtos/sdk-ng/releases/download/v0.16.9/zephyr-sdk-0.16.9_linux-x86_64.tar.xz
tar xvf zephyr-sdk-0.16.9_linux-x86_64.tar.xz
cd zephyr-sdk-0.16.9
./setup.sh
```

### Build

Builds are supported for the two RTSS cores: **rtss_hp** and **rtss_he**.

```bash
python3 -m venv .venv
source ./.venv/bin/activate

pip install -U west
cd video_capture_zephyr/
west init -l .
cd ../
west update
pip install -r zephyr/scripts/requirements.txt

cd video_capture_zephyr/
west build -b alif_e7_dk/ae722f80f55d5xx/rtss_hp
```

`west update` places the Zephyr tree at the repository root (`zephyr/`), which
is where [`video_capture_zephyr/CMakeLists.txt`](video_capture_zephyr/CMakeLists.txt)
expects it. That file also includes TinyUSB's own build support
(`lib/tinyusb/hw/bsp/family_support.cmake`), so the Zephyr application is
linked against TinyUSB rather than Zephyr's USB device stack. The board
overlays in [`video_capture_zephyr/boards/`](video_capture_zephyr/boards)
enable the `alif,xhci` USB controller node.

The resulting image is the usual Zephyr output under `build/zephyr/`.

### Program the board

To program the board using the Alif Security Toolkit in VS Code:

1. Open the compiled binary file (`build/zephyr/zephyr.bin`) in VS Code so it is the active editor tab.

2. Press **F1 → Tasks: Run Task**, then choose the appropriate target core:
- Program opened binary file to M55_HE
- Program opened binary file to M55_HP

---

## Examples in detail

### `video_capture` (bare metal)

A USB Video Class capture device (a webcam) on the Alif Ensemble
USB device controller, driven by TinyUSB with no operating system.
`main()` initialises the board and the TinyUSB device stack, then runs
`tud_task()`, the LED blinker and the video task in a super-loop.

There is no camera sensor. Frames are generated in software: `fill_color_bar()`
writes an EBU colour-bar pattern into a YUY2 frame buffer in RAM, shifted
horizontally by the frame counter, so the host sees the bars scrolling. A new
frame is submitted with `tud_video_n_frame_xfer()` as soon as the previous one
completes.

This example is an end-to-end check of UVC streaming over the
Ensemble device controller — descriptor negotiation, interface alternate
setting selection and sustained isochronous IN traffic — with no sensor,
camera interface or external hardware in the way, and with a frame content
that is easy to judge by eye.

**Run.**

1. Program the board and reset it.
2. Connect the **SoC USB** connector to the host.
3. The host enumerates a standard UVC webcam.
4. Open it in any UVC-capable viewer — see
   [Checking the video stream](#checking-the-video-stream-on-the-host).

**Expected result.**

- The status LED blinks at **250 ms** while the device is not mounted,
  **1000 ms** once the host has enumerated and configured it, and **2500 ms**
  while the bus is suspended.
- The host lists a 128 × 96 YUY2 camera at 10 fps.
- The preview shows vertical EBU colour bars (white, yellow, cyan, green,
  magenta, red, blue, black) scrolling horizontally.
- No trace output is printed by default: TinyUSB logging is off
  (`CFG_TUSB_DEBUG 0`). Raising `CFG_TUSB_DEBUG` in `tusb_config.h` sends
  TinyUSB's log to the PRG USB serial port at 115200 8-N-1.

---

### `video_capture_freertos`

The same UVC device as `video_capture`, with the work split
across FreeRTOS tasks: `usbd` runs the TinyUSB device stack, `video`
produces frames, and `blinky` drives the LED. `usbd` and `video` both run at
`configMAX_PRIORITIES - 1`, `blinky` at priority 1; the `video` task calls
`video_send_frame()` in a loop with a one-tick delay.

This example shows the RTOS integration points for a class with
sustained isochronous traffic: where the device stack must be initialised
relative to the scheduler, and how a frame producer task interacts with the
USB task.

**Target hardware, connections, USB identity, video format, expected LED
behaviour and expected host behaviour.** Same as
[`video_capture`](#video_capture-bare-metal).

---

### `video_capture_zephyr`

The same UVC device on Zephyr, still using TinyUSB as the USB stack
rather than Zephyr's own. `main()` creates a `usbd` thread at
`K_HIGHEST_APPLICATION_THREAD_PRIO - 1` and a `video` thread one priority
below it, plus a `k_timer` that drives the LED. The `video` thread calls
`video_send_frame()` in a loop with `k_msleep(1)`.

This examples shows how to bring the Alif TinyUSB port into a Zephyr
application: the device tree overlay for the Ensemble USB controller, the
board/BSP wiring, and the CMake glue that compiles TinyUSB inside a Zephyr
build.

**Run and expected result.**
Same USB behaviour, video format and LED patterns as the
other two examples. Console output depends on the Zephyr board
configuration and is not produced by the example itself.

---

## Checking the video stream on the host

The device is a standard UVC webcam, so no special host software ships with
this repository — any UVC-capable viewer will do.

On Linux the camera appears as a `/dev/video*` node:

```bash
v4l2-ctl --list-devices
v4l2-ctl -d /dev/video0 --list-formats-ext
ffplay -f v4l2 -input_format yuyv422 -video_size 128x96 -framerate 10 /dev/video0
```

VLC, Cheese and guvcview work as well. On Windows, the Camera app or any
application with a webcam picker; on macOS, QuickTime Player
(**File → New Movie Recording**) or Photo Booth.

Expect vertical EBU colour bars scrolling horizontally at 128 × 96. The image
is small by design — the point of the example is the USB path, not the
resolution.

## Licensing

This repository contains software from multiple sources and under multiple
applicable licences. **No single licence covers the whole repository.**

- **TinyUSB source and TinyUSB-derived source** (the example sources under
  `video_capture*/src/`, `video_capture_zephyr/CMakeLists.txt`, and the
  `lib/tinyusb` submodule except where noted): MIT Licence — see
  [`LICENSE-MIT.txt`](LICENSE-MIT.txt) and the notices in the individual files.
  The copyright notices name Ha Thach (tinyusb.org) and Jerzy Kasenbreg.
- **Alif Semiconductor material** (the Ensemble device controller driver and
  Alif BSP inside `lib/tinyusb`, the `lib/alif_common-app-utils` submodule,
  `video_capture_freertos/dcd_ensemble_isoc_buffers.h`, the CMSIS pack
  configuration under `video_capture*/RTE/`, and the project scaffolding
  derived from the Alif VS Code template): the Alif Semiconductor Software
  License Agreement — see [`LICENSE-ALIF.txt`](LICENSE-ALIF.txt) and the
  notices in the individual files.
- **Other third-party components** (Arm CMSIS under Apache-2.0, the FreeRTOS
  kernel configuration under MIT, Zephyr under Apache-2.0): see
  [`THIRD_PARTY_NOTICES.txt`](THIRD_PARTY_NOTICES.txt) and the components' own
  licence files.

Modifications developed for the Alif Semiconductor platform are marked as such
in the affected files; they do not replace the upstream copyright or licence
notices in those files, which continue to apply to the original material.

---

## Support and issues

This repository is a **published snapshot**. It is maintained on a best-effort
basis by Sirin Software and is not covered by an Alif Semiconductor
customer-support commitment.

- Questions or problems with the content of **this repository** can be raised
  through this repository's issue tracker on GitHub. Opening an issue here does
  not create any support obligation for Alif Semiconductor.
- Questions about **Alif Semiconductor devices, SDKs, packs or tools** belong in
  Alif's own support channels; see [alifsemi.com](https://www.alifsemi.com).
- Questions about **TinyUSB itself** belong upstream at
  [hathach/tinyusb](https://github.com/hathach/tinyusb).

---

## References

- [Alif Ensemble E7 DevKit documentation](https://alifsemi.com/support/kits/ensemble-e7devkit/)
- [Alif VS Code Getting Started Template](https://github.com/alifsemi/alif_vscode-template)
- [TinyUSB documentation](https://docs.tinyusb.org)
- [Zephyr documentation](https://docs.zephyrproject.org)
- [CMSIS-Toolbox](https://github.com/Open-CMSIS-Pack/cmsis-toolbox)
