# 🚑 Laboratorio 01 · Recuperar un sistema que no arranca

**Nivel:** 🔴 Experto · **Sistema:** 🐧 Linux · **Tiempo estimado:** 120 minutos

---

## 🎯 Escenario

Son las 8:00. Un servidor **no arranca** tras un reinicio de madrugada. No hay acceso por SSH, y en la consola solo ves un mensaje de error o un prompt extraño. El servicio está caído y tu jefe pregunta cuánto tardarás.

Tu misión es aprender **cómo arranca Linux** para saber **dónde mirar** y **cómo rescatar** un sistema en distintos tipos de fallo: un `fstab` roto, un GRUB dañado y una contraseña olvidada.

> ⚠️ **Haz snapshot y provoca los fallos de uno en uno**, restaurando el snapshot entre unos y otros. Tendrás que trabajar con la **consola de VirtualBox** y con una **ISO de Ubuntu Server** (modo *live*).

---

## 📖 Conceptos nuevos

### Cómo arranca Linux

| Etapa | Qué ocurre | Si falla... |
| ----- | ---------- | ----------- |
| **1. Firmware** (BIOS/UEFI) | Hace una comprobación inicial y busca un disco de arranque | "No bootable device" |
| **2. Gestor de arranque** (GRUB) | Muestra el menú y carga el kernel y el initramfs | Prompt `grub>` o `grub rescue>` |
| **3. Kernel + initramfs** | El kernel arranca y monta el sistema raíz con ayuda del *initramfs* | Pánico del kernel, "cannot mount root" |
| **4. `systemd`** (PID 1) | Arranca servicios y monta lo de `fstab` | **Modo emergencia** |
| **5. Objetivo (`target`)** | Llega al *target* final (`multi-user` o `graphical`) | Servicios que fallan |

> **Analogía:** es una carrera de relevos: cada etapa le pasa el testigo a la siguiente. Si se cae el testigo, **identificar en qué relevo ocurrió** ya es medio diagnóstico.

### Modos de rescate de `systemd`

| Modo | Qué ofrece |
| ---- | ---------- |
| **rescue** (`rescue.target`) | Sistema mínimo con la raíz montada, algunos servicios, y una consola de `root` |
| **emergency** (`emergency.target`) | **Lo mínimo posible**: solo la raíz, normalmente en **solo lectura** |

### Editar el arranque en GRUB

En el menú de GRUB, con `e` editas **temporalmente** (sin guardar) los parámetros del kernel (la línea `linux`). Dos ejemplos útiles:

| Parámetro añadido | Efecto |
| ----------------- | ------ |
| `systemd.unit=rescue.target` | Arranca en modo rescate |
| `init=/bin/bash` | En vez de `systemd`, lanza **directamente una shell de `root`** |

Con `init=/bin/bash` el sistema raíz suele quedar **solo lectura**: hay que remontarlo con `mount -o remount,rw /`.

> 🔒 **Lección de seguridad:** quien tenga **acceso físico o a la consola** puede recuperar un sistema... y también **saltarse la contraseña de `root`**. Por eso se protege GRUB con contraseña y se **cifra el disco** (reto extra).

### Rescate desde un medio externo (*live* + `chroot`)

Cuando el sistema instalado **no sirve ni para entrar en rescate**, se arranca desde una ISO en modo *live*, se **monta el disco dañado** y se entra en él con **`chroot`**, que "cambia la raíz" del terminal para trabajar **como si estuvieras dentro** del sistema roto:

```
mount /dev/sdaX /mnt                       # monta el sistema raíz roto
mount --bind /dev  /mnt/dev                # comparte los dispositivos del live
mount --bind /proc /mnt/proc
mount --bind /sys  /mnt/sys
chroot /mnt                                # ¡ahora "estás dentro"!
```

### Herramientas útiles

| Comando | Qué hace |
| ------- | -------- |
| `lsblk -f`, `blkid` | Discos, sistemas de ficheros y **UUID reales** (para corregir `fstab`) |
| `journalctl -xb` | Log del arranque actual con explicaciones (`-x`) |
| `systemctl --failed` | Units que han fallado |
| `mount -o remount,rw /` | Vuelve a montar la raíz en **lectura-escritura** |
| `passwd usuario` | Cambia una contraseña |
| `update-grub` | Regenera `/boot/grub/grub.cfg` |
| `grub-install /dev/sda` | Reinstala GRUB en el disco |
| `update-initramfs -u` | Regenera el initramfs |
| En el prompt de GRUB: `ls`, `set root=(hd0,gpt2)`, `linux /boot/vmlinuz-... root=/dev/sda2`, `initrd /boot/initrd.img-...`, `boot` | Arranque **manual** desde el prompt |

---

## 🛠️ Qué debes hacer

Resuelve **tres incidentes**. Para cada uno: **provócalo**, **diagnostica** con el método (en qué etapa falló), **arregla** y **verifica**.

