# ONIE Adventures

This repository contains some resources about my adventures with ONIE. I had to build my own ONIE for Celestica DX010, because Celestica does not provide finished images and there are no real walkthrough guides.

Below you can find a guide on how to build ONIE on your own for Celestica DX010.

Because I wasn't able to find any public ONIE build pipeline / CI (if one exists, please let me know where?), I have includeed the binary results of the ONIE builds for a handful of platforms I use / own (see list below)


## Successful builds
```
- ONIE 2021.08, cel_seastone-r0, Debian 9
- ONIE 2021.08, quanta_common_rglbmc, Debian 9

- ONIE 2024.02, accton_as7716_32x, Debian 11
- ONIE 2024.02, accton_as7716_32xb, Debian 11
- ONIE 2024.02, accton_as7726_32x, Debian 11
- ONIE 2024.02, accton_as7712_32x, Debian 11
```

## Walkthrough for Celestica DX010

PSA: Versions newer than 2021.08 will not build.
Starting at 2021.11, builds will fail with
```
make: *** No rule to make target 'conf/crosstool/gcc-4.9.2/uClibc-ng-1.0.38/crosstool.x86_64.config', needed by '/home/robin/onie-build/build/x-tools/x86_64-g4.9.2-lnx3.2.69-uClibc-ng-1.0.38/build/.config'.  Stop.
```


Download DUE (do not use the one from debian repos, it's v3.0 as of time of writing and too old)
Create buildenv with DUE

```
mkdir onie
cd onie

git clone https://github.com/CumulusNetworks/DUE

cd DUE

./due --create --platform linux/amd64 --name onie-build-debian-9 --prompt ONIE-9 --tag onie-9 --use-template onie --from debian:9 --description 'ONIE Build Debian 9' --image-patch debian/9/filesystem

cd ..
```

Populate the cache.. it will make things much more bearable, especially if you have to go back and rebuild or want to build for a different switch.
```
sudo mkdir -p /var/cache/onie/download
cd /var/cache/onie/download
sudo wget --recursive --cut-dirs=2 --no-host-directories --no-parent --reject="index.html" "http://mirror.opencompute.org/onie"
cd <your top-level onie directory>
```
it will take about 10 minutes... Total size of downloads is about 3.9GB and the opencompute mirror seems to rate limit you after some time?

Time to get started...
```
https://github.com/opencomputeproject/onie.git onie-build
cd onie-build
git checkout 2021.08
cd ../DUE

./due --run-image due-onie-build-debian-9 --home-dir <your top-level onie directory> --mount-dir /var/cache/onie/download/:/var/cache/onie/download/

# Now from container shell:

# For some reason, the DUE default git config does not work... This is taken from the DUE template
git config --global user.name "ONIE build account"
git config --global user.email "oniebuild@localhost"

cd onie-build/build-config/

ONIE_USE_SYSTEM_DOWNLOAD_CACHE=TRUE make -j128 MACHINEROOT=../machine/celestica MACHINE=cel_seastone all demo
```
Adjust the `-j128` to your amount of cores.
This will take a while. About 15 minutes on my 128-Core 2x EPYC 7773X machine.

You will find the resulting file in `<your top-level onie directory>/onie-build/build/images/`:
```
cel_seastone-r0.initrd
cel_seastone-r0.vmlinuz
demo-diag-installer-x86_64-cel_seastone-r0.bin
demo-installer-x86_64-cel_seastone-r0.bin
onie-recovery-x86_64-cel_seastone-r0.iso
onie-updater-x86_64-cel_seastone-r0
```


