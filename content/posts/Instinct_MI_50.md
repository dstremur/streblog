---
title: "All the FP64 without CUDA - AMD Radeon Instinct MI50 Review"
date: 2025-06-25T21:53:35+02:00
draft: false
---
![Radeon Instinct MI50!](/images/InstinctMI50/InstinctMI501.jpg)

## Teardown 

Tearing down the card was pretty straightforward. The outer shroud is held on by 6 small screws. When removed, the shroud can be pulled off the card by slightly wiggling it from side to side. 
Underneath, the card consists of a long black heat sink spanning the whole length of the card. The GPU die is cooled by a copper vapor chamber, easily recognized by its silver fins. 

![Radeon Instinct MI50 without shroud!](/images/InstinctMI50/MI50_noshroud.jpg)

The vapor chamber can be pulled off the die by removing the 4 screws holding it down onto the die. They are the screws which are screw into the silver springy thing on the backside. 

![Radeon Instinct MI50 vapor chamber!](/images/InstinctMI50/MI50_Cooler.jpg)

One particular thing about this card is AMD's choice of TIM (Thermal Interface Material) between the cooler and GPU die. AMD uses a graphite pad, 
which is almost impossible to remove in one piece. It is sometimes strongly advised to not use regular thermal paste and instead just reuse the old pad. The pad is supposed to 
be better at filling in the microimperfections in the cooler and the die. I have repasted most of my MI50's with Noctua NT-H2 thermal paste and have not encountered any issues 
so far.

To remove the black heatsink, the backplate has to be taken off first, since it covers some hidden screws that are securing the aformentioned heat sink. 
After all of them have been removed, the heatsink can be pried off the PCB. Note that the thermal pads over the VRMs should be removed with, such that they 
can be reused. If you need to replace them, I have found that 2 mm thick thermal pads work pretty well. 


## Driver/ROCM Install
The MI50s are currently running inside the GIGABYTE G292-Z00 server. It's a great server that has ample cooling capacity for these
datacenter GPUs. However, it is operating at jet engine noise levels. 
For the operating system, Ubuntu Server 24.04.02 LTS was chosen based on two main points: 

1. Ubuntu is a ROCm supported OS, something which is very important when you're playing with hardware that is not 100% supported. This does not
just apply to the MI50's, but also to Chelsio NICs (blog post coming soon ...), the GIGABYTE server, etc.

2. Ubuntu is the standard OS when it comes to AI and Deep Learning frameworks and applications. 

I can happily report, that the Ubuntu installation has been flawless so far and has caused very little headaches.
The same cannot be said about the ROCm/Driver install, which took about 4 attempts to get running smoothly.

