# 📜 Laboratorio 06 · Logs con `journalctl`

**Nivel:** 🟡 Intermedio · **Sistema:** 🐧 Linux · **Tiempo estimado:** 75 minutos

---

## 🎯 Escenario

Llega un aviso de seguridad: *"Parece que alguien ha intentado entrar en el servidor esta noche"*. Los servidores no hablan, pero **lo apuntan todo**: quién se conecta, qué falla, qué servicio se reinicia. Tu trabajo es **leer esos registros** (logs), filtrar lo que importa y reconstruir qué ha pasado.

Para practicar, primero **provocarás tú mismo los eventos** y después los buscarás como si fueras a investigarlos.

---

## 📖 Conceptos nuevos

### Qué es un log

Un **log** es el diario del sistema: cada línea es un evento con fecha, origen y mensaje. Es la principal herramienta de un sysadmin para **diagnosticar fallos** y **detectar incidentes de seguridad**.

### Dos mundos que conviven

| Sistema | Qué es | Dónde |
| ------- | ------ | ----- |
| **journald** (`journalctl`) | El registro **moderno y centralizado** de `systemd`: estructurado, con filtros potentes | Formato binario, se consulta con `journalctl` |
| **Archivos de texto** (`rsyslog`) | Los logs **clásicos** | `/var/log/` (por ejemplo `auth.log`, `syslog`) |

En Ubuntu **coexisten**: muchos eventos aparecen en los dos sitios.

### Archivos de `/var/log` que debes conocer

| Archivo | Contiene |
| ------- | -------- |
| `/var/log/syslog` | Mensajes generales del sistema |
| `/var/log/auth.log` | **Autenticaciones**: logins, `sudo`, SSH |
| `/var/log/dpkg.log` | Paquetes instalados y quitados |
| `/var/log/kern.log` | Mensajes del kernel |

### Niveles de gravedad (*priority*)

De más a menos grave: `emerg` (0) · `alert` (1) · `crit` (2) · `err` (3) · `warning` (4) · `notice` (5) · `info` (6) · `debug` (7). Al filtrar por `err` ves también los más graves (`crit`, `alert`, `emerg`).

### Rotación de logs

Los logs crecen sin parar. `logrotate` los **rota** de forma automática: archiva el log actual (`auth.log.1`, `auth.log.2.gz`...) y empieza uno nuevo, borrando los más antiguos. La configuración está en `/etc/logrotate.d/`.

### Comandos de este laboratorio

| Comando | Qué hace |
| ------- | -------- |
| `journalctl` | Muestra todo el journal (`q` para salir) |
| `journalctl -n 50` | Últimas 50 entradas |
| `journalctl -f` | **Sigue** el log en vivo (`Ctrl+C` para parar) |
| `journalctl -u ssh` | Entradas de un servicio |
| `journalctl -b` | Solo desde el **último arranque** (`-b -1` = el arranque anterior) |
| `journalctl -p err` | Solo errores y más graves |
| `journalctl --since "1 hour ago"` | Desde hace una hora (también `--until`, o fechas como `"2026-10-11 09:00"`) |
| `journalctl --disk-usage` | Espacio que ocupa el journal |
| `logger "mensaje"` | **Escribe** un mensaje en el log (para pruebas) |
| `sudo tail -n 20 /var/log/auth.log` | Últimas 20 líneas de un log de texto |
| `sudo tail -f /var/log/auth.log` | Sigue un log de texto en vivo |
| `sudo grep "texto" archivo` | Busca líneas que contengan "texto" (`-c` cuenta, `-i` ignora mayúsculas) |
| `sudo zgrep "texto" archivo.gz` | Igual, pero en archivos comprimidos |

Para contar y ordenar, vas a **encadenar** comandos con `|` (nivel 1). Ejemplo de cómo se obtiene un "ranking":

```
comando | sort | uniq -c | sort -rn
```

- `sort` ordena las líneas.
- `uniq -c` agrupa las repetidas y **cuenta** cuántas hay.
- `sort -rn` ordena **numéricamente** de mayor a menor (`-r` = reverso, `-n` = numérico).

---

## 🛠️ Qué debes hacer

### Parte A · Familiarizarte

