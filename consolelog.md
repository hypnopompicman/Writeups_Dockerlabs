# consolelog

* Máquina **atacante** -> 172.17.0.1
* Máquina **objetivo** -> 172.17.0.2 | Linux :penguin:

### Escaneo de puertos

Realizamos un escaneo de puertos con *nmap* para ver puertos abiertos de la máquina **objetivo**:
```shell
nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn 172.17.0.2
```
![escaneo](/Capturas/2024-11-30_13-41.png)

Encontramos tres puertos abiertos, el puerto **80** con un apache, el puerto **3000** con un Node y el puerto **5000** con un servicio ssh corriendo.

### HTTP

Si nos dirigimos al navegador a ver qué tenemos, nos encontramos simplemente con esto:

![bienvenidoamisitio](/Capturas/2024-11-30_13-47.png)

Donde el "Boton en fase beta" no parece realizar ninguna acción.

Al comprobar el código fuente:

![cfuente](/Capturas/2024-11-30_13-48.png)

Observamos que el "Boton en fase beta" llama a la función **autenticate()**, que cuando entramos en ella nos da una pista:

![tokentraviesito](/Capturas/2024-11-30_13-50.png)

### Fuzzing web

Para poder seguir investigando, realizamos **fuzzing web** con *gobuster*:
```shell
gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x .txt,.py,.php,.sh
```
![fuzzing](/Capturas/2024-11-30_13-52.png)

Encontramos interesante el directorio **/backend**, así que procedemos a investigarlo:

![backend](/Capturas/2024-11-30_13-54.png)

Si nos vamos al archivo **server.js**, observamos el código que comprueba si autoriza al usuario o no. Cuando lo autoriza, usa una respuesta que parece ser una contraseña:

![passwd](/Capturas/2024-11-30_13-56.png)

Por lo que debemos intentar usarla, para entrar vía **ssh**.

### Fuerza bruta

Usamos *hydra* para probar la contraseña encontrada intentado entrar por **ssh**:
```shell
hydra -L /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt -p lapassworddebackupmaschingonadetodas ssh://172.17.0.2:5000
```
![fb](/Capturas/2024-11-30_14-04.png)

Obtenemos éxito con las credeneciales:
* Usuario: lovely
* Contraseña: lapasswordmaschingonadetodas

### Intrusión

Entramos vía **SSH** por el puerto que está corriendo, diferente al habitual:
```shell
ssh lovely@172.17.0.2 -p 5000
```
![intrusion](/Capturas/2024-11-30_14-09.png)

Y estamos dentro.

### Escalada de privilegios

Introducimos el comando `sudo -l` para ver si podemos ejecutar algún comando como otro usuario:

![sudo-l](/Capturas/2024-11-30_14-11.png)

Tenemos permisos para ejecutar *nano* como **root**, por lo que lo ejecutamos con un `sudo nano` y tecleamos **Ctr +R** seguido de **Ctrl + X** para que se nos despligue un apartado donde ejecutar comandos:

![ctrlryx](/Capturas/2024-11-30_14-17.png)

A continuación, ejecutamos el comando `reset; sh 1>&0 2>&0` para escalar a **root**:

![escal](/Capturas/2024-11-30_14-20.png)

Nos lanzamos una terminal interactiva con `bash -p` y si hacemos un `whoami`:

![whoami](/Capturas/2024-11-30_14-22.png)

Somos usuario **root** :triangular_flag_on_post: