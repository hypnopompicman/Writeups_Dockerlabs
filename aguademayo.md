# aguademayo

* Máquina **atacante** -> 172.17.0.1
* Máquina **objetivo** -> 172.17.0.2 | Linux :penguin:

### Escaneo de puertos

Realizamos un escaneo de puertos con *nmap* para ver puertos abiertos de la máquina **objetivo**:
```shell
nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn 172.17.0.2
```
![escaneo](/Capturas/2024-10-30_19-21.png)

Encontramos abiertos el puerto **80** y el puerto **22**.

### HTTP

Si nos vamos al navegador y vemos lo que hay corriendo por el puerto **80**, nos encontramos con una plantilla de *Apache* normal, pero si vemos el código fuente de la página, al final del mismo nos encontramos con esto:

> "<!--
++++++++++[>++++++++++>++++++++++>++++++++++>++++++++++>++++++++++>++++++++++>++++++++++++>++++++++++>+++++++++++>++++++++++++>++++++++++>++++++++++++>++++++++++>+++++++++++>+++++++++++>+>+<<<<<<<<<<<<<<<<<-]>--.>+.>--.>+.>---.>+++.>---.>---.>+++.>---.>+..>-----..>---.>.>+.>+++.>.
-->"

Si copiamos y pegamos esto en *Google*, nos sale información sobre *Brainfuck* (un lenguaje de programación esotérico). Por lo que vamos a un **interpreter** de este lenguaje y lo pegamos para ver el output:

![resultado interpreter](/Capturas/2024-10-30_19-40.png)

Vemos que el resultado es: **bebeaguaqueessano**. Por lo que intuimos que esto se trata de alguna credencial de una cuenta de la máquina **objetivo**.

### Fuzzing web

Para seguir obteniendo más información sobre la máquina **objetivo**, procedemos a realizar **fuzzing web** con *dirb*:

![fuzzing web](/Capturas/2024-10-30_20-25.png)

Encontramos un directorio llamado **/images**, y al buscarlo en el navegador obtenemos esto:

![iamgen](/Capturas/2024-10-30_20-26.png)

Si abrimos el archivo, nos encontramos con esta foto:

![imagen agua](/Capturas/2024-10-30_20-27.png)

Esta foto no tiene nada interesante, pero el nombre del archivo **agua_ssh** nos da una pista sobre cual puede ser la segunda credencial que estamos buscando.

### Intrusión

Probamos a acceder vía **ssh** a la máquina **objetivo** con las dos credenciales encontradas:

![intrusion](/Capturas/2024-10-30_20-29.png)

Y estamos dentro de la máquina **objetivo**.

### Escalada de privilegios

Al ejecutar el comando `sudo -l` nos muestran que podemos ejecutar *bettercap* como usuario **root**:

![sudo-l](/Capturas/2024-10-30_20-32.png)

Ejecutamos la herramienta como **root** y vemos las opciones que tenemos con un `help`:

![help](/Capturas/2024-10-30_20-41.png)

Vemos que tenemos la posibilidad de ejecutar comandos si lo realizamos precedido de un signo de **!**. Por lo que cambiamos los permisos de la **bash** para poder ejecutarla como **root** con el comando `! chmod u+s /bin/bash`.

Una vez ejecutado, nos salimos de *bettercap* y ejecutamos el comando `bash -p` para elevar privilegios:

![root](/Capturas/2024-10-30_20-54.png)

Y ya somos **root** :triangular_flag_on_post: