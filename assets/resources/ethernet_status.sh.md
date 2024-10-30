```bash
#!/bin/sh

eth1=$(/usr/sbin/ifconfig eth1 | grep "inet " | awk '{print $2}')

if [ $eth1 ]; then
	echo "%{F#2495e7}󰘘 %{F#ffffff}$(/usr/sbin/ifconfig eth1 | grep "inet " | awk '{print $2}')%{u-}"
else
	echo "%{F#FF6961} %{F#ffffff}Disconnected"
fi
```


El primer icono es el siguiente.

![[icono bridge.png]]

El segundo icono es el siguiente.

![[icono disconnect bridge.png]]