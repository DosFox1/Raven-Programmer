# Raven-Programmer
An implementation of a 68HC705E1 Programmer, to replace the 68HC05E1 Mask ROM Microcontroller that was used as the EGRET and CUDA ICs in 90s Macintoshes, with the EPROM programmable 68HC705E1 part. 

The EGRET and CUDA were used to implement the RTC, PRAM, Soft Power and other functions. As these are located near to the RTC battery, these are well known to be destroyed either by battery leaks or due to capacitor leaks.

 NOTE - THIS PROCESS HAS NOT BEEN TESTED YET, THIS REPOSITORY WILL BE UPDATED WHEN THE PROCESS HAS BEEN VALIDATED.

# How do I use it?

Unfortunately, the programmer is a two step process.
 The first step is to program the desired ROM to a suitable 2764 compatible 8K E/EPROM, using a suitable programmer. (In my case, I used an AT28C64 with a TI866+).

Once programmed, the ROM and a blank 68HC705e1 can then be inserted into the Raven programmer.

First, set switch 1 to on, and switch 2 to off. 
This will set the programmer into “validation” mode.
Bring the 5v rail up first before bringing the 16v programming rail up. 
Bringing the 16v rail up before the 5v rail may result in damage tom the IC. 
When both rails are stable, hit the reset button. 
Note that the 68HC705 will not enter boot loader mode until both power rails are present.

If the device has not been programmed before, neither the program or verify LEDs will light. 

Once satisfied that the 68HC705e1 is likely to be unprogrammed, set both switch 1 and 2 to “on”, and hit the reset button. 
This will set the programmer into program mode. 
At this point, the programmer will begin copying the contents of the external ROM into the onboard ROM of the 68HC705e1.
During this, the “program” LED will become illuminated. 
This process should take approximately 40 seconds, wherein the “program” LED will be extinguished, and the “verify” LED will become illuminated - to indicate that programming was successful.
If there is an issue - the “verify” LED will not become illuminated. 

To ensure that programming was a success, power cycle the setup (whilst ensuring that the 16v rail is powered down first, and powered up first), and set switch 1 to “on” and switch 2 to “off”. 
Again, this will set the programmer into verify mode - however, when the reset LED is hit in this instant, the “verify” LED should become illuminated after a second. 

Once satisfied, power down the programmer, and remove the 68HC705. 

# Where do I get the ROMs? 
At the moment, there is only a single ROM available in the correct format for the Raven programmer. This is the 341S0851 EGRET, that is a commonly used replacement for the majority of the EGRETs. 

The original program data was taken from the internet archive: 
https://archive.org/details/macroms

Which is likely taken from this tool here for dumping EGRET/CUDA ROMs:
https://www.gryphel.com/c/minivmac/extras/egretrom/

These ROMs have been used with a 68HC05 emulator within MAME, so it is likely the data is valid data. 

However, please note that there is some formatting that needs to take place before the ROM files can be burnt to the external 8K EPROM. 

Namely, the program data needs to be offset from 0x0000 to 0x0F00. This program data is 4096 bytes long, and resides from 0x0F00 to 0x1EFF. 

Following this, there needs to be single byte written at 0x1F00 - which is the Mask Option Register. 
From what we can tell, this data is usually part of the mask of the 68HC05e1 itself, and as a result needs to be defined in the 68HC705. As the data from 0x1F00 through to 0x1FEF is the bootloader ROM on the 68HC05e1, and the dumps include the entire bootloader ROM for the EGRET and CUDA, then writing the data from 0x1F00 from the dump will not work. 
Instead, MOR needs to be defined from context clues. 

The MOR defines three things - whether the watchdog is enable, if an external interrupt is enabled, and the rate of the internal interrupt.
In the case of the EGRET, the NMI pin is always held high - meaning it’s likely the external interrupt is not used. According to a decompilation by PhilPem, the Watchdog timer is not used. 
As the EGRET is also used as an RTC, it’s likely that the internal interrupt is 1s long. As a result, 0x1F00 should have the byte “0x04” written to it.

It’s hoped that even if the internal interrupt is wrong, then the clock will just run slightly slow - which can be easily fixed by burning a new EGRET. 


The data from 0x1F01 through to 0x1FEF should then be set to 0x00, as this is the region where the bootloader resides. It’s safer to null out this data region to avoid possibly corrupting the 68hc705e1. 

Finally, the last 16 bytes from 0x1FF0 through to 0x1FFF are the EPROM user vectors, that need to be copied from the original ROM. 

Additionally, there is an errata sheet (included) that details a bug with the exact mask version of the 68HC705 that I have programmed (D32N). Basically a silicon bug means that the 68HC705e1 doesn’t read the data at 0x1000, and reads the data at 0x0000 for a single cycle. This is why the included example ROM has a single byte of 0x12 at 0x0000 - this is a duplication of the data at 0x1000. It does not harm or hinder to have this additional byte present at 0x0000. 

# Errata
VDEV2 is the current version of the Raven Programmer. The VDEV1 version had two issues - one introduced by me, the other by NXP. 
The issue I managed to introduce was accidentally mislabelling R1 and R2. I managed to get the 330K and 20M labels the wrong way around. This resulted in the oscillator not running, and was fixed in VDEV2. 

The major issue was introduced by NXP, who in their datasheet (68hc705e1.pdf in the datasheet folder) failed to mention that PB1 needed to be brought high to 5v, and PB6/PB7 needed to be brought low to 0v. As a result, with those three pins floating, the 68HC705e1 behaved erratically and failed to program. In this instant, both the verify and program LEDs remained illuminated.

Thankfully, the discovery of the original Motorola datasheet (68HC705e1 Better Datasheet.pdf), I was able to correct the issue, and actually program the devices.

# Acknowledgements 

Many thanks to PhilPem (@philpem@digipres.club) for helping with the software side of this headache, hopefully they work!

Also many thanks must be given to Max1zzz (https://github.com/max234252/) for agreeing to testing this new batch of EGRETs. We can only help that these will work, as well as agreeing to send an OTP CUDA over, so we can hopefully have a look at what a proper MOR byte should look like!
