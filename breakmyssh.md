# breakmyssh

* Máquina **atacante** -> 172.17.0.1
* Máquina **objetivo** -> 172.17.0.2 | Linux :penguin:

### Escaneo de puertos

Realizamos un escaneo de puertos con *nmap* para ver puertos abiertos de la máquina **objetivo**:
```shell
nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn 172.17.0.2
```
![escaneo](/Writeups_Dockerlabs/Capturas/2024-10-23_21-10.png)

Vemos que solamente tenemos el puerto **22** abierto con el servicio **ssh** corriendo.

### Fuerza bruta

Procedemos a realizar un ataque de **fuerza bruta** con *hydra* sobre **ssh** en la máquina **objetivo**:

![hydra](/Writeups_Dockerlabs/Capturas/2024-10-23_21-13.png)

Vemos que el ataque surge efecto y nos encuentra la contraseña **estrella**.

### Intrusión

Accedemos a la máquina **víctima** vía **ssh** con el usuario **root** y la contraseña **estrella**:

![acceso ssh](/Writeups_Dockerlabs/Capturas/2024-10-23_21-32.png)

**ROOT** :triangular_flag_on_post: