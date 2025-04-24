#!/bin/bash

mv /home/kali/.config/scripts/ethernet2_status.sh /home/kali/.config/scripts/temp
cp /home/kali/.config/scripts/ethernet_status.sh /home/kali/.config/scripts/ethernet2_status.sh
mv /home/kali/.config/scripts/temp /home/kali/.config/scripts/ethernet_status.sh
