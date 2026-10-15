# 💾 Laboratorio 09 · Backups con `tar` y `rsync`

**Nivel:** 🟡 Intermedio · **Sistema:** 🐧 Linux · **Tiempo estimado:** 90 minutos

---

## 🎯 Escenario

Un día, un compañero borra por error la carpeta de proyectos de Sistemas. Si no hay **copia de seguridad**, esos datos se pierden para siempre.

Tu misión es crear un sistema de backups con dos herramientas clásicas: **`tar`** (para copias completas y comprimidas) y **`rsync`** (para copias incrementales y sincronizadas). Y, lo más importante, **comprobar que sabes restaurar**.

> 📌 Usa el disco del laboratorio 04 y el volumen LVM del 05: uno servirá de **origen** y otro de **destino** de los backups. Si no los tienes, puedes usar dos carpetas distintas, aunque pierdes el sentido de separar los datos de su copia.

---

## 📖 Conceptos nuevos

### Un backup que no se ha probado no es un backup

Es la frase más repetida del oficio. Muchos backups "existen" pero no se pueden restaurar (archivos corruptos, rutas mal copiadas, permisos perdidos). **Siempre** se hace una **prueba de restauración**.

### Tipos de backup

| Tipo | Qué copia | Pros y contras |
| ---- | --------- | -------------- |
| **Completo** (*full*) | Todo | Fácil de restaurar, pero ocupa mucho y tarda |
| **Incremental** | Solo lo que cambió **desde la última copia** (de cualquier tipo) | Rápido y pequeño, pero restaurar exige encadenar varias |
| **Diferencial** | Lo que cambió **desde la última completa** | Punto medio |

### La regla 3-2-1

Una buena estrategia de backups cumple:

- **3** copias de tus datos (el original + 2 copias),
- en **2** tipos de soporte distintos,
- con **1** copia **fuera** del sitio (otra sala, la nube...).

Una copia en el mismo disco que el original **no protege** frente a un fallo del disco.

### `tar`: el empaquetador

`tar` junta muchos archivos en uno solo (un "paquete"), y opcionalmente lo **comprime**. Es la herramienta clásica para copias completas.

### `rsync`: el sincronizador

`rsync` copia **solo las diferencias** entre origen y destino. La primera vez copia todo; las siguientes, solo lo que cambió. Es muy rápido y funciona también por SSH.

Un detalle que confunde a todos, la **barra final**:

| Comando | Resultado |
| ------- | --------- |
| `rsync -a /datos/ /backup/` | Copia el **contenido** de `/datos` dentro de `/backup` |
| `rsync -a /datos /backup/` | Copia la **carpeta** `datos` entera dentro de `/backup` (queda `/backup/datos`) |

### Comandos de este laboratorio

| Comando | Qué hace |
| ------- | -------- |
| `tar -czvf copia.tar.gz carpeta` | **Crea** un paquete comprimido (`c` = crear, `z` = comprimir con gzip, `v` = mostrar archivos, `f` = nombre del archivo) |
| `tar -tzf copia.tar.gz` | **Lista** el contenido sin extraer (`t` = list) |
| `tar -xzf copia.tar.gz -C /destino` | **Extrae** (`x` = extract) en otro directorio (`-C`) |
| `tar --exclude='*.tmp' -czf ...` | Excluye archivos por patrón |
| `rsync -avh origen/ destino/` | Sincroniza (`a` = archivo: conserva permisos, fechas y propietarios; `v` = detalle; `h` = tamaños legibles) |
| `rsync -avh --dry-run origen/ destino/` | **Simula** sin copiar nada |
| `rsync -avh --delete origen/ destino/` | Además **borra del destino** lo que ya no existe en el origen (¡cuidado!) |
| `sha256sum archivo` | Calcula la **huella** de un archivo (si cambia un solo byte, cambia la huella) |
| `diff -r carpeta1 carpeta2` | Compara dos carpetas; sin salida = idénticas |
| `date +%F` | Fecha `AAAA-MM-DD` (útil para nombres de backup) |
| `du -sh carpeta` | Tamaño total de una carpeta |

> ⚠️ **`--delete` y `rm` no perdonan.** Antes de usar `--delete`, ejecuta siempre la misma orden con `--dry-run`.

---

## 🛠️ Qué debes hacer

### Parte A · Datos de prueba

