# Creating the Rpi Images

## Image Creation
1. Using the [Rasbperry Pi Imager](https://github.com/raspberrypi/rpi-imager), create a bootable SD Card, where the public keys for ssh authentication for a user named `rpi` are already configured. This can be done through the Rasbperry Pi Imager's configuration dialogue.


## Image Cloning
There are different ways of cloning the image on other SD Cards. Following is a simple method using `dd` on *nix like systems.
1. Insert the SD Card in your machine and find the device path using tools like `lsblk` or `fdisk -l`
2. Execute `dd if=/dev/sd_card_device_path of=/path/to/image_file.img status=progress`

This will create a byte for byte image from the SD Card.
3. Insert your new SD Card in your machine and run `dd if=/path/to/image_file.img of=/dev/new_sd_card_device_path status=progress`

_Notes:_ The above is a dirty solution as it makes an exact copy of the whole SD Card (also the free space). This can take much longer than using the Rasbperry Pi Imager. It's also worthy to note, that you need your destination SD Cards to be larger or of the same exact make and model, since small differences in number of sectors can lead to problems.

TODO: Find less barbaric solutions to cloning.
Usefull links:
https://stackoverflow.com/questions/19355036/how-to-create-an-img-image-of-a-disc-sd-card-without-including-free-space