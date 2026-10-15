# 🛠️ Laboratorio 11 · Break/fix en Windows y red

**Nivel:** 🔴 Experto · **Sistema:** 🪟 Windows Server y cliente + 🐧 red · **Tiempo estimado:** 150 minutos

---

## 🎯 Escenario

En una empresa real, un problema casi nunca viene etiquetado: *"no puedo entrar"*, *"la carpeta no abre"*, *"no tengo internet"*. La causa puede estar en el **cliente**, en el **servidor**, en el **DNS**, en la **hora**, en una **directiva**... o en **varios sitios a la vez**.

Este laboratorio es la versión Windows y red del anterior: **ocho averías** en el dominio y en la red que has montado. Solo tienes el **síntoma**. Tu misión es **diagnosticar con método**, **arreglar la causa raíz** y **documentar** cada caso en un runbook.

> 💾 **Snapshot** de `dc01`, `pc-cliente01` y (para las averías de red) `router-lab`, `cliente-a` y `cliente-b`. **Vuelve al snapshot tras cada avería.** Idealmente, **otra persona** las provoca sin decirte cuáles son.

---

## 📖 Conceptos nuevos

### El método, igual que en Linux

Repite el ciclo del laboratorio anterior: **observar → hipótesis → comprobar → arreglar → verificar → documentar**. Lo que cambia son las **herramientas** y los **sospechosos habituales** de un dominio Windows.

### Los sospechosos habituales de un dominio

| Sospechoso | Por qué suele ser culpable |
| ---------- | -------------------------- |
| **DNS** | AD **depende de DNS**; casi todo fallo de dominio empieza ahí |
| **Hora** | **Kerberos** rechaza autenticaciones si la hora difiere **más de 5 minutos** entre cliente y DC |
| **Cuenta** | Bloqueada, desactivada, caducada, contraseña caducada |
| **Permisos en dos capas** | Permisos del **recurso compartido** frente a permisos **NTFS**: gana el más restrictivo |
| **GPO** | No vinculada, mal filtrada, el objeto no está en la OU correcta, o aún no se ha actualizado |
| **DHCP** | Ámbito desactivado o servidor no autorizado |
| **Red** | Puerta de enlace, reenvío, NAT, VLAN mal etiquetada |

### Herramientas del diagnóstico en Windows

| Comando | Qué comprueba |
| ------- | ------------- |
| `ipconfig /all` | IP, DNS, servidor DHCP, sufijo del dominio |
| `nslookup nombre` / `Resolve-DnsName nombre` | Resolución de nombres |
| `Test-NetConnection IP -Port N` | **Conectividad a un puerto** (mezcla de `ping` y `telnet`) |
| `nltest /dsgetdc:empresa.local` | ¿Encuentra el cliente **un controlador de dominio**? |
| `Test-ComputerSecureChannel -Verbose` | Estado de la **relación de confianza** del equipo con el dominio (`-Repair` intenta repararla) |
| `w32tm /query /status` y `w32tm /resync` | Estado de la **hora** y resincronización |
| `gpresult /r` y `gpupdate /force` | Qué GPO se aplican y forzar la actualización |
| `Get-WinEvent -LogName System -MaxEvents 20` | Eventos recientes (equivale a `journalctl`) |
| `klist` | Tickets **Kerberos** del usuario (si no hay, la autenticación falla) |
| `Get-ADUser nombre -Properties LockedOut, Enabled, PasswordExpired` | Estado de una cuenta |
| `Get-DhcpServerv4Scope` / `Get-DhcpServerv4Lease` | Estado del DHCP |
| `Get-SmbShareAccess` y `icacls` | Permisos del recurso y NTFS |

Y de red en Linux (laboratorios 10 y 11 del avanzado): `ip a`, `ip r`, `ip route get`, `sysctl net.ipv4.ip_forward`, `sudo nft list ruleset`, `tcpdump`.

---

## 🛠️ Las ocho averías

Para cada una: **lee el síntoma**, **no abras "Cómo provocarla"** hasta resolverla (o hasta que otra persona la provoque), **diagnostica**, **arregla** y **apunta el tiempo**.

---

### Avería 1 · "No puedo iniciar sesión con mi usuario del dominio"

**Síntoma:** en el cliente, al entrar como `EMPRESA\ana`, sale *"No se puede contactar con el dominio"* o *"No hay servidores de inicio de sesión disponibles"*. El día anterior funcionaba.

<details>
<summary>🔧 Cómo provocarla</summary>

En el **cliente**, cambia el DNS a uno que no conoce el dominio:
`Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 8.8.8.8`
</details>

---

### Avería 2 · "Nadie en la red recibe IP"

**Síntoma:** el cliente tiene una IP `169.254.x.x` (autoconfiguración). Antes recibía una del rango de DHCP.

<details>
<summary>🔧 Cómo provocarla</summary>

En `dc01`, desactiva el ámbito:
`Set-DhcpServerv4Scope -ScopeId 192.168.10.0 -State InActive`
(o desautoriza el servidor DHCP con `Remove-DhcpServerInDC`).
</details>

---

### Avería 3 · "La política del Panel de control ya no se aplica"

**Síntoma:** `marta` ya **puede** abrir el Panel de control, y antes no. Nadie ha tocado la GPO... o eso dicen.

<details>
<summary>🔧 Cómo provocarla</summary>

De varias formas (elige una): desvincula la GPO de la OU (`Remove-GPLink`), **quita de la delegación** el permiso de lectura a `Authenticated Users` tras filtrar por un grupo, o **mueve a `marta`** a otra OU.
</details>

---

### Avería 4 · "Nadie puede abrir la carpeta compartida"

**Síntoma:** `\\DC01\Sistemas` da *"Acceso denegado"* a **todos** los usuarios de Sistemas, incluso a quienes tienen permisos NTFS de **Modificar**.

<details>
<summary>🔧 Cómo provocarla</summary>

En el **recurso compartido** (no en NTFS), deja el permiso como **solo lectura** o quita a `Authenticated Users`:
`Revoke-SmbShareAccess -Name Sistemas -AccountName "Todos" -Force` (o equivalente).
</details>

---

### Avería 5 · "Dice que la contraseña es correcta, pero no entra" (y no es la contraseña)

**Síntoma:** en el cliente, el usuario correcto y su contraseña correcta dan error de autenticación, o el acceso a recursos devuelve *"Acceso denegado"* de forma extraña. Los **relojes** no parecen sospechosos... ¿lo son?

<details>
<summary>🔧 Cómo provocarla</summary>

Detén la sincronización y **desajusta la hora del cliente unos 10 minutos**:
`Stop-Service w32time` y `Set-Date (Get-Date).AddMinutes(10)`.
</details>

---

### Avería 6 · "La intranet no se resuelve por nombre"

**Síntoma:** `http://intranet.empresa.local` no carga en el cliente, pero con la **IP** sí funciona. El resto de nombres (por ejemplo `dc01.empresa.local`) resuelven bien.

<details>
<summary>🔧 Cómo provocarla</summary>

En `dc01`, borra el registro A:
`Remove-DnsServerResourceRecord -ZoneName empresa.local -RRType A -Name intranet -Force`
</details>

---

### Avería 7 · (red) "`cliente-b` ya no sale a internet ni ve a `cliente-a`"

**Síntoma:** tras un reinicio del router, `cliente-a` llega a todo, pero **`cliente-b`** (LAN B) no llega ni a internet ni a la otra red. Funcionaba el día anterior.

<details>
<summary>🔧 Cómo provocarla</summary>

En `router-lab`, desactiva el reenvío **solo en caliente** (`sudo sysctl -w net.ipv4.ip_forward=0`) y/o **borra la regla de NAT** (`sudo nft delete rule ...` o `sudo nft flush table ip nat`). Variante: pon una **puerta de enlace equivocada** en `cliente-b`.
</details>

