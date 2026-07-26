# Trick

| Plataforma | Dificultad | Sistema Operativo |
|------------|------------|-------------------|
| Hack The Box | ⭐⭐⭐⭐☆ | Linux |

---

# 📋 Información

- **IP:** `10.129.36.29`
- **Objetivo:** Obtener acceso y escalar privilegios hasta `root`.

---

# 🔍 Reconocimiento

## Descubrimiento de puertos

```bash
nmap -Pn -p- --min-rate 5000 -sS -n <IP> -oN allPorts
```

### Resultado

```text

```

### Captura

![Escaneo completo de puertos](images/01_nmap_all_ports.png)

---

## Enumeración de servicios

```bash
nmap -sCV -p<PUERTOS> <IP> -oN services
```

### Resultado

```text
Tenemos activos los siguientes puertos:
- 22: ssh
- 80: http
- 53: DNS
- 25: smtp
```

![Enumeración de servicios](images/02_nmap_services.png)

---

# 🌐 Enumeración Web

Vemos que en la web solo hay un campo para introducir el email, aquí no hay nada interesante

![Web principal](images/03_web.png)

## Dominio

Recordamos que tenemos abierto el puerto 53, con ello vamos a intentar hacer un ataque de transferencia de zona que consistirá en la busqueda de algun dominio expuesto

```bash
dig @<IP> -x <IP>
```

Vemos que el registro PTR nos ha devuelto un dominio llamado trick.htb

![comando dig](images/04_dig.png)

Vamos a añadirlo al /etc/hosts de nuestra máquina

![/etc/hosts](images/05_etc_hosts.png)


Ahora hacemos una transferencia de zona para ver si hay algun subdominio

```bash
dig @<IP> axfr <DOMAIN>
```
Conseguimos un subdominio llamado preprod-payroll.trick.htb que también vamos a añadir al /etc/hosts

![Transferencia de zona](images/06_axfr.png)

# 🌐 Enumeración Web 2

Con el dominio conseguido, probamos en el navegador a ver si conseguimos que cambie la web

![web2](images/07_web2.png)

En este panel vamos a probar credenciales típicas como admin:admin admin:password user:user etc... Pero no conseguiremos el acceso.
Vamos a probar una inyeccion sql típica

```sql
or1=1
'or'1'='1
'or'1'='1--
'or'1'='1-- -
admin'or1=1
admin'or1=1-- -
admin'or1=1--
admin'or'1'='1
```
Probando tipicas conseguimos acceso con el 'or'1'='1

![admin_panel](images/08_admin_panel.png)

Una vez dentro podemos ver que existe el usuario Enemigoss, un empleado que se llama john y poco mas.

---
# Enumeración de subdominios

Como no encontramos nada, vamos a enumerar subdominios de la parte de preproduccion
Para ello vamos a necesitar la herramienta ffuf y el directorio de diccionarios de [seclists](https://github.com/danielmiessler/seclists)



# 🎯 Acceso inicial

## Vulnerabilidad

Explicación de la vulnerabilidad utilizada.

### Captura

![Vulnerabilidad identificada](images/05_vulnerability.png)

---

## Explotación

```bash

```

### Captura

![Ejecución del exploit](images/06_exploit.png)

---

## Shell obtenida

```text

```

### Captura

![Shell inicial](images/07_initial_shell.png)

---

# 🔎 Enumeración interna

## Usuario actual

```bash
whoami
id
hostname
```

### Captura

![Información del usuario](images/08_user_info.png)

---

## Información del sistema

```bash
uname -a
cat /etc/os-release
```

### Captura

![Información del sistema](images/09_system_info.png)

---

## Enumeración automática

```bash
linpeas.sh
```

### Captura

![Resultados de LinPEAS](images/10_linpeas.png)

---

## Hallazgos

-

---

# 🚀 Escalada de privilegios

## Vector encontrado

Descripción del vector.

### Captura

![Vector de escalada](images/11_privilege_vector.png)

---

## Explotación

```bash

```

### Captura

![Escalada de privilegios](images/12_privilege_escalation.png)

---

## Acceso como root / Administrator

```bash
whoami
```

```text
root
```

### Captura

![Shell como root](images/13_root_shell.png)

---

# 🏁 Flags

## User Flag

```text

```

### Captura

![User Flag](images/14_user_flag.png)

---

## Root Flag

```text

```

### Captura

![Root Flag](images/15_root_flag.png)

---
