```bash
#!/bin/bash

interface=$(cat /home/kali/.config/scripts/ethernet_status.sh | grep -Eo "eth." | head -n 1)

if [ $interface == "eth0" ]; then
	echo "%{F#2495e7}%{F#ffffff}"
elif [ $interface == "eth1" ]; then
	echo "%{F#2495e7}%{F#ffffff}"
else
	echo "%{F#2495e7} %{F#ffffff}"
fi
```

El primer icono es el siguiente.

![[vm icono.png]]

El segundo icono es el siguiente.

![[windows icono.png]]

El tercer icono es el siguiente.

![[no entry icono.png]]