1. Mira las últimas 30 líneas del journal. ¿Qué información trae cada línea?
2. Consulta **solo los errores** del arranque actual. ¿Hay alguno?
3. Consulta los eventos de **solo el servicio `ssh`**.
4. Comprueba cuánto espacio ocupa el journal.

### Parte B · Provocar eventos

5. En una terminal, deja corriendo `journalctl -f`. En otra, haz lo siguiente y observa cómo aparece cada cosa en vivo:
   - Escribe un mensaje propio con `logger`.
   - Reinicia el servicio `cron`.
   - Ejecuta un comando con `sudo`.
6. Desde **tu ordenador**, intenta conectarte por SSH **5 veces con una contraseña incorrecta** y luego **una vez bien**.

### Parte C · Investigar como un detective

7. En `/var/log/auth.log`, localiza los **intentos de SSH fallidos** y cuéntalos.
8. Averigua desde **qué IP** se produjeron.
9. Averigua **qué usuario** intentaron usar.
10. Localiza la **conexión correcta** y la hora exacta.
11. Averigua con `auth.log` **qué usuario usó `sudo`** y qué comando ejecutó.
12. Encuentra el **último reinicio del sistema** y cuándo se arrancó el servicio `cron`.
13. Filtra todo lo ocurrido en **los últimos 15 minutos** con prioridad `warning` o peor.

### Parte D · Rotación

14. Mira `/etc/logrotate.d/rsyslog` y explica con tus palabras cuántas copias se guardan y cada cuánto se rota.
15. Lista los archivos de `/var/log` y localiza alguno ya **comprimido** (`.gz`) o rotado (`.1`).

---

## ✅ Comprobación

| Prueba | Qué debes ver |
| ------ | ------------- |
| `journalctl --since "10 minutes ago" \| grep "latido-prueba"` | Tu mensaje de `logger` (usa un texto reconocible) |
| `sudo grep -c "Failed password" /var/log/auth.log` | Un número mayor o igual a 5 |
| `sudo grep "Accepted" /var/log/auth.log \| tail -n 3` | Tu conexión correcta |
| `sudo grep "sudo:" /var/log/auth.log \| tail -n 3` | El uso de `sudo` con el comando |
| `journalctl -b -p err --no-pager \| head` | Los errores del arranque (si los hay) |
| `journalctl -u cron -n 5` | El reinicio de `cron` que hiciste |

---

## 🧩 Pistas

- Los fallos de SSH dicen **"Failed password for ..."**, y los aciertos **"Accepted password for ..."**. Busca esas frases con `grep`.
- Para quedarte con solo la columna de la IP puedes usar `awk '{print $11}'`, pero la posición depende del formato de la línea: **mira primero una línea completa** para contar bien.
- Un ranking de IPs sería algo así: filtrar con `grep` → extraer el campo → `sort | uniq -c | sort -rn`.
- Si `journalctl` te dice que no tienes permisos para ver todo, usa `sudo` o añade tu usuario al grupo `systemd-journal`.
- `--no-pager` imprime todo de golpe, sin el visor `less`.

---

## 🚀 Retos extra

1. Haz que el journal sea **persistente** (que sobreviva al reinicio): comprueba si existe `/var/log/journal` y averigua cómo activarlo en `journald.conf`.
2. Escribe un **script** que cuente los intentos fallidos por IP y los muestre ordenados (puedes reutilizar lo que sabes de Bash).
3. Mira cómo `fail2ban` automatiza justo esto: lee los logs y **bloquea las IP** que fallan demasiado (lo estudiarás en el nivel avanzado).
4. Prueba `journalctl -o json-pretty -n 1`. ¿Qué campos tiene una entrada? ¿Para qué sirve este formato?
5. Investiga `last` y `lastb`: ¿qué te dicen sobre accesos correctos y fallidos?

---

## 📝 Qué documentar en tu `Solucion.md`

- Los comandos usados para cada pregunta de la Parte C, con la salida (oculta o ficticia cualquier IP que sea real).
- Un **informe de incidente** breve, de cinco o seis líneas: qué pasó, cuándo, desde dónde, cómo se detectó.
- Qué diferencia práctica hay entre `journalctl` y mirar `/var/log`.

---

## 🧠 Lo que has practicado

`journalctl` y sus filtros · `/var/log` · `auth.log` · `logger` · `grep` · `sort | uniq -c | sort -rn` · `logrotate` · `niveles de gravedad` · `análisis básico de incidentes`
