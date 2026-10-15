# 🚨 Laboratorio 12 · Respuesta a incidentes

**Nivel:** 🔴 Experto · **Sistema:** 🐧 Linux · **Tiempo estimado:** 150 minutos

---

## 🎯 Escenario

Es lunes por la mañana. La monitorización avisa de que `ubuntu-server-lab` **ha enviado tráfico extraño por la noche**, y un compañero comenta que ha visto **un usuario que no reconoce** en la máquina. Hay **sospecha de que alguien ha entrado**.

Es el momento de actuar como el **equipo de respuesta**: **investigar sin destruir evidencias**, **contener** el problema, **eliminarlo**, **recuperar** el servicio y **dejar un informe** profesional. Aquí se juntan casi todos los laboratorios anteriores: logs, SSH, cron, procesos, red, backups y monitorización.

> ⚠️ **Todo ocurre en tu laboratorio.** Los "indicadores del intruso" son **inofensivos** y los plantas **tú mismo** (o un compañero) en tu propia VM. **Haz snapshot antes de plantarlos.** No uses estas técnicas de investigación en sistemas que no sean tuyos sin autorización escrita.

---

## 📖 Conceptos nuevos

### Las fases de la respuesta a incidentes

Según el marco más extendido (NIST SP 800-61), un incidente se gestiona en fases:

| Fase | Qué se hace |
| ---- | ----------- |
| **1. Preparación** | Tener logs, copias, contactos, procedimientos **antes** de que pase algo (lo que has ido construyendo en estos niveles) |
| **2. Detección y análisis** | Confirmar que **hay** un incidente, entender **qué**, **cuándo**, **cómo** y **hasta dónde llegó** |
| **3. Contención** | **Limitar el daño**: aislar el equipo, bloquear cuentas o accesos, sin destruir evidencia |
| **4. Erradicación** | **Eliminar la causa**: cuentas, puertas traseras, procesos, la vulnerabilidad de entrada |
| **5. Recuperación** | **Volver a la normalidad** de forma segura y vigilada |
| **6. Lecciones aprendidas** | **Informe post-incidente** y mejoras |

### Reglas de oro

| Regla | Por qué |
| ----- | ------- |
| **No borres lo que no has copiado** | Una vez eliminas una pista, **desaparece para siempre** |
| **Documenta todo con hora** | Qué hiciste, cuándo y por qué: tu propia **bitácora** |
| **Captura primero lo volátil** | La memoria, los procesos y las conexiones **desaparecen al apagar**: guarda su estado **antes de reiniciar** |
| **No confíes en el sistema comprometido** | Un intruso puede haber cambiado herramientas (`ls`, `ps`...). Si puedes, compara con **otro equipo limpio** |
| **No apagues "para ahorrar tiempo"** | Apagar destruye lo volátil. Primero **aísla**, luego decide |
| **Presupón más de lo que ves** | Si encuentras una puerta trasera, **busca otras** |

### Indicadores de compromiso (IOC)

Un **IOC** es una **pista** de que algo ha pasado: una cuenta nueva, un archivo extraño, un proceso desconocido, una conexión a una IP rara. Anotar los IOC permite **buscarlos en otros equipos**.

### Persistencia

Un intruso que entra por una vía quiere **poder volver** aunque la cierres. A esas "llaves de repuesto" se les llama **mecanismos de persistencia**. En Linux, los sitios clásicos donde mirar son:

| Sitio | Qué revisar |
| ----- | ----------- |
| **Cuentas** | `/etc/passwd`, `/etc/group` (¿usuarios nuevos? ¿en el grupo `sudo`?) |
| **Claves SSH** | `~/.ssh/authorized_keys` de **cada** usuario |
| **Tareas programadas** | `crontab -l` de cada usuario, `/etc/cron.*`, **timers** de `systemd` |
| **Servicios** | `systemctl list-unit-files --state=enabled`, units en `/etc/systemd/system/` |
| **Scripts de inicio** | `~/.bashrc`, `~/.profile`, `/etc/profile.d/` |
| **Procesos y puertos** | `ps`, `ss -tulnp` (¿algo escuchando que no debería?) |
| **Archivos modificados** | `find / -mtime -2` (modificados en los últimos 2 días) |
| **Resolución de nombres** | `/etc/hosts` |

### Herramientas

| Comando | Qué hace |
| ------- | -------- |
| `last`, `lastb`, `who`, `w` | Quién ha iniciado sesión (y quién **falló**) |
| `sudo grep -E "Accepted\|Failed\|sudo:\|useradd\|usermod" /var/log/auth.log` | Autenticaciones y cambios de cuentas |
| `sudo journalctl --since "yesterday"` | Todo lo ocurrido desde ayer |
| `ps auxf` | Procesos en forma de **árbol** (quién lanzó a quién) |
| `sudo ss -tupan` | **Todas** las conexiones y puertos con su proceso |
| `sudo ls -la /proc/PID/exe` | **Qué ejecutable real** hay detrás de un proceso |
| `sudo find / -xdev -newermt "2026-10-12" -type f 2>/dev/null \| head` | Archivos **modificados desde una fecha** (`-xdev` no sale del sistema de ficheros) |
| `sudo crontab -l -u usuario` | Tareas programadas de **otro** usuario |
| `getent passwd \| awk -F: '$3>=1000'` | Usuarios con UID de persona |
| `sha256sum archivo` | **Huella** de un archivo, para demostrar que **no se ha alterado** tu evidencia |

---

## 🛠️ Qué debes hacer

### Parte A · Preparación (del equipo y del escenario)

1. **Haz snapshot** de `ubuntu-server-lab`. Comprueba que tienes a mano: logs, **backup reciente**, acceso por consola.
2. **Prepara la plantilla de informe** (la de más abajo) y tu **bitácora** (un archivo donde apuntarás cada acción con hora).
3. **Planta los "indicadores del intruso".** Si trabajas solo, hazlo y **no vuelvas a leer** la lista hasta terminar; mejor si **otra persona** lo hace sin decirte nada.

<details>
<summary>🔧 Cómo plantar el incidente simulado (todo inofensivo)</summary>

Crea indicadores **visibles pero no dañinos**:

1. **Cuenta nueva con privilegios:** `sudo useradd -m -s /bin/bash soporte2` y `sudo usermod -aG sudo soporte2`.
2. **Clave SSH extra:** genera una clave de prueba (`ssh-keygen -t ed25519 -f /tmp/clave_falsa -N ""`) y añade su `.pub` al `authorized_keys` de tu usuario.
3. **Tarea programada sospechosa:** en el `crontab` de `soporte2`, una línea cada minuto que escriba la hora en un archivo oculto: `* * * * * date >> /tmp/.sesion.log`.
4. **Puerto a la escucha:** lanza `nohup python3 -m http.server 4444 --directory /tmp >/dev/null 2>&1 &` (simula un servicio no autorizado).
5. **Entrada en `/etc/hosts`:** añade una línea `203.0.113.50 actualizaciones.empresa.lab` (`203.0.113.0/24` es una red reservada para documentación).
6. **Intentos de acceso fallidos:** desde `ubuntu-cliente`, haz varios intentos de SSH con contraseñas incorrectas al servidor, y **un acceso correcto** con la cuenta `soporte2`.
7. (Opcional) **Un archivo modificado**: cambia el texto de un `index.html` de Nginx.
</details>

### Parte B · Detección y análisis (¡sin tocar nada todavía!)

4. **Captura primero lo volátil** y guárdalo en una carpeta de evidencias (y calcula su `sha256sum`): procesos (`ps auxf`), conexiones y puertos (`ss -tupan`), sesiones (`who`, `w`), tareas, **hora del sistema**.
5. Busca **cada tipo de indicador** de la tabla de persistencia. Para cada hallazgo apunta **qué es, dónde estaba y cuándo se creó** (usa fechas de archivos y logs).
6. **Reconstruye la línea temporal** con `auth.log` y el journal: **cuándo** entró alguien, **desde qué IP**, **con qué cuenta**, **qué hizo** (`sudo`, creación de usuarios...).
7. Haz **tres preguntas** y responde con evidencias: ¿**Cómo entró**? ¿**Qué ha hecho**? ¿**Sigue dentro**?
8. **Alcance:** ¿hay indicios de que **otros equipos** (el cliente, el servidor de logs...) estén afectados? Si tienes los logs centralizados (laboratorio 04), **cruza** los datos.

### Parte C · Contención

9. **Aisla** el servidor **sin apagarlo**: bloquea la IP de origen o limita el acceso con el firewall (`nftables`/`ufw`) y/o **bloquea la cuenta sospechosa** (`usermod -L`, `chage -E 0`), permitiendo solo tu acceso de administración.
10. Haz un **snapshot de evidencias** de la VM (o una copia de los archivos relevantes con su hash) **antes de limpiar**.

### Parte D · Erradicación y recuperación

