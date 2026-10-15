# 📇 Laboratorio 07 · OpenLDAP: autenticación centralizada en Linux

**Nivel:** 🟠 Avanzado · **Sistema:** 🐧 Linux · **Tiempo estimado:** 120 minutos

---

## 🎯 Escenario

En el laboratorio 02 de este nivel viste el problema de los **UID distintos** entre máquinas: `ana` era el 1001 en un servidor y el 1002 en otro. Y en el nivel intermedio resolviste algo parecido en el mundo Windows con **Active Directory**.

En Linux, el equivalente es un **directorio LDAP**. Tu misión: montar un servidor **OpenLDAP** con los usuarios de la empresa y conseguir que **otra máquina Linux** les deje iniciar sesión **sin tener cuentas locales**.

> 💾 **Snapshot** de las dos VMs. Servidor LDAP: `ubuntu-server-lab` (`192.168.56.10`). Cliente: `ubuntu-cliente` (`192.168.56.11`).

---

## 📖 Conceptos nuevos

### Qué es LDAP

**LDAP** (*Lightweight Directory Access Protocol*) es un protocolo para consultar y modificar un **directorio**: una base de datos organizada en **árbol**, optimizada para **lecturas** (buscar usuarios, grupos, equipos...). Es la base de Active Directory (que añade mucha funcionalidad por encima) y de muchos sistemas de identidad.

### Estructura del árbol (DIT)

```
dc=empresa,dc=lab                    ← raíz (base DN)
├── ou=Usuarios
│   ├── uid=ana
│   ├── uid=luis
│   └── uid=marta
└── ou=Grupos
    ├── cn=sistemas
    └── cn=redes
```

### Vocabulario clave

| Término | Qué es |
| ------- | ------ |
| **Entrada** | Cada "nodo" del árbol (un usuario, un grupo...) |
| **DN** | *Distinguished Name*: la **dirección completa** de una entrada: `uid=ana,ou=Usuarios,dc=empresa,dc=lab` |
| **Base DN** | La raíz del árbol: `dc=empresa,dc=lab` |
| **Atributo** | Un dato de la entrada (`uid`, `cn`, `mail`...) |
| **objectClass** | El "tipo" de entrada: define qué atributos tiene |
| **LDIF** | Formato de texto para describir entradas (lo que usarás para añadirlas) |

Un usuario de Linux en LDAP usa, entre otras, las clases `inetOrgPerson` y `posixAccount`, con atributos `uidNumber`, `gidNumber` y `homeDirectory`.

Ejemplo de entrada LDIF (completa tú los datos):

```ldif
dn: uid=ana,ou=Usuarios,dc=empresa,dc=lab
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
uid: ana
cn: ???
sn: ???
uidNumber: ???
gidNumber: ???
homeDirectory: /home/ana
loginShell: /bin/bash
userPassword: ???
```

### Cómo se integra con el inicio de sesión de Linux

| Pieza | Para qué sirve |
| ----- | -------------- |
| **NSS** (*Name Service Switch*, `/etc/nsswitch.conf`) | Le dice a Linux **dónde buscar** usuarios y grupos (local y LDAP) |
| **PAM** | Gestiona **la autenticación** (comprobar la contraseña) |
| **nslcd** | Servicio del cliente que habla con el servidor LDAP |

### Comandos de este laboratorio

| Comando | Qué hace |
| ------- | -------- |
| `sudo apt install slapd ldap-utils` | Instala el **servidor** OpenLDAP y las herramientas |
| `sudo dpkg-reconfigure slapd` | Reconfigura el servidor (nombre de dominio, contraseña de administrador) |
| `slappasswd` | Genera una contraseña **cifrada** para meter en un LDIF |
| `ldapadd -x -D "cn=admin,dc=empresa,dc=lab" -W -f archivo.ldif` | **Añade** entradas (`-x` autenticación simple, `-D` quién eres, `-W` pide tu contraseña, `-f` el archivo) |
| `ldapsearch -x -H ldap://IP -b "dc=empresa,dc=lab" "(uid=ana)"` | **Busca** entradas (`-H` servidor, `-b` base de búsqueda, y al final el filtro) |
| `ldapmodify` / `ldapdelete` / `ldappasswd` | Modifica / borra / cambia contraseñas |
| `sudo slapcat` | Vuelca **toda** la base de datos en formato LDIF (útil para backups) |
| `sudo apt install libnss-ldapd libpam-ldapd` | En el **cliente**: integra LDAP con el sistema |
| `getent passwd ana` | Consulta un usuario **por todos los orígenes** (locales y LDAP) |
| `sudo pam-auth-update` | Activa módulos PAM (por ejemplo, **crear la carpeta personal** al primer login) |

> ⚠️ **LDAP sin cifrar** envía las contraseñas en **texto legible**. En este laboratorio, sobre una red aislada, lo usarás así para aprender; en producción **siempre** con TLS (reto 1).

---

## 🛠️ Qué debes hacer

### Parte A · El servidor

