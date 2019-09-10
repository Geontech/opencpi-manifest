# Xilinx ZCU102/ZCU111 OpenCPI Manifest

This is a Google `repo` manifest that enables installing OpenCPI onto a Xilinx ZCU102 MPSoC (a.k.a., `zcu102-zynqmp`) or ZCU111 RFSoC (a.k.a., `zcu111-zynqmp`).  The following sections will detail how to download the manifest, setup the build environment, build it, and then run it against an existing naming service.

 > NOTE: This build relies on `meta-xilinx-tools`, which requires your host operating system have `Xvfb` and Vivado installed with a license capable of using the ZCU102/ZCU111.

This manifest repository is intended to be at the same hierarchical level as the following repositories:

 * meta-poky
 * meta-openembedded
 * meta-xilinx
 * meta-xilinx-tools
 * meta-opencpi

This repository is expected to be named `build-manifest`.

**Target Release Expectations:**

| ITEM | TYPE | VERSION |
| ----- | ----- | ----- |
| Xilinx Vivado | host | 2018.3 |
| meta-xilinx | branch | rel-v2018.3 |
| meta-xilinx-tools | branch | rel-v2018.3 |
| meta-poky | tag | yocto-2.4.3` |
| meta-openembedded | branch | rocko |
| OpenCPI | branch | release_1.4_zynq_ultra |
| meta-opencpi | branch | zcu1xx |

 > NOTE: We are assuming that Vivado is installed at its standard location: ` /opt/Xilinx/Vivado/<version>`.  If this is not the case, modify the `local.conf` variable `XILINXD_SDK_TOOLCHAIN` to point to the right location.

The manifest file narrows the selection further to a particular commit.  Please see `default.xml`.  If the source revisions are not available in your fork, you do not have the correct fork.  Please contact your vendor as additional patches have been applied.

The only tested `MACHINE` value is `zcu102-zynqmp`.

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
cd <project>
repo init -u <url to the...>/opencpi-manifest.git
repo sync
```

## Setup
Commands in this section are run from your project directory where you ran the `repo sync`.

Assuming the OpenCPI RPMs are installed (including those for the ZCU102 and/or ZCU111), or the OpenCPI framework is installed in some way for use with these boards, there should be a directory with SD card contents in OpenCPI's CDK. Those contents must be copied to the `meta-opencpi` layer at this point.

* For the ZCU102 with OpenCPI `release_1.4_zynq_ultra` RPMs installed:
```
cp -rf /opt/opencpi/cdk/zcu102/*/sdcard-xilinx18_2/opencpi poky/meta-opencpi/recipes-core/opencpi/opencpi-runtime/zcu102-zynqmp/;
```
* For the ZCU102 with OpenCPI `release_1.4_zynq_ultra` installed from source:
```
cp -rf $OCPI_CDK_DIR/zcu102/zcu102-deploy/sdcard-xilinx18_2/opencpi poky/meta-opencpi/recipes-core/opencpi/opencpi-runtime/zcu102-zynqmp/;
```

* For the ZCU111 with OpenCPI `release_1.4_zynq_ultra` RPMs installed:
```
cp -rf /opt/opencpi/cdk/zcu111/*/sdcard-xilinx18_2/opencpi poky/meta-opencpi/recipes-core/opencpi/opencpi-runtime/zcu111-zynqmp/;
```
* For the ZCU111 with OpenCPI `release_1.4_zynq_ultra` installed from source:
```
cp -rf $OCPI_CDK_DIR/zcu111/zcu111-deploy/sdcard-xilinx18_2/opencpi poky/meta-opencpi/recipes-core/opencpi/opencpi-runtime/zcu111-zynqmp/;
```

Now to set up your environment for building:

```
source poky/oe-init-build-env
```

You should now be in the `build` directory.  When you view `conf/local.conf` and `conf/bblayers.conf`, you should see:

 1. (local.conf) The `MACHINE` is weak-set to `zcu102-zynqmp`
 2. (local.conf) The `XILINX_SDK_TOOLCHAIN` is set to the expected default path to Vivado
 3. (bblayers.conf) The layers mentioned at the top of this file are listed in the same order (with sub-layers, as in the case of `meta-openembedded` and `meta-xilinx`).

## Build

You should have run the `source ...` step to enter the `build` directory.  You're now ready to build the OpenCPI Runtime image:

```
bitbake opencpi-runtime-image
```
Note: The default `MACHINE` is set to `zcu102-zynqmp` in build/conf/layers.conf,


When the process completes, you will have a file such as: `tmp/deploy/images/MACHINE/IMAGE.wic`.

## Write to SD Card
You can then write that image using `dd` to your SD card.

* This can be done for the ZCU102 as follows:
```
sudo dd if=tmp/deploy/images/zcu102-zynqmp/redhawk-gpp-image-zcu102-zynqmp.wic of=<path to SD card> bs=1M && sync
```

* or for the ZCU111:
```
sudo dd if=tmp/deploy/images/zcu111-zynqmp/redhawk-gpp-image-zcu111-zynqmp.wic of=<path to SD card> bs=1M && sync
```
