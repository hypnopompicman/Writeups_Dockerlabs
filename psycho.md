# Psycho

* Máquina **atacante** -> 172.17.0.1
* Máquina **objetivo** -> 172.17.0.2 | Linux :penguin:

### Escaneo de puertos

Realizamos un escaneo de puertos con *nmap* para ver puertos abiertos de la máquina **objetivo**:
```shell
nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn 172.17.0.2
```
![escaneo](/Capturas/2025-01-08_13-28.png)

Tenemos abiertos los puertos **22** y **80** de la máquina **objetivo**.

Al investigar la página web que corre por el puerto **80**, lo único que nos llama la atención es el *[!]ERROR[!]* que observamos en la línea 63. Este error parece ser generado por una llamada que obtiene respuesta, por lo que podemos aprovecharla para ver si existe la vulnerabilidad **LFI**.

### Local File Inclusion

Explotar esta vulnerabilidad nos dará acceso a archivos internos de la máquina **objetivo**. Para comprobar si existe dicha vulnerabilidad, vamos a utilizar la herramienta *wfuzz*:
```shell
wfuzz -c --hc 404 --hl 62 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -u http://172.17.0.2/index.php?FUZZ=../../../../../../etc/passwd
```
![secret](/Capturas/2025-01-08_13-38.png)

Parece ser que existe dicha vulnerabilidad, si en el navegador usamos la url http://172.17.0.2/index.php?secret= seguido de la ruta a la que queramos acceder, podremos ver los archivos internos de la máquina **objetivo**.

Por lo que accedemos al archivo **/etc/passwd** y descubrimos que existen dos usuarios: **vaxei** y **luisillo**:

![passwd](/Capturas/2025-01-08_13-49.png)

### Id_rsa

Intentamos realizar un ataque de fuerza bruta con *hydra* de ambos usuarios sin éxito, por lo que buscamos llaves **rsa** de los usuarios para el acceso vía ssh sin uso de contraseñas.

Encontramos, la clave **rsa** del usuario **vaxei**:

![rsa_vaxei](/Capturas/2025-01-08_13-53.png)

Ahora simplemente, copiamos la llave en un archivo, le otorgamos permisos y probamos a iniciar sesión como **vaxei**:

![id_rsa_vaxei](/Capturas/2025-01-08_13-55.png)
![iniciovaxei](/Capturas/2025-01-08_13-56.png)

### Escalada de privilegios

Una vez dentro como el usuario **vaxei**, al ejecutar el comando `sudo -l`, vemos que podemos ejecutar **perl** como el usuario **luisillo**:

![luisillo](/Capturas/2025-01-08_14-16.png)

Si volvemos a ejecutar `sudo -l`, nos percatamos de que podemos ejecutar **paw.py** como **root**. Por lo que procedemos a ver ese archivo.py.

![sudo-l](/Capturas/2025-01-08_14-20.png)
![paw](/Capturas/2025-01-08_14-21.png)

Aquí lo interesante no es el archivo.py en sí, sino que en la carpeta en la que se encuentra el archivo, tenemos permisos para hacer lo que queramos. Por lo que borraremos el archivo y crearemos un pequeño script con *python* que nos cambie los permisos de la bash para poder escalar privilegios a **root**.

![archivonuevo](/Capturas/2025-01-08_14-26.png)

Ejecutamos el **script** y nos lanzamos una shell como **root** :triangular_flag_on_post:

![root](/Capturas/2025-01-08_14-28.png)