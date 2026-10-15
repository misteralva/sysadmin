# 🧱 Laboratorio 05 · LVM: volúmenes que crecen

**Nivel:** 🟡 Intermedio · **Sistema:** 🐧 Linux · **Tiempo estimado:** 75 minutos

---

## 🎯 Escenario

Los proyectos de Sistemas crecen más rápido de lo previsto. En el laboratorio anterior montaste un disco con una partición **de tamaño fijo**: si se llena, no hay forma cómoda de ampliarla. Tu misión es usar **LVM** para crear un volumen que puedas **agrandar en caliente**, sin desmontar nada y sin parar el servidor.

> 💾 Haz un **snapshot** antes de empezar. Vas a trabajar con discos nuevos: **comprueba el nombre de cada disco por su tamaño** antes de usarlo.

---

## 📖 Conceptos nuevos

### El problema de las particiones fijas

Una partición clásica tiene un tamaño decidido de antemano. Ampliarla implica tocar la tabla de particiones, y si hace falta usar otro disco, es muy incómodo.

### LVM: *Logical Volume Manager*

LVM añade una **capa intermedia** entre los discos y los sistemas de ficheros. Analogía: imagina **depósitos de agua**.

| Concepto | Siglas | Qué es | Analogía |
| -------- | ------ | ------ | -------- |
| **Volumen físico** | PV | Un disco (o partición) entregado a LVM | Un depósito |
| **Grupo de volúmenes** | VG | Un "pozo común" formado por uno o varios PV | Todos los depósitos conectados entre sí |
| **Volumen lógico** | LV | Un trozo del VG, del tamaño que tú quieras. Es sobre lo que pones el sistema de ficheros | Una botella que llenas del pozo |

La gracia: **puedes añadir más depósitos al pozo** (más discos) y **agrandar las botellas** cuando quieras.

```
  disco sdc      disco sdd
     │              │
    [PV]           [PV]       ← volúmenes físicos
      └──────┬───────┘
           [VG]               ← grupo de volúmenes (un "pozo" de 6 GB)
        ┌────┴─────┐
      [LV]       (libre)      ← volumen lógico (2 GB) + espacio sin usar
        │
   sistema de ficheros → montado en /srv/datos
```

### Comandos de este laboratorio

| Comando | Qué hace |
| ------- | -------- |
| `sudo pvcreate /dev/sdc` | Convierte un disco en volumen físico |
| `sudo vgcreate nombre /dev/sdc ...` | Crea un grupo de volúmenes con esos PV |
| `sudo lvcreate -L 2G -n nombre vg` | Crea un LV de 2 GB llamado `nombre` dentro del grupo `vg` (`-L` = tamaño exacto, `-n` = nombre) |
| `sudo pvs` / `vgs` / `lvs` | Resumen de PV / VG / LV (con `pvdisplay`, `vgdisplay`, `lvdisplay` el detalle) |
| `sudo vgextend vg /dev/sdd` | Añade otro disco al grupo |
| `sudo lvextend -r -L +2G /dev/vg/lv` | **Amplía** el LV en 2 GB y, con `-r`, **también el sistema de ficheros** en el mismo paso |
| `lsblk` | Árbol de discos, con los LV |

Los LV aparecen como `/dev/NOMBRE_VG/NOMBRE_LV` (por ejemplo `/dev/vg_datos/lv_proyectos`).

> ⚠️ **Ampliar es fácil y seguro. Reducir es peligroso**: hay que reducir primero el sistema de ficheros y después el LV, y `xfs` no se puede reducir. No lo hagas en datos que te importen.

---

## 🛠️ Qué debes hacer

### Parte A · Preparar los discos

1. **Apaga** la VM y añade **dos discos virtuales nuevos de 3 GB cada uno**.
2. **Enciende** y localiza los discos nuevos con `lsblk`, **por su tamaño**.

### Parte B · Crear el volumen

3. Convierte **el primer disco nuevo** en PV.
4. Crea un grupo de volúmenes llamado `vg_datos` con **ese PV**.
5. Crea un LV de **2 GB** llamado `lv_proyectos`.
6. Formatéalo con `ext4`, móntalo en `/srv/lvm` y comprueba con `df -h`.
7. Añádelo a `/etc/fstab` con `nofail`, usando la **ruta del LV** o su UUID. Pruébalo con `mount -a`.
8. Rellena parte del volumen con datos de prueba (por ejemplo, un archivo de 1 GB con `fallocate -l 1G /srv/lvm/prueba.bin`).

### Parte C · Ampliar en caliente

9. Mira cuánto espacio **libre** hay en el VG (`vgs`).
10. **Añade el segundo disco** al VG (primero conviértelo en PV).
11. **Amplía** el LV en 2 GB **con el volumen montado** y en un solo comando (que amplíe también el sistema de ficheros).
12. Comprueba que `df -h` muestra el nuevo tamaño y que el archivo de prueba **sigue ahí**, sin haber desmontado nada.

---

## ✅ Comprobación

| Prueba | Qué debes ver |
| ------ | ------------- |
| `sudo pvs` | Dos PV (`/dev/sdc` y `/dev/sdd`), ambos en `vg_datos` |
| `sudo vgs` | `vg_datos` con unos 6 GB en total y el espacio libre que queda |
| `sudo lvs` | `lv_proyectos` con unos 4 GB tras ampliar |
| `df -h /srv/lvm` | Unos 4 GB |
| `ls -lh /srv/lvm` | El archivo `prueba.bin` intacto |
| Tras reiniciar | El volumen se monta solo |

---

## 🧩 Pistas

- Si `lvextend` te dice que no hay espacio, mira `vgs` (columna `VFree`): el VG debe tener espacio libre, y para eso necesita el segundo PV.
- `-L 2G` pone el tamaño **final**; `-L +2G` **añade** 2 GB. No es lo mismo.
- Con `-r` (`--resizefs`), LVM llama a la herramienta adecuada para ampliar el sistema de ficheros. Sin él, el LV crece pero el sistema de ficheros sigue igual (la "botella" es más grande, pero el contenido sigue ocupando lo mismo).
- Si olvidas el nombre de un dispositivo: `lsblk` y `sudo lvs` te lo recuerdan.
- Puedes ver el LV con dos rutas equivalentes: `/dev/vg_datos/lv_proyectos` y `/dev/mapper/vg_datos-lv_proyectos`.

---

## 🚀 Retos extra

1. Comprueba si el sistema de tu VM ya usa LVM para `/` (mira `lsblk`). ¿Cómo ampliarías el sistema raíz si se llenara?
2. Crea un **segundo LV** en el mismo VG con `xfs` y compáralo: ¿qué comando usarías para ampliar un `xfs`? (Pista: `xfs_growfs`; con `lvextend -r` ya lo hace por ti.)
3. Crea una **instantánea LVM** del LV (`lvcreate -s`), modifica archivos y piensa cómo la usarías para volver atrás. ¿En qué se parece a un snapshot de VirtualBox?
4. Mueve los datos de un PV a otro con `pvmove` y retira un disco del grupo con `vgreduce`.
5. Prueba a **reducir** el LV de un volumen de prueba **vacío** y razona por qué hay que hacerlo en un orden concreto.

---

## 📝 Qué documentar en tu `Solucion.md`

- Un esquema PV → VG → LV de **tu** configuración, antes y después de ampliar.
- La salida de `pvs`, `vgs` y `lvs` en cada paso.
- Una explicación con tus palabras de por qué `-r` es importante.
- Cuándo usarías LVM y cuándo bastaría con una partición normal.

---

## 🧠 Lo que has practicado

`PV / VG / LV` · `pvcreate / vgcreate / lvcreate` · `vgextend / lvextend -r` · `ampliar en caliente` · `/dev/vg/lv` · `fstab`
