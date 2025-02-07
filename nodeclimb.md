# Nodeclimb

* Máquina **atacante** -> 172.17.0.1
* Máquina **objetivo** -> 172.17.0.2 | Linux :penguin:

### Escaneo de puertos

Realizamos un escaneo de puertos con *nmap* para ver puertos abiertos de la máquina **objetivo**:
```shell
nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn 172.17.0.2
```
![scaneo](/Capturas/2025-01-13_11-51.png)

Descubrimos muchas cosas con este escaneo de puertos. Tenemos abiertos dos puertos, el **21** y el **22**. Dentro del puerto **21**, tenemos un servicio FTP corriendo con la **vulnerabilidad Anonymous FTP** activa y un archivo .zip llamado **secretitopicaron.zip** cuanto menos interesante.

### Explotación vulnerabilidad Anonymous

Nos conectamos al servidor FTP aprovechándonos de la vulnerabilidad y nos descargamos el archivo **secretitopicaron.zip**:

![ftp](/Capturas/2025-01-13_11-58.png)

### Fuerza bruta

Como el archivo descargado necesita de una contraseña para poder descomprimirlo, procedemos a extraer el hash y realizarle un ataque de fuerzar bruta con *jhonTheRipper*:
```shell
zip2john secretitopicaron.zip > hash

john --wordlist=/usr/share/wordlists/rockyou.txt hash
```
![fb](/Capturas/2025-01-13_12-01.png)

Obtenemos unas credenciales con las que probaremos accediendo a la máquina vía ssh.

### Escalada de privilegios

Una vez habiendo accedido a la máquina con el usuario **mario** y la contraseña **laKontraseñAmasmalotaHdelbarrioH**, vemos que podemos ejecutar *node* usando el archivo script.js como sudo ejecutando el comando `sudo -l`:

![sudol](/Capturas/2025-01-13_12-05.png)

Por lo que ponemos el siguiente comando en el archivo **script.js** y lo ejecutamos usando *node*:

![script.js](/Capturas/2025-01-13_12-08.png)
![root](/Capturas/2025-01-13_12-10.png)

Y somos **root** :triangular_flag_on_post: