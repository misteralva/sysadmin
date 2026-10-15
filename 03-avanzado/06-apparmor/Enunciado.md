# 🔒 Laboratorio 06 · Control de acceso obligatorio con AppArmor

**Nivel:** 🟠 Avanzado · **Sistema:** 🐧 Linux · **Tiempo estimado:** 90 minutos

---

## 🎯 Escenario

Los permisos de Linux (`rwx`, propietario, grupo) tienen un punto débil: si un programa se ejecuta como `root`, **puede hacer cualquier cosa**. Y si un atacante consigue controlar ese programa (por un fallo en él), **hereda todos sus privilegios**.

Tu misión es aprender a **confinar** un programa con **AppArmor**: limitar exactamente qué archivos puede tocar, de modo que **ni siquiera siendo `root`** pueda salirse de lo que le has permitido. Es el principio del **mínimo privilegio** aplicado a los programas.

> 💾 Haz **snapshot** antes de empezar.

---

## 📖 Conceptos nuevos

### DAC frente a MAC

| Modelo | Quién decide | Cómo funciona |
| ------ | ------------ | ------------- |
| **DAC** (*Discretionary Access Control*) | El **propietario** del archivo | Los permisos `rwx` que ya conoces. Si eres `root`, te saltas casi todo |
| **MAC** (*Mandatory Access Control*) | El **administrador del sistema**, con reglas globales | Además de los permisos normales, un **módulo del kernel** comprueba reglas que **ni el propio `root` puede cambiar** desde dentro del programa |

> **Analogía:** el DAC es la llave de tu despacho. El MAC es una **norma de la empresa** que dice "aunque tengas llave, **este empleado concreto** no puede entrar al archivo de nóminas".

### AppArmor

**AppArmor** es el sistema MAC que usa **Ubuntu** (en Red Hat/Fedora se usa **SELinux**, con otro enfoque). Funciona con **perfiles**: un archivo por programa, que lista qué puede hacer.

### Modos de un perfil

| Modo | Qué hace |
| ---- | -------- |
| **enforce** | **Bloquea** lo no permitido y lo registra |
| **complain** | **No bloquea**, solo **registra** lo que bloquearía (ideal para ir afinando un perfil) |
| **unconfined** | Sin perfil: el programa no está confinado |

### Dónde viven los perfiles

`/etc/apparmor.d/`. El nombre del archivo es la ruta del programa con los `/` convertidos en `.`: el perfil de `/usr/local/bin/micat` se llama `usr.local.bin.micat`.

### Sintaxis de un perfil

Completa las `???`:

```
#include <tunables/global>

/ruta/al/programa {
  #include <abstractions/base>

  ???    r,    # lectura
  ???    w,    # escritura
}
```

| Permiso | Significa |
| ------- | --------- |
| `r` | Leer |
| `w` | Escribir |
| `x` | Ejecutar (`ix` = ejecutar **heredando** el perfil) |
| Acabar en `,` | Todas las reglas terminan en coma |

`#include <abstractions/base>` aporta lo mínimo que cualquier programa necesita (librerías básicas, etc.).

### Comandos de este laboratorio

| Comando | Qué hace |
| ------- | -------- |
| `sudo aa-status` | Estado de AppArmor: perfiles cargados, en enforce y en complain |
| `sudo apt install apparmor-utils` | Instala las herramientas `aa-*` |
| `sudo aa-complain /ruta/perfil` / `aa-enforce /ruta/perfil` | Pasa un perfil a modo complain / enforce |
| `sudo apparmor_parser -r /etc/apparmor.d/archivo` | **Carga o recarga** un perfil |
| `sudo journalctl -k \| grep -i apparmor` | Mensajes de AppArmor del kernel (los `DENIED`) |
| `sudo dmesg \| grep -i "apparmor"` | Alternativa para ver bloqueos |
| `sudo aa-logprof` | Asistente que lee el log y **propone reglas** para actualizar un perfil |
| `sudo cp /usr/bin/cat /usr/local/bin/micat` | Copia un programa (lo usarás como "cobaya") |

---

## 🛠️ Qué debes hacer

### Parte A · Explorar lo que ya hay

1. Instala `apparmor-utils` y mira `aa-status`: ¿cuántos perfiles hay? ¿Cuántos en **enforce**? ¿Qué procesos están confinados?
2. Localiza en `/etc/apparmor.d/` algún perfil de un programa que conozcas (por ejemplo, `tcpdump`) y **léelo**: ¿qué permite?

### Parte B · Tu propio perfil

Vas a confinar un programa inofensivo, una **copia** de `cat`, para demostrar la idea.

3. Crea la copia `/usr/local/bin/micat`.
4. **Sin perfil**, comprueba que, como `root`, `micat` puede leer cualquier archivo: `sudo /usr/local/bin/micat /etc/hostname` y `sudo /usr/local/bin/micat /etc/shadow`.
5. Escribe el perfil `/etc/apparmor.d/usr.local.bin.micat` que permita **solo** leer `/etc/hostname`, sin nada más.
6. **Cárgalo en modo complain** primero, y ejecuta de nuevo las dos lecturas. ¿Qué ocurre? ¿Qué aparece en el log?
7. Pasa el perfil a **enforce** y repite.
8. Comprueba que ahora `micat /etc/shadow` es **denegado**, aunque lo ejecutes con `sudo` (es decir, como `root`).
9. Localiza el mensaje `DENIED` en el log y **léelo**: ¿qué operación, qué archivo y qué perfil?

### Parte C · Afinar

10. Permite **también** leer `/etc/os-release` y comprueba que ahora funciona.
11. Intenta que `micat` **escriba**: con redirección no (lo hace tu shell), pero puedes probar con `micat /etc/hostname > /tmp/x` (es la shell quien escribe: ¿entiendes por qué no se bloquea?).
12. Describe con tus palabras **qué protege** AppArmor aquí y **qué no**.

### Parte D · Un caso realista

13. Pon un perfil existente (por ejemplo el de `tcpdump`) en modo **complain**, ejecútalo (`sudo tcpdump -i lo -c 3`) y observa si se registra algo. Devuélvelo a **enforce**.

---

## ✅ Comprobación

| Prueba | Qué debes ver |
| ------ | ------------- |
| `sudo aa-status` | Tu perfil `/usr/local/bin/micat` en la lista de **enforce** |
| `sudo micat /etc/hostname` | Funciona |
| `sudo micat /etc/shadow` | `Permission denied` (aunque eres `root`) |
| `sudo journalctl -k \| grep -i 'apparmor="DENIED"'` | Una línea con `profile="/usr/local/bin/micat"` y `name="/etc/shadow"` |
| Perfil en complain | `micat /etc/shadow` **funciona**, pero se registra `ALLOWED` |

---

## 🧩 Pistas

- Cada vez que edites el perfil, **recárgalo** con `apparmor_parser -r`.
- Si `micat` no puede ni arrancar, falta algo básico: ¿has incluido `#include <abstractions/base>`?
- Si el perfil no parece aplicarse, comprueba que el **nombre del archivo** coincide con la ruta del programa (con puntos en vez de barras) y que ejecutas `micat` por su **ruta exacta**.
- Para vaciar la vista del log y leer solo lo nuevo: `sudo journalctl -k --since "2 minutes ago"`.
- `aa-logprof` automatiza la lectura del log y te pregunta qué reglas añadir. Es muy cómodo, pero **léelas antes de aceptar**.

---

## 🚀 Retos extra

1. Usa `aa-genprof /usr/local/bin/micat` para generar un perfil **de forma guiada** y compáralo con el tuyo.
2. Confina un **script** tuyo (por ejemplo, el de backup del laboratorio 09 del nivel intermedio) y descubre qué reglas necesita realmente.
3. Averigua para qué sirven las **abstracciones** (`/etc/apparmor.d/abstractions/`): abre `base` y `nameservice`.
4. Investiga cómo **SELinux** lo enfoca distinto (etiquetas en archivos y procesos frente a rutas). Haz una tabla comparativa.
5. Razona: si un atacante compromete `nginx` y `nginx` tiene un perfil AppArmor bien hecho, ¿qué le impide leer `/etc/shadow`?

---

## 📝 Qué documentar en tu `Solucion.md`

- Tu perfil completo, explicado línea a línea.
- Capturas del bloqueo de `/etc/shadow` y del mensaje `DENIED`.
- Con tus palabras: **qué aporta MAC** que no aportan los permisos normales.
- Qué pasaría **sin** AppArmor si `micat` fuera un programa real con un fallo explotable.

---

## 🧠 Lo que has practicado

`DAC frente a MAC` · `AppArmor` · `perfiles` · `enforce y complain` · `abstractions` · `mínimo privilegio` · `lectura de logs DENIED` · `apparmor_parser`
