# 💽 Laboratorio 04 · Discos, particiones y `fstab`

**Nivel:** 🟡 Intermedio · **Sistema:** 🐧 Linux · **Tiempo estimado:** 75 minutos

---

## 🎯 Escenario

El servidor se está quedando sin espacio, y el departamento de Sistemas necesita un **disco nuevo** para guardar sus proyectos. Tu misión es añadirlo, prepararlo y conseguir que se **monte solo** cada vez que el servidor arranque.

> ⚠️ **Este laboratorio puede destruir datos si te equivocas de disco.** Haz un **snapshot** antes de empezar y **comprueba dos veces** el nombre del dispositivo antes de formatear nada.

---

## 📖 Conceptos nuevos

### Cuatro capas

Piensa en un terreno:

| Capa | Qué es | Analogía |
| ---- | ------ | -------- |
| **Disco** | El dispositivo físico (o virtual) | El terreno entero |
| **Partición** | Un trozo del disco | Una parcela |
| **Sistema de ficheros** | La forma de organizar los datos dentro (`ext4`, `xfs`...) | El sistema de archivo y numeración de cada parcela |
| **Montaje** | Conectar la partición a un directorio del árbol | Abrir una calle que llegue a la parcela |

Hasta que no está **montada**, una partición no se puede usar: no aparece en ninguna carpeta.

### Nombres de dispositivo

Los discos se llaman `/dev/sda`, `/dev/sdb`, `/dev/sdc`... Y sus particiones llevan número: `/dev/sdb1`, `/dev/sdb2`. El **primer disco** suele ser `sda` (el del sistema); el nuevo será `sdb`, **pero compruébalo siempre por el tamaño**.

> 💡 Esos nombres **pueden cambiar** entre reinicios (si añades o quitas discos). Por eso en `fstab` se usa el **UUID**, un identificador único que no cambia.

### `/etc/fstab`

Es la lista de sistemas de ficheros que se montan **automáticamente al arrancar**. Cada línea tiene 6 campos:

```
UUID=xxxx-xxxx   /srv/proyectos   ext4   defaults,nofail   0   2
   (1)               (2)           (3)        (4)         (5) (6)
```

| Campo | Qué es |
| ----- | ------ |
| 1 | Qué montar (por UUID) |
| 2 | Dónde montarlo (el directorio debe existir) |
| 3 | Tipo de sistema de ficheros |
| 4 | Opciones (`defaults` = las normales; `nofail` = si el disco falla, no impide arrancar) |
| 5 | Copia con `dump` (casi siempre `0`) |
| 6 | Orden de comprobación del disco al arrancar (`1` para `/`, `2` para los demás, `0` para no comprobar) |

> ⚠️ **Un error en `fstab` puede impedir que el servidor arranque.** Por eso siempre se prueba con `sudo mount -a` **antes** de reiniciar.

### Comandos de este laboratorio

| Comando | Qué hace |
| ------- | -------- |
| `lsblk` | Muestra discos y particiones en forma de árbol (`-f` añade sistema de ficheros y UUID) |
| `sudo fdisk -l` | Lista los discos con detalles |
| `sudo fdisk /dev/sdb` | Abre el particionador interactivo del disco (`m` = ayuda, `n` = nueva partición, `p` = mostrar, `w` = **guardar y salir**, `q` = salir sin guardar) |
| `sudo mkfs.ext4 /dev/sdb1` | **Formatea** la partición con `ext4` (borra lo que haya) |
| `sudo mkdir -p /srv/proyectos` | Crea el punto de montaje |
| `sudo mount /dev/sdb1 /srv/proyectos` | Monta ahora (temporal) |
| `sudo umount /srv/proyectos` | Desmonta |
| `sudo blkid /dev/sdb1` | Muestra el UUID y el tipo |
| `sudo mount -a` | Monta todo lo que dice `fstab` (sirve para **probarlo**) |
| `df -h` | Espacio libre por sistema de ficheros |
| `findmnt /srv/proyectos` | Muestra qué hay montado en esa ruta |

---

## 🛠️ Qué debes hacer

