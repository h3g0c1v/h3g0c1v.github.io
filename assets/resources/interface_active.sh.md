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

![Icono VM](/assets/img/vm-icono.png)

El segundo icono es el siguiente.

![Icono Windows](/assets/img/windows-icono.png)

El tercer icono es el siguiente.

![No entry icono](/assets/img/no-entry-icono.png)