At the time of writing, the latest ROCm version is 6.4.3. For this project I chose to install version 6.3.0, since AMD decided to [drop support](https://rocm.docs.amd.com/projects/install-on-linux/en/docs-6.4.0/reference/system-requirements.html) for the MI50's completely beginning with version 6.4.0. 

To install ROCm, AMD provides a quick install, which in theory should be as easy as one copy-paste. 
However, I did not have great luck with it, instead resorting to the detailed installation. This is the preferred method for most people. 
I won't go over every step of the installation, you can refer to AMD's detailed instructions. [^1]

I'm just sharing the eventual command, which installs ROCm and additional libraries. I strongly recommend to reading through all the options under 
`amdgpu-install --list-usecases` and selecting the one's that you need.




## Issues


Another interesting quirk of this card is that it still features a beeper on the PCB. Yes, the kind that is usually found on motherboards.
The utility of it on a GPU is still not that clear to me. 

I first noticed this when turning on my Gigabyte G292-Z00, thinking that the noise must be the result of an error message given by the server.
But after checking the IPMI and running some tests on the GPU, no errors could be found. All GPUs were running as expected.
After investigating and removing one of the four Instinct MI50's, the noise suddenly stopped. 
So, I installed the "faulty" card in another motherboard and was greeted by the same friendly beeping. 
At this point, I thought it must be relatively clear, especially as the card in question was the only one that I took apart to repaste, 
that the card must have some defect, which however does not impact its functionality. 
I took the card apart once again and cleaned it even more. (It should be noted that I got this particular card at a reduced price since it was very beat up.)
Unable to find the true cause, I just removed the card from the server and put it on the shelf. 
In the meantime, another MI50 developed similar behavior, but which stopped after a day or two. 
While browsing the LevelOneTechs forum a week later, I stumbled upon a thread covering the same issue, 
with one user suggesting that insufficient power may be the cause.[^2] 
I'm not able to verify this at the moment, but it does not seem unlikely, considering that on some cards the beeping is only temporary.

## Appendix 0 Flashing VGPUBIOS

In recent years a couple different versions of MI50s have surfaced. There is the authentic MI50 by AMD, recognized as `Radeon Instinct MI50 16GB`.
The other type is a chinese model, which seems to be a Radeon VII that has been modified to look like a Radeon Instinct card.
In software, they are easily distinguished, as the chinese version is recognized as `Radeon VII`.
There is also a 32 GB variant of the card, making it basically a MI60 that also seems to be coming out of China. However, I do not own one of those cards. 



In the following part, I am going to show how to flash the VGPUBIOS of a "fake" Instinct MI50 to a real one. 
Most of this info is from a blog post on a forum about Mersenne primes.[^3] 
The required tool is easily obtainable from [this](https://github.com/stylesuxx/amdvbflash) GitHub repo. 

First, we list all installed GPUs with `sudo ./amdvbflash -i`.

```
AMDVBFLASH version 4.71, Copyright (c) 2020 Advanced Micro Devices, Inc.


adapter seg  bn dn dID       asic           flash      romsize test    bios p/n    
======= ==== == == ==== =============== ============== ======= ==== ================
   0    0000 07 00 66A1 Vega20          GD25Q80C        100000 pass 113-D1631400-X11
   1    0000 45 00 66AF Vega20          GD25Q80C        100000 pass 113-D3600200-105
   2    0000 8A 00 66A1 Vega20          GD25Q80C        100000 pass 113-D1631400-X11
   3    0000 C6 00 66A1 Vega20          GD25Q80C        100000 pass 113-D1631400-X11
```
The two versions of the 16 GB MI50 can be distinguished by different device IDs (dID). 
`66A1` points to the genuine cards, while `66AF` belongs to the chinese version.

I obtained the VBIOS by just copying it out of a genuine MI50 (e.g. adapter 0). 
By running `sudo ./amdvbflash -biosfileinfo bios.rom.0` we get a detailed overview of the VBIOS file.  



```
AMDVBFLASH version 4.71, Copyright (c) 2020 Advanced Micro Devices, Inc.

    Product Name is :    Vega20 A1 SERVER XL D16314 Hynix/Samsung 16GB Gen 24HI 600m 
    Device ID is    :    66A1
    Bios Version    :    016.004.000.056.013521
    Bios P/N is     :    113-D1631400-X11
    Bios SSID       :    0834
    Bios SVID       :    1002
    Bios Date is    :    05/12/21 05:25 
```

By comparison, the "Radeon VII" VBIOS looks like this. 

```
AMDVBFLASH version 4.71, Copyright (c) 2020 Advanced Micro Devices, Inc.

    Product Name is :    Vega20 A1 XT MOONSHOT D36002 16GB 1000m 
    Device ID is    :    66AF
    Bios Version    :    016.004.000.030.011639
    Bios P/N is     :    113-D3600200-105
    Bios SSID       :    081E
    Bios SVID       :    1002
    Bios Date is    :    01/19/19 13:25 

```

The VBIOS from card 0 (genuine MI50) is now flashed to card 1 ("Radeon VII") with the following command.

```
sudo ./amdvbflash -fv -fs -fp -p 1 bios.rom.0
```
The `-fv -fs -fp` flags are overrides because the SSIDs do not match between the cards. 
When all went well, you should see an output like this.
```
sudo ./amdvbflash -fv -fs -fp -p 1 bios.rom.0
AMDVBFLASH version 4.71, Copyright (c) 2020 Advanced Micro Devices, Inc.

Old SSID: 081E
New SSID: 0834
Old P/N: 113-D3600200-105
New P/N: 113-D1631400-X11
The result of RSA signature verify is PASS.
Old DeviceID: 66AF
New DeviceID: 66A1
Old Product Name: Vega20 A1 XT MOONSHOT D36002 16GB 1000m 
New Product Name: Vega20 A1 SERVER XL D16314 Hynix/Samsung 16GB Gen 24HI 600m 
Old BIOS Version: 016.004.000.030.011639
New BIOS Version: 016.004.000.056.013521
Flash type: GD25Q80C
Burst size is 256 
100000/100000h bytes programmed
100000/100000h bytes verified

Restart System To Complete VBIOS Update.
```

[^1]: https://rocm.docs.amd.com/projects/install-on-linux/en/docs-6.4.0/install/detailed-install.html
[^2]: https://forum.level1techs.com/t/amd-instinct-mi50-beeping-constantly/232359
[^3]: https://www.mersenneforum.org/node/1070049
