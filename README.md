# Oneblox Manager

Set of simple tools to control hardware in a rooted Exablox Oneblox 3308 storage appliance.  Unfortunately, the company behind this hardware has completely abandoned the product.  As typical with products that depend on some cloud component, the management interface for this device depends on Exablox servers and therefore this product has been rendered e-waste.  I bought one on eBay thinking it'd look pretty nice sitting in my rack and might actually be usable for something.  For a while, it ran fine in it's very default mode as a simple, not very fast SMB server.  I was able to fill it with 600GB SAS drives, over time swapping in 4TB drives, and Exablox's scalable RAID worked fine, if not giving me any actual feedback of what it was doing in the background.  

I could even swap 2 drives at a time and it maintained the files.  Unfortunately, when I almost had the drive slots filled entirely with HGST 8TB drives, the filesystem disappeared.  Oh well. 

So I decided to pop the cover open and look inside.  Surprisingly, there's a 500GB SSD.  And woohoo, to my surprise it had a Linux root pawrtition on it.  So, after cloning the drive, I very easily re-enabled root account and telnet so I could pop in and do a little exploration.

The system used is a very old version of Debian running kernel 3.10 (Linux version 3.10.49oneblox-rel-2.6.x-44) on a 6-core MIPS64 Cavium 6335 Octeon-II with 16GB of main memory.  It's quite a bit more powerful than most dedicated NAS products and mine currently runs PostgreSQL, CouchDB, ProFTPD and several Node webapps.  The processor was designed for network appliances so there is no hardware floating-point. Disk i/o is handled through an LSI SAS/SATA controller that offers no built-in RAID functionality.  Spinning up drives makes them appear within /dev/sdX as one would expect.  I've found some weird issue where a single drive appears under multiple devices but mounting drives by partition label makes this not an issue.

Exploration of the oneblox system files in /opt/exablox revealed a lot of hardware details.  There is a bash script called oneblox-i2c which sets up the GPIO bus and is used to enable power to any of the 8 drive slots, as well as toggling green & red lights next to each drive slot, as well as several LEDs above the info screen.  Several 8-bit registers map to each of these functions.  For instance, poking 0x11 into the power register enabled the two drive slots on the left.  Poking 0x0f turns on all 4 drive slots on the top row and 0xf0 does for the bottom row.

I'm hoping to explore more aspects of the hardware and write a small C daemon that makes hardware management simple.  I still need to tackle the LCD info panel, temperature monitoring and fan controls, which fortunately looks pretty trivial.  

All the tools are here to manage Exablox's SmashFS dynamic RAID system but I don't intend on using it.

I was able to get the old version of Debian linked to the online repo (deb http://archive.debian.org/debian wheezy main) so I could install apps.  The bad news is that root is actually mounted to some main-board flash memory with very little space and corrupting this could render the system unbootable (unless there's UART on board I can find) and a little symlinking was able to clear up a few 100MBs.  Used gparted to resize partitions on the SSD gave a lot more free system space.  

I should note that the Oneblox 3308 does not make a very fast fileserver and I'm not sure why.  Network i/o is not particularly great, and neither is disk i/o.  The Cavium network processor has some really neat on-board packet processing, hardware data compression and hardware REGEX processing that can handle gigabytes per second, but using these features in your own software is a bit difficult without the proper version of the Octeon SDK that matches the kernel.  I've tried.

