# 🚫 Laboratorio 04 · Protección automática con `fail2ban`

**Nivel:** 🟠 Avanzado · **Sistema:** 🐧 Linux · **Tiempo estimado:** 75 minutos

---

## 🎯 Escenario

En el laboratorio 06 del nivel intermedio descubriste cómo se ven los intentos de acceso fallidos en `auth.log`. En un servidor expuesto a internet, esos intentos llegan **por miles al día**, de bots que prueban contraseñas sin parar. Vigilarlos a mano es imposible.

Tu misión es instalar **`fail2ban`**, un "guardia" que lee los logs, detecta a quien falla demasiadas veces y **lo bloquea automáticamente** en el firewall durante un tiempo.

> 💾 Haz **snapshot** de las dos VMs. Usarás `ubuntu-cliente` (IP `192.168.56.11`) como **"intruso"** de pruebas, para no bloquear a tu propio ordenador. Todo ocurre entre tus VMs.

---

## 📖 Conceptos nuevos

### Cómo funciona `fail2ban`

1. **Lee** los logs de un servicio (por ejemplo, SSH).
2. Con un **filtro** (una expresión regular), reconoce las líneas que indican un **fallo** y de qué IP vienen.
3. Si una IP falla **N veces** en un tiempo **T**, aplica una **acción**: la **bloquea** en el firewall durante un tiempo.
4. Cuando termina la condena, la **desbloquea**.

> **Analogía:** un portero que apunta quién llama a la puerta con la clave equivocada. Si alguien se equivoca cinco veces seguidas, **le prohíbe la entrada durante diez minutos**.

### Cárceles (*jails*)

Cada servicio vigilado se configura en una **jail** (cárcel). Los parámetros más importantes:

| Parámetro | Qué significa |
| --------- | ------------- |
| `enabled` | Si la jail está activa |
| `maxretry` | Fallos permitidos antes de bloquear |
| `findtime` | Ventana de tiempo en la que se cuentan los fallos |
| `bantime` | Cuánto dura el bloqueo |
| `ignoreip` | IP que **nunca** se bloquean (tu red de confianza) |
| `logpath` / `backend` | De dónde lee los logs |

### Archivos de configuración

| Archivo | Para qué |
| ------- | -------- |
| `/etc/fail2ban/jail.conf` | Configuración **por defecto**. **No se edita** (se pisa al actualizar) |
| `/etc/fail2ban/jail.local` | **Tu** configuración: lo que pongas aquí sobrescribe lo anterior |
| `/etc/fail2ban/jail.d/*.local` | Alternativa: un archivo por jail |
| `/etc/fail2ban/filter.d/` | Los **filtros** (expresiones regulares) |

Plantilla mínima de `jail.local` (completa las `???`):

```ini
[DEFAULT]
ignoreip = ???
bantime  = ???
findtime = ???
maxretry = ???

[sshd]
enabled = true
```

### Comandos de este laboratorio

| Comando | Qué hace |
| ------- | -------- |
| `sudo apt install fail2ban` | Instala |
| `sudo systemctl enable --now fail2ban` | Arranca y habilita el servicio |
| `sudo fail2ban-client status` | Lista las jails activas |
| `sudo fail2ban-client status sshd` | Estado de la jail SSH: fallos, IP bloqueadas |
| `sudo fail2ban-client set sshd unbanip IP` | **Desbloquea** una IP manualmente |
| `sudo fail2ban-client reload` | Recarga la configuración |
| `sudo fail2ban-regex /var/log/auth.log /etc/fail2ban/filter.d/sshd.conf` | **Prueba** un filtro contra un log y dice cuántas líneas casa |
| `sudo tail -f /var/log/fail2ban.log` | Sigue en vivo lo que hace `fail2ban` |
| `sudo nft list ruleset \| grep -i f2b` | Muestra las reglas de bloqueo que ha creado |

---

## 🛠️ Qué debes hacer

### Parte A · Preparar el escenario

1. En el **servidor** (`192.168.56.10`), asegúrate de que SSH acepta **contraseña** (en el laboratorio 08 del nivel intermedio la desactivaste): actívala de nuevo **solo para este laboratorio**, sobre un snapshot, y recarga SSH.
2. Comprueba que desde `ubuntu-cliente` puedes conectar por SSH al servidor con contraseña.

### Parte B · Instalar y configurar