### Incidente 1 · "Arranca en modo emergencia" (`fstab` roto)

**Síntoma:** al arrancar, tras un rato de espera, aparece *"You are in emergency mode"*.

<details>
<summary>🔧 Cómo provocarlo (no lo mires hasta tenerlo claro)</summary>

Añade a `/etc/fstab` una línea con un **UUID inventado** y **sin** `nofail`, por ejemplo:

```
UUID=1111-2222  /srv/inexistente  ext4  defaults  0  2
```

(`/srv/inexistente` debe existir como carpeta.) Reinicia.
</details>

**Objetivo:** devolver el sistema a un arranque normal desde el modo emergencia, **sin ISO**, y dejar `fstab` robusto (`nofail` donde proceda).

### Incidente 2 · "No sé la contraseña" (usuario bloqueado fuera)

**Síntoma:** tienes acceso a la consola pero **no recuerdas** la contraseña de tu usuario (ni la de `root`).

<details>
<summary>🔧 Cómo provocarlo</summary>

Cambia la contraseña de tu usuario por una **aleatoria que no apuntes**: `sudo passwd tuusuario`. Reinicia.
</details>

**Objetivo:** recuperar el acceso **editando el arranque en GRUB**, sin ISO, y fijar una contraseña nueva. Escribe después **qué medidas** impedirían a un intruso hacer esto sin ser detectado.

### Incidente 3 · "Se queda en `grub>`" (GRUB dañado)

**Síntoma:** al encender, solo aparece un prompt `grub>` o `grub rescue>`.

<details>
<summary>🔧 Cómo provocarlo</summary>

Renombra el archivo de configuración de GRUB:

```
sudo mv /boot/grub/grub.cfg /boot/grub/grub.cfg.roto
```

Reinicia. (Con UEFI, el comportamiento puede variar; si no te funciona, razona qué cambia y adapta el fallo.)
</details>

**Objetivo:** primero, **arranca manualmente** desde el prompt de GRUB (eligiendo la partición y el kernel a mano). Después, ya dentro del sistema, **repara** GRUB de forma permanente. Y si no logras arrancar así, repáralo **desde la ISO *live* con `chroot`**.

---

## ✅ Comprobación

| Prueba | Qué debes ver |
| ------ | ------------- |
| `systemctl is-system-running` | `running` (o `degraded` solo si lo explicas) |
| `findmnt --verify` | Sin errores en `fstab` |
| Iniciar sesión con la contraseña nueva | Funciona |
| Reiniciar tras reparar GRUB | Arranca solo, sin intervención |
| Tu documento de recuperación | Un **runbook** por incidente |

---

## 🧩 Pistas

- En **emergencia**, la raíz suele estar en solo lectura: antes de editar, `mount -o remount,rw /`.
- `blkid` te da los UUID **reales**: compara con los de `fstab`.
- En GRUB, `e` edita; **`Ctrl+X`** o **`F10`** arranca con tus cambios (no son permanentes).
- Si tras `init=/bin/bash` no puedes cambiar la contraseña, ¿has remontado la raíz con permisos de escritura? Y al terminar, no puedes usar `reboot` normal: investiga cómo reiniciar (`exec /sbin/init` o `reboot -f`, con cuidado).
- En el prompt de GRUB, `ls` lista discos y particiones; **tabulador** autocompleta nombres de kernels.
- Si ves `grub rescue>` en lugar de `grub>`, el modo es más limitado: necesitarás `set prefix` y `insmod normal` antes.

---

## 🚀 Retos extra

1. **Protege GRUB con contraseña** (`grub-mkpasswd-pbkdf2` y `/etc/grub.d/40_custom`) para que no se pueda editar el arranque. Comprueba que **ya no puedes** repetir el incidente 2.
2. Investiga el **cifrado de disco completo** (LUKS): ¿cómo cambiaría el incidente 2 si el disco está cifrado?
3. Provoca un **fallo de initramfs** (por ejemplo, borra el initrd de un kernel) y rescátalo con `chroot` + `update-initramfs`.
4. Ejecuta una **comprobación del sistema de ficheros** (`fsck`) sobre una partición **desmontada** y razona por qué nunca se hace en una montada.
5. Prepara una **ISO de rescate** personalizada o una checklist impresa de "qué hacer si no arranca" para tu futura empresa.

---

## 📝 Qué documentar en tu `Solucion.md`

- Un **runbook por incidente**: síntoma → diagnóstico (en qué etapa del arranque falla) → causa → pasos de recuperación → cómo prevenirlo.
- El **tiempo** que tardaste en cada uno.
- Una reflexión: *"acceso físico = game over"* y qué medidas lo mitigan.

---

## 🧠 Lo que has practicado

`proceso de arranque` · `GRUB` · `initramfs` · `modos rescue y emergency` · `live ISO + chroot` · `fstab y UUID` · `recuperación de contraseña` · `medidas contra acceso físico`
