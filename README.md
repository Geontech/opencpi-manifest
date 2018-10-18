# Xilinx ZCU102 REDHAWK Manifest

This is a Google `repo` manifest that enables installing REDHAWK SDR onto a Xilinx ZCU102 MPSoC (a.k.a., `zcu102-zynqmp`).  The following sections will detail how to download the manifest, setup the build environment, build it, and then run it against an existing naming service.

 > NOTE: This build relies on `meta-xilinx-tools`, which requires your host operating system have `Xvfb` and Vivado installed with a license capable of using the ZCU102.

This manifest repository is intended to be at the same hierarchical level as the following repositories:

 * meta-poky
 * meta-openembedded
 * meta-xilinx
 * meta-xilinx-bsp
 * meta-redhawk-sdr

This repository is expected to be named `build-manifest`.

**Target Release Expectations:**

| ITEM | TYPE | VERSION |
| ----- | ----- | ----- |
| Xilinx Vivado | host | 2018.2 |
| meta-xilinx | branch | rel-v2018.2 |
| meta-xilinx-bsp | rel-v2018.2 |
| meta-poky | tag | yocto-2.4.3` |
| meta-openembedded | branch | rocko |
| meta-redhawk-sdr | branch | master |

 > NOTE: We are assuming that Vivado is installed at its standard location: ` /opt/Xilinx/Vivado/<version>`.  If this is not the case, modify the `local.conf` variable `XILINXD_SDK_TOOLCHAIN` to point to the right location.

The manifest file narrows the selection further to a particular commit.  Please see `default.xml`.  If the source revisions are not available in your fork, you do not have the correct fork.  Please contact your vendor as additional patches have been applied.

The only tested `MACHINE` is `zcu102-zynqmp`.

## Downloading

If you do not have `repo` installed, do so now:

```
curl http://commondatastorage.googleapis.com/git-repo-downloads/repo > repo
chmod a+x repo
sudo mv repo /usr/local/bin
```

Then, clone the manifest:

```
mkdir <project>
repo init <url to the...>/project-manifest.git
repo sync
```

## Setup

From your project directory where you ran the `repo sync`:

```
source poky/oe-init-build-env
```

You should now be in the `build` directory.  When you view `conf/local.conf` and `conf/bblayers.conf`, you should see:

 1. (local.conf) The `MACHINE` is weak-set to `zcu102-zynqmp`
 2. (local.conf) The `XILINX_SDK_TOOLCHAIN` is set to the expected default path to Vivado
 3. (bblayers.conf) The layers mentioned at the top of this file are listed in the same order (with sub-layers, as in the case of `meta-openembedded` and `meta-xilinx`).

## Build

You should have run the `source ...` step to enter the `build` directory.  You're now ready to build the REDHAWK images, such as:

```
bitbake redhawk-gpp-image
```

When the process completes, you will have a file such as: `tmp/deploy/images/MACHINE/IMAGE.wic`.  You can then write that image using `dd` to your SD card:

```
sudo dd if=tmp/deploy/images/zcu102-zynqmp/redhawk-gpp-image-zcu102-zynqmp.wic of=<path to SD card> bs=1M && sync
```

## Run

 > NOTE: If you chose an image definition that installed the Domain, the following instructions will prove your device can join a naming and event service as its own Domain.

This assumes you have an OmniORB Naming and Event service running on the network to which the device is connected.  If this is not the case, consider one of the other image definitions or mix-and-match to get a device that can run as its own domain, naming, and event service.

First, update the `/etc/omniORB.cfg` to point at your naming and event services IP.

Second, run `nodeBooter -D ...`, where the `...` are additional options to deconflict the new Domain's name with what is available on the naming service.  If successful, remote REDHAWK systems should see your domain name either in the REDHAWK IDE or, for example, the python shell:

```
from ossie.utils import redhawk
dom = redhawk.attach('YOUR_DOMAIN_NAME')
```
