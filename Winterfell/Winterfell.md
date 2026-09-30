Dificultad -> Fácil

Enlace a la máquina -> [Dockerlabs](https://dockerlabs.es/maquinas/121/descargar)

# Archivos
.sh → Script de Linux con comandos. Se ejecuta en la terminal.

.tar → Archivo que agrupa otros archivos/carpetas.

# Despliegue

La máquina se descarga en formato .zip, deberemos extraer su contenido desde nuestra máquina Kali. 
Abriremos una terminal en la carpeta donde hemos extraído el contenido del .zip y deberemos ejecutar:

```
sudo bash auto_deploy.sh winterfell.tar
```

Obtenemos la IP de la máquina.

![[Pasted image 20260930142630.png]]

Veremos si tenemos conexión con la máquina, para esto realizaremos un ping

```bash
ping 172.17.0.2
```

![[Pasted image 20260930142939.png]]

Vemos que tenemos conexión con la máquina y además podemos deducir que la máquina es un sistema operativo Linux debido al valor de ttl, ya que para Linux el valor de ttl suele iniciar en 64 y para Windows suele iniciar en 128.


# Escaneo

Una vez que sabemos que tenemos conexión con la máquina realizaremos el escaneo de puertos.

```bash
nmap -p- -sCV 172.17.0.2 -vvv
```

Vemos que hay 4 puertos abiertos:

* 22: OpenSSH 9.2p1 debian 2+deb12u3: Esta versión no es vulnerable al fallo regreSSHion ya que incluye el parche que lo corrige.
* 80: Apache httpd 2.4.61
* 139: Samba smbd 4
* 445 Samba smbd 4

Vemos que hay un aplicativo web con el nombre de Juego de Tronos al que podemos acceder.

Podemos guardarnos los nombres que aparecen en la web como posibles nombres de usuarios

![[Pasted image 20260930145320.png]]

```bash
nano users.txt

jon
arya
aria
daenerys
```
# Enumeración

## Enumeración Web

Vamos a enumerar el directorio web para ver todos los directorios.

![[Pasted image 20260930145916.png]]

Vemos que ha encontrado directorios a los que podemos acceder, como /dragon. Este nos da un código 301 lo cual significa que nos redireccionará.
![[Pasted image 20260930150033.png]]

Nos encontramos en una carpeta donde podemos acceder a un archivo llamado EpisodiosT1

Si entramos al archivo nos encontramos lo siguiente:
![[Pasted image 20260930150425.png]]

Este listado podria servirnos como posibles contraseñas.

```bash
nano pass.txt

seacercaelinvierno
elcaminoreal
lordnieve
tullidosbastardosycosasrotas
elloboyelleon
unacoronadeoro
ganasomueres
porelladodelapunta
baelor
fuegoyhielo
```

## Enumeración Samba

Vamos a pasar a enumerar el recurso de samba, en el que encontramos dos recursos interesantes:
- Shared
- nobody: Apunta a Home Directories. Podría tratarse del directorio de un usuario sin privilegios o una configuración incorrecta.

```bash
smbclient -L //172.17.0.2/
```

![[Pasted image 20260930151036.png]]

Listamos los permisos

```bash
smbmap -H 172.17.0.2
```

![[Pasted image 20260930151315.png]]

Vemos que no podemos acceder con sesión nula a ningún recurso.


Con Enum4Linux listaremos los usuarios

```bash
enum4linux -a 127.17.0.2
```

Encontrará los usuarios que anteriormente listamos como posibles.

![[Pasted image 20260930152403.png]]

# Acceso a Samba

Vamos a intentar acceder a Samba con los usuarios y las posibles contraseñas que hemos encontrado.

![[Pasted image 20260930173845.png]]

Hemos encontrado la contraseña de un usuario, por lo que ahora podemos entrar a samba con estas credenciales.

![[Pasted image 20260930174343.png]]

Hemos conseguido entrar al recurso con el usuario jon.

Vemos que hay un archivo llamado paraJon
![[Pasted image 20260930174448.png]]

Vamos a llevárnoslo con el comando get a nuestra máquina.
![[Pasted image 20260930174618.png]]

Ahora vamos al recurso compartido y nos llevaremos el archivo que existe

![[Pasted image 20260930175221.png]]

![[Pasted image 20260930175251.png]]

Descifraremos la contraseña, está en base64
![[Pasted image 20260930175335.png]]
Tendremos la contraseña de Jon para ssh

# Acceso SSH
![[Pasted image 20260930175739.png]]

Hemos logrado entrar en el SSH, leeremos el passwd

![[Pasted image 20260930175817.png]]

Vamos a ver los permisos.
![[Pasted image 20260930175940.png]]

Jon puede ejecutar .mensaje.py como aria sin introducir contraseña

![[Pasted image 20260930180257.png]]
Aria es la propietaria del archivo, vamos a clonarlo para que nosotros seamos el propietario e introduciremos código arbitrario.

Primero borraremos el archivo y lo volveremos a crear e introduciremos lo siguiente.
![[Pasted image 20260930180849.png]]

```python
import os
os.system("/bin/bash")
```

Ahora podremos ejecutar el archivo.
![[Pasted image 20260930181110.png]]

De esta forma habremos conseguido entrar al usuario de aria.

Listaremos el contenido de la carpeta de aria y los permisos de este usuario
![[Pasted image 20260930181349.png]]

Vemos que puede usar cat y ls suplantando el usuario de daenerys.

![[Pasted image 20260930181704.png]]
Hemos conseguido la contraseña de daenerys.

Podemos seguir leyendo archivos
![[Pasted image 20260930182009.png]]

![[Pasted image 20260930182213.png]]

![[Pasted image 20260930182230.png]]

Este es un codigo para crear una reverse shell.

Vamos a usar el usuario daenerys

![[Pasted image 20260930182340.png]]

Vemos que el archivo .shell.sh lo puede ejecutar como root sin contraseña

Cambiamos el contenido de .shell.sh para cambiar los permisos de bin/bash a SUID

![[Pasted image 20260930184014.png]]
![[Pasted image 20260930184253.png]]

Somos root y hemos completado la máquina