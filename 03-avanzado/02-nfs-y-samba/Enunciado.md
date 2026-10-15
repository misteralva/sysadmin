# 📁 Laboratorio 02 · Compartir ficheros con NFS y Samba

**Nivel:** 🟠 Avanzado · **Sistema:** 🐧 Linux (y acceso desde Windows) · **Tiempo estimado:** 100 minutos

---

## 🎯 Escenario

La empresa tiene dos mundos: **servidores Linux** que necesitan acceder a una carpeta compartida, y **ordenadores Windows** cuyos usuarios quieren la misma información en su explorador de archivos. Para cada mundo hay un protocolo:

- **NFS** para Linux con Linux.
- **Samba** (protocolo **SMB**) para compartir con Windows, el mismo que usaste en el nivel 1 con las carpetas compartidas de Windows Server.

Tu misión es montar **los dos** en tu servidor Ubuntu.

> 📌 Necesitas el cliente `ubuntu-cliente` (un clon con IP `192.168.56.11`; ver `03-avanzado/README.md`) y los usuarios `ana`, `luis` y `marta` del laboratorio 03 del nivel 1 (recréalos si hace falta). Haz **snapshot** de las dos VMs.

---

## 📖 Conceptos nuevos

### Compartir ficheros por red

Un servidor **expone** una carpeta; los clientes la **montan** en su propio árbol de directorios y la usan como si fuera local. Es el mismo concepto de "montaje" del laboratorio 04 del nivel intermedio, pero el disco está en **otra máquina**.

### NFS: el estándar de Linux/Unix

**NFS** (*Network File System*) integra la carpeta remota en el sistema de ficheros del cliente de forma casi transparente. Se configura con el archivo **`/etc/exports`**:

```
/ruta/carpeta   red_o_equipo(opciones)
```

Opciones importantes:

| Opción | Significado |
| ------ | ----------- |
| `rw` / `ro` | Lectura y escritura / solo lectura |
| `sync` | El servidor confirma una escritura **solo cuando ya está en disco** (más seguro) |
| `no_subtree_check` | Evita comprobaciones innecesarias y problemas de rendimiento |
| `root_squash` (por defecto) | El `root` del cliente se convierte en un usuario sin privilegios en el servidor. **Protege**: evita que un administrador del cliente sea `root` en tu servidor |
| `no_root_squash` | Lo contrario. **Peligroso**, evítalo |

**Detalle crítico:** NFS identifica a los usuarios por su **UID numérico**. Si `ana` es UID 1001 en el servidor y UID 1002 en el cliente, los permisos no cuadrarán. Por eso en empresas se centralizan los usuarios (por ejemplo, con LDAP: laboratorio 07).

### Samba: SMB para interoperar con Windows

**Samba** implementa el protocolo **SMB/CIFS** en Linux. Permite que un Windows vea una carpeta de Linux como un recurso compartido normal (`\\servidor\recurso`). Se configura en **`/etc/samba/smb.conf`**.

Plantilla de un recurso (completa las `???`):

```ini
[Empresa]
   path = ???
   valid users = ???
   read only = ???
   browseable = yes
```

**Detalle crítico:** Samba tiene su **propia base de contraseñas**, separada de la de Linux. Un usuario debe existir en Linux **y además** darse de alta en Samba (`smbpasswd -a`).

### Comandos de este laboratorio

| Comando | Qué hace |
| ------- | -------- |
| `sudo apt install nfs-kernel-server` / `nfs-common` | Servidor NFS / cliente NFS |
| `sudo exportfs -ra` | Relee `/etc/exports` y aplica los cambios |
| `sudo exportfs -v` | Muestra lo que se está exportando |
| `showmount -e IP` | Desde un cliente, lista las carpetas que exporta un servidor |
| `sudo mount -t nfs IP:/ruta /mnt/punto` | Monta una carpeta NFS (`-t` = tipo) |
| `sudo apt install samba` | Instala Samba |
| `testparm` | **Valida** `smb.conf` y muestra la configuración efectiva |
| `sudo smbpasswd -a usuario` | Da de alta a un usuario Linux en Samba y fija su contraseña |
| `smbclient -L //IP -U usuario` | Lista los recursos de un servidor Samba |
| `sudo systemctl restart smbd` | Reinicia el servicio de Samba |
| `sudo ufw allow from 192.168.56.0/24 to any port 2049 proto tcp` | Abre NFS (versión 4 usa solo el puerto 2049) para tu red |
| `sudo ufw allow samba` | Abre los puertos de Samba (perfil predefinido de `ufw`) |

---

## 🛠️ Qué debes hacer

### Parte A · NFS

