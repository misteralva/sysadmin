# 🛠️ Laboratorio 10 · Break/fix en Linux

**Nivel:** 🔴 Experto · **Sistema:** 🐧 Linux · **Tiempo estimado:** 150 minutos

---

## 🎯 Escenario

Un buen sysadmin no se mide por lo que sabe montar, sino por **lo rápido y ordenado que diagnostica un problema** que no ha visto nunca. Aquí te enfrentas a **ocho averías** en servidores Linux. Para cada una tienes solo el **síntoma**, igual que en la vida real: *"no funciona"*.

Tu misión es **diagnosticar con método**, **arreglar la causa** (no el síntoma), **verificar** y **documentar** cada avería en un **runbook**.

> 💾 **Snapshot al inicio**, y **vuelve a él después de cada avería** para empezar limpio. Trabaja con la **consola de VirtualBox** abierta como plan B. **Ideal: que otra persona provoque las averías** sin decirte cuáles son, o hacerlas en **orden aleatorio**.

---

## 📖 Conceptos nuevos

### El método de diagnóstico

Para **todas** las averías, sigue siempre el mismo ciclo:

1. **Observar:** ¿cuál es el **síntoma exacto**? ¿Qué mensaje de error sale? ¿Desde cuándo? ¿Qué cambió?
2. **Hipótesis:** ¿qué **capa** puede estar fallando? (hardware → sistema de ficheros → red → servicio → aplicación → permisos)
3. **Comprobar:** una prueba **que confirme o descarte** la hipótesis, con la herramienta adecuada.
4. **Arreglar:** corrige **la causa raíz** (un cambio cada vez).
5. **Verificar:** comprueba que el síntoma **desapareció** y que no has roto otra cosa.
6. **Documentar:** causa, solución y cómo evitarlo.

### Las preguntas de oro

| Pregunta | Ejemplo |
| -------- | ------- |
| **¿Qué ha cambiado?** | Un reinicio, una actualización, un cambio de configuración |
| **¿Se puede reproducir?** | Siempre o a veces |
| **¿A quién afecta?** | A un usuario, a todos, a una red |
| **¿Qué dicen los logs?** | `journalctl`, `/var/log/`, `systemctl status` |
| **¿Qué funciona y qué no?** | Acotar: si el `ping` a la IP va y al nombre no → DNS |

### Tu caja de herramientas (la de los niveles anteriores)

| Capa | Herramientas |
| ---- | ------------ |
| **Servicios** | `systemctl status`, `journalctl -u`, `ss -tulnp`, `nginx -t`, `sshd -t` |
| **Archivos y disco** | `df -h`, `df -i`, `lsblk`, `mount`, `ls -l`, `namei -l ruta` (permisos de **cada carpeta** del camino) |
| **Red** | `ip a`, `ip r`, `ping`, `ss`, `resolvectl`, `curl -v`, `tcpdump` |
| **Procesos y carga** | `top`, `ps`, `vmstat`, `iostat` |
| **Permisos y seguridad** | `id`, `sudo -u`, `ufw status`, `nft list ruleset`, `aa-status` |
| **Logs** | `journalctl -xe`, `tail -f`, `grep` |

---

## 🛠️ Las ocho averías

Para cada una: **lee el síntoma**, **no mires "Cómo provocarla"** hasta que esté resuelta (o hasta que alguien las provoque por ti), **diagnostica**, **arregla** y **apunta el tiempo**.

---

### Avería 1 · "La intranet no carga"

**Síntoma:** al abrir `http://192.168.56.10` el navegador **no conecta**. Hace una hora funcionaba.

<details>
<summary>🔧 Cómo provocarla</summary>

Introduce un error de sintaxis en un sitio de Nginx (por ejemplo, borra un `;` de una directiva en `/etc/nginx/sites-available/empresa`) y ejecuta `sudo systemctl restart nginx` (fallará, y el servicio quedará **parado**).
</details>

---

### Avería 2 · "No se puede guardar nada"

**Síntoma:** una aplicación y los usuarios reciben *"No space left on device"* al escribir en `/srv/proyectos`, pero `df -h` muestra... que **hay sitio** (o no). **Averigua por qué.**

<details>
<summary>🔧 Cómo provocarla</summary>

Llena el volumen: `sudo fallocate -l 4G /srv/proyectos/relleno.bin` (ajusta el tamaño al de tu disco). Variante difícil: **agota los inodos** creando miles de archivos vacíos (`for i in $(seq 1 200000); do touch /srv/proyectos/f$i; done`) y fíjate en `df -i`.
</details>

---

### Avería 3 · "Ya no puedo entrar por SSH con mi clave"

**Síntoma:** `ssh` te pide la contraseña (o te rechaza) aunque **nada ha cambiado** en tu clave. Hasta ayer entrabas.

<details>
<summary>🔧 Cómo provocarla</summary>

En el servidor, cambia los permisos para que SSH desconfíe: `chmod 777 ~/.ssh` (y/o `chmod 666 ~/.ssh/authorized_keys`). Entra por la **consola de VirtualBox** para diagnosticar.
</details>

---

### Avería 4 · "No funciona internet… pero sí puedo hacer ping"

**Síntoma:** `ping 8.8.8.8` **responde**, pero `apt update` falla y `ping google.com` dice *"Temporary failure in name resolution"*.

