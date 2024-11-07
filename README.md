# Raspberry Pi Docker Swarm instructions

## Initial SD card operating system installation

1. Download [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
2. Start up Raspberry Pi Imager and use "CHOOSE DEVICE" to select your
   Raspberry Pi model
3. Select "CHOOSE OS" and navigate to
   "Other general-purpose OS" -> Ubuntu -> "Ubuntu Server 24.04.1 LTS (64-bit)"

   NOTE: If you are using a Raspberry Pi 2 or below, or a Raspberry Pi Zero,
   you'll need to pick the 32-bit version instead, and some apps may take some
   effort to get working
4. Insert your SD Card into your computer
5. Select "CHOOSE STORAGE" and select the SD Card you've just inserted. Ensure
   you have the correct device, as it'll be completely wiped and you'll lose
   anything on it
6. Select "NEXT"
7. When asked "Would you like to apply OS customisation settings?", Select "NO"
8. You'll now receive your final prompt to confirm this is the correct storage
   device to wipe. Check this, and then select "YES"
9. When prompted, remove the SD card and close Raspberry Pi Imager

## Customising operating system configuration

1. Insert the SD card you've just freshly flashed with Ubuntu Server
2. Your computer should detect the card and you should be able to find a new
   drive in your file manager. On MacOS this will be called "system-boot"
3. Copy all of the files from this repository's `cloud-init` directory into the
   top directory of the SD card
4. Eject the SD card from your computer

## Raspberry Pi first boot

1. Insert a configured SD card into your Raspberry Pi, and connect ethernet,
   then power
2. Wait a few minutes, and then you should be able to run `ssh
   pi@raspberrypi.local` with the default password `mbt`