1. En el **servidor**, instala `nfs-kernel-server` y crea la carpeta `/srv/nfs/compartido` con algunos archivos de prueba.
2. **Expórtala** solo a la red `192.168.56.0/24`, con lectura/escritura, `sync` y `no_subtree_check`.
3. Aplica los cambios y comprueba con `exportfs -v` que se está exportando.
4. Abre el firewall para NFS **solo desde tu red de laboratorio**.
5. En el **cliente**, instala `nfs-common`, comprueba con `showmount -e` qué ofrece el servidor y **móntalo** en `/mnt/nfs`.
6. Escribe un archivo desde el cliente y comprueba que aparece en el servidor. Fíjate en **quién es el propietario** (UID) del archivo creado: ¿qué ha pasado con `root_squash`?
7. Haz el montaje **permanente** en `fstab` del cliente con la opción `_netdev` (que indica "es un disco de red, espera a que haya red") y `nofail`.

### Parte B · Samba

8. En el servidor, instala Samba y crea la carpeta `/srv/samba/empresa`.
9. Crea un recurso compartido `[Empresa]` accesible **solo** por `ana` y `luis`, con escritura.
10. Da de alta a `ana` y `luis` en Samba (con contraseña), valida con `testparm` y reinicia el servicio.
11. Abre el firewall para Samba.
12. Desde el cliente Linux, lista el servidor con `smbclient`, y conéctate como `ana`.
13. **Desde Windows** (tu ordenador real o `pc-cliente01`), abre `\\192.168.56.10\Empresa` en el Explorador e identifícate como `ana`. Crea un archivo.
14. Comprueba en el servidor el **propietario** del archivo creado desde Windows. Prueba a entrar con `marta`: debe **rechazarte**.

---

## ✅ Comprobación

| Prueba | Qué debes ver |
| ------ | ------------- |
| `sudo exportfs -v` (servidor) | La carpeta exportada con sus opciones |
| `showmount -e 192.168.56.10` (cliente) | `/srv/nfs/compartido 192.168.56.0/24` |
| `df -h /mnt/nfs` (cliente) | El montaje `192.168.56.10:/srv/nfs/compartido` |
| Archivo creado como `root` en NFS | En el servidor aparece como `nobody` (por `root_squash`) |
| `testparm` | Sin errores y con la sección `[Empresa]` |
| `smbclient -L //192.168.56.10 -U ana` | Lista el recurso `Empresa` |
| Windows → `\\192.168.56.10\Empresa` | `ana` entra; `marta`, no |

---

## 🧩 Pistas

- Si `mount` NFS se queda colgado, casi siempre es el **firewall** (puerto 2049) o la IP de la red en `/etc/exports`.
- No pongas **espacios** entre la red y las opciones en `exports`: `192.168.56.0/24(rw,...)` y no `192.168.56.0/24 (rw,...)` (con el espacio, significa algo muy distinto: abierto a todos).
- Si Windows no autentica, comprueba que la contraseña es la de `smbpasswd`, **no** la de Linux.
- Los permisos finales dependen de **dos capas**: los de Samba (`valid users`, `read only`) y los de Linux (propietario y modo de la carpeta). Revisa las dos.
- `testparm` es tu amigo: ejecútalo **siempre** antes de reiniciar Samba.

---

## 🚀 Retos extra

1. Haz que el recurso Samba use un **grupo** (`@sistemas`) en `valid users` en lugar de nombres sueltos, y que los archivos nuevos se creen con **el grupo** `sistemas` (investiga `force group` y el bit `setgid` del nivel 1).
2. Configura **`autofs`** para que el NFS se monte **bajo demanda** y se desmonte solo al no usarse.
3. Exporta una carpeta NFS en **solo lectura** a un segundo cliente y prueba a escribir.
4. Investiga cómo se uniría Samba a **Active Directory** para que los usuarios de `empresa.local` accedan sin cuentas locales.
5. Compara **NFS vs SMB**: ¿cuándo usarías cada uno? Haz una tabla.

---

## 📝 Qué documentar en tu `Solucion.md`

- Tu `/etc/exports` y tu sección `[Empresa]` de `smb.conf`, explicados línea a línea.
- La demostración de `root_squash` con tu salida.
- La diferencia entre **permisos de recurso** y **permisos de sistema de ficheros**.
- Tabla comparativa NFS frente a Samba.

---

## 🧠 Lo que has practicado

`NFS` · `/etc/exports` · `root_squash` · `UID y permisos remotos` · `Samba / SMB` · `smb.conf` · `smbpasswd` · `fstab con _netdev` · `firewall para servicios de red`
