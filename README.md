# Creating-virtual-partition-and-virtual-image
Those instructions by Eng - Moatasem El sayed https://www.youtube.com/@moatasemelsayed6226

- Install qemu-system-arm
- If you build an executable using a cross-compiler, you can build it using qemu-system-aarch64 for example (If the code build is static)
   ``` qemu-arm main```
- To run the same executable (Not statically builded)
    ``` qemu-arm -L /usr/lib/gcc-cross/...linker main```
- To see what machines your qemu supports
    ``` qemu-system-arm --machine ?```
- Configure your u-boot and build it accrding to your qemu machine
    ``` make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- vexpress_ca9x4_defconfig```
- Launch your qemu using
    ```qemu-system-arm -M vexpress-a9 -m 128M -nographic -kernel u-boot```


## Now we have the bootloader and the machine. We need a virtual memory card
- Making your virtual sdcard directory ```mkdir sdcard```
- Making a virtul memory card partition : ```sudo dd if=/dev/zero of=file.imgg count=1024 bs=1M status=progress```
- Dividing it into partitions : ```cfdisk file.img```
    - The first partition should be named boot, ID : 6, Type : FAT 16
    - Second partition should be named rootfs, ID : , Type : Linux
- Now we have our SD card. we just need to access it the same way we access our hdd or real SD-Card. to be mounted like /dev/sda
``` sudo losetup -f --show --partition file.img```
output : ```/dev/loop53```
- now i made my filesystem like that
      --- sdcard
            --- file.img
            --- mmcstorage/
                  --- boot/    -> represents our entrypoint to the boot
                  --- rootfs/  -> represents our entry point to the rootfs
- ```sudo mount /dev/loop53p1 boot/```
- you will face the error ```mount: /home/abdu/Study/Embedded_Linux/git_ignoring/qemu/rpi4_32/sdcard/mmcStorage/rootfs: wrong fs type, bad option, bad superblock on /dev/loop53p2, missing codepage or helper program, or other error.```
- To fix it use ```sudo mkfs.vfat -F 16 /dev/loop53p1```
- ```sudo mount /dev/loop53p1 boot/```
- same thing  with rootfs use ```sudo mkfs.ext4 /dev/loop53p2```
- ```sudo mount /dev/loop53p2 rootfs/```

- To unmount your partitions : ``` sudo umound boot```
  ``` sudo umount rootfs```
- ```sudo losetup -d /dev/loop53```

- Right now we have an sdCard, which I can represent my image inside.

# Using my virtual SD card in qemu
``` sudo qemu-system-arm -M vexpress-a9 -m 128M -nographic -kernel u-boot -sd ../qemu/rpi4_32/sdcard/file.img```

# Now I want to build my kernel
- download linux
- ```make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- vexpress_defconfig```
- ```make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j 30```
- Now we can try to build it with qemu ```qemu-system-arm -M vexpress-a9 -m 120M -kernel ../linux/arch/arm/boot/zImage -nographic``` It will fail because there is no device tree.
- Now we can try to build it with qemu ```qemu-system-arm -M vexpress-a9 -m 120M -kernel ../linux/arch/arm/boot/zImage -nographic``` It will fail because there is no device tree.

- Passing dtb (device-tree binary) to the kernel ```sudo qemu-system-arm -M vexpress-a9 -m 128M -nographic -kernel linux/arch/arm/boot/zImage -dtb linux/arch/arm/boot/dts/arm/vexpress-v2p-ca9.dtb```
It will boot successfully, but it doesn't have any rootfs. So it can't mount rootfs because it doesn't exist. Also there is no bootargs.

- passing the bootargs to the kernel in qemu using ```--append cmdline```
- ```sudo qemu-system-arm -M vexpress-a9 -m 128M -nographic -kernel linux/arch/arm/boot/zImage -dtb linux/arch/arm/boot/dts/arm/vexpress-v2p-ca9.dtb --append "console=ttyAMA0,115200"``` Again it won't continue because there is no rootfs

# Initramfs


