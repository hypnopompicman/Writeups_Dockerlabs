# firsthacking

* Máquina **atacante** -> 172.17.0.1
* Máquina **objetivo** -> 172.17.0.2 | Linux :penguin:

### Escaneo de puertos

Realizamos un escaneo de puertos con *nmap* para ver puertos abiertos de la máquina **objetivo**:
```shell
nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn 172.17.0.2
```
![escaneo](/Writeups_Dockerlabs/Capturas/2024-10-23_22-03.png)

Observamos que el servicio **ftp** que corre por el puerto **21** posee una versión con una vulnerabilidad conocida.

### Explotación

Vamos a *metasploit* y buscamos la vulnerabilidad por la versión `search vsftpd 2.3.4`:

![busqueda](/Writeups_Dockerlabs/Capturas/2024-10-23_22-19.png)

La seleccionamos (`use`) y vemos los parámetros que tenemos que completar (`show options`):

![opciones](/Writeups_Dockerlabs/Capturas/2024-10-23_22-21.png)

Completamos el host remoto al que nos queremos conectar, seleccionamos el payload automático por defecto (cmd/unix/interact) y ejecutamos el **exploit**:

* set RHOSTS 172.17.0.2
* use 0
* exploit

Nos abre una sesión y obtenemos una shell ejecutando el comando `shell`:

![shell](/Writeups_Dockerlabs/Capturas/2024-10-23_22-29.png)

Si ejecutamos el comando `whoami`, vemos que somos **root** :triangular_flag_on_post:

![whoami](/Writeups_Dockerlabs/Capturas/2024-10-23_22-31.png)