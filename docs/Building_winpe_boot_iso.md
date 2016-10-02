# Building a customized WinPE image

Why the page is appeared this because of Microsoft has good tools for making Windows OSes being installed over network using PXE. But their documentation is a mix of retired and new tools and I didn't find a complete guide how to make possible to run setup of Windows without the Google searcher help.

I am writing this README for Windows 7 because this one is suitable for my family. Actually all but the adding a network driver possible to build very fast but a default winpe image does not contain a network driver for my laptop so I decided to memorize my efforts.


## Prerequisites

Before it is possible to do something useful need to complete a few steps in advance.


## Common prerequisites

1. We need to have installed Windows 7. I'm running one in a KVM virtual machine.
2. A Linux machine that is configured to run as a PXE server. I use for this an ARM board [Banana R1](http://www.banana-pi.org/r1.html).
3. A running Samba service. I have another ARM based board [Cubieboard 2](http://cubieboard.org/) that's running it.

Actually it doesn't matter where and how you running the services they just should be avaiable. It is considered that all required PXE services and Samba are already configured and running so once you boot a PC over network it gets booting PXE boot menu. Here describes only the things related to Windows OS.


### Windows 7: prerequsites

(to do) Installation of Automated Installation Kit (AIK).


## Building WinPE ISO image

*All in this chapter is a compilation of [Microsoft TechNet's articles][https://TechNet.microsoft.com/en-us/library/dd799281(v=ws.10).aspx]*.

As of this step we'll work inside a special enviroment running by installed AIK. To go in, run as an administrator **Start -> Programs -> Microsoft Windows AIK -> Deployment Tools Image Manager**. This is a command shell like cmd.exe will be run within preconfigured environment variables.

### Set up a Windows PE build environment

MS TechNet article [here](https://TechNet.microsoft.com/en-us/library/dd799303(v=ws.10).aspx).


Initialization of a working directory

	copype.cmd amd64 c:\winpe_amd64

Copying a base image

	copy C:\winpe_amd64\winpe.wim C:\winpe_amd64\ISO\sources\boot.wim

At this place it is possible to make any customization that needed to be in the end WinPE image.


### Adding a network driver

It is not clear what reasons why Microsoft does not include a lot of network drivers in a default WinPE image, I think it because of end size.


Mounting winpe image into a directory

	Dism /Mount-Wim /Wim-Image:"C:\winpe_amd64\ISO\sources\boot.wim" /index:1 /MountDir:"C:\winpe_amd64\mount"

Adding a network card driver 
	
	Dism /Add-Driver /Image:"C:\WinPE_amd64\mount" /Driver:"\\192.168.168.103\install\drivers\network\win7_rtl8101e\64\rt64win7.inf"

The path `\\192.168.168.103\...` could be any that Windows can recognize.

And one more driver

	Dism /Add-Driver /Image:"C:\WinPE_amd64\mount" /Driver:"\\192.168.168.103\install\drivers\network\win7_rtl8106e_rtl8111g\64\rt64win7.inf"

This way you can insert so many drivers as you need for any kind of devices and not only for network cards. At this moment we can think that we have done what we wanted and now need to sync performed modifications and built an iso image.


### Verify included drivers

This command shows the extra drivers that has been added manually.

	Dism /Get-Drivers /Image:"C:\winpe_amd64\mount"


### Commit and unmount winpe image

Run this

	Dism /Unmount-Image /MountDir:"C:\winpe_amd64\mount" /commit


### Create a bootable ISO image

When you have finished all preparations and customizations you will want to have a ready to write an ISO image. Next command do the thing

	oscdimg -n -bC:\winpe_amd64\etfsboot.com C:\winpe_amd64\ISO C:\winpe_amd64\winpe_amd64.iso

If do nothing and just built a bare ISO image it gets size around 170 MB.

Since now this lightweight ISO image can be burnt on a blank CD/DVD disc. But I haven't been using CD/DVD disc for a long time at all and will use the ISO image for loading it via PXE.


### Copy the ISO image to PXE server