3. Instala `fail2ban` y comprueba que está activo.
4. Mira el estado de las jails: ¿está `sshd` activa por defecto en Ubuntu?
5. Crea tu `/etc/fail2ban/jail.local`: tras **3 fallos en 5 minutos**, bloquear **10 minutos**; y pon tu **ordenador anfitrión** (`192.168.56.1`) en `ignoreip` para no bloquearte tú, pero **no** `ubuntu-cliente`.
6. Recarga y comprueba que la configuración se ha cargado.
7. Prueba el filtro con `fail2ban-regex` contra `auth.log`: ¿cuántas líneas reconoce?

### Parte C · Provocar el bloqueo

8. En una terminal del servidor, sigue el log de `fail2ban` en vivo.
9. Desde **`ubuntu-cliente`**, intenta conectarte por SSH **con contraseña incorrecta** varias veces.
10. Observa cómo el servidor **bloquea** la IP. Intenta conectar otra vez desde `ubuntu-cliente`: ¿qué ocurre? (¿Error o conexión que se queda colgada? ¿Por qué?)
11. Mira en el servidor el estado de la jail y las reglas de firewall que se han añadido.
12. **Desbloquea** la IP manualmente y comprueba que `ubuntu-cliente` puede volver a conectar.
13. Espera a que **expire el bloqueo** en otro intento y comprueba que se levanta solo.

### Parte D · Reforzar

14. Añade un **`bantime` incremental** (que cada reincidencia dure más): investiga `bantime.increment`.
15. Cuando termines, **desactiva de nuevo** la contraseña en SSH (deja solo claves).

---

## ✅ Comprobación

| Prueba | Qué debes ver |
| ------ | ------------- |
| `sudo fail2ban-client status` | `Jail list: sshd` |
| `sudo fail2ban-client status sshd` | `Currently failed`, `Total failed`, `Currently banned` con la IP del cliente |
| Cuarto intento desde el cliente | **Sin respuesta** o rechazo (la IP está bloqueada) |
| `sudo tail /var/log/fail2ban.log` | Líneas `Ban 192.168.56.11` y, más tarde, `Unban` |
| `sudo nft list ruleset \| grep -i f2b` | Una regla/ conjunto con la IP bloqueada |
| Tras `unbanip` | El cliente vuelve a poder conectar |

---

## 🧩 Pistas

- Si `fail2ban` no bloquea, mira primero `fail2ban-regex`: si el filtro no reconoce líneas, no hay nada que contar.
- En Ubuntu moderno, SSH escribe en el **journal**: la jail puede necesitar `backend = systemd`. Mira `jail.d/defaults-debian.conf`, que ya lo define.
- Una IP puesta en `ignoreip` **nunca** se bloquea, aunque falle cien veces. Si no te bloquea el cliente, comprueba que **no** está ahí.
- El tiempo se escribe como `10m`, `1h`, `1d`.
- Cada vez que cambies `jail.local`, `fail2ban-client reload`.

---

## 🚀 Retos extra

1. Crea una **jail para Nginx** (`nginx-http-auth`) y pruébala con autenticación básica del laboratorio 10 del nivel intermedio.
2. Escribe un **filtro propio** para detectar, por ejemplo, demasiados **404** en el log de Nginx (en `filter.d/`), y pruébalo con `fail2ban-regex`.
3. Cambia la **acción de bloqueo** para que use `nftables` o `ufw` en vez de la predeterminada, y observa en qué se nota.
4. Configura un **aviso por correo** (solo la configuración, sin servidor de correo real) y razona qué necesitarías para que funcione.
5. Piensa en la **limitación de `fail2ban`**: ¿qué ataque no detendría? (Pista: piensa en ataques lentos o desde muchas IP.)

---

## 📝 Qué documentar en tu `Solucion.md`

- Tu `jail.local` explicado línea a línea.
- El fragmento de `fail2ban.log` con el `Ban` y el `Unban`.
- Una comparación con `ufw limit` y con SSH por clave: ¿qué capa de protección da cada una?
- Cuándo **no** conviene usar `fail2ban` (por ejemplo, si ya solo permites claves).

---

## 🧠 Lo que has practicado

`fail2ban` · `jails y filtros` · `maxretry / findtime / bantime / ignoreip` · `fail2ban-regex` · `bloqueo dinámico en el firewall` · `defensa en profundidad`
