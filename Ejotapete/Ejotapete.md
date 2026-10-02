Dificultad -> Fácil

Enlace a la máquina -> [Dockerlabs](https://dockerlabs.es/)

# Archivos
.sh → Script de Linux con comandos. Se ejecuta en la terminal.

.tar → Archivo que agrupa otros archivos/carpetas.

# Despliegue

La máquina se descarga en formato .zip, deberemos extraer su contenido desde nuestra máquina Kali. 
Abriremos una terminal en la carpeta donde hemos extraído el contenido del .zip y deberemos ejecutar:

```
sudo bash auto_deploy.sh ejotapete.tar
```

Obtenemos la IP de la máquina.

![](Fotos/Pasted%20image%2020261002164405.png)

# Escaneo

Vamos a escanear con nmap para ver los puertos y servicios activos.

Vemos que hay únicamente un servicio activo

![](Fotos/Pasted%20image%2020261002164614.png)

* 80: Apache httpd 2.4.25. Codigo 403 forbidden

Vamos a enumerar el directorio web

# Enumeración

![](Fotos/Pasted%20image%2020261002164944.png)

Vemos que ha encontrado una ruta accesible, la cual es /drupal.

Drupal es un CMS de código abierto y gratuito.


# Drupal

![](Fotos/Pasted%20image%2020261002165218.png)

Vamos a enumerar bajo el directorio drupal

![](Fotos/Pasted%20image%2020261002170123.png)

Vemos un install.php el cual nos dirá la versión actual de drupal.

![](Fotos/Pasted%20image%2020261002170217.png)

La versión es Drupal 8.5.0 el cual es vulnerable a Drupalggedon2

En mi caso utilizaré para la explotación Metasploit.

![](Fotos/Pasted%20image%2020261002170952.png)

Rellenamos las opciones y lanzamos el exploit

![](Fotos/Pasted%20image%2020261002171042.png)

Hemos conseguido una sesión de meterpreter
![](Fotos/Pasted%20image%2020261002171155.png)

Vemos que somos el usuario www-data, vamos a intentar abrir una shell

![](Fotos/Pasted%20image%2020261002171350.png)

Necesitaremos que la shell sea ptty asi que lo haremos de la siguiente manera

![](Fotos/Pasted%20image%2020261002171634.png)

Se ha intentado usar python para crear la shell pero no la máquina no lo tiene asi que tendremos que hacerlo mediante

```bash
script /dev/null -c bash
```

Con esto ya tenemos la shell.

El siguiente paso será buscar binarios con SUID
```bash
find / -perm -4000 -user root 2>/dev/null
```

Encontramos los siguientes

![](Fotos/Pasted%20image%2020261002172140.png)

Hemos encontrado que el binario /usr/bin/find tiene el SUID activo y le pertenece a root asi que vamos a aprovecharlo para escalar privilegios

![](Fotos/Pasted%20image%2020261002172905.png)

```bash
/usr/bin/find . -exec /bin/bash -p \; -quit
```

Hemos conseguido root y hemos completado la máquina
