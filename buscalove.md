# buscalove

* Máquina **atacante** -> 172.18.0.1
* Máquina **objetivo** -> 172.18.0.2 | Linux :penguin:

### Escaneo de puertos

Realizamos un escaneo de puertos con *nmap* para ver puertos abiertos de la máquina **objetivo**:
```shell
nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn 172.18.0.2
```
![escaneo](/Capturas/2024-11-11_21-24.png)

Encontramos los puertos **80** y **22** abiertos.

### Fuzzing web

Como no vemos nada más allá de una plantilla básica de apache:

![plantilla_apache](/Capturas/2024-11-11_21-29.png)

Procedemos a realizar **fuzzing web** para encontrar directorios ocultos:
```shell
gobuster dir -u http://172.18.0.2/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x .txt,.py,.php,.sh
```
![fuzzing](/Capturas/2024-11-11_21-31.png)

Observamos que exite el directorio `/wordpress` y dentro solamente `index.php`. Al observar la página web, solamente nos topamos con un comentario interesante en el código fuente:

![c_fuente](/Capturas/2024-11-11_21-34.png)

Aunque luego veremos que se trata de una pista, en el momento no me sirvió.

### Local File Inclusion

En cambio, cuando decidimos comprobar si existe la vulnerabilidad de **Local File Inclusion** con *wfuzz* podemos observar, que la pista cobra sentido:
```shell
wfuzz -c --hc 404 --hl 40 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -u http://172.18.0.2/wordpress/index.php?FUZZ=../../../../../../etc/passwd
```
![fuzz](/Capturas/2024-11-11_21-43.png)

Vemos que con la palabra **love** podemos realizar un **LFI** y ver el contenido del fichero `/etc/passwd`. Así que procedemos a comprobarlo y resulta que existen dos usuarios:

![passwd](/Capturas/2024-11-11_21-45.png)

**pedro** y **rosa**.

### Fuerza bruta

Probamos a realizar un ataque de **fuerza bruta** con *hydra* a los usuarios encontrados vía **ssh**:
```shell
hydra -l rosa -P /usr/share/wordlists/rockyou.txt ssh://172.18.0.2
```
![fb](/Capturas/2024-11-11_21-49.png)

Encontramos que el usuario **rosa** posee la contraseña **lovebug**.

### Escalada de privilegios

Entramos con las credenciales obtenidas:
```shell
ssh rosa@172.18.0.2
```
![ssh](/Capturas/2024-11-11_21-51.png)

Comprobamos si podemos ejecutar algún comando como **sudo**:

![sudo-l](/Capturas/2024-11-11_21-53.png)

Obtenemos que podemos ver directorios (**ls**) y ficheros (**cat**) como si fuéramos **root**. Así que, probamos a ver el contenido de lo existente en `/root`:
```shell
sudo -u root ls -la /root
```
![root](/Capturas/2024-11-11_21-57.png)

Y ahora vemos el contenido del fichero **secret.txt**:
```shell
sudo -u root cat /root/secret.txt
```
![secret](/Capturas/2024-11-11_21-58.png)

Recibimos una cadena hexadecimal, que deberemos pasarla a texto plano para poder interpretar su contenido:

![hexa](/Capturas/2024-11-11_22-00.png)

Parecer ser que lo que nos devuelve, podría tratarse de una contraseña. Por lo que intentamos acceder al usuario **pedro** con la posible credencial:

![noacertarasosi](/Capturas/2024-11-11_22-02.png)

Al entrar como **pedro**, probamos nuevamente a ver qué comandos podemos ejecutar como **sudo**:

![sudo-l2](/Capturas/2024-11-11_22-03.png)

Como podemos ejecutar **env** como si fueramos **root**, ejecutamos:
```shell
sudo env /bin/sh
```
![rooty](/Capturas/2024-11-11_22-05.png)

Y obtenemos una shell como **root** :triangular_flag_on_post: