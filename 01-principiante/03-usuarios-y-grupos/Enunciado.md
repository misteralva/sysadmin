# 👥 Laboratorio 03 · Usuarios y grupos

**Nivel:** 🟢 Principiante · **Sistema:** 🐧 Linux · **Tiempo estimado:** 60 minutos

---

## 🎯 Escenario

Llegan tres empleados nuevos a la empresa y te toca darles acceso al servidor:

| Empleado | Departamento | Necesita |
| -------- | ------------ | -------- |
| `ana` | Sistemas | Acceso normal + poder administrar el servidor con `sudo` |
| `luis` | Sistemas | Acceso normal |
| `marta` | Redes | Acceso normal |

Más adelante, `luis` dejará la empresa y habrá que eliminar su cuenta correctamente.

> 💾 **Antes de empezar:** haz un snapshot de la VM. Gestionar usuarios mal puede dejarte sin acceso al sistema.

---

## 📖 Conceptos nuevos

### Usuarios y grupos

Piensa en una empresa. Cada empleado tiene una **credencial personal** (el *usuario*) y pertenece a uno o varios **departamentos** (los *grupos*). Los permisos se dan normalmente al departamento, no a cada persona.

- Cada usuario tiene un número único: el **UID**.
- Cada grupo tiene el suyo: el **GID**.
- Cada usuario tiene un **grupo principal** y puede tener **grupos secundarios**.
- El usuario **`root`** (UID 0) es el superadministrador: puede hacer cualquier cosa. Por eso casi nunca se trabaja como `root`: se usa `sudo` solo cuando hace falta.

### Usuarios normales y de sistema

Los usuarios con UID **a partir de 1000** son personas. Los de UID más bajo son **cuentas de sistema**: las usan servicios (por ejemplo, el servidor web) y normalmente no pueden iniciar sesión.

### Dónde se guarda todo

| Archivo | Contiene |
| ------- | -------- |
| `/etc/passwd` | La lista de usuarios (UID, grupo principal, carpeta personal, shell). **No** contiene contraseñas, a pesar del nombre |
| `/etc/shadow` | Las contraseñas **cifradas**. Solo `root` puede leerlo |
| `/etc/group` | La lista de grupos y sus miembros |

### Comandos de este laboratorio

| Comando | Qué hace |
| ------- | -------- |
| `whoami` | Dice con qué usuario estás trabajando |
| `id usuario` | Muestra UID, GID y todos los grupos de un usuario |
| `groups usuario` | Lista solo los grupos de un usuario |
| `sudo comando` | Ejecuta ese comando con privilegios de administrador |
| `groupadd grupo` | Crea un grupo |
| `useradd` | Crea un usuario. Opciones clave: `-m` (crea su carpeta personal), `-s ruta` (su shell), `-G grupos` (grupos secundarios) |
| `passwd usuario` | Establece o cambia la contraseña de un usuario |
| `usermod` | Modifica un usuario. Opción clave: `-aG grupo` (**a**ñade al grupo sin quitarle los demás) |
| `userdel` | Elimina un usuario. Con `-r` borra también su carpeta personal |
| `getent group grupo` | Consulta un grupo en el sistema |

> ⚠️ **Trampa clásica:** `usermod -G grupo usuario` (sin la `a`) **reemplaza** todos los grupos secundarios del usuario por ese único grupo. Con `-aG` solo se **añade**. Esta confusión ha quitado permisos a más de un administrador.

---

## 🛠️ Qué debes hacer

1. **Comprueba** con qué usuario estás trabajando y a qué grupos perteneces.
2. **Crea los grupos** `sistemas` y `redes`.
3. **Crea los usuarios** `ana`, `luis` y `marta`, cada uno con su carpeta personal y la shell `/bin/bash`.
4. **Asigna una contraseña** a cada uno.
5. **Añade a los grupos** correspondientes:
   - `ana` y `luis` → grupo `sistemas`
   - `marta` → grupo `redes`
6. **Da permisos de administración** a `ana`. Pista: en Ubuntu, quien pertenece al grupo `sudo` puede usar `sudo`.
7. **Inicia sesión como `ana`** y comprueba que puede ejecutar un comando con `sudo`. Haz lo mismo con `luis` y comprueba que **no puede**.
8. **Examina** los archivos `/etc/passwd` y `/etc/group` y localiza las líneas de tus nuevos usuarios y grupos.
9. **Elimina a `luis`** junto con su carpeta personal.

---

## ✅ Comprobación

| Comando | Qué hace | Qué debes ver |
| ------- | -------- | ------------- |
| `id ana` | Datos y grupos de `ana` | Pertenece a `sistemas` y a `sudo` |
| `id marta` | Datos y grupos de `marta` | Pertenece a `redes` |
| `getent group sistemas` | Miembros del grupo | `ana` y `luis` (antes de eliminar a luis) |
| `grep ana /etc/passwd` | Busca la línea de `ana` en el archivo (`grep` filtra líneas por texto) | Una línea con su UID, carpeta y shell |
| `ls /home` | Lista las carpetas personales | `ana` y `marta`, pero no `luis` al final |
| `sudo -l -U luis` | Muestra qué puede ejecutar `luis` con `sudo` | Que no tiene permisos |

Para cambiar de usuario en la terminal puedes usar `su - ana` (`su` = *switch user*; el `-` carga su entorno completo). Para volver, escribe `exit`.

---

## 🧩 Pistas

- `useradd` por sí solo **no** crea la carpeta personal. Si te olvidas de `-m`, el usuario existirá pero sin `/home/usuario`.
- Hay un comando más amigable, `adduser`, que te va preguntando los datos. Pero `useradd` es el que verás en scripts y documentación, así que practícalo.
- Si te equivocas en algo, `userdel -r usuario` y vuelve a empezar.
- Con `man useradd` tienes todas las opciones.

---

## 🚀 Retos extra

1. Crea un usuario **sin posibilidad de iniciar sesión** (shell `/usr/sbin/nologin`). ¿Para qué se usan en la práctica?
2. **Bloquea temporalmente** la cuenta de `marta` y comprueba que ya no puede entrar. Pista: busca `usermod -L` y su contrario `-U`.
3. Investiga `chage -l ana` y haz que su contraseña **caduque** a los 90 días.
4. Usa `last` para ver quién ha iniciado sesión últimamente.
5. En `/etc/passwd`, ¿qué significa cada uno de los campos separados por `:`?

---

## 📝 Qué documentar en tu `Solucion.md`

- Los comandos usados, con su explicación.
- La salida de `id` para cada usuario.
- Qué pasó cuando `luis` intentó usar `sudo` y cómo lo interpretas.
- Qué errores cometiste (¿se te olvidó `-m`?, ¿usaste `-G` sin `-a`?).

---

## 🧠 Lo que has practicado

`UID/GID` · `/etc/passwd` · `/etc/shadow` · `/etc/group` · `groupadd` · `useradd` · `passwd` · `usermod -aG` · `userdel` · `id` · `su` · `sudo`
