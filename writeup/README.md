
# Write Up - Easy Peasy

### **Nmap**

Primero escanearemos los puertos de la máquina a la que nos hemos conectado utilizando el programa Nmap y ejecutando el siguiente comando:

```bash
**Nmap -sCV -p- 10.10.218.202**

Starting Nmap 7.92 ( https://nmap.org ) at 2022-12-16 18:26 CET
Nmap scan report for 10.10.218.202
Host is up (0.057s latency).
Not shown: 65532 closed tcp ports (conn-refused)
PORT      STATE SERVICE VERSION
80/tcp    open  http    nginx 1.16.1
| http-robots.txt: 1 disallowed entry 
|_/
|_http-title: Welcome to nginx!
|_http-server-header: nginx/1.16.1
6498/tcp  open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 30:4a:2b:22:ac:d9:56:09:f2:da:12:20:57:f4:6c:d4 (RSA)
|   256 bf:86:c9:c7:b7:ef:8c:8b:b9:94:ae:01:88:c0:85:4d (ECDSA)
|_  256 a1:72:ef:6c:81:29:13:ef:5a:6c:24:03:4c:fe:3d:0b (ED25519)
65524/tcp open  http    Apache httpd 2.4.43 ((Ubuntu))
|_http-title: Apache2 Debian Default Page: It works
| http-robots.txt: 1 disallowed entry 
|_/
|_http-server-header: Apache/2.4.43 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 92.46 seconds
```

| Argumentos | Descripción |
| --- | --- |
| -sCV | Utiliza los script que trae Nmap por defecto aparte de detectar las versiones de los servicios encontrados |
| -p- | Indica que analice los puertos del 1 al 65535 |

Como podemos observar estos son los siguientes puertos abiertos:

| Puerto | Servicio | Versión |
| --- | --- | --- |
| 80 | http | nginx 1.16.1 |
| 6498 | ssh | OpenSSH 7.6p1 |
| 65524 | http | Apache httpd 2.4.43 |

Con esta información buscaremos dentro de las páginas http primero para recopilar más información que podría servirnos, como servicios que ejecute la página web, rutas o código que hayan dejado escrito los programadores y se les haya olvidado eliminar.

![Captura de pantalla 2022-12-16 174527.png](writeup/Captura_de_pantalla_2022-12-16_174527.png)

![apache.png](writeup/apache.png)

Como podemos observar, las páginas que nos aparecen son las del servicio por defecto, así que tendremos que realizar un ataque con diccionario sobre los directorios para encontrar rutas que puedan existir.

### Gobuster

En este caso utilizaremos el programa Gobuster y ejecutaremos el siguiente comando:

```bash
gobuster dir -u http://10.10.218.202:80 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.218.202:80
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2022/12/16 18:49:44 Starting gobuster in directory enumeration mode
===============================================================
/hidden               (Status: 301) [Size: 169] [--> http://10.10.218.202/hidden/]
                                                                                  
===============================================================
2022/12/16 19:09:59 Finished
===============================================================
```

| Argumentos | Descripción |
| --- | --- |
| dir | Realiza un ataque de fuerza bruta |
| -u | Indica la URL a la que realizar el ataque |
| -w | Indica el diccionario que utilizar |

Utilizando este programa hemos encontrado una ruta que se llama hidden en el puerto 80, navegaremos a está para observar si hay información útil en este enlace.

![hidden.png](writeup/hidden.png)

No hemos encontrado nada en está página, por lo que seguiremos utilizando Gobuster pero esta vez indicando el directorio hidden para encontrar más rutas dentro de este enlace.

```bash
gobuster dir -u http://10.10.218.202/hidden -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.218.202/hidden
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2022/12/16 19:20:13 Starting gobuster in directory enumeration mode
===============================================================
/whatever             (Status: 301) [Size: 169] [--> http://10.10.218.202/hidden/whatever/]
                                                                                           
===============================================================
2022/12/16 19:39:06 Finished
===============================================================
```

En este caso hemos encontrado la ruta whatever dentro de la ruta hidden, entraremos dentro de la página y analizaremos el código fuente de esta para ver si encontramos información importante.

![hidden_whatever.png](writeup/hidden_whatever.png)

