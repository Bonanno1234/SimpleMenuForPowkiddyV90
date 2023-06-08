# SimpleMenuForPowkiddyV90
Simple Menu For Powkiddy V90
This is a port version of Simple Menu to Powkiddy V90 and any device with MiyooCFW

Main Simple Menu: 

https://github.com/fgl82/simplemenu

Thanks Taichi For the guide, you can see the guide in here : 

https://gist.github.com/taichikuji/8070ddf3349a4ec57053fc9e417289bc


In Final edit, main file is:

#!/bin/sh
set -e

boot_logo_path="/mnt/boot-logo"

export HOME="/mnt"
export SDL_NOMOUSE="1"

start_script_path="/mnt/autoexec.sh"

echo -e "\e[?1c" #replace 1 with 3 to see text
#/mnt/kernel/setcolors /mnt/kernel/colors

modprobe r61520fb.ko

# Detect low battery and prevent booting up if so
if test "$(cat /sys/class/power_supply/miyoo-battery/voltage_now)" -lt '3400'; then
clear; echo -e "\n\n\n\n\n\n\n\n\n\n\n\n\n\n            \e[1;33m[ LOW BATTERY ]\e[0m"
sync
sleep 3
poweroff
exit
fi

#Check if fat32 is flagged as "dirty", and if so unmount, repair, remount
if dmesg | grep "mmcblk0p4" > /dev/null; then
    echo -e "\e[31mUnclean shutdown detected.\e[0m"
    echo -e "\e[32mChecking FAT32 partition...\e[0m"
    if umount "/dev/mmcblk0p4"; then
        echo "Unmounted FAT32 partition."
    else
        echo "Could not unmount FAT32 partition."
    fi
    if fsck.vfat -y "/dev/mmcblk0p4" > /dev/null; then
        echo "Repaired FAT32 partition."
    else
        echo "Could not repair FAT32 partition."
    fi
    if mount "/dev/mmcblk0p4" "/mnt" -t "vfat" -o "rw,sync,utf8"; then
        echo "Remounted FAT32 partition."
    else
        echo "Could not remount FAT32 partition."
    fi
    echo -e "\e[32mCheck complete.\e[0m"
fi
clear

/mnt/kernel/daemon > /dev/null 2>&1

function check_battery {
while true; do
    sleep 300
    if test "$(cat /sys/class/power_supply/miyoo-battery/voltage_now)" -lt '3400'; then
    clear; echo -e "\n\n\n\n\n\n\n\n\n\n\n\n\n\n            \e[1;33m[ LOW BATTERY ]\e[0m"
    kill -TERM "${1}"
    fi
done
}

while true; do
    if [ -f "$start_script_path" ]; then
        . "$start_script_path"
    else
        cd "/mnt/gmenu2x"
        ./gmenu2x > /dev/null 2>&1
    fi
    clear
    pid="${!}"
    sleep 3
    clear; echo -e "\n\n\n\n\n\n\n\n\n\n\n\n\n\n               \e[1;33m[ SAVING ]\e[0m"
    check_battery "${pid}" &
    wait "${pid}"
    sync
    poweroff
done
