# Using panfrost GPU driver with BE-M1000 SoC

## Kernel

[5.4.y with Baikal-M and Mali T6xx support](https://github.com/asheplyakov/linux/tree/mali-t6xx-baikalm-5.4.y)

use `baikal_minimal_defconfig`

## Mesa

[master with Mali T6xx support](https://github.com/asheplyakov/mesa/tree/panfrost-mali-t628)

Unfortunately it has to be compiled natively (due to LLVM libs being incompatible
with multiarch)

### Install dependencies

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

### Building Mesa


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

## Xorg

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