Hemos tenido suerte y hemos encontrado una cadena de texto que por el formato se puede observar que está en base64, por lo tanto tendremos que decofidicarlo, utilizaremos la terminal de Linux para realizar esto.

```bash
echo ZmxhZ3tmMXJzN19mbDRnfQ== | base64 -d
flag{f1r****l4g}
```

| Comando | Descripción |
| --- | --- |
| echo | Muestra la cadena de caracteres escrita. |
| base64 | Sirve para codificar o decodificar en base64 |

| Argumento | Descripción |
| --- | --- |
| | | Se utiliza para encadenar comandos |
| -d | Se utiliza para decodificar |

Tras llegar a este punto, utilizaremos Gobuster pero esta vez con la página de Apache.

```bash
gobuster dir -u http://10.10.218.202:65524 -w /usr/share/wordlists/dirb/common.txt 
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.218.202:65524
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2022/12/16 20:20:41 Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 281]
/.htaccess            (Status: 403) [Size: 281]
/.htpasswd            (Status: 403) [Size: 281]
/index.html           (Status: 200) [Size: 10818]
/robots.txt           (Status: 200) [Size: 153]  
/server-status        (Status: 403) [Size: 281]  
                                                 
===============================================================
2022/12/16 20:21:04 Finished
===============================================================
```

En este caso hemos utilizado otro diccionario y podemos observar un enlace con un status 200 que se llama robot.txt, en este ruta se suelen ubicar enlaces que no se quieren mostrar el los navegadores webs.

![robots.png](writeup/robots.png)

### Hash-identifier

Para saber que tipo de codificación utiliza usaremos el programa hash-identifier.

```bash
hash-identifier a18672860d0510e5ab6699730763b250
   #########################################################################
   #     __  __                     __           ______    _____           #
   #    /\ \/\ \                   /\ \         /\__  _\  /\  _ `\         #
   #    \ \ \_\ \     __      ____ \ \ \___     \/_/\ \/  \ \ \/\ \        #
   #     \ \  _  \  /'__`\   / ,__\ \ \  _ `\      \ \ \   \ \ \ \ \       #
   #      \ \ \ \ \/\ \_\ \_/\__, `\ \ \ \ \ \      \_\ \__ \ \ \_\ \      #
   #       \ \_\ \_\ \___ \_\/\____/  \ \_\ \_\     /\_____\ \ \____/      #
   #        \/_/\/_/\/__/\/_/\/___/    \/_/\/_/     \/_____/  \/___/  v1.2 #
   #                                                             By Zion3R #
   #                                                    www.Blackploit.com #
   #                                                   Root@Blackploit.com #
   #########################################################################
--------------------------------------------------

