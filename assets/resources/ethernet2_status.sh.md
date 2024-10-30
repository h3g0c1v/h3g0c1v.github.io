```bash
#!/bin/sh

eth0=$(/usr/sbin/ifconfig eth0 | grep "inet " | awk '{print $2}')

if [ $eth0 ]; then
	echo "%{F#2495e7}󰈀 %{F#ffffff}$(/usr/sbin/ifconfig eth0 | grep "inet " | awk '{print $2}')%{u-}"
else
	echo "%{F#FF6961}󰌙 %{F#ffffff}Disconnected"
fi
```

El primer icono es el siguiente.

![[icono ethernet.png]]

El segundo icono es el siguiente.

![[eth1 icono disconnect.png]]