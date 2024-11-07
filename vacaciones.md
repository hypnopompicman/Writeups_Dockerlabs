# vacaciones

* Máquina **atacante** -> 172.17.0.1
* Máquina **objetivo** -> 172.17.0.2 | Linux :penguin:

### Escaneo de puertos

Realizamos un escaneo de puertos con *nmap* para ver puertos abiertos de la máquina **objetivo**:
```shell
nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn 172.17.0.2
```
![escaneo](/Capturas/2024-10-24_22-22.png)

Encontramos abiertos el puerto **80** y el puerto **22**.

### HTTP

Vamos a la página web y la vemos en blanco, pero al ver el **código fuente** podemos leer el siguiente mensaje:
> De : Juan Para: Camilo , te he dejado un correo es importante...

Encontramos dos posibles **usuarios** de la máquina **objetivo**.

### Fuerza bruta

Probamos a realizar un ataque de **fuerza bruta** con ambos nombres:

![fbruta](/Capturas/2024-10-24_22-28.png)

Encontramos la contraseña del usuario **camilo** y procedemos al acceso vía **ssh**:

![entrada](/Capturas/2024-10-24_22-32.png)

### Escalada de privilegios

Al no poder ejecutar nada como **root** ni encontrar algún binario vulnerable, solo nos queda investigar la máquina. Recordemos que **Juan** había dejado un correo a **Camilo** importante, por lo que nos dirigimos a `/var/mail/camilo` y leemos el archivo `.txt` que hay en el interior:

![correo](/Capturas/2024-10-24_22-38.png)

Accedemos como el usuario **Juan** con la contraseña contenida en el correo, y comprobamos que con `sudo -l` podemos ejecutar **ruby** como **root**:

![usuariojuan](/Capturas/2024-10-24_22-41.png)

Ejecutamos el comando `sudo ruby -e 'exec "/bin/sh"'` y somos **root** :triangular_flag_on_post:

![root](/Capturas/2024-10-24_22-43.png)