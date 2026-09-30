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

<img width="545" height="355" alt="image" src="https://github.com/user-attachments/assets/41b9e1fe-91be-47b2-9e39-a1480c4bb683" />


Veremos si tenemos conexión con la máquina, para esto realizaremos un ping

```bash
ping 172.17.0.2
```

<img width="615" height="231" alt="image" src="https://github.com/user-attachments/assets/63d507a9-4cfc-4571-8d5e-8ed56803f847" />


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

<img width="1505" height="696" alt="image" src="https://github.com/user-attachments/assets/d54a784e-a69c-408a-aff8-dd41c30ba43d" />


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

<img width="805" height="592" alt="image" src="https://github.com/user-attachments/assets/326ff349-88b6-4762-ae55-27db80854120" />


Vemos que ha encontrado directorios a los que podemos acceder, como /dragon. Este nos da un código 301 lo cual significa que nos redireccionará.
<img width="967" height="417" alt="image" src="https://github.com/user-attachments/assets/f532c2df-4ed9-4120-8f9f-e26186385f8c" />


Nos encontramos en una carpeta donde podemos acceder a un archivo llamado EpisodiosT1

Si entramos al archivo nos encontramos lo siguiente:
<img width="812" height="353" alt="image" src="https://github.com/user-attachments/assets/57d82694-0d47-4bea-b7e0-33ece63e78a7" />


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

<img width="1080" height="252" alt="image" src="https://github.com/user-attachments/assets/bef19231-4153-4782-972a-68a9ec12fbe5" />


Listamos los permisos

```bash
smbmap -H 172.17.0.2
```

<img width="995" height="445" alt="image" src="https://github.com/user-attachments/assets/06c0bb4b-56c1-40d2-8b4c-d6391515d82c" />


Vemos que no podemos acceder con sesión nula a ningún recurso.


Con Enum4Linux listaremos los usuarios

```bash
enum4linux -a 127.17.0.2
```

Encontrará los usuarios que anteriormente listamos como posibles.

<img width="905" height="82" alt="image" src="https://github.com/user-attachments/assets/4ccd17cd-e4bd-4a57-8ef7-4234ee93cb38" />


# Acceso a Samba

Vamos a intentar acceder a Samba con los usuarios y las posibles contraseñas que hemos encontrado.

<img width="1134" height="90" alt="image" src="https://github.com/user-attachments/assets/5607acbb-2299-4f39-a834-495136070b14" />


Hemos encontrado la contraseña de un usuario, por lo que ahora podemos entrar a samba con estas credenciales.

<img width="672" height="381" alt="image" src="https://github.com/user-attachments/assets/721c23d5-a80d-4a57-9384-edf8c55b2d22" />


Hemos conseguido entrar al recurso con el usuario jon.

Vemos que hay un archivo llamado paraJon
<img width="614" height="191" alt="image" src="https://github.com/user-attachments/assets/47790353-113d-4cc4-a38d-b5dd5f356c1d" />


Vamos a llevárnoslo con el comando get a nuestra máquina.
<img width="932" height="123" alt="image" src="https://github.com/user-attachments/assets/cc4987c6-2ab1-4d30-97c4-b1b26b5efeab" />


Ahora vamos al recurso compartido y nos llevaremos el archivo que existe

<img width="1007" height="235" alt="image" src="https://github.com/user-attachments/assets/75c3c38a-83f0-435b-88b1-6e06a0b1f498" />


<img width="1136" height="97" alt="image" src="https://github.com/user-attachments/assets/5cea80fe-3267-4ef6-a063-9a128eb72881" />


Descifraremos la contraseña, está en base64
<img width="455" height="61" alt="image" src="https://github.com/user-attachments/assets/589131c9-dd4f-4c2f-a9d6-5bb088bbb54f" />

Tendremos la contraseña de Jon para ssh

# Acceso SSH
<img width="874" height="254" alt="image" src="https://github.com/user-attachments/assets/83e393cb-b891-4591-a97c-799a0b87eab0" />


Hemos logrado entrar en el SSH, leeremos el passwd

<img width="614" height="415" alt="image" src="https://github.com/user-attachments/assets/08a0867a-5d26-4b60-b771-d70c84c3a17d" />


Vamos a ver los permisos.
<img width="947" height="97" alt="image" src="https://github.com/user-attachments/assets/6c8dd2c2-753d-4ffa-8593-c54f4dd3006b" />


Jon puede ejecutar .mensaje.py como aria sin introducir contraseña

<img width="485" height="183" alt="image" src="https://github.com/user-attachments/assets/a3f382b4-967c-4713-9c5e-49025aa70451" />

Aria es la propietaria del archivo, vamos a clonarlo para que nosotros seamos el propietario e introduciremos código arbitrario.

Primero borraremos el archivo y lo volveremos a crear e introduciremos lo siguiente.
<img width="441" height="243" alt="image" src="https://github.com/user-attachments/assets/3cc49adc-4e2b-4daa-914e-3250a6b693f3" />


```python
import os
os.system("/bin/bash")
```

Ahora podremos ejecutar el archivo.
<img width="594" height="29" alt="image" src="https://github.com/user-attachments/assets/0a1dedc4-112e-4f92-869c-f39195543d8e" />


De esta forma habremos conseguido entrar al usuario de aria.

Listaremos el contenido de la carpeta de aria y los permisos de este usuario
<img width="1006" height="219" alt="image" src="https://github.com/user-attachments/assets/4821bcc0-57aa-49f2-9ed8-674ad7934200" />


Vemos que puede usar cat y ls suplantando el usuario de daenerys.

<img width="1132" height="143" alt="image" src="https://github.com/user-attachments/assets/ec82e7b9-f7e3-4605-afc7-f9fe4c091be7" />

Hemos conseguido la contraseña de daenerys.

Podemos seguir leyendo archivos
<img width="640" height="158" alt="image" src="https://github.com/user-attachments/assets/71b53479-b34d-4dff-b60a-891bb337aad7" />


<img width="718" height="76" alt="image" src="https://github.com/user-attachments/assets/3adcec5e-a625-4bee-8e3c-dee26fa9b173" />


<img width="765" height="62" alt="image" src="https://github.com/user-attachments/assets/53b9b4bd-aaf5-455d-ad2b-5fb4d8798c5f" />


Este es un codigo para crear una reverse shell.

Vamos a usar el usuario daenerys

<img width="1061" height="143" alt="image" src="https://github.com/user-attachments/assets/a6b9333a-6a08-4db2-8886-d9822a2f5afc" />


Vemos que el archivo .shell.sh lo puede ejecutar como root sin contraseña

Cambiamos el contenido de .shell.sh para cambiar los permisos de bin/bash a SUID

<img width="955" height="173" alt="image" src="https://github.com/user-attachments/assets/08ac01c0-5ad9-4a2b-b69f-e3330f3d2367" />

<img width="365" height="66" alt="image" src="https://github.com/user-attachments/assets/69922a76-6195-4d1b-974c-8bd1f3d82486" />


Somos root y hemos completado la máquina