Possible Hashs:
[+] MD5
[+] Domain Cached Credentials - MD4(MD4(($pass)).(strtolower($username)))
```

Como se observa en este caso es un MD5, utilizaremos una [web](https://md5hashing.net) para averiguar la información de este.

![hash decode.png](writeup/hash_decode.png)

Como hemos comprobado, hemos conseguido otra flag, ahora utilizaremos el inspector de elementos en la página principal de apache para buscar más a fondo alguna información relevante.

![flag_3.png](writeup/flag_3.png)

Utilizando el buscador del inspector de elementos y poniendo la palabra flag hemos encontrado otra flag más, pero no terminaremos de buscar aquí, ahora buscaremos hidden para encontrar código oculto que haya en la página web.

![base62.png](writeup/base62.png)

Hemos encontrado un texto codificado con la pista de que la codificación empieza por ba.., por lo que utilizaremos la web CyberChef para ver que tipo de hash ha utilizado.

![base_62_decode.png](writeup/base_62_decode.png)

Probando los distintos tipo de codificadores que empiezan por ba hemos encontrado que esta encriptado en base62 y el resultado nos arroja la siguiente ruta /n0th1ng3ls3m4tt3r por lo que buscaremos que hay dentro de esta ruta.

![nothing.png](writeup/nothing.png)

Como podemos observar, en esta página se encuentra tanto una cadena de caracteres como una imagen, por lo que seguramente haga falta hacer esteganografía, pero primero tendremos que descodificar la cadena de caracteres utilizando la siguiente [web](https://md5hashing.net)

![decrypt.png](writeup/decrypt.png)

### Steghide

Ahora que tenemos la contraseña utilizaremos el programa steghide con el archivo .jpg de la página para obtener la información que seguramente este oculta dentro de esta.

```bash
steghide extract -sf binarycodepixabay.jpg -p mypasswordforthatjob
anot� los datos extra�dos e/"secrettext.txt".
```

| Argumentos | Descripción |
| --- | --- |
| -sf | Sirve para indicar el archivo |
| -p | Declarar salvo conducto (contraseña) |

Cuando analizamos el archivo de secrettext.txt nos saldrá la siguiente información:

```bash
cat secrettext.txt 
username:boring
password:
01101001 01100011 01101111 01101110 01110110 01100101 01110010 01110100 01100101 01100100 01101101 01111001 01110000 01100001 01110011 01110011 01110111 01101111 01110010 01100100 01110100 01101111 01100010 01101001 01101110 01100001 01110010 01111001
```

Como podemos comprobar la contraseña está en binario y el usuario es boring, con esto puedo sospechar que son las credenciales para conectarme por ssh, pero primero pasaremos a texto el binario con la página [web](https://www.traductorbinario.com/) traductor binario.

![binario a texto.png](writeup/binario_a_texto.png)

Una vez tenemos la contraseña usaremos ssh para conectarnos a la máquina con las credenciales obtenidas.

```bash
ssh boring@10.10.113.202 -p 6498
The authenticity of host '[10.10.113.202]:6498 ([10.10.113.202]:6498)' can't be established.
ECDSA key fingerprint is SHA256:hnBqxfTM/MVZzdifMyu9Ww1bCVbnzSpnrdtDQN6zSek.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[10.10.113.202]:6498' (ECDSA) to the list of known hosts.
*************************************************************************
**        This connection are monitored by government offical          **
**            Please disconnect if you are not authorized	       **
** A lawsuit will be filed against you if the law is not followed      **
*************************************************************************
boring@10.10.113.202's password: 
You Have 1 Minute Before AC-130 Starts Firing
XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
!!!!!!!!!!!!!!!!!!I WARN YOU !!!!!!!!!!!!!!!!!!!!
You Have 1 Minute Before AC-130 Starts Firing
XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
!!!!!!!!!!!!!!!!!!I WARN YOU !!!!!!!!!!!!!!!!!!!!
boring@kral4-PC:~$
```

| Argumento | Descripción |
| --- | --- |
| @ | Se utiliza para separar el usuario de la dirección IP |
| -p | Se utiliza para indicar el puerto |

Una vez hemos entrado utilizaremos el comando ls para ver el directorio y cat para ver la información que hemos encontrado dentro del archivo.

```bash
boring@kral4-PC:~$ ls
user.txt
boring@kral4-PC:~$ cat user.txt
User Flag But It Seems Wrong Like It`s Rotated Or Something
synt{a0jvgf33zfa0ez4y}
boring@kral4-PC:~$
```

Como podemos ver, hay una flag, pero como se puede observar está encriptado en Cesar, por lo que utilizaremos la [web](https://www.dcode.fr/caesar-cipher) dcode para esto.

![cesar.png](writeup/cesar.png)

Como podemos comprobar hemos obtenido la flag del usuario, ahora nos queda obtener la flag del usuario root, pero como seguramente no tengamos permiso para entrar en la carpeta tendremos que buscar alguna vulnerabilidad que aprovechar.

![carpeta con vulnerabilidad.png](writeup/carpeta_con_vulnerabilidad.png)

Aquí hemos encontrado un archivo secreto donde seguramente podremos elevar privilegios.

![aaa.png](writeup/aaa.png)

Como podemos ver, los comandos que se ejecuten aquí se ejecutarán como root, así que con el comando que hemos puesto crearemos una reverse shell a nuestro equipo con la cuenta de administrador cuando se actualice la tarea de cron en la máquina.

```bash
root@kral4-PC:~# cat .root.txt
cat .root.txt
flag{63a9f0ea7bb98******b649e85481845}
root@kral4-PC:~#
```

Como podemos observar, la conexión como root se ha realizado y hemos obtenido la última flag que era del usuario root.

## Contact

[📧 danielruizraposo02@gmail.com](mailto:adalovelace@mail.com)