<details>
<summary>🔧 Cómo provocarla</summary>

En la configuración de `netplan` del adaptador NAT, pon un DNS inexistente, por ejemplo `nameservers: addresses: [192.0.2.1]` (`192.0.2.0/24` es una red reservada para documentación y no responde), y aplica con `sudo netplan apply`.
</details>

---

### Avería 5 · "Nginx no arranca"

**Síntoma:** tras un reinicio, `systemctl start nginx` falla con *"Address already in use"*.

<details>
<summary>🔧 Cómo provocarla</summary>

Lanza otro programa que ocupe el puerto 80 **antes** de arrancar Nginx: `sudo python3 -m http.server 80 &`.
</details>

---

### Avería 6 · "El backup de cada noche no se ha hecho"

**Síntoma:** a la mañana siguiente, la carpeta de backups **no tiene copia nueva**. Nadie ha tocado `cron` ni el script.

<details>
<summary>🔧 Cómo provocarla</summary>

Quita el permiso de ejecución al script del backup (`chmod -x /usr/local/bin/backup.sh`) y deja que `cron` lo intente, o cambia **la ruta** de `cron` a una que no exista. Mira **qué log** te lo diría.
</details>

---

### Avería 7 · "La carpeta compartida: unos pueden escribir y otros no"

**Síntoma:** `ana` guarda archivos en `/srv/compartido`, pero `luis` (**del mismo grupo**) recibe *"Permission denied"* al modificarlos.

<details>
<summary>🔧 Cómo provocarla</summary>

Crea la carpeta con grupo `sistemas`, pero **sin permiso de escritura para el grupo** (`chmod 755`), o haz que los archivos nuevos **no hereden el grupo** (sin `setgid`) y con `umask 022`. Recuerda `namei -l`.
</details>

---

### Avería 8 · "El servidor va lentísimo y no sabemos por qué"

**Síntoma:** la VM va muy lenta desde hace un rato. `top` no muestra un culpable obvio... ¿o sí? Mira más allá del primer proceso.

<details>
<summary>🔧 Cómo provocarla</summary>

Programa en `cron` (cada minuto) una tarea que lance un proceso que consuma CPU durante mucho tiempo (por ejemplo, `yes > /dev/null` con `timeout 3000`), de modo que **se acumulen** varios procesos. Alternativa: lanza varios `stress-ng` con nombres poco evidentes.
</details>

---

## ✅ Comprobación

Para **cada avería**, tu runbook debe tener estos campos rellenos:

| Campo | Contenido |
| ----- | --------- |
| **Síntoma** | Lo que se veía |
| **Capa afectada** | Qué parte del sistema fallaba |
| **Diagnóstico** | **Los comandos** que usaste y **qué salida** te llevó a la causa |
| **Causa raíz** | El **porqué real**, no el síntoma |
| **Solución** | Qué cambiaste, exactamente |
| **Verificación** | Cómo comprobaste que está arreglado |
| **Prevención** | Qué evitaría que **volviera a pasar** (monitorización, alerta, proceso...) |
| **Tiempo** | Cuánto tardaste |

---

## 🧩 Pistas

- **No reinicies "a ver si se arregla"**: puedes borrar la evidencia (logs, procesos, estado).
- **Cambia una cosa cada vez**. Si cambias tres a la vez y funciona, no sabrás **cuál** lo arregló.
- Si dos cosas parecen igual de probables, busca **una prueba que las separe**.
- **`systemctl status` + `journalctl -u`** resuelven más de la mitad de las averías de servicio.
- Un error que dice *"permission denied"* **no siempre son permisos de archivo**: puede ser un firewall, AppArmor, o una carpeta superior sin `x`.
- Si te atascas más de 20 minutos, **para**, escribe lo que sabes y lo que has descartado, y revisa tus **suposiciones**.

---

## 🚀 Retos extra

1. **Averías compuestas:** provoca **dos a la vez** (por ejemplo, la 4 y la 5) y comprueba si sabes **separarlas**.
2. **Pasa el reto a un compañero**: dile solo el síntoma y mide **su** tiempo.
3. Diseña **tres averías nuevas** tuyas, de entornos que hayas montado (LVM, WireGuard, fail2ban...), con su "Cómo provocarla".
4. Automatiza un **script de comprobación rápida** (`salud.sh`) que ejecute en segundos las 10 comprobaciones que más te han servido: servicios caídos, disco, inodos, DNS, hora, carga...
5. Pasa cada **prevención** a la monitorización del laboratorio 08 del avanzado: ¿qué **alerta** de Prometheus te habría avisado antes?

---

## 📝 Qué documentar en tu `Solucion.md`

- **Ocho runbooks** con la tabla de arriba.
- Una **lista de los cinco comandos que más te han salvado**.
- Tu **checklist personal de diagnóstico** (de diez puntos).
- Una **autoevaluación**: ¿qué averías te costaron más y **por qué**? ¿Qué **suposiciones** erróneas hiciste?

---

## 🧠 Lo que has practicado

`método de diagnóstico` · `hipótesis y pruebas` · `causa raíz` · `servicios, disco, red, permisos y rendimiento` · `runbooks` · `prevención` · `trabajar bajo presión`
