# Corralito : Versión de Firewarden en castellano

Firewarden es un guión en bash que se usa para abrir un programa o una dirección 
web adentro de un arenero privado [Firejail][1]. Por eso, elegí traducirlo como 
'corralito', que hace juego con 'arenero'. Mientras que firewarden debería 
traducirse como guardabosque, lo que confunde un poco. 

La utilidad de mi traducción estimo que tiene una duración tan breve como la que 
tarde el desarrollador en encontrar la forma de traducir la salida esperada por 
el administrador de redes al idioma del usuario. En este caso, lo que espera leer 
en la salida de NetworkManager es "connected" lo que en mi equipo, nunca ocurrirá, 
porque la salida está traducida como "conectado". Igual que en cualquier sistema 
operativo Linux en castellano...

Igual, es una oportunidad para todos los que quieran acostumbrarse a usar FireJail 
en castellano. Firewarden es mucho mas simple de optimizar. Y es muy poco lo que 
hay que aprender a usar... Voy a reemplazar en este documento, donde dice Firewarden 
por Corralito, pero entiéndase que solo uso este nombre al llamar a mi guión. 
Internamente, se conserva el nombre original Firewarden. Incluso al crear el directorio 
virtual (aunque debería llamarlo FireJail)...

Corralito (Firewarden) lanzará el programa esperado, usando un arenero Firejail con 
un directorio home privado en un sistema temporario. La red está activada predeterminadamente,
aunque puede desactivarse.

Este guion puede resultar util con muchas aplicaciones, aunque se creó específicamente para 
usar Chromium. <(n.t.)> Hay webs de video que no funcionan en otros navegadores.
Puede usarse para visitar un sitio "sospechoso" ( e.i. [shady site][2]  ) dentro
de un arenero aislado y temporario -- o simplemente para proteger nuestro banco
web -- solo hay que agregarle a nuestro comando habitual la palabra 'corralito'
(<(n.t.)> contando con que hemos puesto el guión en un directorio con derechos de ejecutable):

    $ corralito chromium http://www.forbes.com

Al usar corralito (firewarden) para lanzar `chromium` o `google-chrome`, el guión evitará
el saludo al nuevo usuario, deshabilitará la comprobación de navegador privilegiado y 
evitará filtraciones de IP WebRTC ( e.i. [WebRTC IP leak][3] ). 

## Archivos Locales

Cuando el último argumento parece ser un archivo local, corralito (firewarden) 
copiará el archivo en un directorio temporario. Este directorio lo usará Firejail
como home de usuario. Al cerrar el programa, tanto el directorio temporal como 
todo su contenido se borrarrán.

La red estará desactivada, por omisión. Y para usar el archivo local, también se
creará un `/dev` privado.

Esto es especialmente útil, para mitigar el daño que podría causar la apertura de 
archivos potencialmente maliciosos. Tales como PDFs o JPGs. Asócielo como mailcap
(n.t. Mime Type o tipo de archivo de apertura) para proteger sl sistema contra 
archivos adjunto sospechosos en nuestros emails.

Por ejemplo, si queremos ver un archivo `~/notatrap.pdf` con el lector de PDF `zatura`...
   
    $ corralito zathura ~/notatrap.pdf

Es lo mismo que si hiciéramos lo siguiente:

    $ export now=`date --iso-8601=s`
    $ mkdir -p $XDG_RUNTIME_DIR/$USER/firewarden/$now
    $ cp ~/notatrap.pdf $XDG_RUNTIME_DIR/$USER/firewarden/$now/
    $ firejail --net=none --private-dev --private=$XDG_RUNTIME_DIR/$USER/firewarden/$now zathura notatrap.pdf
    $ rm -r $XDG_RUNTIME_DIR/$USER/firewarden/$now

## Opciones

### Redes

Predeterminadamente, las redes están habilitadas, a menos que queramos usar un archivo
local (que en la mayoría de los casos, no necesita acceder a ninguna red).

El usuario puede explícitamente habilitar o deshabilitar el acceso a redes, desestimando
el comportamiento por omisión.

    # (-n) negando el acceso a redes, desestimando su comportamiento por omisión.
    $ corralito -n ...
    # (-N) habilitando el acceso a redes, desestimando su comportamiento por omisión.
    $ corralito -N ...

Opcionalmente, el arenero puede activarse con un nombre-espacio de red aislado
y restringido. A menos que se declare lo contrario, se usará [NetworkManager][4]
para conocer el primer interfaz de red conectado. Este interfaz se usará para 
crear el nuevo nombre-espacio de red.

    # aislar la red usando el primer interfaz conectado.
    $ corralito -i ...
    # o, aislar la red, usando el interfaz ya conectado.
    $ corralito -I eth0 ...

When isolating the network, Firejail's default client network filter will be
used in the new network namespace.

```
*filter
:INPUT DROP [0:0]
:FORWARD DROP [0:0]
:OUTPUT ACCEPT [0:0]
-A INPUT -i lo -j ACCEPT
-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
# allow ping
-A INPUT -p icmp --icmp-type destination-unreachable -j ACCEPT
-A INPUT -p icmp --icmp-type time-exceeded -j ACCEPT
-A INPUT -p icmp --icmp-type echo-request -j ACCEPT
# drop STUN (WebRTC) requests
-A OUTPUT -p udp --dport 3478 -j DROP
-A OUTPUT -p udp --dport 3479 -j DROP
-A OUTPUT -p tcp --dport 3478 -j DROP
-A OUTPUT -p tcp --dport 3479 -j DROP
COMMIT

```


### /dev

Optionally, a new `/dev` can be created to further restrict the sandbox. This
has the effect of preventing access to audio input and output, as well as any
webcams. It is enabled by default when viewing local files.

    # create a private /dev, regardless of the defaults.
    $ firewarden -d ...
    # do not create a private /dev, regardless of the defaults.
    $ firewarden -D ...

## Examples

    $ firewarden -d -i chromium https://www.nsa.gov/ia/ &
    $ firewarden zathura /mnt/usb/nsa-ant.pdf &
    $ firewarden chromium https://www.youtube.com/watch?v=bDJb8WOJYdA &
    $ firejail --list
    630:pigmonkey:/usr/bin/firejail --private --net=enp0s25 --netfilter --private-dev chromium --no-first-run --no-default-browser-check https://www.nsa.gov/ia/
    31788:pigmonkey:/usr/bin/firejail --private=/run/user/1000/firewarden/2016-01-31T16:09:14-0800 --net=none --private-dev zathura nsa-ant.pdf
    32255:pigmonkey:/usr/bin/firejail --private chromium --no-first-run --no-default-browser-check https://www.youtube.com/watch?v=bDJb8WOJYdA


[1]: https://github.com/netblue30/firejail
[2]: http://www.engadget.com/2016/01/08/you-say-advertising-i-say-block-that-malware/
[3]: https://www.privacytools.io/webrtc.html
[4]: https://wiki.gnome.org/Projects/NetworkManager
