# borazuwarahctf

* Máquina **atacante** -> 172.17.0.1
* Máquina **objetivo** -> 172.17.0.2 | Linux :penguin:

### Escaneo de puertos

Realizamos un escaneo de puertos con *nmap* para ver puertos abiertos de la máquina **objetivo**:
```shell
nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn 172.17.0.2
```
![resultado escaneo](/Capturas/2024-10-23_19-40.png)

Vemos que tenemos en el puerto **22** corriendo un servicio de *ssh*  y en el **80** un *apache*.

### Investigación servidor web

Abrimos el navegador y ponemos la ip de la máquina **objetivo** para ver qué tenemos:

![kinder](/Capturas/2024-10-23_19-45.png)

Vemos una imagen de un huevo Kinder. Al mirar el **código fuente** de la página no encontramos nada, al igual que al realizar **fuzzing web** con *dirb*, *gobuster* u otras herramientas. Por lo que optamos por las opción más simple, nos descargamos la **imagen** y usamos el comando `cat` para ver el "interior del **huevo**":

![interior huevo](/Capturas/2024-10-23_19-49.png)

Descubrimos la existencia del usuario **borazuwarah**.

### Fuerza bruta

Procedemos a realizar **fuerza bruta** sobre el usuario **borazuwarah** para el servicio **ssh** con la herramienta *hydra*:
```shell
hydra -l borazuwarah -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
```
![resultado hydra](/Capturas/2024-10-23_19-55.png)

Encontramos la contraseña del usuario y accedemos a la máquina **víctima** con las credenciales vía **ssh**.

![acceso ssh](/Capturas/2024-10-23_19-57.png)

### Escalada de privilegios

Escribimos el comando `sudo -l` para ver si hay algún comando que podamos correr como **root**:

![comando sudo-l](/Capturas/2024-10-23_20-04.png)

Al parecer podemos correr la **bash** como **root**, así que simplemente ejecutamos `sudo bash`:

![sudo bash](/Capturas/2024-10-23_20-06.png)

Y ya somos **root** :triangular_flag_on_post: