# 🗝️ Laboratorio 08 · Autenticación SSH con claves

**Nivel:** 🟡 Intermedio · **Sistema:** 🐧 Linux · **Tiempo estimado:** 75 minutos

---

## 🎯 Escenario

En el laboratorio 06 viste cómo un servidor recibe intentos de acceso con contraseñas incorrectas, y en el 07 limitaste los reintentos. Pero una contraseña, por buena que sea, **se puede adivinar o robar**. En los servidores profesionales, SSH se protege de otra forma: **sin contraseña**, solo con **claves criptográficas**.

Tu misión es configurar el acceso por clave y **desactivar el acceso por contraseña**.

> ⚠️ **Riesgo de quedarte fuera.** Haz un **snapshot**, ten la **consola de VirtualBox** a mano, y **mantén siempre una sesión SSH abierta** mientras cambias la configuración. Necesitas la red solo-anfitrión del laboratorio 03.

---

## 📖 Conceptos nuevos

### Criptografía de clave pública (asimétrica)

Una pareja de claves matemáticamente enlazadas:

| Clave | Quién la tiene | Analogía |
| ----- | -------------- | -------- |
| **Pública** (`.pub`) | La repartes por donde quieras (servidores) | Un **candado** que cualquiera puede ver y copiar |
| **Privada** | **Solo tú**, nunca sale de tu equipo | La **llave** que abre ese candado |

Instalas tu candado (clave pública) en el servidor. Cuando te conectas, el servidor te "reta" a demostrar que tienes la llave correspondiente, **sin que la llave viaje por la red**. Por eso es mucho más seguro que una contraseña.

> 🚨 **La clave privada es como tu contraseña: no la compartas, no la subas a GitHub, no la mandes por correo.** Si alguien la consigue, es como si fuera tú.

### Passphrase

Puedes proteger la clave privada con una **frase de paso** (*passphrase*): si alguien roba el archivo, no puede usarlo sin ella.

### Dónde se guarda todo

| Archivo | Dónde | Qué es |
| ------- | ----- | ------ |
| `~/.ssh/id_ed25519` | **Tu ordenador** | Tu clave **privada** |
| `~/.ssh/id_ed25519.pub` | Tu ordenador | Tu clave **pública** |
| `~/.ssh/authorized_keys` | **El servidor** (en la carpeta del usuario) | Lista de claves públicas autorizadas a entrar |
| `/etc/ssh/sshd_config` y `/etc/ssh/sshd_config.d/` | El servidor | Configuración del servidor SSH |

### Permisos obligatorios

SSH **se niega a usar claves** si los permisos son demasiado abiertos:

- La carpeta `~/.ssh` del servidor: `700` (solo el propietario).
- El archivo `authorized_keys`: `600`.
- Tu clave privada: `600`.

### Comandos de este laboratorio

| Comando | Qué hace |
| ------- | -------- |
| `ssh-keygen -t ed25519 -C "comentario"` | Genera una pareja de claves (`ed25519` es un algoritmo moderno y seguro; `-C` añade un comentario identificativo) |
| `ssh-copy-id usuario@host` | Copia tu clave pública a `authorized_keys` del servidor (en Linux/macOS) |
| `ssh -i ruta/clave usuario@host` | Conecta usando una clave concreta |
| `ssh-keygen -lf clave.pub` | Muestra la **huella** de una clave |
| `sudo sshd -t` | **Comprueba** que la configuración de SSH es válida, sin reiniciar |
| `sudo systemctl reload ssh` | Recarga la configuración (sin cortar sesiones abiertas) |
| `sudo sshd -T \| grep -i passwordauthentication` | Muestra el valor **efectivo** de una directiva |
| `ssh-add` / `ssh-agent` | Guardan tu clave descifrada en memoria para no escribir la passphrase cada vez |
| `chmod 700 ~/.ssh` / `chmod 600 ~/.ssh/authorized_keys` | Ajustan los permisos |

---

## 🛠️ Qué debes hacer

### Parte A · Generar y copiar la clave

1. En **tu ordenador** (no en la VM), genera una clave `ed25519` con un comentario identificativo. **Ponle una passphrase.**
2. Localiza los dos archivos creados y comprueba cuál es la pública y cuál la privada.
3. **Copia la clave pública** al servidor (usuario de tu VM). En Linux/macOS, `ssh-copy-id`; en **Windows** no existe, así que tendrás que añadir el contenido de tu `.pub` al archivo `~/.ssh/authorized_keys` de la VM por otro medio (investiga cómo hacerlo con `ssh` y `cat`, o pegándolo con un editor).
4. En el servidor, comprueba los **permisos** de `~/.ssh` y de `authorized_keys` y corrígelos si hace falta.

