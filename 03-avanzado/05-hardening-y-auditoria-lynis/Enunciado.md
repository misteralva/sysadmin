# 🛡️ Laboratorio 05 · Hardening y auditoría con Lynis

**Nivel:** 🟠 Avanzado · **Sistema:** 🐧 Linux · **Tiempo estimado:** 100 minutos

---

## 🎯 Escenario

Un servidor recién instalado es **funcional, no seguro**. Trae servicios que quizá no necesitas, parámetros de red permisivos y contraseñas sin requisitos. Antes de entregar un servidor a producción, un buen sysadmin lo **endurece** (*hardening*): reduce todo lo que un atacante podría aprovechar.

Tu misión es **auditar** tu servidor con una herramienta profesional, aplicar mejoras y **medir** cuánto has mejorado.

> ⚠️ **El hardening mal hecho rompe cosas** (un servicio deja de funcionar, te quedas sin SSH...). **Snapshot antes de empezar**, aplica los cambios **de uno en uno** y comprueba después de cada uno.

---

## 📖 Conceptos nuevos

### Superficie de ataque

La **superficie de ataque** es todo lo que un atacante podría intentar usar: servicios en escucha, puertos abiertos, usuarios, paquetes instalados, configuraciones débiles. **Hardening = reducirla.** La regla de oro: *"lo que no está instalado ni activo no se puede atacar"*.

### Estándares de referencia

Hay guías reconocidas con listas de controles. La más famosa son los **CIS Benchmarks** (*Center for Internet Security*), con recomendaciones concretas por sistema. Herramientas como **Lynis** se basan en estas ideas para evaluar un sistema.

### Lynis

**Lynis** es una herramienta de auditoría de código abierto que recorre el sistema y genera:
- Un **índice de hardening** (de 0 a 100).
- **Advertencias** (*warnings*) y **sugerencias** (*suggestions*).

Es **solo de lectura**: no cambia nada. Tú decides qué aplicar.

### Áreas habituales de hardening

| Área | Qué se revisa | Ejemplos |
| ---- | ------------- | -------- |
| **Acceso remoto** | SSH | `PermitRootLogin no`, `MaxAuthTries 3`, `X11Forwarding no`, `AllowUsers` |
| **Autenticación** | Contraseñas y cuentas | Calidad de contraseñas (`libpam-pwquality`), bloqueo por intentos |
| **Kernel y red** | Parámetros `sysctl` | No aceptar redirecciones ICMP, activar `tcp_syncookies`, filtrado de ruta inversa `rp_filter` |
| **Actualizaciones** | Parches | `unattended-upgrades` |
| **Servicios** | Qué escucha | Desactivar lo innecesario |
| **Ficheros** | Permisos | Permisos de `/etc/shadow`, ficheros con SUID |
| **Auditoría** | Quién hace qué | `auditd` y reglas de observación |
| **Avisos legales** | Banners | `/etc/issue.net` |

### `sysctl`: parámetros del kernel

`sysctl` permite ver y cambiar parámetros del kernel **en caliente**. Para que sobrevivan al reinicio, se guardan en `/etc/sysctl.d/*.conf`. Ejemplo de formato de línea: `clave.del.parametro = valor`.

### `auditd`: el "cuaderno de bitácora" del sistema

`auditd` es el servicio de **auditoría** del kernel: puede registrar, por ejemplo, **quién toca** un archivo sensible. Con `auditctl -w /etc/passwd -p wa -k identidad` se vigila ese archivo (`-w` = ruta, `-p wa` = escrituras y cambios de atributos, `-k` = etiqueta para buscar después).

### Comandos de este laboratorio

| Comando | Qué hace |
| ------- | -------- |
| `sudo apt install lynis` | Instala la herramienta |
| `sudo lynis audit system` | Realiza la **auditoría completa** (tarda unos minutos; pulsa Enter cuando pida pausa) |
| `sudo grep -E "warning\|suggestion" /var/log/lynis-report.dat` | Extrae avisos y sugerencias del informe |
| `sudo grep hardening_index /var/log/lynis-report.dat` | Muestra tu **índice de hardening** |
| `sudo lynis show details TEST-ID` | Explica una sugerencia concreta (el ID aparece entre corchetes) |
| `sysctl net.ipv4.tcp_syncookies` | Consulta un parámetro del kernel |
| `sudo sysctl --system` | Recarga **todos** los archivos de `/etc/sysctl.d/` |
| `sudo sshd -t` | Valida la configuración SSH |
| `sudo apt install libpam-pwquality auditd unattended-upgrades` | Instala herramientas de hardening |
| `sudo ausearch -k identidad` | Busca eventos de auditoría por etiqueta |
| `systemctl list-unit-files --state=enabled` | Servicios habilitados al arrancar |
| `sudo find / -perm -4000 -type f 2>/dev/null` | Lista archivos con **SUID** (se ejecutan con privilegios del propietario; deben revisarse) |

---

## 🛠️ Qué debes hacer

### Parte A · Punto de partida

