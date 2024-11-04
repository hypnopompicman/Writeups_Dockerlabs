# trust

* Máquina **atacante** -> 172.18.0.1
* Máquina **objetivo** -> 172.18.0.2 | Linux :penguin:

### Escaneo de puertos

Realizamos un escaneo de puertos con *nmap* para ver puertos abiertos de la máquina **objetivo**:
```shell
nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn 172.18.0.2
```
![escaneo](/Capturas/2024-10-24_21-28.png)

Vemos abiertos los puertos **22** y **80**.

### Fuzzing web

Ya que al entrar a la página web solamente observamos una plantilla por defecto de apache, procedemos a hacer **fuzzing web** con *gobuster*:
```shell
gobuster dir -u http://172.18.0.2/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x .txt,.py,.php,.sh
```
![fuzzing](/Capturas/2024-10-24_21-39.png)

Encontramos un archivo llamado **secret.php** en el cual, si accedemos a él por la url, nos muestra la siguiente imagen:

![secret](/Capturas/2024-10-24_21-41.png)

Suponemos que **Mario** puede ser un potencial usuario de la máquina **objetivo**.

### Fuerza bruta

Probamos a hacer un ataque de **fuerza bruta** con *hydra*:
```shell
hydra -l mario -P /usr/share/wordlists/rockyou.txt ssh://172.18.0.2
```
![fuerza bruta](/Capturas/2024-10-24_21-46.png)

Obtenemos las credenciales y entramos vía **ssh** a la máquina **objetivo**:

![acceso](/Capturas/2024-10-24_21-47.png)

### Escalada de privilegios

Una vez dentro, ejecutamos el comando `sudo -l` y vemos que podemos ejecutar *vim* como **root**, por lo que usamos el comando `sudo /usr/bin/vim -c ':!/bin/sh'` para escalar privilegios:

![escalada](/Capturas/2024-10-24_21-51.png)

Y somos **root** :triangular_flag_on_post: