# 🛟 Laboratorio 01 · RAID por software con `mdadm`

**Nivel:** 🟠 Avanzado · **Sistema:** 🐧 Linux · **Tiempo estimado:** 90 minutos

---

## 🎯 Escenario

Los discos se estropean, es cuestión de tiempo. Si el disco que guarda los datos de Sistemas falla, el servicio cae y se pierde el trabajo desde la última copia. La solución es la **redundancia**: guardar los datos en **dos discos a la vez**, de modo que si uno muere, el otro sigue funcionando.

Tu misión es montar un **RAID 1** (espejo) por software, **romperlo a propósito** simulando un fallo de disco y **reconstruirlo** con un disco nuevo, sin perder datos.

> ⚠️ Vas a trabajar con discos nuevos y a "romperlos". **Haz snapshot**, añade discos **vacíos** para este laboratorio y **comprueba cada nombre por su tamaño** antes de usarlo. Nunca uses los discos que ya tienen datos (`sda`, ni los de los laboratorios 04 y 05 del nivel intermedio).

---

## 📖 Conceptos nuevos

### Qué es RAID

**RAID** (*Redundant Array of Independent Disks*) combina varios discos para conseguir **más velocidad, más seguridad o ambas cosas**. Puede hacerse con un controlador físico (*hardware RAID*) o por **software**, gestionado por el sistema operativo. En Linux, el software RAID se maneja con **`mdadm`**.

### Niveles más usados

| Nivel | Cómo funciona | Discos mínimos | Capacidad útil | Tolera |
| ----- | ------------- | -------------- | -------------- | ------ |
| **RAID 0** | Reparte los datos entre discos (*striping*). **Sin** redundancia | 2 | Suma de todos | **Nada**: si falla uno, todo se pierde |
| **RAID 1** | **Espejo**: los mismos datos en dos discos | 2 | Un disco | Fallo de 1 disco |
| **RAID 5** | Reparto con **paridad** | 3 | (n-1) discos | Fallo de 1 disco |
| **RAID 6** | Como RAID 5 con doble paridad | 4 | (n-2) discos | Fallo de 2 discos |
| **RAID 10** | Espejos que se reparten (1+0) | 4 | La mitad | 1 disco por espejo |

> **Analogía del RAID 1:** tienes dos copias idénticas de un libro en dos estanterías distintas. Si una estantería se cae, la otra sigue ahí.

### ⚠️ RAID no es un backup

Si alguien **borra** un archivo, se borra en **los dos discos a la vez**. RAID te protege de un **fallo de hardware**, no de errores humanos, ransomware ni corrupciones. Los backups del laboratorio 09 del nivel intermedio **siguen siendo necesarios**.

### Estados del array

| Estado | Significado |
| ------ | ----------- |
| **clean / active** | Todo en orden |
| **degraded** | Falta un disco: sigue funcionando, pero **sin protección** |
| **recovering / resync** | Reconstruyendo (copiando los datos al disco nuevo) |

### Comandos de este laboratorio

| Comando | Qué hace |
| ------- | -------- |
| `sudo apt install mdadm` | Instala la herramienta |
| `sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdX /dev/sdY` | **Crea** un RAID 1 llamado `md0` con dos discos (`--level` = nivel, `--raid-devices` = cuántos discos) |
| `cat /proc/mdstat` | Estado en vivo de los arrays. `[UU]` = ambos discos bien; `[U_]` = falta uno |
| `sudo mdadm --detail /dev/md0` | Detalle completo del array |
| `sudo mdadm --examine /dev/sdX` | Lee los metadatos RAID de un disco |
| `sudo mdadm /dev/md0 --fail /dev/sdY` | **Simula** que un disco falla |
| `sudo mdadm /dev/md0 --remove /dev/sdY` | **Retira** el disco fallido |
| `sudo mdadm /dev/md0 --add /dev/sdZ` | **Añade** un disco nuevo (comienza la reconstrucción) |
| `sudo mdadm --detail --scan` | Genera la línea de configuración del array |
| `watch -n 1 cat /proc/mdstat` | Observa el progreso cada segundo (`Ctrl+C` para salir) |

---

## 🛠️ Qué debes hacer

### Parte A · Crear el RAID 1

1. **Apaga** la VM y añade **tres discos virtuales nuevos de 2 GB** cada uno. **Enciende** y localízalos con `lsblk` **por su tamaño**.
2. Instala `mdadm`.
3. Crea un **RAID 1** con **dos** de los discos nuevos. Deja el tercero libre (será tu "disco de repuesto").
4. Observa en `/proc/mdstat` la sincronización inicial y espera a que termine.
5. Mira el detalle del array: estado, nivel, tamaño y los discos que lo forman.

### Parte B · Usarlo

6. Crea un sistema de ficheros `ext4` sobre **`/dev/md0`**, móntalo en `/srv/raid` y copia algunos datos de prueba (por ejemplo, un archivo de 500 MB y otros pequeños). Calcula su `sha256sum`.
7. Haz que **se monte al arrancar**: añádelo a `fstab` **por UUID**.
8. Haz que el array **se reconozca al arrancar**: guarda su configuración en `/etc/mdadm/mdadm.conf` (con `mdadm --detail --scan`) y actualiza el `initramfs` (`sudo update-initramfs -u`).

### Parte C · Romperlo y repararlo

9. **Simula el fallo** de uno de los dos discos y observa el estado `[U_]` y `degraded`.
10. **Comprueba que los datos siguen accesibles** (lee los archivos y compara los `sha256sum`).
11. **Retira** el disco fallido.
12. **Añade el disco de repuesto** y observa la **reconstrucción** hasta `[UU]`.
13. Comprueba otra vez la integridad de los datos.
14. **Reinicia** la VM y verifica que el array y el montaje siguen intactos.

---

## ✅ Comprobación

| Prueba | Qué debes ver |
| ------ | ------------- |
| `cat /proc/mdstat` | `md0 : active raid1 ...` con `[UU]` |
| `sudo mdadm --detail /dev/md0 \| grep -E "State\|Level"` | `Raid Level : raid1` y `State : clean` |
| Tras el fallo simulado | `[U_]` y `degraded`; los datos siguen leyéndose |
| `sha256sum` antes y después | **Idénticos** |
| Tras añadir el repuesto | Estado `recovering` y luego `clean`, `[UU]` |
| Tras reiniciar | `/srv/raid` montado y los datos presentes |

---

## 🧩 Pistas

- Si `mdadm --create` pregunta algo sobre metadatos, acepta; si se queja de que el disco está en uso, **comprueba que es el correcto** con `lsblk`.
- Si el array se llama `md127` tras reiniciar en lugar de `md0`, es que la configuración no se guardó bien en `mdadm.conf`.
- Para evitar errores de arranque por un array que no aparece, usa `nofail` en `fstab`.
- La reconstrucción de discos pequeños es rápida; usa `watch` para verla.
- Para ver qué pasa en vivo, abre dos terminales: en una, `watch cat /proc/mdstat`; en la otra, lanza los comandos.

---

## 🚀 Retos extra

1. Crea un **RAID 5** con tres discos (necesitarás otro disco más) y compara su capacidad con la del RAID 1. ¿Cuánto espacio "pierdes"?
2. Añade el tercer disco como **repuesto activo** (*spare*) del RAID 1 desde el principio: cuando falle un disco, ¿se reconstruye **solo**?
3. Investiga `mdadm --monitor` y cómo recibir un **aviso por correo** cuando un disco falla.
4. **Detén** el array (`--stop`) y vuélvelo a **ensamblar** (`--assemble`). ¿Para qué te serviría si cambias los discos de máquina?
5. Monta **LVM sobre RAID** (el RAID como PV): combina lo que has aprendido en el laboratorio 05 del nivel intermedio. ¿Qué ventajas tiene?

---

## 📝 Qué documentar en tu `Solucion.md`

- Un esquema de tu RAID con los discos implicados.
- La salida de `mdstat` en **cada fase** (sano, degradado, reconstruyendo, sano).
- Una explicación con tus palabras de por qué **RAID no es un backup** y qué falta para estar realmente protegido.

---

## 🧠 Lo que has practicado

`RAID 0/1/5/6/10` · `mdadm` · `/proc/mdstat` · `array degradado` · `reconstrucción` · `mdadm.conf` · `fstab con UUID` · `integridad con sha256sum`