11. **Elimina cada mecanismo de persistencia** que encontraste (cuenta, clave SSH, tarea, proceso, entrada de `hosts`, archivo modificado), **y anota cada uno** en tu bitácora.
12. **Busca más:** repasa de nuevo **todos** los sitios de la tabla: *¿hay algo que se te haya escapado?*
13. **Cierra la vía de entrada**: ¿qué falló? (¿Contraseña débil? ¿SSH con contraseña? ¿Sin `fail2ban`? ¿Cuenta sin protección?) **Corrígelo** (claves SSH, `fail2ban`, hardening: laboratorios del nivel avanzado).
14. **Cambia las credenciales** que puedan haberse visto comprometidas.
15. **Recupera** el servicio con confianza: si dudas de la integridad, **restaura desde un backup limpio** (laboratorio 08 del experto) y **vigila** con la monitorización reforzada.

### Parte E · Lecciones aprendidas

16. Escribe el **informe de incidente** con la plantilla.
17. **Propón mejoras**: ¿qué **alerta** te habría avisado antes? ¿Qué control habría impedido la entrada? ¿Qué te habría ayudado a **investigar más rápido**?

---

## ✅ Comprobación

| Prueba | Qué debes ver |
| ------ | ------------- |
| Carpeta de evidencias | Con **hash** de cada archivo, **antes** de limpiar nada |
| Bitácora | Con **hora y acción** de cada paso |
| `getent passwd soporte2` tras limpiar | Sin resultado |
| `sudo ss -tulnp \| grep 4444` | Sin resultado |
| Todos los `authorized_keys` | **Sin** la clave extraña |
| `crontab -l` de todos los usuarios | Sin tareas sospechosas |
| `auth.log` | Línea temporal reconstruida **con evidencias** |
| Informe de incidente | **Completo** (plantilla) |

---

## 🧩 Pistas

- **No te lances a borrar.** Primero documenta y copia; luego contén; después limpia.
- La fecha de creación de un usuario no se guarda directamente: **deduce** cuándo se creó con `/etc/passwd`, el directorio `/home/usuario` (`stat`) y las líneas `useradd` de `auth.log`.
- Si un proceso aparece con **nombre normal**, compruébalo con `ls -la /proc/PID/exe` y `ps auxf`: ¿quién es su padre?
- Un servicio en un puerto raro (`4444`) no es malo **por el número**, pero **todo servicio no autorizado es un hallazgo**.
- Si sospechas del sistema, **no confíes solo en sus herramientas**: compara con las de otro equipo limpio (por ejemplo, ejecutando `ss` desde fuera con `nmap` sobre tu propio servidor).
- Lo que **no** ves no demuestra que no esté: **un buen informe distingue lo confirmado de lo supuesto**.

---

## 🚀 Retos extra

1. Escribe un **script de triaje** (`triaje.sh`) que, en **menos de un minuto**, recoja automáticamente **todas las evidencias volátiles** y los chequeos de persistencia y las guarde con hash.
2. Busca el incidente **en tus logs centralizados** (experto, 04) y en las **alertas de Suricata** (experto, 06): ¿cuántas pistas habrían saltado **antes** de que te avisara el compañero?
3. Repite el ejercicio con un **segundo intruso más discreto** (sin cuenta nueva: **solo** una clave SSH y una tarea programada) y mide si lo encuentras.
4. Investiga **volcados de memoria** (*memory forensics*) y qué se puede averiguar **solo** de la RAM. Es concepto, no práctica.
5. Redacta un **plan de respuesta a incidentes** de una página para una pyme: contactos, quién decide aislar, cómo se comunica, a quién hay que avisar (por ejemplo, **notificación obligatoria** de brechas de datos personales en el **RGPD**, con plazos).

---

## 📝 Qué documentar en tu `Solucion.md`

**Plantilla de informe de incidente:**

| Sección | Contenido |
| ------- | --------- |
| **Resumen ejecutivo** | 3-4 líneas, sin tecnicismos: qué pasó, impacto, estado actual |
| **Cronología** | Tabla con **fecha/hora, evento, evidencia** |
| **Vector de entrada** | Cómo accedió el intruso (**lo confirmado** y **lo supuesto**) |
| **Acciones del atacante** | Qué hizo en el sistema |
| **Indicadores de compromiso** | Lista de IOC (cuentas, claves, IP, archivos, puertos) |
| **Contención y erradicación** | Qué hiciste, en qué orden y por qué |
| **Recuperación** | Cómo se verificó que el sistema vuelve a ser fiable |
| **Impacto** | Datos, servicios y tiempo afectados |
| **Causa raíz** | El **porqué** de que fuera posible |
| **Lecciones y mejoras** | Acciones concretas, con **responsable y plazo** |

---

## 🧠 Lo que has practicado

`fases de respuesta a incidentes` · `recogida de evidencias y cadena de custodia` · `persistencia e IOC` · `análisis de logs` · `contención sin destruir pruebas` · `erradicación y recuperación` · `informe post-incidente`
