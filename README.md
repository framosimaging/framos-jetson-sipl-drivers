# FRAMOS IMX900 Camera-over-Ethernet driver for Jetson SIPL

This repository adds the FRAMOS **IMX900** camera module driver to NVIDIA's Jetson SIPL API
package. The IMX900 sends its image data over Ethernet (Camera-over-Ethernet, CoE). A
Holoscan Sensor Bridge connects the sensor to the Jetson's **QSFP port**, not to the
RJ45 Ethernet port. One build supports both the colour and the mono sensor. The driver
detects which one is connected at runtime.

| JetPack / L4T  | Framos branch | Platform |
|----------------|---------------|----------|
| 7.2.1 / 39.2.1 | fr-l4t-39.2.1 | AGX Thor |

The repository contains:

- `framos-imx900-sipl-l4t39.2.1.patch`: the driver, as a patch for NVIDIA's SIPL sources
- `nito-files/`: ISP tuning files, `IMX900.nito` (colour) and `mono/IMX900.nito`
- `LICENSE`, `LICENSE.NVIDIA`: license texts, see [License](#license)
- `README.md`: this file

## 1. Equipment

- [NVIDIA Jetson AGX Thor Developer Kit](https://www.nvidia.com/en-us/autonomous-machines/embedded-systems/jetson-thor/)
- [Lattice CPNX100-ETH-SENSOR-BRIDGE](https://www.latticesemi.com/products/designsoftwareandip/intellectualproperty/referencedesigns/referencedesigns05/lattice-nvidia-edge-ai)
  Setup: NVIDIA [hardware setup](https://docs.nvidia.com/holoscan/sensor-bridge/getting-started/hardware-setup).
- [FRAMOS IMX900 sensor module](https://docs.framos.com/en/latest/FSMEcosystem/ProductDocumentation/FSMGO/IMX900.html) (colour or mono), on camera connector J5 or J6
- QSFP cables for options please read [Nvidia documentation](https://docs.nvidia.com/holoscan/sensor-bridge/2-7-latest/support/notes#network-cables-and-adapters). We have tested
	- Copper [SFP+ to SFP+ cable](https://www.amazon.com/10G-SFP-DAC-Cable-SFP-H10GB-CU2M/dp/B00U8BL09Q/ref=sr_1_4?dib=eyJ2IjoiMSJ9.Cf-3YlRVvPfOvuT9WkBirl136H7mBcMmsk3GZo6CrIgg6twUeFibkg2B33myyuT9gB0QLyjJtTm3HKhnEhweaz73ZteuRh32EQoRms2iNgX8I3HM6_CTTqjm7Pt6x1HMSCNBpbtGP2UjMWH1_LROIHSpFF3SEf53-aG4o0kkVvDVVWeTVvr-bQHiGkMqKCv9EDZCMso3MU8BX9zT_-sZOHfCBMpOPHcU_-uPGAdl47o.SFlNj6GVEC-arkVjg8PX91PbzSpRKD5cWOWvg3hEyPI&dib_tag=se&keywords=sfp%2B%2Bto%2Bsfp%2B%2Bcable&qid=1708561658&sr=8-4&th=1) with [SFP+ to QSFP adapter](https://www.amazon.com/Converter-CVR-QSFP-SFP10G-Mellanox-MAM1Q00A-QSA-All-Metal/dp/B082V1TLHH?th=1)
	- [QSFP to 4x SFP+ 10 Gtek cable](https://www.amazon.de/-/en/10Gtek-QSFP-4SFP10G-CU1M-Direct-Breakout-Passive/dp/B0C36SJ8QN/ref=sr_1_3?crid=10FYQM0IEONAE&dib=eyJ2IjoiMSJ9.jNPbyy6wpZnT6LSHnVb-YdzfB2soiulPuHVX5UOiWQ-MY7NwlboegT7oNfKf9eOYLje1mBNfFyMA1A5F71_3yiEr3ROHro_zKN82pP5pbV3iItKcToGG6SS5trAOu6GLo5tdbB170fn6UMdse7XXIw77kypbdVG1NIBE161E0T5RRA3AINn0Va-XfB_Uu-VBFgu7PMAfXF0pFjUkN3ERwAorVik5ueyHW7KZQtB7hB4.OdwYWHWAVIJjFK54reDchvlDy76ERxEw5HSet97SHsA&dib_tag=se&keywords=40G%2BQSFP%2B%2Bto%2B4%2Bx%2B10G%2BSFP%2B%2BEthernet%2BPassive%2BDirect%2BAttach%2BCopper%2BBreakout%2BCable&qid=1772463868&sprefix=40g%2Bqsfp%2B%2Bto%2B4%2Bx%2B10g%2Bsfp%2B%2Bethernet%2Bpassive%2Bdirect%2Battach%2Bcopper%2Bbreakout%2Bcable%2Caps%2C126&sr=8-3&th=1)



## 2. Installation

1. Flash JetPack 7.2.1 on the AGX Thor. The simplest route is the ISO installer on a USB
   stick, no host PC needed, as described in the
   [AGX Thor Developer Kit Quick Start Guide](https://docs.nvidia.com/jetson/agx-thor-devkit/user-guide/latest/quick_start.html).
   The [JetPack download page](https://developer.nvidia.com/embedded/jetpack/downloads/)
   has the ISO and the alternatives: [SDK Manager](https://developer.nvidia.com/sdk-manager) or
   [manual flashing](https://docs.nvidia.com/jetson/archives/r39.2.1/DeveloperGuide/SD/FlashingSupport.html)
   from an Ubuntu host.

2. To configure network port follow the instructions from Nvidia for
   [Holoscan Sensor Bridge host setup](https://docs.nvidia.com/holoscan/sensor-bridge/getting-started/host-setup).
   The configs expect the bridge at `192.168.0.2`, which means a cable in SFP+ port 0 of
   the bridge, connected to the Thor's QSFP port (interface `mgbe0_0`):

3. If you need to update firmware version of the Lattice board you will need to [build the Holoscan Sensor Bridge container](https://docs.nvidia.com/holoscan/sensor-bridge/getting-started/build) and [flash the board](https://docs.nvidia.com/holoscan/sensor-bridge/firmware/firmware-setup)

4. Install [SIPL and dependencies](https://docs.nvidia.com/jetson/archives/r39.2.1/DeveloperGuide/SD/CameraDevelopment/SIPLFramework/SIPL-for-L4T/SIPL-App-Development-Guide.html#build-prerequisites)
Install the build dependencies. For other ways to install the CUDA headers, see
   [sipl/README.md](sipl/README.md). Run one test build to confirm everything works as expected.


5. Clone this repository (branch `fr-l4t-39.2.1`) to your home folder on the Jetson,
   apply the FRAMOS patch to the installed SIPL sources, then build and install:

   ```bash
   sudo apt-get install -y git
   cd ~
   git clone -b fr-l4t-39.2.1 https://github.com/framosimaging/framos-jetson-sipl-drivers.git
   cd /usr/src/jetson_sipl_api
   sudo git apply --check ~/framos-jetson-sipl-drivers/framos-imx900-sipl-l4t39.2.1.patch
   sudo git apply ~/framos-jetson-sipl-drivers/framos-imx900-sipl-l4t39.2.1.patch
   cd sipl
   sudo bash build.sh
   ```

   `git apply --check` changes nothing; it only confirms that the patch matches the
   installed SIPL version. The build installs the drivers to `/usr/lib/nvsipl_drv/` and
   `nvsipl_camera` / `nvsipl_query` to `/usr/sbin/`.

6. Install the ISP tuning files from `nito-files/`. The mono tuning has its own folder
   because it must also be named `IMX900.nito`:

   ```bash
   cd ~/framos-jetson-sipl-drivers
   sudo cp nito-files/IMX900.nito /var/nvidia/nvcam/settings/sipl/
   sudo mkdir -p /var/nvidia/nvcam/settings/sipl/mono
   sudo cp nito-files/mono/IMX900.nito /var/nvidia/nvcam/settings/sipl/mono/
   ```

## 3. Streaming

To list the available configs, run `sudo nvsipl_query -l`.

| Config               | Resolution | FPS   | Setup                           |
|----------------------|------------|-------|---------------------------------|
| `IMX900`             | 2048x1552  | 72.1  | one camera on J5, colour or mono |
| `IMX900_BIN2X2_MONO` | 1024x720   | 160.1 | one camera on J5, mono only     |


### nvsipl_camera

```bash
sudo nvsipl_camera -c IMX900 -H -R -s -r 15                  # stream 15 s, print FPS every 2 s
sudo nvsipl_camera -c IMX900 -H -s -r 15 --egl-display       # on-screen preview
sudo nvsipl_camera -c IMX900 -H -s -r 15 \
     -N /var/nvidia/nvcam/settings/sipl/mono/                # mono sensor via ISP
```

Without `-N`, the ISP uses the colour tuning file, and a mono sensor's output looks magenta.
For more info on nvsipl_camera application check -h flag and [nvsipl_camera documentation](https://docs.nvidia.com/jetson/archives/r39.2/DeveloperGuide/SD/CameraDevelopment/SIPLFramework/SIPL-for-L4T/SIPL-App-Development-Guide.html#nvsipl-camera)

For GStreamer (`nvsiplsrc`) and Holoviz, see the
[Streaming](https://github.com/framosimaging/framos-coe-sipl-drivers/wiki/Streaming) page
in the wiki.

## Further reading

- [NVIDIA SIPL framework (r39.2.1)](https://docs.nvidia.com/jetson/archives/r39.2.1/DeveloperGuide/SD/CameraDevelopment/SIPLFramework/Introduction-to-SIPL.html)
- [Home.md](https://docs.nvidia.com/jetson/archives/r39.2.1/DeveloperGuide/SD/CameraDevelopment/SIPLFramework/Introduction-to-SIPL.html)

## License

- FRAMOS files whose header carries `SPDX-License-Identifier: Apache-2.0` are licensed
  under the Apache License 2.0 - see `LICENSE` in this repository.
- NVIDIA files, including the ones FRAMOS modified, remain under NVIDIA's Jetson Linux
  license (NVIDIA Driver License Agreement) - see `LICENSE.NVIDIA` in this repository.
  The NVIDIA SIPL package itself is not included; install it from NVIDIA first.
