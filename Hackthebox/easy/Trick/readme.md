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

### Captura

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


### Captura

![comando dig](images/04_dig.png)

Vamos a añadirlo al /etc/hosts de nuestra máquina

### Captura

![/etc/hosts](images/05_etc_hosts.png)

```text
```

### Captura

![Enumeración de directorios](images/03_gobuster.png)

---

## Tecnologías detectadas

- Servidor:
- Framework:
- CMS:
- Versión:

### Captura

![Tecnologías detectadas](images/04_whatweb.png)

---

## Hallazgos

-

---

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
