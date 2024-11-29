---
title: Personalización de entorno en Linux
description: En este artículo vamos a personalizar un entorno Linux, con el fin de mejorar la eficacia.
date: 2024-10-30
categories: [personalizacion, entorno]
tags: [personalizar, entorno, bspwm, sxhkd, polybar, kali-linux, parrot-os, kitty, picom, rofi, fzf, lsd, batcat, nvchad, neovim, nvim, hack4u, s4vitaar, burpsuite, modulos, virtualizacion, vmware, virtualbox, feh, powerlevel10k]
img_path: /assets/img/entorno.png
image: /assets/img/entorno.png
---

## **Introducción**

Usar *Kali Linux* o *Parrot OS* por defecto está genial, sin embargo, si queremos llevar a otro nivel nuestro movimiento por el entorno y ser más eficientes, deberemos de personalizar nuestro entorno de manera que nos sintamos cómodos y seamos rápidos en él. Es por esto, por lo que traigo este artículo en el que personalizaremos nuestro entorno en un sistema *Kali Linux*, aunque esta misma configuración se puede realizar en un sistema *Parrot OS*.

## **Descargando Kali Linux**

Para descargarnos el SO Kali Linux, nos deberemos de dirigir a la [página de Kali Linux](https://www.kali.org/get-kali/). Seleccionaremos la descarga a nuestra preferencia. En mi caso descargaré la de VMWare por lo que seleccionaré la opción correspondiente a la de VMWare.

Una vez descargado e instalado en nuestro software de virtualización, ejecutaremos un **upgrade** completo del sistema.

>**IMPORTANTE**<br>
>Ten en cuenta que, si estás en *Parrot OS* los comandos que tendrás que usar para realizar el *upgrade completo del sistema* son los siguientes:
>
>`sudo apt update -y && sudo apt parrot-upgrade -y`

```bash
sudo apt update -y && sudo apt upgrade -y
```

Al finalizar al upgrade, realizaremos un *Snapshot* de la máquina ya que, en caso de que nos falle posteriormente, tendremos un punto de partida al que volver.

## **Instalación de paquetes necesarios**

Antes que nada, vamos a instalar algunos paquetes, algunos **necesarios para la configuración del entorno**, y otros adicionales. Puedes ajustar estos paquetes a tu gusto.

```bash
sudo apt install build-essential git vim libxcb1 libxcb-util0-dev libxcb-ewmh-dev libxcb-randr0-dev libxcb-icccm4-dev libxcb-keysyms1-dev libxcb-xinerama0-dev libasound2-dev libxcb-xtest0-dev libxcb-shape0-dev make zsh caja locate -y
```

## **Instalación y configuración de Bspwm y Sxhkd**

Instalaremos *bspwm* y *sxhkd* desde los repositorios oficiales. Para ello, nos dirigiremos a nuestra carpeta `Downloads` y clonaremos los repositorios **desde un usuario NO privilegiado**.

```bash
git clone https://github.com/baskerville/bspwm.git
git clone https://github.com/baskerville/sxhkd.git
```

Seguidamente, nos meteremos dentro de la carpeta `bspwm` y le instalaremos.

```bash
cd bspwm
make
sudo make install
```

Si ahora ejecutamos el comando `which bspwm` deberíamos de ver el binario.

```bash
which bspwm
```

A continuación, realizaremos la misma acción pero para `sxhkd`.

```bash
cd sxhkd
make
sudo make install
```

Si ahora ejecutamos el comando `which sxhkd`, veremos el binario correspondiente.

```bash
which sxhkd
```

Algo que debemos de tener en cuenta son sus ficheros de configuración.

- **bspwm** ➜ `bspwmrc`
- **sxhkd** ➜ `sxhkdrc`

Para poder configurar cada uno de ello, crearemos los directorios de configuración de cada uno en `~/.config/`.

```bash
mkdir ~/.config/{bspwm,sxhkd}
```

Seguidamente, copiaremos los ficheros de configuración que nos comparte por defecto el repositorio, en las carpetas que hemos creado.

```bash
cp /home/kali/bspwm/examples/bspwmrc ~/.config/bspwm/
cp /home/kali//bspwm/examples/sxhkdrc ~/.config/sxhkd/
```

Comprobaremos que en cada uno de los directorios tenemos los ficheros que hemos copiado.

```bash
ls -l /home/kali/.config/bspwm
ls -l /home/kali/.config/sxhkd
```

A continuación, empezaremos con la configuración de `sxhkd`.

```bash
cd /home/kali/.config/sxhkd
```

Configuraremos el primer *shortcut* para abrir una terminal `kitty`. Para ello, instalaremos `kitty` de la siguiente manera.

```bash
sudo apt install kitty -y
```

Comprobamos que la versión de `kitty` es la última con el comando `kitty -v`.

En caso de que no sea la última, tendremos que descargar el último *release* del [repositorio oficial de Github](https://github.com/neovim/neovim/). Si tenemos la última versión, configuraremos la siguiente línea:


```bash
## terminal emulator
super + Return
        /usr/bin/kitty
```

También, modificaremos algunas líneas del fichero para añadir unos cuantos *shortcuts*.

```bash
## quit/restart bspwm
super + shift + {q,r}
        bspc {quit,wm -r}

## close and kill
super + {_,shift + }q
        bspc node -{c,k}

## focus the node in the given direction
super + {_,shift + }{Left,Down,Up,Right}
        bspc node -{f,s} {west,south,north,east}

#
## preselect
#

## preselect the direction
super + ctrl + alt {Left,Down,Up,Right}
        bspc node -p {west,south,north,east}

## cancel the preselection for the focused node
super + ctrl + alt + space
        bspc node -p cancel

#
## move/resize
#

## move a floating window
super + shift + {Left,Down,Up,Right}
        bspc node -v {-20 0,0 20,0 -20,20 0}
```

También, quitaremos las siguientes líneas:

```bash
#
## move/resize
#
 
## expand a window by moving one of its side outward
super + alt + {h,j,k,l}
        bspc node -z {left -20 0,bottom 0 20,top 0 -20,right 20 0}

## contract a window by moving one of its side inward
super + alt + shift + {h,j,k,l}
        bspc node -z {right -20 0,top 0 20,bottom 0 -20,left 20 0}
```

Y ahora, añadiremos las líneas de a continuación al final del archivo.

```bash
## Custom Resize
super + alt + {Left,Down,Up,Right} 
        /home/kali/.config/bspwm/scripts/bspwm_resize {west,south,north,east}

## Open Firefox
super + shift + f
		/usr/bin/firefox
```

Dentro del fichero `/home/kali/.config/bspwm/scripts/bspwm_resize` meteremos el contenido que podemos encontrar en [bspwm_resize.txt](https://hack4u.io/wp-content/uploads/2022/09/bspwm_resize.txt).

```bash
#!/usr/bin/env dash

if bspc query -N -n focused.floating > /dev/null; then
	step=20
else
	step=100
fi

case "$1" in
	west) dir=right; falldir=left; x="-$step"; y=0;;
	east) dir=right; falldir=left; x="$step"; y=0;;
	north) dir=top; falldir=bottom; x=0; y="-$step";;
	south) dir=top; falldir=bottom; x=0; y="$step";;
esac

bspc node -z "$dir" "$x" "$y" || bspc node -z "$falldir" "$x" "$y"
```

Para que funcione correctamente, le asignaremos permisos de ejecución.

```bash
chmod u+x bspwm_resize
```
## **Instalación de la polybar, Picom y Rofi**

Instalaremos la `polybar` ejecutando el comando que se muestra a continuación.

```bash
sudo apt install polybar -y
```

Instalaremos y configuraremos [picom](https://github.com/yshui/picom). Para ello, instalaremos todos los siguientes paquetes.

```bash
sudo apt install libconfig-dev libdbus-1-dev libegl-dev libev-dev libgl-dev libepoxy-dev libpcre2-dev libpixman-1-dev libx11-xcb-dev libxcb1-dev libxcb-composite0-dev libxcb-damage0-dev libxcb-glx0-dev libxcb-image0-dev libxcb-present-dev libxcb-randr0-dev libxcb-render0-dev libxcb-render-util0-dev libxcb-shape0-dev libxcb-util-dev libxcb-xfixes0-dev meson ninja-build uthash-dev -y
```

Clonaremos el repositorio de `picom` en `Downloads`.

```bash
git clone https://github.com/yshui/picom
```

Nos meteremos dentro de la carpeta de `picom` y tendremos que construirlo. Seguiremos los pasos del fichero [README.md](https://github.com/yshui/picom/blob/next/README.md#to-build) del repositorio.

```bash
cd picom
meson setup --buildtype=release build
ninja -C build
```

Por último, construiremos con `ninja` el binario de `picom`.

```bash
ninja -C build install
```

Si en este momento, ejecutamos un `which picom` veremos el binario que ya está instalado.

```bash
which picom
```

Instalaremos `rofi` con el siguiente comando.

```bash
sudo apt install rofi -y
```

Y configuraremos un *shortcut* para abrir `rofi`. Editaremos el fichero de configuración `/home/kali/.config/sxhkd/sxhkdrc`.

```bash
## program launcher
super + d     
        /usr/bin/rofi -show run 
```

Ahora sí, reiniciaremos el equipo e iniciaremos el entorno de `bspwm`.

```bash
reboot
```

Una vez reiniciado, para iniciar el entorno `bspwm` lo seleccionaremos en la parte del login.

![Iniciando Bspwm](/assets/img/iniciando-bspwm.png)

En caso de que no nos aparezca, deberemos de instalar `bspwm` con `apt`.

```bash
sudo apt install bspwm -y
```

Reiniciaremos el equipo de nuevo y lo seleccionaremos en la parte superior derecha del inicio de sesión.

Al iniciar en el entorno `bspwm`, puede que se nos desconfigure la configuración del teclado. Para volver a poner el idioma en **Español**, deberemos de ejecutar el siguiente comando.

```bash
sudo dpkg-reconfigure keyboard-configuration
```

Seleccionaremos el layout por defecto que nos aparece en la primera pantalla. Seguidamente, seleccionaremos `Other` + `Spanish` y las demás opciones las dejaremos por defecto.

Una vez configurado, reiniciaremos el equipo y ya tendremos el teclado nuevamente configurado.

```bash
reboot
```
## **Habilitando la clipboard bidireccional**

Vamos a habilitar la clipboard bidireccional. Para ello, abriremos el fichero `/home/kali/.config/bspwm/bspwmrc` y meteremos el siguiente contenido al final de este.

```bash
vmware-user-suid-wrapper &
```

Reiniciaremos el `bspwmrc` con `Win + Shift + R` y ya podríamos copiar y pegar bidireccionalmente.
## **Configurando las fuentes, la Kitty e instalación de Feh**

A continuación, instalaremos la fuente *Hack Nerd Fonts*. Para ello, iremos al [siguiente enlace](https://www.nerdfonts.com/).

>**Enlace a la página de fuente**<br>
>https://www.nerdfonts.com/font-downloads

Y nos descargaremos el comprimido de la fuente de *Hack Nerd Fonts*.

![Hack Nerd Fonts](/assets/img/hack-nerd-fonts.png)

Nos dirigiremos al siguiente directorio, moveremos el comprimido y lo descomprimiremos como el usuario *root*. Además, eliminaremos los ficheros que no necesitamos.

```bash
sudo su
cd /usr/local/share/fonts
mv /home/kali/Downloads/Hack.zip
unzip Hack.zip
rm Hack.zip LICENSE.md README.md
```

Ahora, vamos a configurar la `kitty` y para ello, nos dirigiremos a `~/.config/kitty` como un usuario **NO** privilegiado.

```bash
cd ~/.config/kitty
```

Y crearemos dos archivos. El primero, será `kitty.conf` con el siguiente contenido. Este fichero lo podemos encontrar en [kitty.conf.txt](https://hack4u.io/wp-content/uploads/2022/09/kitty.conf_.txt).

```bash
enable_audio_bell no

include color.ini

font_family HackNerdFont
font_size 13

disable_ligatures never

url_color #61afef

url_style curly

map ctrl+left neighboring_window left
map ctrl+right neighboring_window right
map ctrl+up neighboring_window up
map ctrl+down neighboring_window down

map f1 copy_to_buffer a
map f2 paste_from_buffer a
map f3 copy_to_buffer b
map f4 paste_from_buffer b

cursor_shape beam
cursor_beam_thickness 1.8

mouse_hide_wait 3.0
detect_urls yes

repaint_delay 10
input_delay 3
sync_to_monitor yes

map ctrl+shift+z toggle_layout stack
tab_bar_style powerline

inactive_tab_background #e06c75
active_tab_background #98c379
inactive_tab_foreground #000000
tab_bar_margin_color black

map ctrl+shift+enter new_window_with_cwd
map ctrl+shift+t new_tab_with_cwd

background_opacity 0.95

shell zsh
```

Y otro con nombre `color.ini` con el siguiente contenido. Este fichero lo podemos encontrar en [color.ini.txt](https://hack4u.io/wp-content/uploads/2022/09/color.ini_.txt).

```bash
cursor_shape          Underline
cursor_underline_thickness 1
window_padding_width  20

## Special
foreground #a9b1d6
background #1a1b26

## Black
color0 #414868
color8 #414868

## Red
color1 #f7768e
color9 #f7768e

## Green
color2  #73daca
color10 #73daca

## Yellow
color3  #e0af68
color11 #e0af68

## Blue
color4  #7aa2f7
color12 #7aa2f7

## Magenta
color5  #bb9af7
color13 #bb9af7

## Cyan
color6  #7dcfff
color14 #7dcfff

## White
color7  #c0caf5
color15 #c0caf5

## Cursor
cursor #c0caf5
cursor_text_color #1a1b26

## Selection highlight
selection_foreground #7aa2f7
selection_background #28344a
```

Ahora, en caso de que no lo tengamos instalado, instalaremos `imagemagick` para el tratamiento de imágenes.

```bash
sudo apt install imagemagick -y
```

Supongamos que queremos ver una imagen. Para ello, podemos usar el siguiente comando.

```bash
kitty +kitten icat imagen.png
```

Esto, es bastante tedioso a la hora de escribir, por lo que configuraremos en nuestro `~/.zshrc` una función que nos permita efectuar esto.

```bash
## Visualizar imagenes con kitten
function show () {
	kitty +kitten icat $1
}
```

Para que la configuración de la `kitty` sea igual con el usuario *root*, nos deberemos de traer la configuración que hemos aplicado en el directorio `~/.config/kitty`

```bash
sudo su
cp -r /home/kali/.config/kitty /root/.config/kitty
```

Para configurar un fondo de pantalla, necesitaremos `feh`.

```bash
sudo apt install feh -y
```

Nos crearemos una carpeta `Fondos` dentro del escritorio para ir metiendo todos los fondos ahí.

```bash
mkdir ~/Desktop/Fondos
```

Nos descargamos cualquier imagen que nos guste como fondo de pantalla. En mi caso, hare una copia de ella y la renombraré como `Wallpaper.jpg`, para que de esta manera, no tener que cambiar el nombre de la imagen y siempre se llame `Wallpaper.jpg`.

En mi caso, este es el fondo que estaré usando yo.

![Im groot Root](/assets/img/im-groot-root.jpg)

Y configuramos el fondo de pantalla en el fichero `~/.config/bspwm/bspwmrc`.

```bash
/usr/bin/feh --bg-fill /home/kali/Desktop/Fondos/Wallpaper.jpg
```

## **Despliegue de la Polybar**

Nos vamos a ir al directorio `Downloads` como un usuario **NO** privilegiado y nos clonaremos un repositorio que trae ciertas configuraciones de la `polybar` que vamos a estar usando.

```bash
cd ~/Downloads
git clone https://github.com/VaughnValle/blue-sky.git
cd blue-sky/polybar
```

Ahora, copiaremos todo el contenido de este directorio `blue-sky/polybar` en nuestro directorio `~/.config/polybar`.

```bash
mkdir ~/.config/polybar
cp -r * ~/.config/polybar
```

Seguidamente, configuraremos que se inicie la `polybar` en el inicio del sistema.

```bash
echo '~/.config/polybar/./launch.sh &' >> ~/.config/bspwm/bspwmrc
```

Y como el usuario *root* copiaremos las fuentes al directorio `/usr/share/fonts/truetype`.

```bash
sudo su
cp fonts/* /usr/share/fonts/truetype
```

Sincronizaremos y actualizaremos la cache de fuentes con el siguiente comando.

```bash
fc-cache -v
```

De esta manera, si nos salimos de la sesión con `CTRL + SHIFT + q` e iniciamos sesión, veremos cómo nos carga correctamente la polybar.

![Polybar recién instalada](/assets/img/polybar-recien-instalada.png)

## **Configurando los bordeados, las sombras y los difuminados con Picom**

Vamos a configurar `picom` y para ello deberemos de crear un fichero de configuración para este.

```bash
mkdir ~/.config/picom
touch ~/.config/picom/picom.conf
```

E introduciremos el contenido del fichero que encuentra en [picom.conf.txt](https://hack4u.io/wp-content/uploads/2022/09/picom.conf_.txt) y/o en [picom.conf.txt](https://github.com/h3g0c1v/h3g0c1v.github.io/blob/main/assets/resources/picom.conf.md).

Y deberemos de actualizar el fichero `~/.config/bspwm/bspwmrc` para correr el demonio de `picom`.

```bash
picom &
```

A demás, vamos a quitar el borde blanco de la terminal, desde el fichero `~/.config/bspwm/bspwmrc`.

```bash
bspc config border_width 0
```

## **Configurando la ZSH e instalando Powerlevel10k**

Vamos a instalar unos plugins para la  *zsh*. Nos ponemos como *root* y ejecutamos el siguiente comando.

```bash
sudo su
sudo apt install zsh-autcomplete zsh-autosuggestions zsh-syntax-highlighting
```

En caso de que el plugin `zsh-autcomplete` no nos funcione con `apt install`, lo tendremos que instalar manualmente. Nos iremos a su [repositorio de GitHub](https://github.com/marlonrichert/zsh-autocomplete) y lo clonaremos desde el directorio `/usr/share`.

```bash
cd /usr/share
git clone --depth 1 -- https://github.com/marlonrichert/zsh-autocomplete.git
```

Seguidamente, añadiremos en este mismo fichero las siguientes líneas de configuración arriba del todo.

```bash
## Cargando el plugin de zsh-autocomplete
source /usr/share/zsh-autocomplete
git -C /usr/share/zsh-autocomplete pull &>/dev/null
```

Por último, cargaremos la configuración del `.zshrc` por comandos o *reiniciando la terminal* (recomendado).

```bash
source ~/.zshrc
```

Ahora, configuraremos la `powerlevel10k`. Para ello, como indica en su [repositorio oficial de GitHub](https://github.com/romkatv/powerlevel10k) ejecutaremos los siguientes comandos.

```bash
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ~/powerlevel10k
echo 'source /home/kali/powerlevel10k/powerlevel10k.zsh-theme' >> ~/.zshrc
```

De forma que, ejecutaremos el comando `zsh` y elegiremos las siguientes opciones de configuración. Estas opciones podemos cambiarlas si las deseamos ya que es una personalización personal.

```bash
## Opciones a seleccionar después de ejecutar el comando zsh en la terminal

y - y - y - 1 - y

2 - 1 - 2 - n - 1 - 3 - 4 - 1 - 2 - 2 - 2

y - 1 - y
```

Después, nos vamos al directorio `/home/kali/` con el comando `cd` y configuramos el fichero de configuración `.p10k.zsh`.

```bash
cd
nano .p10k.zsh
```

Comentaremos todo lo que se muestra en la parte derecha de la terminal, por lo que deberíamos de dejar la siguiente parte del fichero de la forma que se muestra a continuación.

```bash
  ## The list of segments shown on the right. Fill it with less important segments.
  ## Right prompt on the last prompt line (where you are typing your commands) gets
  ## automatically hidden when the input line reaches it. Right prompt above the
  ## last prompt line gets hidden if it would overlap with left prompt.
  typeset -g POWERLEVEL9K_RIGHT_PROMPT_ELEMENTS=(
##    status                  ## exit code of the last command
##    command_execution_time  ## duration of the last command
##    background_jobs         ## presence of background jobs
##    direnv                  ## direnv status (https://direnv.net/)
##    asdf                    ## asdf version manager (https://github.com/asdf-vm/asdf)
##    virtualenv              ## python virtual environment (https://docs.python.org/3/library/venv.html)
##    anaconda                ## conda environment (https://conda.io/)
##    pyenv                   ## python environment (https://github.com/pyenv/pyenv)
##    goenv                   ## go environment (https://github.com/syndbg/goenv)
##    nodenv                  ## node.js version from nodenv (https://github.com/nodenv/nodenv)
##    nvm                     ## node.js version from nvm (https://github.com/nvm-sh/nvm)
##    nodeenv                 ## node.js environment (https://github.com/ekalinin/nodeenv)
    ## node_version          ## node.js version
    ## go_version            ## go version (https://golang.org)
    ## rust_version          ## rustc version (https://www.rust-lang.org)
    ## dotnet_version        ## .NET version (https://dotnet.microsoft.com)
    ## php_version           ## php version (https://www.php.net/)
    ## laravel_version       ## laravel php framework version (https://laravel.com/)
    ## java_version          ## java version (https://www.java.com/)
    ## package               ## name@version from package.json (https://docs.npmjs.com/files/package.json)
##    rbenv                   ## ruby version from rbenv (https://github.com/rbenv/rbenv)
##    rvm                     ## ruby version from rvm (https://rvm.io)
##    fvm                     ## flutter version management (https://github.com/leoafarias/fvm)
##    luaenv                  ## lua version from luaenv (https://github.com/cehoffman/luaenv)
##    jenv                    ## java version from jenv (https://github.com/jenv/jenv)
##    plenv                   ## perl version from plenv (https://github.com/tokuhirom/plenv)
##    perlbrew                ## perl version from perlbrew (https://github.com/gugod/App-perlbrew)
##    phpenv                  ## php version from phpenv (https://github.com/phpenv/phpenv)
##    scalaenv                ## scala version from scalaenv (https://github.com/scalaenv/scalaenv)
##    haskell_stack           ## haskell version from stack (https://haskellstack.org/)
##    kubecontext             ## current kubernetes context (https://kubernetes.io/)
##    terraform               ## terraform workspace (https://www.terraform.io)
    ## terraform_version     ## terraform version (https://www.terraform.io)
##    aws                     ## aws profile (https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-profiles.html)
##    aws_eb_env              ## aws elastic beanstalk environment (https://aws.amazon.com/elasticbeanstalk/)
##    azure                   ## azure account name (https://docs.microsoft.com/en-us/cli/azure)
##    gcloud                  ## google cloud cli account and project (https://cloud.google.com/)
##    google_app_cred         ## google application credentials (https://cloud.google.com/docs/authentication/production)
##    toolbox                 ## toolbox name (https://github.com/containers/toolbox)
##    context                 ## user@hostname
##    nordvpn                 ## nordvpn connection status, linux only (https://nordvpn.com/)
##    ranger                  ## ranger shell (https://github.com/ranger/ranger)
##    yazi                    ## yazi shell (https://github.com/sxyazi/yazi)
##    nnn                     ## nnn shell (https://github.com/jarun/nnn)
##    lf                      ## lf shell (https://github.com/gokcehan/lf)
##    xplr                    ## xplr shell (https://github.com/sayanarijit/xplr)
##    vim_shell               ## vim shell indicator (:sh)
##    midnight_commander      ## midnight commander shell (https://midnight-commander.org/)
##    nix_shell               ## nix shell (https://nixos.org/nixos/nix-pills/developing-with-nix-shell.html)
##    chezmoi_shell           ## chezmoi shell (https://www.chezmoi.io/)
##    vi_mode                 ## vi mode (you don't need this if you've enabled prompt_char)
    ## vpn_ip                ## virtual private network indicator
    ## load                  ## CPU load
    ## disk_usage            ## disk usage
    ## ram                   ## free RAM
    ## swap                  ## used swap
##    todo                    ## todo items (https://github.com/todotxt/todo.txt-cli)
##    timewarrior             ## timewarrior tracking status (https://timewarrior.net/)
##    taskwarrior             ## taskwarrior task count (https://taskwarrior.org/)
##    per_directory_history   ## Oh My Zsh per-directory-history local/global indicator
    ## cpu_arch              ## CPU architecture
    ## time                  ## current time
    ## ip                    ## ip address and bandwidth usage for a specified network interface
    ## public_ip             ## public IP address
    ## proxy                 ## system-wide http/https/ftp proxy
    ## battery               ## internal battery
    ## wifi                  ## wifi speed
    ## example               ## example user-defined segment (see prompt_example function below)
  )
```

También, configuraremos algunas opciones adicionales para que nos salgan en la parte izquierda.

```bash
  ## The list of segments shown on the left. Fill it with the most important segments.
  typeset -g POWERLEVEL9K_LEFT_PROMPT_ELEMENTS=(
    os_icon                 ## os identifier
    dir                     ## current directory
    vcs                     ## git status
    context
    command_execution_time
    status 
    ## prompt_char           ## prompt symbol
  )
```

Haremos lo mismo para el usuario *root*:

```bash
sudo su
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ~/powerlevel10k
echo 'source /root/powerlevel10k/powerlevel10k.zsh-theme' >> ~/.zshrc
```

Ejecutamos el comando `zsh` y lo configuramos de la siguiente manera.

```bash
## Opciones a seleccionar después de ejecutar el comando zsh en la terminal

y - y - y - 1 - y

2 - 1 - 2 - n - 1 - 3 - 4 - 1 - 2 - 2 - 2

y - 1 - y
```

Volvemos a comentar todo lo que se muestra en la parte derecha de la terminal configurando el fichero `~/.p10k.zsh` de *root*.

```bash
  ## The list of segments shown on the right. Fill it with less important segments.
  ## Right prompt on the last prompt line (where you are typing your commands) gets
  ## automatically hidden when the input line reaches it. Right prompt above the
  ## last prompt line gets hidden if it would overlap with left prompt.
  typeset -g POWERLEVEL9K_RIGHT_PROMPT_ELEMENTS=(
##    status                  ## exit code of the last command
##    command_execution_time  ## duration of the last command
##    background_jobs         ## presence of background jobs
##    direnv                  ## direnv status (https://direnv.net/)
##    asdf                    ## asdf version manager (https://github.com/asdf-vm/asdf)
##    virtualenv              ## python virtual environment (https://docs.python.org/3/library/venv.html)
##    anaconda                ## conda environment (https://conda.io/)
##    pyenv                   ## python environment (https://github.com/pyenv/pyenv)
##    goenv                   ## go environment (https://github.com/syndbg/goenv)
##    nodenv                  ## node.js version from nodenv (https://github.com/nodenv/nodenv)
##    nvm                     ## node.js version from nvm (https://github.com/nvm-sh/nvm)
##    nodeenv                 ## node.js environment (https://github.com/ekalinin/nodeenv)
    ## node_version          ## node.js version
    ## go_version            ## go version (https://golang.org)
    ## rust_version          ## rustc version (https://www.rust-lang.org)
    ## dotnet_version        ## .NET version (https://dotnet.microsoft.com)
    ## php_version           ## php version (https://www.php.net/)
    ## laravel_version       ## laravel php framework version (https://laravel.com/)
    ## java_version          ## java version (https://www.java.com/)
    ## package               ## name@version from package.json (https://docs.npmjs.com/files/package.json)
##    rbenv                   ## ruby version from rbenv (https://github.com/rbenv/rbenv)
##    rvm                     ## ruby version from rvm (https://rvm.io)
##    fvm                     ## flutter version management (https://github.com/leoafarias/fvm)
##    luaenv                  ## lua version from luaenv (https://github.com/cehoffman/luaenv)
##    jenv                    ## java version from jenv (https://github.com/jenv/jenv)
##    plenv                   ## perl version from plenv (https://github.com/tokuhirom/plenv)
##    perlbrew                ## perl version from perlbrew (https://github.com/gugod/App-perlbrew)
##    phpenv                  ## php version from phpenv (https://github.com/phpenv/phpenv)
##    scalaenv                ## scala version from scalaenv (https://github.com/scalaenv/scalaenv)
##    haskell_stack           ## haskell version from stack (https://haskellstack.org/)
##    kubecontext             ## current kubernetes context (https://kubernetes.io/)
##    terraform               ## terraform workspace (https://www.terraform.io)
    ## terraform_version     ## terraform version (https://www.terraform.io)
##    aws                     ## aws profile (https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-profiles.html)
##    aws_eb_env              ## aws elastic beanstalk environment (https://aws.amazon.com/elasticbeanstalk/)
##    azure                   ## azure account name (https://docs.microsoft.com/en-us/cli/azure)
##    gcloud                  ## google cloud cli account and project (https://cloud.google.com/)
##    google_app_cred         ## google application credentials (https://cloud.google.com/docs/authentication/production)
##    toolbox                 ## toolbox name (https://github.com/containers/toolbox)
##    context                 ## user@hostname
##    nordvpn                 ## nordvpn connection status, linux only (https://nordvpn.com/)
##    ranger                  ## ranger shell (https://github.com/ranger/ranger)
##    yazi                    ## yazi shell (https://github.com/sxyazi/yazi)
##    nnn                     ## nnn shell (https://github.com/jarun/nnn)
##    lf                      ## lf shell (https://github.com/gokcehan/lf)
##    xplr                    ## xplr shell (https://github.com/sayanarijit/xplr)
##    vim_shell               ## vim shell indicator (:sh)
##    midnight_commander      ## midnight commander shell (https://midnight-commander.org/)
##    nix_shell               ## nix shell (https://nixos.org/nixos/nix-pills/developing-with-nix-shell.html)
##    chezmoi_shell           ## chezmoi shell (https://www.chezmoi.io/)
##    vi_mode                 ## vi mode (you don't need this if you've enabled prompt_char)
    ## vpn_ip                ## virtual private network indicator
    ## load                  ## CPU load
    ## disk_usage            ## disk usage
    ## ram                   ## free RAM
    ## swap                  ## used swap
##    todo                    ## todo items (https://github.com/todotxt/todo.txt-cli)
##    timewarrior             ## timewarrior tracking status (https://timewarrior.net/)
##    taskwarrior             ## taskwarrior task count (https://taskwarrior.org/)
##    per_directory_history   ## Oh My Zsh per-directory-history local/global indicator
    ## cpu_arch              ## CPU architecture
    ## time                  ## current time
    ## ip                    ## ip address and bandwidth usage for a specified network interface
    ## public_ip             ## public IP address
    ## proxy                 ## system-wide http/https/ftp proxy
    ## battery               ## internal battery
    ## wifi                  ## wifi speed
    ## example               ## example user-defined segment (see prompt_example function below)
  )
```

A continuación, comentamos la siguiente línea.

```bash
  ## Custom icon.
  ## typeset -g POWERLEVEL9K_CONTEXT_VISUAL_IDENTIFIER_EXPANSION='⭐'
  ## Custom prefix.
  typeset -g POWERLEVEL9K_CONTEXT_PREFIX=''
```

Y configuramos la siguiente línea de la manera de a continuación. Podemos cambiar el icono de la línea `typeset -g POWERLEVEL9K_CONTEXT_ROOT_TEMPLATE=` por el que queramos.

```bash
  ## Context format when running with privileges: bold user@hostname.
  typeset -g POWERLEVEL9K_CONTEXT_ROOT_TEMPLATE='ICONO'
  ## Context format when in SSH without privileges: user@hostname.
  typeset -g POWERLEVEL9K_CONTEXT_{REMOTE,REMOTE_SUDO}_TEMPLATE='%n@%m'
  ## Default context format (no privileges, no SSH): user@hostname.
  typeset -g POWERLEVEL9K_CONTEXT_TEMPLATE='%n@%m'
```

En cada uno de los usuarios (*root* y *kali*), configuraremos a `false` la siguiente directiva.

```bash
  ## Display anchor directory segments in bold.
  typeset -g POWERLEVEL9K_DIR_ANCHOR_BOLD=false
```

Ahora, configuraremos para que la `.zshrc` del usuario *root* sea la misma que la del usuario *kali*, por lo que vamos a crear un link simbólico.

```bash
sudo su
ln -s -f /home/kali/.zshrc /root/.zshrc
```

Vamos a configurar los permisos del fichero `/usr/local/share/zsh/site-functions/_bspc`, ya que tiene que ser propietario *root*.

```bash
chown root:root /usr/local/share/zsh/site-functions/_bspc
```

Seguidamente, cargamos los plugins que hemos instalado anteriormente, en la `~/.zshrc` del usuario **NO** privilegiado.

```bash
## ZSH Auto Complete
if [ -f /usr/share/zsh-autocomplete/zsh-autocomplete.plugin.zsh ]; then
	source /usr/share/zsh-autocomplete/zsh-autocomplete.plugin.zsh
fi

## ZSH Auto Suggestions
if [ -f /usr/share/zsh-autosuggestions/zsh-autosuggestions.zsh ]; then
	source /usr/share/zsh-autosuggestions/zsh-autosuggestions.zsh
fi

## ZSH Syntax Highlighting
if [ -f /usr/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh ]; then
	source /usr/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
fi
```

En mi caso, el plugin de *zsh-autocomplete* no lo voy a usar por lo que eliminaré el directorio `/usr/share/zsh-autocomplete/` y lo quitare del `~/.zshrc`. También, eliminaré las líneas del `~/.zshrc` de cada usuario que hacían referencia a la instalación (`source /usr/share/zsh-autocomplete` y `git -C /usr/share/zsh-autocomplete pull &>/dev/null`).

También, configuraremos el plugin `zsh-sudo`. Para ello, como *root*, ejecutamos los siguientes comandos.

```bash
sudo su
mkdir /usr/share/zsh-sudo
cd /usr/share/zsh-sudo
wget https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/refs/heads/master/oh-my-zsh.sh
```

Y lo metemos en el `/.zshrc` del usuario **NO** privilegiado.

```bash
## ZSH Sudo
if [ -f /usr/share/zsh-sudo/oh-my-zsh.sh ]; then
	source /usr/share/zsh-sudo/oh-my-zsh.sh
fi
```

Por último, desde el directorio `/usr/share/zsh-sudo`, nos traeremos los siguientes ficheros.

```bash
wget https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/refs/heads/master/tools/check_for_upgrade.sh
wget https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/refs/heads/master/lib/compfix.zsh
```

Y los meteremos en `tools/` y `lib/`.

```bash
mv check_for_upgrade.sh tools/
mv compfix.zsh lib/
```

También, añadiremos la siguiente línea para configurar un autocompletado moderno.

```bash
## Use modern completion system
autoload -Uz compinit
compinit

zstyle ':completion:*' auto-description 'specify: %d'
zstyle ':completion:*' completer _expand _complete _correct _approximate
zstyle ':completion:*' format 'Completing %d'
zstyle ':completion:*' group-name ''
zstyle ':completion:*' menu select=2
eval "$(dircolors -b)"
zstyle ':completion:*:default' list-colors ${(s.:.)LS_COLORS}
zstyle ':completion:*' list-colors ''
zstyle ':completion:*' list-prompt %SAt %p: Hit TAB for more, or the character to insert%s
zstyle ':completion:*' matcher-list '' 'm:{a-z}={A-Z}' 'm:{a-zA-Z}={A-Za-z}' 'r:|[._-]=* r:|=* l:|=*'
zstyle ':completion:*' menu select=long
zstyle ':completion:*' select-prompt %SScrolling active: current selection at %p%s
zstyle ':completion:*' use-compctl false
zstyle ':completion:*' verbose true

zstyle ':completion:*:*:kill:*:processes' list-colors '=(#b) #([0-9]#)*=0=01;31'
zstyle ':completion:*:kill:*' command 'ps -u $USER -o pid,%cpu,tty,cputime,cmd
```

## **Instalación de Batcat y Lsd**

A continuación, instalaremos `bat` y `lsd` desde los repositorios oficiales de cada uno, que son [bat](https://github.com/sharkdp/bat) y [lsd](https://github.com/lsd-rs/lsd/). Nos ponemos como un usuario **NO** privilegiado y lo descargamos en `~/Downloads/`.

```bash
cd ~/Downloads
wget https://github.com/sharkdp/bat/releases/download/v0.24.0/bat-musl_0.24.0_amd64.deb
wget https://github.com/lsd-rs/lsd/releases/download/v1.1.5/lsd-musl_1.1.5_amd64.deb
```

Ahora, nos ponemos como *root* y los instalamos.

```bash
sudo su
dpkg -i bat-musl_0.24.0_amd64.deb
dpkg -i lsd-musl_1.1.5_amd64.deb
```

Vamos a quitar la negrita a la hora de hacer un `ls`, por lo que vamos a modificar el fichero `~/.zshrc` como el usuario *kali* para añadir la línea que configure la variable `LS_COLORS`.

```bash
## LS_COLORS sin negrita (01;)
export LS_COLORS="rs=0:di=34:ln=36:mh=00:pi=40;33:so=35:do=35:bd=40;33;01:cd=40;33;01:or=40;31;01:mi=00:su=37;41:sg=30;43:ca=00:tw=30;42:ow=34;42:st=37;44:ex=32:*.tar=31:*.tgz=31:*.arc=31:*.arj=31:*.taz=31:*.lha=31:*.lz4=31:*.lzh=31:*.lzma=31:*.tlz=31:*.txz=31:*.tzo=31:*.t7z=31:*.zip=31:*.z=31:*.dz=31:*.gz=31:*.lrz=31:*.lz=31:*.lzo=31:*.xz=31:*.zst=31:*.tzst=31:*.bz2=31:*.bz=31:*.tbz=31:*.tbz2=31:*.tz=31:*.deb=31:*.rpm=31:*.jar=31:*.war=31:*.ear=31:*.sar=31:*.rar=31:*.alz=31:*.ace=31:*.zoo=31:*.cpio=31:*.7z=31:*.rz=31:*.cab=31:*.wim=31:*.swm=31:*.dwm=31:*.esd=31:*.avif=35:*.jpg=35:*.jpeg=35:*.mjpg=35:*.mjpeg=35:*.gif=35:*.bmp=35:*.pbm=35:*.pgm=35:*.ppm=35:*.tga=35:*.xbm=35:*.xpm=35:*.tif=35:*.tiff=35:*.png=35:*.svg=35:*.svgz=35:*.mng=35:*.pcx=35:*.mov=35:*.mpg=35:*.mpeg=35:*.m2v=35:*.mkv=35:*.webm=35:*.webp=35:*.ogm=35:*.mp4=35:*.m4v=35:*.mp4v=35:*.vob=35:*.qt=35:*.nuv=35:*.wmv=35:*.asf=35:*.rm=35:*.rmvb=35:*.flc=35:*.avi=35:*.fli=35:*.flv=35:*.gl=35:*.dl=35:*.xcf=35:*.xwd=35:*.yuv=35:*.cgm=35:*.emf=35:*.ogv=35:*.ogx=35:*.aac=00;36:*.au=00;36:*.flac=00;36:*.m4a=00;36:*.mid=00;36:*.midi=00;36:*.mka=00;36:*.mp3=00;36:*.mpc=00;36:*.ogg=00;36:*.ra=00;36:*.wav=00;36:*.oga=00;36:*.opus=00;36:*.spx=00;36:*.xspf=00;36:*~=00;90:*#=00;90:*.bak=00;90:*.crdownload=00;90:*.dpkg-dist=00;90:*.dpkg-new=00;90:*.dpkg-old=00;90:*.dpkg-tmp=00;90:*.old=00;90:*.orig=00;90:*.part=00;90:*.rej=00;90:*.rpmnew=00;90:*.rpmorig=00;90:*.rpmsave=00;90:*.swp=00;90:*.tmp=00;90:*.ucf-dist=00;90:*.ucf-new=00;90:*.ucf-old=00;90:"
```

También, vamos a crear unos aliases para poder ejecutar y `cat` y `lsd`.

```bash
## Custom Aliases
#------------------------------------
## bat
alias cat='bat'
alias catn='bat --style=plain'
alias catnp='bat --style=plain --paging=never'

## ls
alias ll='lsd -lh --group-dirs=first'
alias la='lsd -a --group-dirs=first'
alias l='lsd --group-dirs=first'
alias lla='lsd -lha --group-dirs=first'
alias ls='lsd --group-dirs=first'
```

En este punto, vamos a arreglar los posibles problemas que nos puede dar *BurpSuite*.

>**Blog para arreglar los problemas de BurpSuite**<br>
>https://medium.com/@fidjolakoka/burpsuite-no-longer-launches-after-parrot-upgrade-d1c6b17cb70d

Vamos a seguir sus pasos. Primeramente, nos vamos al directorio de `Downloads/` como un usuario **NO** privilegiado y ejecutamos los siguientes comandos.

```bash
wget https://download.java.net/java/GA/jdk21.0.2/f2283984656d49d69e91c558476027ac/13/GPL/openjdk-21.0.2_linux-x64_bin.tar.gz
tar xvf openjdk-21.0.2_linux-x64_bin.tar.gz
sudo mv jdk-21.0.2/ /usr/lib/jvm/jdk-21
```

Actualizamos la configuración de *BurpSuite* con el comando `geany`. Si no lo tenemos instalado, lo instalaremos.

```bash
sudo apt install geany -y
```

Ejecutamos el comando de `geany` para sustituir toda la ruta de `/usr/lib/jvm/java-17-openjdk-$(dpkg-architecture -q DEB_HOST_ARCH)/bin/java` por la línea `/usr/lib/jvm/jdk-21/bin/java`.

```bash
sudo geany /usr/bin/burpsuite
```

El fichero debería de quedar de la siguiente manera.
```bash
#!/usr/bin/env sh

set -e

export JAVA_CMD=/usr/lib/jvm/jdk-21/bin/java

## Include the wrappers utility script
. /usr/lib/java-wrappers/java-wrappers.sh

run_java -jar /usr/share/burpsuite/burpsuite.jar "$@"
```

Una vez configurado, lo guardamos seleccionando `File + Save`. Pero, si lo abrimos, veremos cómo se nos abre en una esquina sin llegar a abrirlo en tamaño completo. Para arreglar esto, vamos a meter el siguiente contenido en el fichero `~/.zshrc` del usuario *kali*.

```bash
## Fix the Java Problem
export _JAVA_AWT_WM_NONREPARENTING=1
```

Y ahora, editaremos el fichero `~/.config/bspwm/bspwmrc`.

```bash
wmname LG3D &
```

Seguidamente, hacemos un `super + shift + q` para volver a iniciar sesión y que se apliquen los cambios. De esta manera, ya podemos abrir *BurpSuite* perfectamente sin fallos.

Ahora, vamos a hacer que funcione correctamente si lo abrimos desde `rofi`. Para ello, nos vamos a cualquier ruta del `$PATH` como `/usr/bin` y creamos el fichero `burpsuite-launcher`.

```bash
sudo su
cd /usr/bin
touch burpsuite-launcher
```

El contenido de este fichero `burpsuite-launcher` es el siguiente.

```bash
#!/bin/bash

## Fix the Java Problem
export _JAVA_AWT_WM_NONREPARENTING=1

vmname LG3d &
/usr/bin/burpsuite
```

Con esto, podremos ejecutar *BurpSuite* correctamente desde `rofi` seleccionando `burpsuite-launcher`.

## **Configurando y creando nuevos módulos en la Polybar**

Dentro del directorio `~/.config/` del usuario *kali* tenemos un directorio `polybar`.

```bash
cd ~/.config/polybar
```

El fichero `launch.sh` definen los elementos / barras que estamos cargando y, con el fichero `current.ini` podemos manejar las dimensiones y posición de las barras que hemos creado en el `launch.sh`. Introduciremos el contenido de los ficheros de configuración completos. Podemos copiar este contenido desde [launch.sh](https://github.com/h3g0c1v/h3g0c1v.github.io/blob/main/assets/resources/launch.sh.md) y [current.ini](https://github.com/h3g0c1v/h3g0c1v.github.io/blob/main/assets/resources/current.ini.md).

Una vez que hemos copiado el contenido de `launch.ini` y `current.ini`, crearemos una carpeta `~/.config/scripts` con los siguientes scripts.

- [ethernet_status.sh](https://github.com/h3g0c1v/h3g0c1v.github.io/blob/main/assets/resources/ethernet_status.sh.md) ➜ Muestra la interfaz eth0 en la polybar.
- [ethernet2_status.sh](https://github.com/h3g0c1v/h3g0c1v.github.io/blob/main/assets/resources/ethernet2_status.sh.md) ➜ Muestra la interfaz eth1 en la polybar.
- [change_ethernet.sh](https://github.com/h3g0c1v/h3g0c1v.github.io/blob/main/assets/resources/change_ethernet.sh.md) ➜ Con el shortcut `alt + shift + c` cambiará que interfaz nos muestra (eth0 o eth1) cambiando el contenido del fichero `ethernet_status.sh` por el de `ethernet2_status.sh`
- [interface_active.sh](https://github.com/h3g0c1v/h3g0c1v.github.io/blob/main/assets/resources/interface_active.sh.md) ➜ Muestra si estamos con la eth0 (icono de un pc) o con la eth1(icono de windows / ventanas).
- [vpn_status.sh](https://github.com/h3g0c1v/h3g0c1v.github.io/blob/main/assets/resources/vpn_status.sh.md) ➜ Muestra la VPN a la que estamos conectados (tun0 o tun1).
- [victim_to_hack.sh](https://github.com/h3g0c1v/h3g0c1v.github.io/blob/main/assets/resources/victim_to_hack.sh.md) ➜ Muestra el target a hackear. Se cambia con `settarget` y se borra con `cleartarget`.

Una vez que tengamos esto en la carpeta de scripts les daremos permisos de ejecución a todos.

```bash
chmod 755 ~/.config/scripts/*
```

A continuación, vamos a configurar el shortcut `alt + shift + c` par que funcione correctamente el script de `change_ethernet.sh`. Para ello, nos vamos a `~/.config/sxhkd/sxhkdrc` y metemos lo siguiente.

```bash
## Cambiar para ver la IP eth0 o la eth1
alt + shift + c
    /home/kali/.config/scripts/change_ethernet.sh
```

Después, configuraremos las siguientes funciones `settarget` y `cleartarget` en la `~/.zshrc` para que funcione correctamente la polybar de `victim_to_hack.sh`.

```bash
## Configurar target
function settarget(){
    ip_address=$1
    machine_name=$2
    echo "$ip_address $machine_name" > /home/kali/.config/bin/target
}

## Limpiar el target
function cleartarget(){
    echo '' > /home/kali/.config/bin/target
}
```

Ahora, crearemos el directorio `~/.config/bin` para que se pueda crear el fichero `target`.

```bash
mkdir ~/.config/bin/
```

Por último, vamos a modificar el fichero `~/.config/polybar/workspace.ini` para modificar los colores de la barra de ventanas.

```bash
label-active-foreground = ${color.red}
label-active-background = ${color.bg}

label-occupied = "%icon% "
label-occupied-foreground = ${color.yellow}
label-occupied-background = ${color.bg}
```

## **Instalando fzf**

Para instalar `fzf`, nos iremos al [repositorio oficial de GitHub](https://github.com/junegunn/fzf?tab=readme-ov-file#using-git) y como indican, ejecutaremos los siguientes comandos tanto como el usuario **NO** privilegiado como con el *privilegiado*.

```bash
git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf
~/.fzf/install

sudo su
git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf
~/.fzf/install
```

Las opciones de cada uno son las de por defecto, por lo que le daremos `ENTER` a las tres opciones que te preguntan.

## **Configuración e integración de NVchad en Neovim**

Nos ponemos como un usuario **NO** privilegiado y nos vamos al `/home/kali/`. Ahora, seguiremos los pasos que indican en la instalación del [repositorio oficial de GitHub](https://github.com/NvChad/NvChad) aunque con algunos cambios.

```bash
cd
git clone https://github.com/NvChad/starter ~/.config/nvim
```

Ahora, nos descargamos el último release de `nvim` desde el [repo de GitHub](https://github.com/NvChad/NvChad) en el directorio `/opt/nvim`

```bash
sudo mkdir /opt/nvim
cd /opt/nvim
sudo wget https://github.com/neovim/neovim/releases/download/v0.10.2/nvim-linux64.tar.gz
sudo tar -xf nvim-linux64.tar.gz
```

Deberemos de contemplar en el *$PATH* la ruta `/opt/nvim/nvim-linux64/bin` que es donde se encuentra el binario de `nvim`. Para ello, en caso de no tener ninguna ruta configurada en el `~/.zshrc` que defina el *PATH* la crearemos nosotros.

```bash
## Export PATH
export PATH=$PATH:/opt/nvim/nvim-linux64/bin
```

A continuación, abriremos una nueva ventana en la consola y ejecutamos el comando `nvim`.

```bash
nvim
```

Una vez terminada la instalación, nos salimos con `:q!`(puede que tengamos que ejecutar ese comando *dos o tres veces* hasta salir definitivamente).

Finalizando, vamos a irnos al fichero de configuración `~/.config/nvim/init.lua` e introduciremos debajo de las dos primeras líneas `vim.opt.listchars = "tab:»·,trail:·"`. También, podemos copiarlo del [pastebin](https://pastebin.com/Lbk9vQLZ).

```bash
vim.g.base46_cache = vim.fn.stdpath "data" .. "/base46/"
vim.g.mapleader = " "

vim.opt.listchars = "tab:»·,trail:·" ## Esta es la línea que hemos añadido

-- bootstrap lazy and all plugins
```

Ahora, ejecutaremos el comando `updatedb`.

```bash
updatedb
```

Después, vamos a abrir cualquier fichero `.lua` y ejecutaremos el comando `:MasonInstallAll` para evitar que nos den errores a la hora de abrir ciertos tipos de archivos como los `.lua`.

```bash
:MasonInstallAll
```

Realizaremos estos mismos pasos de integración y configuración de `nvchad` con el usuario *root*.

## **Configurando tema para rofi**

Nos vamos a ir a `/opt` y nos clonaremos el siguiente [repositorio](https://github.com/newmanls/rofi-themes-collection).

```bash
cd /opt
sudo git clone https://github.com/newmanls/rofi-themes-collection
```

Seguidamente, en caso de que no esté creado, nos crearemos el directorio `~/.config/rofi/themes`.

```bash
mkdir -p ~/.config/rofi/themes
```

Y copiaremos todo el contenido de `/opt/rofi-themes-collection/themes` en `~/.config/rofi/themes`.

```bash
cp /opt/rofi-themes-collection/themes/* ~/.config/rofi/themes
```

Ejecutamos el comando `rofi-themes-selector` y seleccionamos el tema que más nos guste. Para ir viendo cada uno, nos iremos moviendo con las flechas de arriba y abajo y dándole a `ENTER` en el tema que queramos ver. En mi caso, el que más me gustó fue el `rounded-green-dark`. Una vez que tengamos el tema que nos gusta seleccionado, lo configuraremos con la combinación de teclas `ALT + a`.

```bash
ALT + a
```

## **Configurando certificado de BurpSuite**

Por último, configuraremos el certificado de *BurpSuite*. Para ello, abrimos BurpSuite y configuramos el proxy utilizando una extensión como FoxyProxy o directamente configurándolo en el navegador. Una vez hecho esto, accedemos a la siguiente URL para continuar con el proceso.

>**URL para descargar el certificado de BurpSuite**<br>
>http://burp/

Nos descargamos el certificado desde el botón de *Ca Certificate*.

![Descargandonos el certificado de BurpSuite](/assets/img/descargandonos-el-certificado-de-burpsuite.png)

Y ahora lo importamos.

![Importando el certificado de BurpSuite](/assets/img/importando-el-certificado-de-burpsuite.png)

Seleccionamos la siguiente opción y le damos a *Aceptar*.

![Confiar en esta CA de BurpSuite para identificar sitios web](assets/img/confiar-en-esta-ca-de-burpsuite-para-identificar-sitios-web.png)

## **Comandos Nvchad**

Con `nvchad`, podemos configurar *Temas*. Ejecutaremos la combinación de teclas `ESC + ESPACIO + th` y seleccionaremos el tema deseado.

```bash
ESC + ESPACIO + th
```

Con `CTRL + n` podemos abrir una barra lateral que nos permita movernos entre directorios.

```bash
CTRL + n
```

Podemos buscar archivos dentro de nuestro directorio actual de trabajo con `ESC + ESPACIO + ff`.

```bash
ESC + ESPACIO + ff
```

Si queremos ver el mismo archivo pero para comparar diferentes lugares de este y no perder el punto de vista actual, podemos ejecutar el comando `ESC + :vsp`  o `ESC + :sp` y nos creará una copia idéntica con la que podemos movernos.

```bash
ESC + :vsp ## Nos lo abre a la derecha
ESC + :sp ## Nos lo abre debajo
```

Si queremos ver el *CheatSheet* del propio `nvchad`, podemos efectuar la combinatoria `ESC + ESPACIO + ch`.

```bash
ESC + ESPACIO + ch
```

## **Borrando el contenido de descargas**

Eliminamos todo el contenido de descargas y hemos terminado de personalizar el laboratorio

```bash
rm -rf ~/Downloads
```

## **Créditos**

Quiero agradecer a **Marcelo Vázquez, aka s4vitar** quien ha configurado el entorno y me ha dado permiso para publicar este blog. **Marcelo Vázquez** es CEO y fundador de la academia de hacking ético *[Hack4u](https://hack4u.io)*, pentester, YouTuber, Streamer y un gran ejemplo a seguir. A continuación, os comparto sus redes sociales para que podáis mostrarle vuestro apoyo.

- [![LinkedIn](https://img.shields.io/badge/LinkedIn-%230077B5.svg?logo=linkedin&logoColor=white)](https://www.linkedin.com/in/s4vitar/)
- [![Github](https://img.shields.io/badge/Github-000?style=rounded-square&logo=Github&logoColor=white&link=https://github.com/h3g0c1v)](https://github.com/s4vitar)
- [![Canal Principal de YouTube](https://img.shields.io/badge/-Canal%20Principal%20de%20YouTube-F00?&logo=youtube&labelColor=F00)](https://www.youtube.com/@s4vitar)
- [![Canal Secundario de YouTube](https://img.shields.io/badge/-Canal%20Secundario%20de%20YouTube-F00?&logo=youtube&labelColor=F00)](ttps://www.youtube.com/@S4viOnLive)
- [![Twitch](https://img.shields.io/badge/Twitch-9347FF?logo=twitch&logoColor=white)](https://www.twitch.tv/s4vitaar)

## **Despedida**
Espero que con este entorno os sintáis más cómodos y mejoréis vuestra eficiencia. Muchas gracias por leer este blog. ¡Nos vemos en el siguiente!

![Entorno finalizado](/assets/img/entorno-finalizado.png)
