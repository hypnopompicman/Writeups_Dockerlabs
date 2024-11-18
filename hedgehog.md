# hedgehog

* Máquina **atacante** -> 172.17.0.1
* Máquina **objetivo** -> 172.17.0.2 | Linux :penguin:

### Escaneo de puertos

Realizamos un escaneo de puertos con *nmap* para ver puertos abiertos de la máquina **objetivo**:
```shell
nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn 172.17.0.2
```
![escaneo](/Capturas/2024-11-17_13-49.png)

Tenemos abiertos los puertos **22** y **80**.

### HTTP

Exploramos lo que tenemos corriendo por el puerto **80** y vemos que hay una página web simplemente con la palabra **tails**:

![tails](/Capturas/2024-11-17_13-52.png)

### Fuerza bruta

Probamos a realizar un **ataque de fuerza bruta** con *hydra*:
```shell
hydra -l tails -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
```
![tails_hydra](/Capturas/2024-11-17_13-59.png)

Al probar diferentes combinaciones y diccionarios sin resultado alguno, y viendo que no tenemos otro camino abierto para avanzar con la máquina, intentamos darle la vuelta al diccionario para empezar desde atrás el ataque.

Para ello, he decidido aplicar mis conocimientos de **python** y realizar un script que automatice el proceso de invertir el diccionario deseado. Este **script**, lo subiré también a mi **Github** por si lo quiere utilizar alguien más.

Para usarlo, simplemente lo ejecutamos donde tengamos el archivo de texto deseado
```shell
python3 Inverter.py
```
Y seguimos los pasos que nos indica el programa:

![script](/Capturas/2024-11-17_14-07.png)

Si ahora probamos a realizar el **ataque de fuerza bruta** con nuestro nuevo archivo, obtenemos las credenciales del usuario **tails**:
```shell
hydra -l tails -P rockyou_invertido.txt ssh://172.17.0.2
```
![rockiinvertido](/Capturas/2024-11-17_14-30.png)

### Escalada de privilegios

Entramos a la máquina **objetivo**:

![entrada](/Capturas/2024-11-17_14-32.png)

Vemos que podemos ejecutar todos los comandos como el usuario **sonic**, así que accedemos a él:

![sonic](/Capturas/2024-11-17_14-33.png)

Y desde el usuario sonic, vemos que podemos ejecutar todos los comandos desde cualquier usuario, así que escalamos privilegios:

![root](/Capturas/2024-11-17_14-34.png)

Y ya somos usuario **root** :triangular_flag_on_post: