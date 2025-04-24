#!/bin/sh

IFACE1=$(/usr/sbin/ifconfig | grep tun0 | awk '{print $1}' | tr -d ':')
IFACE2=$(/usr/sbin/ifconfig | grep tun1 | awk '{print $1}' | tr -d ':')

if [ "$IFACE1" ]; then
    echo "%{F#77DD77}󰆧 %{F#ffffff}$(/usr/sbin/ifconfig tun0 | grep "inet " | awk '{print $2}')%{u-}"

elif [ "$IFACE2" ]; then
    echo "%{F#FF6961}  %{F#ffffff}$(/usr/sbin/ifconfig tun1 | grep "inet " | awk '{print $2}')%{u-}"

else
	echo "%{F#FF6961} %{u-} Disconnected"
fi

-------------------------------------------------------------------------------------------------------------

El primer icono es el siguiente.

![Icono cubo](/assets/img/cube-icono.png)

El segundo icono es el siguiente.

![Icono nube](/assets/img/cloud-icono.png)

El tercer icono es el siguiente.

![Icono bridge desconectado](/assets/img/icono-disconnect-bridge.png)
