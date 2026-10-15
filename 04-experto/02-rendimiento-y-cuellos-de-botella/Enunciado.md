# 🏎️ Laboratorio 02 · Rendimiento y cuellos de botella

**Nivel:** 🔴 Experto · **Sistema:** 🐧 Linux · **Tiempo estimado:** 120 minutos

---

## 🎯 Escenario

*"El servidor va lento."* Es la frase más vaga y más frecuente de la administración de sistemas. En el laboratorio 06 del nivel 1 la resolviste con `top`. Aquí vas a hacerlo **con método**: averiguar **qué recurso** está saturado (CPU, memoria, disco o red), **quién** lo consume y **qué hacer**.

Tu misión es aprender a **diagnosticar** un problema de rendimiento con evidencias, no con intuiciones, y a **limitar** el daño de un proceso descontrolado.

> 💾 **Snapshot**, y VM con **al menos 2 CPU y 2 GB de RAM**. Vas a **saturar tu propia VM** a propósito; ten la consola de VirtualBox a mano por si se pone muy lenta.

---

## 📖 Conceptos nuevos

### El método USE

Para cada recurso del sistema, hazte tres preguntas (método de Brendan Gregg):

| Letra | Pregunta | Ejemplo |
| ----- | -------- | ------- |
| **U**tilization | ¿Cuánto del recurso está **en uso**? | CPU al 95 % |
| **S**aturation | ¿Hay **cola de espera** por el recurso? | Muchos procesos esperando CPU |
| **E**rrors | ¿Hay **errores**? | Errores de disco o de red |

Recorrer **CPU → memoria → disco → red** con USE evita saltar a conclusiones y es muy rápido.

### La carga media (*load average*)

`uptime` muestra tres números: la **carga media de 1, 5 y 15 minutos**. Es el número medio de procesos **ejecutándose o esperando** (en Linux, también los bloqueados esperando disco). **Se interpreta frente al número de CPU** (`nproc`): en una máquina de 2 CPU, una carga de 2 es "llena"; una de 8 es una cola de 6 procesos esperando.

### Herramientas por recurso

| Recurso | Herramientas | Qué miras |
| ------- | ------------ | --------- |
| **CPU** | `top`/`htop`, `vmstat 1`, `mpstat -P ALL 1` | `%us` (usuario), `%sy` (kernel), `%wa` (espera de E/S), `%id` (libre), cola `r` |
| **Memoria** | `free -h`, `vmstat 1`, `ps aux --sort=-%mem` | `available`, uso de **swap** (`si`/`so` en `vmstat` ≠ 0 = problema) |
| **Disco** | `iostat -xz 1`, `iotop`, `df -h`, `df -i` | `%util`, `await` (latencia), uso de inodos |
| **Red** | `ss -s`, `ip -s link`, `sar -n DEV 1` | Errores y descartes, conexiones |

`vmstat`, `iostat`, `mpstat` y `sar` están en el paquete **`sysstat`**.

### Generar carga controlada: `stress-ng`

`stress-ng` crea carga artificial para practicar. Ejemplos (cada uno con límite de tiempo):

| Comando | Carga |
| ------- | ----- |
| `stress-ng --cpu 2 --timeout 120s` | Satura **2 CPU** durante 2 minutos |
| `stress-ng --vm 1 --vm-bytes 80% --timeout 120s` | Consume el **80 % de la RAM** |
| `stress-ng --hdd 1 --timeout 120s` | Provoca **escrituras de disco** |
| `stress-ng --io 2 --timeout 120s` | Muchas **operaciones de E/S** |

### Limitar un proceso: prioridades y `cgroups`

| Mecanismo | Qué hace |
| --------- | -------- |
| `nice` / `renice` | Cambia la **prioridad** relativa (nivel 1, laboratorio 06) |
| **`cgroups`** (vía `systemd`) | Pone **límites duros** de CPU y memoria a un servicio: `CPUQuota=50%`, `MemoryMax=500M` |
| `systemd-run --scope -p CPUQuota=50% comando` | Lanza un comando con un límite temporal |

> **Analogía:** `nice` es pedirle a un cocinero que **ceda el paso**; un `cgroup` es **cerrar la llave del gas** a partir de cierto punto.

---

## 🛠️ Qué debes hacer

### Parte A · Línea base

1. Instala `sysstat`, `stress-ng` y `htop`. Anota la **línea base** de tu VM en reposo: carga, CPU, memoria, disco y red.
2. Prepara una **tabla de evidencias** (plantilla en la sección de documentación).

### Parte B · Cuatro diagnósticos a ciegas

Para cada uno, **otra persona** (o tú, **sin mirar la terminal donde lo lanzas**) inicia una carga. Tú diagnosticas con **solo las herramientas del sistema**: **qué recurso** está saturado, **qué proceso** lo causa y **qué evidencias** tienes. Cada diagnóstico debe incluir **al menos dos herramientas distintas que lo confirmen**.

3. **CPU:** una carga de CPU sobre todos los núcleos.
4. **Memoria:** una carga que casi agote la RAM y provoque **swap**.
5. **Disco:** una carga de escritura intensa.
6. **Mixta:** lanza **dos tipos de carga a la vez** y consigue **separarlas** con tus herramientas.

### Parte C · Soluciones

7. Con el proceso de CPU activo: pásalo a baja prioridad con `renice` y observa el efecto en otro proceso interactivo. Después, **límitalo con `systemd-run`** (`CPUQuota`) y comprueba el límite con `top`.
8. Crea un **servicio de `systemd`** que ejecute `stress-ng` con **`MemoryMax`** pequeño y comprueba **qué le ocurre** al superar el límite (mira los logs).
9. Investiga el ajuste **`vm.swappiness`** y razona cuándo cambiarlo.
10. Investiga qué ocurre cuando se acaba **la memoria**: busca en `journalctl -k` mensajes del **OOM killer** (*Out Of Memory*) y averigua cómo decide a quién matar.

---

## ✅ Comprobación

| Prueba | Qué debes ver |
| ------ | ------------- |
| Diagnóstico de CPU | `uptime` con carga alta, `%us` alto, un proceso concreto en `top` |
| Diagnóstico de memoria | `free -h` con poca memoria disponible, `vmstat` con `si/so` > 0 |
| Diagnóstico de disco | `iostat -xz` con `%util` cercano a 100 y `await` alto, `%wa` alto en CPU |
| Con `CPUQuota=50%` | El proceso no pasa de ~50 % de **un** núcleo |
| Con `MemoryMax` | El proceso es **terminado** y queda registrado |
| Tu tabla de evidencias | **Cuatro filas**, cada una con dos fuentes |

---

## 🧩 Pistas

- **Un `%wa` alto con CPU libre** significa que el sistema espera al **disco**, no que le falte CPU. Es la confusión clásica.
- En `vmstat`, mira la columna **`r`** (procesos esperando CPU) y **`b`** (bloqueados en E/S).
- `iostat` sin `-x` es casi inútil: las columnas `await` y `%util` están en el modo extendido.
- `top` ordena por CPU; pulsa `M` para ordenar por memoria y `1` para ver cada núcleo por separado.
- Para ver un resumen del consumo por servicio de `systemd`: `systemd-cgtop`.
- Cuando algo se atasca de verdad, **no reinicies**: abre otra sesión y mata **solo** el proceso culpable.

---

## 🚀 Retos extra

1. Mide el **rendimiento de disco** con `fio` (lecturas/escrituras secuenciales y aleatorias) y compara el disco de tu VM con el LVM o RAID de los laboratorios anteriores.
2. Usa `sar` para **guardar histórico** de métricas y consultar qué pasó "ayer a las 3 de la madrugada".
3. Relaciona este laboratorio con **Prometheus/Grafana** (avanzado, 08): ¿qué panel y qué alerta te habrían avisado **antes** de que nadie se quejara?
4. Investiga `perf` o `strace` para averiguar **qué está haciendo** un proceso por dentro (qué llamadas al sistema hace) y compara con `top`.
5. Fíjate en la **red**: genera tráfico con `iperf3` entre las dos VMs y diagnostica con `ip -s link` y `sar -n DEV`.

---

## 📝 Qué documentar en tu `Solucion.md`

Una **tabla de evidencias** como esta:

| Síntoma | Recurso afectado (USE) | Proceso causante | Herramienta 1 (evidencia) | Herramienta 2 (evidencia) | Acción tomada |
| ------- | ---------------------- | ---------------- | ------------------------- | ------------------------- | ------------- |

Y un **procedimiento de diagnóstico en 5 pasos** que le darías a un compañero novato cuando le dicen *"el servidor va lento"*.

---

## 🧠 Lo que has practicado

`método USE` · `load average` · `vmstat / iostat / mpstat / sar` · `stress-ng` · `iowait y swap` · `nice y renice` · `cgroups y systemd` · `OOM killer` · `diagnóstico con evidencias`