1. En el servidor, instala `slapd` y `ldap-utils`. Reconfigura con `dpkg-reconfigure slapd` usando el dominio **`empresa.lab`** y una organización a tu elección. Anota la contraseña del administrador.
2. Comprueba con `ldapsearch -x -b "dc=empresa,dc=lab"` que el árbol base existe.
3. Escribe un LDIF `estructura.ldif` que cree las dos **OU**: `Usuarios` y `Grupos`. Cárgalo con `ldapadd`.
4. Escribe un LDIF `usuarios.ldif` con **`ana`, `luis` y `marta`**: UID y GID distintos y coherentes (a partir del 10000), shell `/bin/bash`, y contraseña cifrada con `slappasswd`.
5. Escribe un LDIF `grupos.ldif` con los grupos **`sistemas`** (ana, luis) y **`redes`** (marta), con clase `posixGroup`.
6. Cárgalos, y comprueba con `ldapsearch` que puedes **buscar a cada usuario** y listar todos los de la OU.
7. Abre el firewall para LDAP (puerto 389/tcp) **solo para tu red de laboratorio**.

### Parte B · El cliente

8. En `ubuntu-cliente`, instala `libnss-ldapd` y `libpam-ldapd`. Durante la instalación te preguntará la **URI** del servidor, la **base** y los servicios en los que usar LDAP (`passwd`, `group`, `shadow`).
9. Comprueba con `getent passwd ana` que el cliente **ve a `ana`** sin que exista en el sistema local. Comprueba también `id ana`.
10. Activa la **creación automática de la carpeta personal** en el primer inicio de sesión (`pam-auth-update`).
11. **Inicia sesión** en el cliente como `ana` (por SSH desde tu ordenador o con `su - ana`) y comprueba que funciona con la contraseña de LDAP.
12. Comprueba que `ana` tiene **el mismo UID** en cliente y servidor.

### Parte C · Gestión del día a día

13. **Cambia la contraseña** de `marta` con `ldappasswd`, y comprueba que la antigua deja de funcionar.
14. **Modifica** una entrada con `ldapmodify` (por ejemplo, añade un `mail` a `ana`).
15. **Elimina** a `luis` con `ldapdelete` y comprueba que ya no puede iniciar sesión.
16. Haz un **backup** de la base de datos con `slapcat`.

---

## ✅ Comprobación

| Prueba | Qué debes ver |
| ------ | ------------- |
| `ldapsearch -x -b "dc=empresa,dc=lab" "(uid=ana)" cn uidNumber` (servidor) | La entrada de `ana` con sus datos |
| `getent passwd ana` (cliente) | Una línea con UID 10000+ y su carpeta `/home/ana` |
| `id ana` (cliente) | UID y grupos (`sistemas`) desde LDAP |
| `ssh ana@192.168.56.11` | Entra con la contraseña de LDAP y se crea `/home/ana` |
| `getent passwd luis` tras borrarlo | **Sin resultado** |
| `sudo slapcat \| head -20` | El LDIF con tu directorio |

---

## 🧩 Pistas

- Los LDIF son **muy estrictos**: una línea en blanco separa las entradas, **sin espacios** al final de línea y con los `dn` bien escritos. Si `ldapadd` da error, lee **qué línea** y **qué atributo** señala.
- Para la contraseña: `slappasswd -s 'tu_contraseña'` devuelve algo como `{SSHA}...`: eso va en `userPassword`.
- Si `getent` no ve a `ana`, revisa `/etc/nsswitch.conf`: en `passwd`, `group` y `shadow` debe aparecer `ldap` además de `files`.
- Si `nslcd` no conecta, comprueba la URI en `/etc/nslcd.conf` y reinicia el servicio (`sudo systemctl restart nslcd`).
- Si falla el login, mira el log de autenticación (`/var/log/auth.log`) en el cliente y en el servidor.

---

## 🚀 Retos extra

1. **Activa TLS** (LDAPS o `StartTLS`) con un certificado autofirmado. ¿Por qué es imprescindible en producción?
2. Prueba **SSSD** (`sssd-ldap`) como alternativa a `nslcd`: es lo que se usa hoy en la mayoría de empresas.
3. Usa **ACL de OpenLDAP** para que un usuario pueda cambiar **solo su propia contraseña**.
4. Repite la prueba de **NFS** del laboratorio 02 con los usuarios de LDAP: ¿se arreglan ahora los UID?
5. Compara con **Active Directory**: ¿qué tiene en común con tu directorio LDAP y qué añade AD (Kerberos, GPO, DNS integrado)?

---

## 📝 Qué documentar en tu `Solucion.md`

- Un dibujo del **árbol** (DIT) que has creado.
- Tus archivos LDIF con la explicación de cada atributo.
- Cómo se conectan **NSS y PAM** al cliente, con tus palabras.
- Una tabla **LDAP frente a Active Directory** con las similitudes y diferencias.

---

## 🧠 Lo que has practicado

`LDAP` · `DIT, DN, entradas y atributos` · `LDIF` · `OpenLDAP` · `ldapadd / ldapsearch / ldapmodify / ldapdelete` · `NSS y PAM` · `identidad centralizada` · `slapcat`