1. Crea el origen `/srv/lvm/datos` (o la carpeta que prefieras) con unos 20-30 archivos de texto y alguna subcarpeta. Puedes generarlos con un bucle de Bash (`for i in $(seq 1 20); do echo "contenido $i" > archivo$i.txt; done`).
2. Crea el destino de los backups, por ejemplo `/srv/proyectos/backups` (en **otro disco o volumen** distinto del origen).

### Parte B · Copia completa con `tar`

3. Crea una copia completa comprimida del origen, con **la fecha en el nombre** (`datos-AAAA-MM-DD.tar.gz`), en la carpeta de backups.
4. **Lista** su contenido sin extraer y comprueba su tamaño.
5. Calcula su **`sha256sum`** y guárdalo en un archivo.
6. **Prueba de restauración:** borra el origen entero, restáuralo desde el backup y comprueba con `diff -r` que **no falta nada**.

### Parte C · Sincronización con `rsync`

7. Haz una **simulación** (`--dry-run`) de sincronizar el origen con una carpeta `espejo` en el destino. Léela con calma.
8. Ejecútala de verdad. Anota cuánto ha copiado.
9. **Modifica** 3 archivos, **crea** 2 nuevos y **borra** 1 del origen. Vuelve a sincronizar **sin** `--delete`: ¿qué pasa con el archivo borrado?
10. Repite **con** `--delete` (primero en simulación) y comprueba la diferencia.
11. Confirma con `diff -r` que origen y espejo son idénticos.

### Parte D · Automatizar

12. Escribe un script `backup.sh` que haga la copia completa con `tar` con la fecha en el nombre y que **escriba en un log** si ha ido bien o mal (usa el código de salida `$?`).
13. Prográmalo con `cron` o con un **timer** (laboratorio 02) a una hora determinada. Pruébalo con una frecuencia corta y déjalo luego a diario.

---

## ✅ Comprobación

| Prueba | Qué debes ver |
| ------ | ------------- |
| `ls -lh /srv/proyectos/backups` | El `.tar.gz` con la fecha en el nombre |
| `tar -tzf datos-*.tar.gz \| head` | Los archivos del origen |
| `diff -r origen restauracion` | **Sin salida** (idénticos) |
| `sha256sum -c datos.sha256` | `OK` (comprueba que el backup no se ha corrompido) |
| `rsync -avh --dry-run origen/ espejo/` tras sincronizar | Sin archivos pendientes |
| Log del script | Una línea por ejecución con fecha y resultado |

---

## 🧩 Pistas

- Al extraer con `tar`, comprueba **dónde** quedan los archivos (rutas relativas y absolutas se comportan distinto). `tar -tzf` te lo enseña antes de extraer.
- Si `diff -r` muestra diferencias de contenido, algo se ha restaurado mal; si solo muestra "Only in...", falta un archivo.
- Con `rsync`, **la barra final** del origen decide si copias el contenido o la carpeta.
- En el script, `if [ $? -eq 0 ]; then ... else ... fi` te permite distinguir éxito de error.
- Los backups viven mejor con nombres que se **ordenan solos** por fecha (`AAAA-MM-DD`).

---

## 🚀 Retos extra

1. Haz que el script **conserve solo los últimos 7 backups** y borre los demás (usa `find ... -mtime +7` primero **sin** `-delete`).
2. Usa `rsync` **a través de SSH** hacia otra máquina (otra VM o tu ordenador), con la clave del laboratorio 08.
3. Prueba `rsync --link-dest` para hacer **incrementales con enlaces duros**: cada backup parece completo pero solo ocupa lo que cambió. Investiga cómo funciona.
4. Excluye archivos temporales y carpetas de caché de la copia (`--exclude`).
5. Calcula cuánto espacio necesitarías para guardar 30 días de backups diarios de tus datos, con completas frente a incrementales.

---

## 📝 Qué documentar en tu `Solucion.md`

- Los comandos con la explicación de cada opción.
- El resultado de la prueba de restauración.
- Tu script `backup.sh`, explicado.
- Una reflexión: ¿qué parte de la regla 3-2-1 cumples ahora y cuál te falta?

---

## 🧠 Lo que has practicado

`backup completo / incremental` · `regla 3-2-1` · `tar` · `rsync` · `--dry-run / --delete` · `sha256sum` · `diff -r` · `prueba de restauración` · `automatización con cron / timers`