1. Instala `lynis` y ejecuta una auditoría completa **antes de tocar nada**.
2. Anota tu **índice de hardening inicial** y copia el informe a un archivo (no lo subas a GitHub, puede contener datos sensibles de tu sistema).
3. Lee la lista de **advertencias** y **sugerencias**. Clasifícalas por área (SSH, red, autenticación, etc.).

### Parte B · Aplicar mejoras (una a una)

Elige **al menos 8 mejoras** de tu informe y aplícalas, **comprobando tras cada una** que el servidor sigue funcionando. Debes incluir al menos una de cada familia:

4. **SSH:** endurece la configuración con un archivo en `/etc/ssh/sshd_config.d/` (por ejemplo: deshabilitar `root`, limitar los intentos, desactivar el reenvío X11, permitir solo tu usuario). Valida con `sshd -t` antes de recargar y **mantén una sesión abierta**.
5. **Red/kernel:** crea `/etc/sysctl.d/99-hardening.conf` con al menos **tres** parámetros de red de tu informe (por ejemplo, `tcp_syncookies`, no aceptar `redirects`, `rp_filter`). Aplica con `sysctl --system` y comprueba cada valor.
6. **Contraseñas:** instala `libpam-pwquality` y exige una **longitud mínima** y variedad de caracteres. Pruébalo cambiando una contraseña de un usuario de prueba.
7. **Actualizaciones:** activa `unattended-upgrades` (actualizaciones de seguridad automáticas) y comprueba su configuración.
8. **Servicios:** lista los servicios habilitados, identifica **uno innecesario** en tu servidor y **desactívalo** con criterio.
9. **Banner legal:** configura un aviso en `/etc/issue.net` y actívalo en SSH con la directiva `Banner`.
10. **Auditoría:** instala `auditd` y crea una regla que **vigile `/etc/passwd`** o `/etc/ssh/sshd_config`. Provoca un cambio (por ejemplo, crea un usuario de prueba) y búscalo con `ausearch`.

### Parte C · Medir

11. Vuelve a ejecutar `lynis audit system` y **compara** el nuevo índice con el inicial.
12. Anota: ¿qué mejoras subieron más el índice? ¿Hay alguna sugerencia que **decidiste no aplicar**? ¿Por qué?

---

## ✅ Comprobación

| Prueba | Qué debes ver |
| ------ | ------------- |
| `sudo grep hardening_index /var/log/lynis-report.dat` | Un número **mayor** que el inicial |
| `sudo sshd -T \| grep -E "permitrootlogin\|maxauthtries\|x11forwarding"` | Tus valores endurecidos |
| `sysctl net.ipv4.tcp_syncookies` | `= 1` |
| Cambiar una contraseña con una débil | **Rechazada** por `pwquality` |
| `systemctl status unattended-upgrades` | Activo |
| `sudo ausearch -k identidad` | El evento que provocaste |
| Conectar por SSH | **Sigue funcionando** |

---

## 🧩 Pistas

- No apliques todas las sugerencias **a ciegas**: algunas son discutibles según el uso del servidor. Entender el *porqué* de cada una es parte del ejercicio.
- Aplica **un cambio, una comprobación**. Si algo se rompe, sabrás qué fue.
- Para los parámetros `sysctl`, el nombre completo (por ejemplo, `net.ipv4.conf.all.rp_filter`) suele aparecer en la sugerencia de Lynis.
- Si Lynis te dice que no encuentra un componente, instala el paquete que sugiere.
- Si una mejora rompe un servicio, **restaura el snapshot** y vuelve a empezar con más cuidado.

---

## 🚀 Retos extra

1. Revisa los archivos **SUID** de tu sistema y averigua para qué sirven los cinco primeros. ¿Cuáles serían peligrosos si estuvieran mal configurados?
2. Haz que `/tmp` se monte con las opciones `noexec,nosuid,nodev` (en `fstab`). ¿Qué ataque dificulta? ¿Qué podría romper?
3. Investiga **los CIS Benchmarks** de Ubuntu: elige tres controles de su lista y compáralos con lo que has hecho.
4. Escribe un **script** que compruebe automáticamente diez de tus controles de hardening y te diga cuáles cumples (reutiliza lo que sabes de Bash).
5. Reflexiona: ¿en qué se diferencian **hardening** y **firewall**? ¿Pueden sustituirse?

---

## 📝 Qué documentar en tu `Solucion.md`

- Una tabla con cada mejora: **qué cambiaste, por qué, cómo lo comprobaste y el riesgo de aplicarla**.
- Los índices de Lynis antes y después.
- Las sugerencias que **decidiste no aplicar** y la razón.
- Una reflexión sobre el **equilibrio entre seguridad y usabilidad**.

---

## 🧠 Lo que has practicado

`superficie de ataque` · `hardening` · `Lynis` · `CIS` · `sysctl` · `SSH seguro` · `pwquality` · `unattended-upgrades` · `auditd` · `SUID` · `medir la mejora`