---

### Avería 8 · (red) "En la VLAN 20 no hay manera"

**Síntoma:** en la práctica de VLAN, `cliente-b` (VLAN 20) **no puede ni hacer ping a su propia puerta de enlace** `10.10.20.1`. `cliente-a` (VLAN 10) va perfecto.

<details>
<summary>🔧 Cómo provocarla</summary>

En `cliente-b`, **crea la subinterfaz con otro número de VLAN** (`ip link add link enp0s8 name enp0s8.20 type vlan id 21`) o haz lo mismo en el router. Comprueba qué ve `tcpdump -e vlan`.
</details>

---

## ✅ Comprobación

Para **cada avería**, tu runbook debe incluir:

| Campo | Contenido |
| ----- | --------- |
| **Síntoma** | Lo que se veía |
| **Ámbito** | ¿A un usuario, a un equipo, a todos? |
| **Hipótesis descartadas** | Qué pensaste y **cómo lo descartaste** |
| **Diagnóstico** | Comandos y salidas **que señalan la causa** |
| **Causa raíz** | El **porqué real** |
| **Solución** | El cambio exacto |
| **Verificación** | Cómo confirmaste que está arreglado |
| **Prevención** | Qué lo evitaría (monitorización, alerta, proceso, documentación) |
| **Tiempo** | Cuánto tardaste |

---

## 🧩 Pistas

- **"El problema empieza en DNS"** es cierto con tanta frecuencia que debería ser tu **primera comprobación** en cualquier fallo de dominio.
- Si un login falla, mira **ambos lados**: el **cliente** (`Eventos`, `nltest`, `klist`) y el **DC** (`Eventos de seguridad`, estado de la cuenta).
- Una **diferencia de hora de más de 5 minutos** produce errores de Kerberos que **no mencionan la hora**. Es uno de los fallos más desconcertantes.
- Un permiso efectivo es la **intersección** de recurso compartido y NTFS. **Revisa las dos capas.**
- Una GPO que "no se aplica": revisa **vínculo, filtrado, delegación, OU del objeto, tipo de configuración (equipo/usuario) y `gpupdate`**.
- En red, **acota por capas** (interfaz → IP → puerta de enlace → IP externa → DNS) como en el laboratorio 07 del nivel 1.
- `tcpdump` (Linux) te dice **si el paquete llega y por dónde se pierde**.

---

## 🚀 Retos extra

1. **Averías compuestas:** combina dos (por ejemplo, la 1 y la 5) y comprueba si sabes **separarlas**.
2. **Pásalas a un compañero** (o a una versión futura de ti mismo, pasadas dos semanas).
3. Crea **dos averías nuevas** de tus laboratorios (GPO avanzadas, PSO, `dsa.msc`, backups de AD...) con su "Cómo provocarla".
4. Escribe un script de **comprobación rápida de salud** del cliente (`Salud-Cliente.ps1`) que ejecute en segundos: DNS, DC localizable, hora, canal seguro, GPO y red.
5. Cruza con la **monitorización**: ¿qué **alerta** (Prometheus/Grafana, eventos de Windows o logs centralizados) te habría avisado de cada avería **antes** que el usuario?

---

## 📝 Qué documentar en tu `Solucion.md`

- **Ocho runbooks** con la tabla de arriba.
- Una **checklist de diagnóstico de dominio** de diez puntos, en el **orden** que seguirías.
- Los **cinco comandos de Windows** que más te han servido.
- Una **autoevaluación**: qué averías te costaron más y qué **suposiciones erróneas** hiciste.

---

## 🧠 Lo que has practicado

`diagnóstico de dominio` · `DNS, hora y Kerberos` · `cuentas` · `permisos en dos capas` · `GPO` · `DHCP` · `diagnóstico de red por capas` · `runbooks` · `prevención`
