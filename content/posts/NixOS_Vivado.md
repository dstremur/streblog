---
title: "Running Vivado 2025.2 on NixOS"
date: 2026-03-13T22:04:36+01:00
draft: false
tags: ["NixOS"]
categories: ["Guides"]
---

## TLDR

I should preface this by saying that Vivado probably has most painful installation process of any program I know. It is as if someone had implemented [Wirth's law](https://en.wikipedia.org/wiki/Wirth%27s_law).
This post is based on Oliver Kovacs great [article](https://oliverkovacs.dev/blog/2025/05/02/installing-vivado-on-nixos.html) about his setup for Vivado 2019.2 on NixOS. My changes are based around the new version of Vivadoa and getting the USB connection to an FPGA board to work.

## Downloading Vivado

The first step is to download Vivado from the official [website](https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/vivado-design-tools.html). Note that you need to download the offline installer for this to work.
To be able to download you will also need to create an AMD account.
Then you also need to fill out the export restriction form and handover all your
personal information to the US government. 

When your download is finished, move it to a directory where you would like
to install Vivado and create `shell.nix` file with the following content in it.[^1] 
```nix 
{ pkgs ? import <nixpkgs> { } }:
(pkgs.buildFHSEnv {
  name = "vivado-env";
  targetPkgs = pkgs: (
 with pkgs; [
   ncurses5
   ncurses
   libxcrypt-legacy
   libpng
   libusb1
   systemd
   pixman
   zlib libuuid
   bash coreutils zlib stdenv.cc.cc
   xorg.libXext xorg.libX11 xorg.libXrender xorg.libXtst
   xorg.libXi xorg.libXft xorg.libxcb xorg.libxcb
   freetype fontconfig glib gtk2 gtk3
   graphviz gcc unzip nettools
 ]);

  profile = ''
    export LD_LIBRARY_PATH=/usr/lib:/usr/lib64:$LD_LIBRARY_PATH
  '';

  runScript = ''
   env LIBRARY_PATH=/usr/lib \
   C_INCLUDE_PATH=/usr/include \
   CPLUS_INCLUDE_PATH=/usr/include \
   CMAKE_LIBRARY_PATH=/usr/lib \
   CMAKE_INCLUDE_PATH=/usr/include \
   bash
  '';
}).env 
```
Since Vivado expects a typical Linux distro, the shell needs to create a FHS 
environment with all the necessary dependencies. 

## Installing Vivado
Now you can enter the shell by typing `nix-shell` into the terminal. To start the setup, 
type the following in to the terminal. 
```bash
./FPGAs_AdaptiveSoCs_Unified_SDI_2025.2_****_****/xsetup
```
There you can customize your install, choose for which FPGA chips you want support and set the default install directory.
Then the install process will begin, and you'll need to wait. Really wait. 
It might seem that it does nothing or got stuck,
but believe me, if you close the installer the whole install will be erased, and you need to start over.

## Running Vivado

Now you should be able to run Vivado with the following command. 
```bash
./YOUR_INSTALL_DIR/2025.2/Vivado/bin/vivado
```
Vivado should launch and work as expected.

## Connecting the FPGA

At last, we need to connect the FPGA to be able to program it. This is mostly done 
via USB in an educational or testing environment. Note that the following is only tested 
on a Framework 13 and a Basys 3 board.

When you try to connect to a device, you will realise that none can be found over USB.
This is due to the need of certain [udev](https://en.wikipedia.org/wiki/Udev) rules being needed. Add the following to `configuration.nix` file.
```nix 
  services.udev.extraRules = ''
    # Digilent JTAG cables 
    ATTRS{idVendor}=="0403", ATTRS{idProduct}=="6010", MODE="666", GROUP="dialout"
    # Xilinx Platform Cable USB II
    ATTRS{idVendor}=="03fd", ATTRS{idProduct}=="0008", MODE="666", GROUP="dialout"
    ATTRS{idVendor}=="03fd", ATTRS{idProduct}=="0013", MODE="666", GROUP="dialout"
  '';
```
To apply the udev rules, you can reboot or use the following commands. 
```
sudo udevadm control --reload-rules
sudo udevadm trigger
```
Now the auto connect feature in Vivado will find your board and you can start to program the FPGA.  

[^1]: Adapted from: https://oliverkovacs.dev/blog/2025/05/02/installing-vivado-on-nixos.html
