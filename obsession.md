# obsession

* Máquina **atacante** -> 172.17.0.1
* Máquina **objetivo** -> 172.17.0.2 | Linux :penguin:

### Escaneo de puertos

Realizamos un escaneo de puertos con *nmap* para ver puertos abiertos de la máquina **objetivo**:
```shell
nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn 172.17.0.2
```
![scaneo](/Capturas/2024-10-23_23-17.png)

Encontramos un servicio **ftp** con la vulnerabilidad **anonymous** activa, un servicio **ssh** y un **apache**.

### HTTP

Vamos al navegador y buscamos lo que hay corriendo por el puerto **80**. Vemos una página web, y al consultar el **código fuente**, encontramos un comentario que nos desvela información interesante:

![infosensible](/Capturas/2024-10-23_23-21.png)

### FTP

Entramos al servicio con **anonymous** y descargamos los archivos existentes en busca de más información:

![archivos en ftp](/Capturas/2024-10-23_23-25.png)

Al leer los archivos descargados, junto con la información extraída del comentario en el código fuente de la web, deducimos que el usuario de la máquina **objetivo** debe ser **russoski**.

### Intrusión

Realizamos un ataque de **fuerza bruta** con *hydra* con el usuario **russoski** mediante el servicio **ssh**:

![intrusion](/Capturas/2024-10-23_23-29.png)

Obtenemos la contraseña del usuario **russoski** y procedemos a conectarnos a la máquina **objetivo** mediante **ssh**:

![connect ssh](/Capturas/2024-10-23_23-30.png)

### Escalada de privilegios

Una vez dentro, ejecutamos el comando `sudo -l` y vemos que podemos ejecutar *vim* como **root**, por lo que usamos el comando `sudo /usr/bin/vim -c ':!/bin/sh'` para escalar privilegios:

![escalada](/Capturas/2024-10-23_23-33.png)

Y somos **root** :triangular_flag_on_post: