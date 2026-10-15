# ⚙️ Laboratorio 01 · Servicios con `systemd`

**Nivel:** 🟡 Intermedio · **Sistema:** 🐧 Linux · **Tiempo estimado:** 75 minutos

---

## 🎯 Escenario

En la empresa hay un pequeño programa que debe estar **siempre en marcha**: escribe un "latido" en un registro cada pocos segundos, para que el equipo de monitorización sepa que el servidor está vivo. Hoy lo lanzan a mano, y si el servidor se reinicia o el programa se cae, nadie lo vuelve a arrancar.

Tu misión: convertirlo en un **servicio de `systemd`** que arranque con el sistema y se reinicie solo si falla.

> 💾 Haz un **snapshot** antes de empezar.

---

## 📖 Conceptos nuevos

### Qué es `systemd` y qué es una *unit*

`systemd` es el **primer proceso** que arranca (PID 1, lo viste en el nivel 1) y el encargado de arrancar y vigilar todo lo demás. Es como el **encargado de un edificio**: abre las puertas por la mañana, comprueba que todo funciona y avisa si algo se rompe.

Lo que gestiona se llama **unit** (unidad). Las más importantes:

| Tipo | Extensión | Qué es |
| ---- | --------- | ------ |
| Servicio | `.service` | Un programa que corre en segundo plano |
| Temporizador | `.timer` | Lanza algo según el calendario (lo verás en el laboratorio 02) |
| Montaje | `.mount` | Un disco montado |
| Destino | `.target` | Un grupo de units (por ejemplo, "modo multiusuario") |

### Estados importantes

Un servicio puede estar **activo** (*running*), **inactivo** (*dead*) o **fallido** (*failed*). Y aparte, **habilitado** (*enabled*: arranca solo al iniciar el sistema) o **deshabilitado** (*disabled*). Son dos cosas independientes: un servicio puede estar activo ahora y no arrancar en el próximo reinicio.

### Dónde viven los archivos de unit

| Ruta | Para qué |
| ---- | -------- |
| `/lib/systemd/system/` | Units que instalan los paquetes. **No se tocan** |
| `/etc/systemd/system/` | Units tuyas o modificaciones. **Aquí se crean las propias** |

### Anatomía de un archivo `.service`

Tiene tres secciones. Esta es la plantilla; completa tú las `???` investigando con `man systemd.service`:

```ini
[Unit]
Description=???
After=???

[Service]
ExecStart=???
Restart=???
RestartSec=???

[Install]
WantedBy=???
```

| Sección | Qué contiene |
| ------- | ------------ |
| `[Unit]` | Descripción y orden de arranque (`After=` = "arranca después de…") |
| `[Service]` | Qué ejecutar (`ExecStart=`), con qué usuario, y qué hacer si se cae (`Restart=`) |
| `[Install]` | En qué momento del arranque se activa al habilitarlo (`WantedBy=`; lo habitual es `multi-user.target`) |

### Comandos de este laboratorio

| Comando | Qué hace |
| ------- | -------- |
| `systemctl status servicio` | Estado, últimos logs y PID |
| `sudo systemctl start / stop / restart servicio` | Arranca / para / reinicia ahora |
| `sudo systemctl reload servicio` | Recarga la configuración sin parar el servicio (si lo soporta) |
| `sudo systemctl enable servicio` | Hace que arranque al iniciar el sistema (`--now` lo arranca también ahora) |
| `sudo systemctl disable servicio` | Quita el arranque automático |
| `systemctl is-active servicio` / `is-enabled servicio` | Dice si está activo / habilitado |
| `systemctl cat servicio` | Muestra el archivo de unit de un servicio |
| `systemctl --failed` | Lista las units que han fallado |
| `sudo systemctl daemon-reload` | Obliga a `systemd` a releer los archivos de unit (**obligatorio** tras crear o editar uno) |
| `journalctl -u servicio -n 20` | Últimas 20 líneas de log de ese servicio |

> ⚠️ No practiques parando o deshabilitando `ssh`, `systemd-networkd` o `systemd-resolved`: podrías quedarte sin acceso. Para los ejercicios de parar y arrancar, usa el servicio **`cron`**, que es inofensivo.

---

## 🛠️ Qué debes hacer

### Parte A · Gestionar un servicio existente

1. Mira el estado del servicio `cron` y lee con calma la salida: ¿qué significa cada línea?
2. **Páralo**, comprueba que está parado, **arráncalo** de nuevo y **reinícialo**.
3. Con `systemctl cat`, mira su archivo de unit y localiza sus tres secciones.
4. Comprueba si `cron` está **habilitado**. Deshabilítalo y vuelve a habilitarlo.
5. Lista los servicios **en ejecución** (`systemctl list-units --type=service --state=running`) y los **fallidos**.

### Parte B · Crear tu propio servicio

6. Crea este script en `/usr/local/bin/latido.sh` y hazlo ejecutable (`chmod +x`):

```bash
#!/bin/bash
# Escribe un "latido" con fecha y hora cada 10 segundos
while true; do
  echo "$(date '+%F %T') latido" >> /var/log/latido.log
  sleep 10
done
```

(`$(date '+%F %T')` ejecuta `date` y pega su resultado: la fecha `AAAA-MM-DD` y la hora. `>>` **añade** al final del archivo sin borrarlo.)

7. **Prueba el script a mano** unos segundos (`Ctrl+C` para pararlo) y mira `/var/log/latido.log`.
8. Crea `/etc/systemd/system/latido.service` completando la plantilla: debe describir el servicio, arrancar el script y **reiniciarlo siempre que se caiga**, esperando 5 segundos.
9. Avisa a `systemd` del archivo nuevo, **arranca** el servicio y comprueba su estado y el log.
10. **Mátalo a la fuerza** (`kill -9` sobre su PID, que te da `systemctl status`) y comprueba que `systemd` lo **revive solo**.
11. **Habilítalo**, reinicia la VM y comprueba que sigue funcionando sin hacer nada.
12. Para terminar, **elimínalo correctamente**: parar, deshabilitar, borrar el archivo de unit y recargar `systemd`.

---

## ✅ Comprobación

| Prueba | Qué debes ver |
| ------ | ------------- |
| `systemctl is-active cron` | `active` (y `inactive` mientras esté parado) |
| `systemctl status latido` | `active (running)` y el PID |
| `tail -n 5 /var/log/latido.log` | Líneas nuevas cada 10 s (`tail -n 5` muestra las 5 últimas) |
| Tras `kill -9` | Un PID **distinto** y el servicio de nuevo `active` |
| Tras reiniciar la VM | `systemctl is-enabled latido` = `enabled` y el log sigue creciendo |
| `systemctl --failed` | Sin units de tu servicio |

---

## 🧩 Pistas

- Si `systemctl start` da error, mira `journalctl -u latido -n 20` y `systemctl status latido`: suelen decir qué falla.
- `ExecStart=` necesita la **ruta absoluta** del programa.
- Si cambias el archivo de unit y no notas el cambio, te falta `daemon-reload`.
- Para el valor de `Restart=`, compara `on-failure` y `always` en `man systemd.service`: ¿en qué se diferencian?
- Con `systemctl show -p NRestarts latido` ves cuántas veces se ha reiniciado.

---

## 🚀 Retos extra

1. Haz que el servicio corra con un **usuario sin privilegios** (crea uno con `useradd --system`) en vez de `root`. Te dará un problema de permisos con el log: ¿cómo lo resuelves?
2. Investiga `systemd-analyze blame`: ¿qué servicio tarda más en arrancar en tu VM?
3. Añade a la unit medidas de seguridad como `ProtectSystem=full` o `NoNewPrivileges=true`. ¿Qué hace cada una?
4. Crea una **segunda unit** que dependa de la primera (`Requires=` y `After=`) y observa qué pasa al parar la primera.
5. Averigua la diferencia entre `Type=simple` y `Type=forking`.

---

## 📝 Qué documentar en tu `Solucion.md`

- Tu archivo `latido.service` completo, explicando cada línea.
- La salida de `status` antes y después del `kill -9`.
- Qué errores te dio y cómo los encontraste con `journalctl`.

---

## 🧠 Lo que has practicado

`units` · `systemctl` · `enable / disable / start / stop` · `archivos .service` · `daemon-reload` · `Restart=` · `journalctl -u`
