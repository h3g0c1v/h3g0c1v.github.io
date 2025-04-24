#!/bin/bash
 
ip_address=$(/bin/cat /home/kali/.config/bin/target | awk '{print $1}')
machine_name=$(/bin/cat /home/kali/.config/bin/target | awk '{print $2}')
 
if [ $ip_address ] && [ $machine_name ]; then
    echo "%{F#FF6961}󰓾 %{F#ffffff}$ip_address%{u-} - $machine_name"
else
    echo "%{F#77DD77}󰓾 %{u-}%{F#ffffff} No target"
fi

-------------------------------------------------------------------------------------------------------------

Ambos iconos son el siguiente.

![Icono target](/assets/img/target-icono.png)
