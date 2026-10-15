# ⏰ Laboratorio 02 · Tareas programadas: `cron` y timers

**Nivel:** 🟡 Intermedio · **Sistema:** 🐧 Linux · **Tiempo estimado:** 75 minutos

---

## 🎯 Escenario

Cada noche, alguien debería guardar un **informe del estado del disco** del servidor. Hasta ahora lo hace una persona a mano... cuando se acuerda. Tu misión es **automatizarlo**, primero con `cron` (el clásico) y después con un **timer de `systemd`** (el moderno), y comparar ambos.

> 📌 Necesitas el conocimiento de `systemd` del laboratorio 01.

---

## 📖 Conceptos nuevos

### `cron`: el despertador del sistema

`cron` es un servicio que lee una lista de tareas ("crontab") y las ejecuta a la hora indicada. Es como un **despertador programable**: *"todos los días a las 2:00, haz esto"*.

### La sintaxis: cinco campos y un comando

```
┌───────── minuto (0-59)
│ ┌─────── hora (0-23)
│ │ ┌───── día del mes (1-31)
│ │ │ ┌─── mes (1-12)
│ │ │ │ ┌─ día de la semana (0-7; 0 y 7 = domingo)
│ │ │ │ │
* * * * *  comando
```

| Símbolo | Significa | Ejemplo |
| ------- | --------- | ------- |
| `*` | Cualquier valor | `* * * * *` = cada minuto |
| `*/5` | Cada 5 | `*/5 * * * *` = cada 5 minutos |
| `1-5` | Rango | `0 9 * * 1-5` = 9:00 de lunes a viernes |
| `,` | Lista | `0 8,20 * * *` = a las 8:00 y a las 20:00 |

### Trampas típicas de `cron`

- `cron` arranca con un **entorno mínimo**: el `PATH` es muy corto. Usa **rutas absolutas** (`/usr/bin/df`, no `df`).
- **No te enseña** los errores: tienes que redirigirlos tú. `>> archivo 2>&1` guarda la salida normal (`>>`, añadiendo) **y** los errores (`2>&1` = "manda los errores al mismo sitio").
- `crontab -r` borra **todo** tu crontab **sin preguntar**.

### Timers de `systemd`: la alternativa moderna

Un **timer** es una unit `.timer` que activa una unit `.service` según un calendario. Siempre van **en pareja**:

| Archivo | Qué define |
| ------- | ---------- |
| `informe.service` | **Qué** se ejecuta |
| `informe.timer` | **Cuándo** se ejecuta |

Ventajas frente a `cron`: logs integrados en `journalctl`, puede ejecutar tareas perdidas si el equipo estaba apagado (`Persistent=true`), y se ve su estado con `systemctl`.

Plantilla del timer (completa las `???` con `man systemd.timer`):

```ini
[Unit]
Description=???

[Timer]
OnCalendar=???
Persistent=???

[Install]
WantedBy=timers.target
```

### Comandos de este laboratorio

| Comando | Qué hace |
| ------- | -------- |
| `crontab -e` | Edita tu crontab (te deja elegir editor la primera vez) |
| `crontab -l` | Lista tu crontab |
| `sudo crontab -l -u usuario` | Lista el crontab de otro usuario |
| `journalctl -u cron -n 20` | Últimos mensajes del servicio `cron` |
| `systemctl list-timers` | Lista los timers y cuándo se ejecutarán |
| `systemd-analyze calendar "expresión"` | **Prueba** una expresión de calendario y dice cuándo se ejecutaría |
| `sudo systemctl enable --now informe.timer` | Habilita y arranca un timer (se habilita el **timer**, no el servicio) |
| `df -h` | Espacio de disco (del nivel 1) |
| `date +%F` | Fecha en formato `AAAA-MM-DD` |

---

## 🛠️ Qué debes hacer

### Parte A · `cron`

1. Crea la carpeta `/var/log/informes` (con `sudo`) y el script `/usr/local/bin/informe.sh` con este contenido, y hazlo ejecutable:

```bash
#!/bin/bash
# Guarda un informe del estado del disco con la fecha en el nombre
FICHERO="/var/log/informes/disco-$(date +%F_%H%M).txt"
echo "Informe generado: $(date)" > "$FICHERO"
df -h >> "$FICHERO"
```

2. Ejecútalo a mano una vez y comprueba el resultado.
3. Programa una tarea en tu `crontab` que ejecute el script **cada minuto** (para poder probarlo rápido), guardando la salida y los errores en `/tmp/cron-informe.log`.
4. Espera 2-3 minutos y verifica que se generan archivos nuevos.
5. Cambia la tarea para que se ejecute **todos los días a las 02:00** y comprueba que `crontab -l` lo refleja.
6. Escribe con tus palabras qué significan estas expresiones: `*/10 * * * *`, `30 6 * * 1`, `0 0 1 * *`.

### Parte B · Timer de `systemd`

7. **Elimina** la tarea de `cron` (con `crontab -e`, borrando la línea; **no** con `crontab -r`).
8. Crea `informe.service` y `informe.timer` en `/etc/systemd/system/`. El servicio ejecuta el script una vez (`Type=oneshot`); el timer, **cada día a las 02:00**.
9. Prueba primero la expresión con `systemd-analyze calendar`.
10. Para probarlo rápido, cambia temporalmente el calendario a **cada minuto**, recarga y **habilita el timer**. Comprueba con `systemctl list-timers` y con los archivos generados.
11. Mira los logs de ejecución con `journalctl -u informe.service`.
12. Vuelve a poner el calendario real (02:00) y deja el timer habilitado.

---

## ✅ Comprobación

| Prueba | Qué debes ver |
| ------ | ------------- |
| `ls /var/log/informes` | Archivos nuevos con fecha y hora en el nombre |
| `cat /tmp/cron-informe.log` | Vacío si todo va bien (o los errores si algo falla) |
| `crontab -l` | La línea a las 02:00 (en la Parte A) |
| `systemctl list-timers \| grep informe` | Tu timer, con la próxima ejecución |
| `journalctl -u informe.service -n 5` | Ejecuciones del servicio |
| `systemd-analyze calendar "*-*-* 02:00:00"` | La próxima fecha correcta |

---

## 🧩 Pistas

- Si `cron` no ejecuta tu script, lo primero: ruta absoluta del script, permiso de ejecución y mira `/tmp/cron-informe.log`.
- El primer campo es el **minuto**, no la hora: es el error más común (`2 0 * * *` ≠ `0 2 * * *`).
- Si en el timer no pasa nada, ¿has habilitado el `.timer` y no el `.service`? ¿Has hecho `daemon-reload`?
- `Type=oneshot` es para servicios que **hacen algo y terminan**, a diferencia del `latido` del laboratorio 01, que no terminaba nunca.
- El script no necesita `sudo` dentro porque `systemd` y el `crontab` de `root` ya corren con privilegios. Si usas el `crontab` de tu usuario, fíjate bien en los permisos de `/var/log/informes`.

---

## 🚀 Retos extra

1. **Limpieza automática:** crea otra tarea que borre los informes de más de 7 días (`find /var/log/informes -mtime +7 -delete`). Ojo: antes ejecútalo **sin `-delete`** para ver qué borraría.
2. Mira las carpetas `/etc/cron.daily`, `/etc/cron.weekly` y el archivo `/etc/crontab`. ¿Qué tareas del sistema ya se ejecutan solas?
3. Prueba `Persistent=true`: apaga la VM antes de la hora programada, enciéndela después y mira si se ejecuta la tarea perdida.
4. Investiga `@reboot` en `cron`: ¿para qué sirve?
5. Haz una tabla comparativa `cron` frente a timers, con ventajas e inconvenientes de cada uno.

---

## 📝 Qué documentar en tu `Solucion.md`

- La línea de `crontab` con la explicación campo por campo.
- Los archivos `.service` y `.timer` completos y explicados.
- La tabla comparativa y cuál usarías tú y por qué.

---

## 🧠 Lo que has practicado

`cron` · `sintaxis de crontab` · `rutas absolutas` · `redirección de salida y errores` · `timers de systemd` · `OnCalendar` · `Type=oneshot` · `journalctl`