### Parte A · Añadir el disco

1. **Apaga** la VM.
2. En VirtualBox, añade un **nuevo disco virtual de 5 GB** (en *Almacenamiento → Controlador SATA → añadir disco duro*, creando uno nuevo).
3. **Enciende** la VM y con `lsblk` localiza el disco nuevo **por su tamaño**. Anota su nombre.

### Parte B · Particionar y formatear

4. Crea **una partición** que ocupe todo el disco (tabla de particiones `GPT`, la opción por defecto).
5. Formatea la partición con `ext4`.
6. Comprueba con `lsblk -f` que ahora tiene sistema de ficheros y UUID.

### Parte C · Montaje manual

7. Crea el directorio `/srv/proyectos` y **móntala** a mano. Comprueba con `df -h`.
8. Crea un archivo dentro, **desmonta** y comprueba que el archivo "desaparece" (en realidad está en el disco, no en la carpeta). Vuelve a montar y comprueba que reaparece.

### Parte D · Montaje automático

9. Obtén el **UUID** de la partición.
10. Añade una línea en `/etc/fstab` para montarla automáticamente en `/srv/proyectos`, con `nofail`.
11. **Antes de reiniciar**, desmonta y prueba con `sudo mount -a`. No debe dar errores.
12. **Reinicia** y comprueba que el disco se ha montado solo.

### Parte E · Un fallo a propósito

13. Equivócate **adrede** en una letra del UUID de `fstab`, ejecuta `sudo mount -a` y lee el error. Arréglalo.

---

## ✅ Comprobación

| Prueba | Qué debes ver |
| ------ | ------------- |
| `lsblk -f` | `sdb1` con `ext4`, su UUID y el punto de montaje `/srv/proyectos` |
| `df -h /srv/proyectos` | Una partición de unos 5 GB montada ahí |
| `findmnt /srv/proyectos` | El dispositivo y el tipo `ext4` |
| `sudo mount -a; echo $?` | `0` (sin errores; `$?` muestra el código de salida del último comando) |
| Tras reiniciar | El disco sigue montado y el archivo que creaste, presente |

---

## 🧩 Pistas

- En `fdisk`, `n` crea la partición: acepta los valores por defecto (Enter) y verás que ocupa todo el disco. **Nada se escribe hasta que pulses `w`.**
- Si `mkfs` te dice que el dispositivo está en uso, comprueba que no está montado.
- Si `mount` dice "wrong fs type", probablemente no has formateado la partición o te equivocas de dispositivo.
- Para extraer solo el UUID: `sudo blkid -s UUID -o value /dev/sdb1`.
- Si te equivocas en `fstab` y la VM no arranca, usa el **snapshot** o entra en modo recuperación.

---

## 🚀 Retos extra

1. Crea una **segunda partición** con `xfs` en otro disco (o dividiendo el primero) y compara `ext4` con `xfs`.
2. Pon una **etiqueta** al sistema de ficheros (`e2label` o `tune2fs -L`) y monta usando `LABEL=` en vez de UUID. ¿Qué ventaja tiene el UUID?
3. Intenta desmontar mientras estás **dentro** del directorio. ¿Qué error da? Investiga `lsof` o `fuser` para averiguar quién lo usa.
4. Crea un **archivo de intercambio (swap)** de 512 MB y actívalo (`fallocate`, `mkswap`, `swapon`). ¿Para qué sirve el swap?
5. Usa `du -sh /var/*` para averiguar qué directorio de `/var` ocupa más espacio.

---

## 📝 Qué documentar en tu `Solucion.md`

- Los comandos de `fdisk` paso a paso y qué significa cada tecla que pulsaste.
- Tu línea de `fstab` explicada campo por campo.
- Un esquema de las cuatro capas aplicado a tu disco.
- El error de `mount -a` que provocaste y cómo lo leíste.

---

## 🧠 Lo que has practicado

`disco / partición / sistema de ficheros / montaje` · `fdisk` · `mkfs` · `mount / umount` · `blkid` · `UUID` · `/etc/fstab` · `mount -a` · `nofail`