### Parte B · Entrar con la clave

5. Cierra la sesión y **vuelve a conectar**: debe pedirte la **passphrase de la clave**, no la contraseña del usuario.
6. Observa en el servidor (`auth.log` o `journalctl -u ssh`) cómo se registra un acceso con clave: busca **"Accepted publickey"**.

### Parte C · Desactivar la contraseña

7. **Mantén abierta** la sesión SSH actual (y la consola de VirtualBox).
8. Edita la configuración del servidor para **desactivar la autenticación por contraseña**. Hazlo con un archivo propio en `/etc/ssh/sshd_config.d/` (por ejemplo `99-hardening.conf`) con la directiva `PasswordAuthentication no`.
9. **Valida** la configuración con `sshd -t` y, solo si no da errores, **recárgala**.
10. **Sin cerrar** la sesión anterior, abre **otra** desde tu ordenador y comprueba que entra con la clave.
11. Comprueba que **ya no es posible** entrar con contraseña: fuerza el intento (`ssh -o PubkeyAuthentication=no usuario@host`). Debe rechazarte sin siquiera pedirla.
12. Comprueba el valor efectivo con `sshd -T`.

### Parte D · Más de una clave

13. Genera una **segunda** pareja de claves y autorízala también en el servidor. Comprueba que ambas funcionan.
14. **Revoca** la segunda: bórrala de `authorized_keys` y comprueba que deja de funcionar.

---

## ✅ Comprobación

| Prueba | Qué debes ver |
| ------ | ------------- |
| `ls -la ~/.ssh` (servidor) | `drwx------` en `.ssh` y `-rw-------` en `authorized_keys` |
| `ssh usuario@192.168.56.10` | Pide la **passphrase de tu clave** y entra |
| `sudo grep "Accepted publickey" /var/log/auth.log \| tail -n 2` | Tu acceso con clave |
| `ssh -o PubkeyAuthentication=no usuario@192.168.56.10` | `Permission denied (publickey)`: sin pedir contraseña |
| `sudo sshd -T \| grep -i passwordauthentication` | `passwordauthentication no` |
| Clave revocada | `Permission denied (publickey)` |

---

## 🧩 Pistas

- Si SSH te sigue pidiendo la contraseña del usuario tras copiar la clave, casi seguro son los **permisos** de `~/.ssh` o de `authorized_keys`, o la clave está mal pegada (una sola línea).
- Windows (PowerShell): tu clave suele estar en `$HOME\.ssh\id_ed25519`. Para ver su contenido: `Get-Content $HOME\.ssh\id_ed25519.pub`.
- `authorized_keys` debe tener **una clave por línea**, sin saltos de línea dentro de la clave.
- En Ubuntu, algunos archivos de `/etc/ssh/sshd_config.d/` se leen **antes** que el principal y el **primero que fija una directiva gana**: si el valor efectivo no es el esperado, revisa qué archivo lo está fijando.
- Si te quedas fuera: consola de VirtualBox → corrige el archivo → `sudo systemctl reload ssh`, o restaura el snapshot.

---

## 🚀 Retos extra

1. Usa `ssh-agent` y `ssh-add` para teclear la passphrase **una sola vez** por sesión.
2. Crea un archivo `~/.ssh/config` con un alias para tu servidor (`Host lab`, `HostName`, `User`, `IdentityFile`) y conéctate solo con `ssh lab`.
3. Añade `PermitRootLogin no` y `AllowUsers tuusuario` a tu configuración. ¿Qué mejora de seguridad aporta cada una?
4. Restringe una clave para que **solo pueda ejecutar un comando concreto** (opción `command="..."` delante de la clave en `authorized_keys`). ¿Para qué se usa en la práctica?
5. Compara: vuelve a mirar el ranking de intentos fallidos del laboratorio 06. ¿Por qué con `PasswordAuthentication no` los ataques de fuerza bruta por contraseña **dejan de ser un riesgo real**?

---

## 📝 Qué documentar en tu `Solucion.md`

- Los comandos usados (explicados), **sin pegar nunca tu clave privada**.
- La huella de tu clave y qué te indica.
- El orden de pasos que seguiste para **no quedarte fuera**.
- Una explicación con tus palabras de por qué una clave es más segura que una contraseña.

---

## 🧠 Lo que has practicado

`criptografía de clave pública` · `ssh-keygen` · `authorized_keys` · `permisos de ~/.ssh` · `sshd_config` · `sshd -t` · `desactivar contraseñas` · `revocar claves` · `hardening de SSH`
