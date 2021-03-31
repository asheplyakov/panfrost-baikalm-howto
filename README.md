# Using panfrost GPU driver with BE-M1000 SoC

The instructions are valid for ALT Linux distro

## TL;DR version

```
sudo apt-get install kernel-image-un-def kernel-modules-drm-un-def xorg-dri-armsoc
cat <<-EOF | sudo tee /etc/modprobe.d/panfrost_enable.conf
options panfrost enable_broken_machines=y
EOF
cat <<-EOF | sudo tee /etc/X11/xorg.conf.d/panfrost.conf
Section "ServerFlags"
	Option	"AutoAddGPU" "off"
EndSection

Section "OutputClass"
	Identifier "BaikalM"
	MatchDriver "baikal_vdu_drm"
	Driver "modesetting"
	Option "PrimaryGPU" "true"
EndSection
EOF
```

## Brief explanation

### Prerequisites:

* Kernel 5.10.y with BE-M1000 and Mali T6xx patches: kernel-image-un-def version 5.10.y where y >= 18
* Mesa 21.0.x with BE-M1000 and Mali T6xx patches: xorg-dri-armsoc version 21.0.0 and newer

### Xorg configuration

Xorg needs the following config to start successfully


```
Section "ServerFlags"
	Option	"AutoAddGPU" "off"
EndSection

Section "OutputClass"
	Identifier "BaikalM"
	MatchDriver "baikal_vdu_drm"
	Driver "modesetting"
	Option "PrimaryGPU" "true"
EndSection
```

## Benchmarking notes

Use [glmark2](https://github.com/glmark2/glmark2) version 2021.02 to avoid artefacts.

```
sudo apt-repo add 268669
sudo apt-get update
sudo apt-get install glmark-es2 glmark-es2-wayland
glmark-es2 --fullscreen --annotate --run-forever
```

## Building from the source

### Kernel

[5.10.y with Baikal-M and Mali T6xx support](https://github.com/asheplyakov/linux/tree/baikalm-5.10.y)

use `baikal_minimal_defconfig`

### Mesa

[Mesa 21.0 branch](https://github.com/mesa3d/mesa/tree/21.0) with these patches

* [panfrost: mark T6xx as supported](http://git.altlinux.org/gears/M/Mesa.git?p=Mesa.git;a=patch;h=046df28d274ca1632b17febfed1d185dc61eee8b)
* [kmsro: added entry points for baikal_vdu (BE-M1000 SoC)](http://git.altlinux.org/gears/M/Mesa.git?p=Mesa.git;a=patch;h=aa7f229d6574d7d4358b95aee8fda3d68446195c)

Unfortunately it has to be compiled natively (due to LLVM libs being incompatible
with multiarch)

#### Install dependencies

The following packages should be installed on the board

```
   gcc
   gcc-c++
   bison
   ccache
   cmake
   flex
   meson
   ninja-build
   pkg-config
   python3-module-mako
   libdrm-devel
   libexpat-devel
   libxcb-devel
   libXext-devel
   libXfixes-devel
   libxshmfence-devel
   libXrandr-devel
   libXxf86vm-devel
   libudev-devel
   llvm10.0-devel
   libunwind-devel
   libwayland-client-devel
   libwayland-egl-devel
   libwayland-server-devel
   xorg-proto-devel
   wayland-devel
   wayland-protocols
   zlib-devel
```

#### Building Mesa


```bash
git clone --depth=1 -b panfrost-mali-t628 git://github.com/asheplyakov/mesa.git
cd mesa
mkdir _build
cd _build
meson .. . -Ddri-drivers= -Dvulkan-drivers= -Dgallium-drivers=panfrost,kmsro
ninja
sudo ninja install
```

After the installation adjust `/etc/ld.so.conf`, so the first line
in this file is

```
/usr/local/lib64
```

and run

```bash
sudo ldconfig
```
