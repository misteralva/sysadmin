# 🏁 Proyecto final · La infraestructura completa de NovaTech Sistemas

**Nivel:** 🏁 Proyecto final · **Sistemas:** 🐧 Linux + 🪟 Windows Server · **Tiempo estimado:** 80-100 horas (6-8 semanas)

---

## 🎯 Escenario

**NovaTech Sistemas S.L.** es una pequeña consultora ficticia con **25 empleados** en una sola sede, repartidos en tres departamentos (**Sistemas**, **Redes** y **Administración**) más una zona de **Invitados** (clientes y visitas). Tres personas trabajan **a distancia** parte de la semana. La empresa maneja datos de clientes y en **seis semanas** tiene una **auditoría externa de seguridad**.

Hoy la "infraestructura" es un desastre típico de pyme: un servidor para todo, contraseñas compartidas, sin copias de seguridad probadas y sin forma de saber si algo falla hasta que alguien se queja. **El presupuesto es cero**: solo software libre y las versiones de evaluación de Microsoft.

**Tú eres el consultor de sistemas.** Tu misión es **diseñar, construir, proteger, documentar y demostrar** la infraestructura completa de la empresa, usando **todo** lo que has practicado en los cuatro niveles, y entregarla como lo haría un profesional.

> 🧠 Este proyecto **no tiene pasos guiados**. Se te da un escenario, unos **requisitos con criterios de aceptación**, una arquitectura de referencia y unos entregables. **Cómo llegues** es decisión tuya, y **justificar esas decisiones** forma parte de la nota.

---

## 🧭 Cómo trabajar: tres niveles de ambición

Cada requisito está marcado con un nivel. Elige **hasta dónde llegar** según tu tiempo y tu equipo:

| Nivel | Qué incluye | Para quién |
| ----- | ----------- | ---------- |
| 🥉 **Mínimo (M)** | Solo los requisitos **M**. Infraestructura funcional, segura y documentada | Un proyecto completo y presentable |
| 🥈 **Estándar (E)** | **M + E**. Añade automatización, más controles de seguridad y más operación | El objetivo recomendado |
| 🥇 **Avanzado (A)** | **M + E + A**. Alta disponibilidad, IDS, VLAN, auditoría y simulacros | Para sacarle todo el partido |

> 💡 Un **Mínimo muy bien hecho y documentado** vale más que un Avanzado a medias.

### Ajuste a tu equipo

La arquitectura pide varias VMs. Elige la variante que tu ordenador aguante (usa **clones vinculados** y apaga lo que no uses):

| Tu RAM | Variante | VMs |
| ------ | -------- | --- |
| **16 GB** | **Compacta** | `fw-router`, `dc01`, `srv-linux` (web **y** servicios de infraestructura juntos), `pc-cliente01` (o un Windows Server como "servidor miembro") |
| **32 GB o más** | **Completa** | `fw-router`, `dc01`, `srv-web`, `srv-infra`, `pc-cliente01`, `pc-remoto` |

En la variante compacta, las funciones de `srv-web` y `srv-infra` conviven en `srv-linux`, y el cliente remoto puede ser **tu propio ordenador**.

---

## 🏗️ Arquitectura de referencia

```
                   INTERNET simulada ("WAN": red solo-anfitrión 192.168.56.0/24)
                                   │
                        ┌──────────┴──────────┐
 pc-remoto ──(VPN)────► │     fw-router       │  ← firewall, router, NAT, VPN, (IDS)
 (empleado a distancia) │  WireGuard 51820    │
                        └───┬─────────────┬───┘
                            │             │
              LAN 192.168.10.0/24     DMZ 192.168.20.0/24
          ┌────────┬─────────┐            │
        dc01    srv-infra  pc-cliente01   srv-web
   (AD, DNS,   (logs,                    (intranet
 DHCP, GPO,   monitorización,             HTTPS)
  ficheros)    backups)
```

### Plan de direccionamiento sugerido

| Zona | Red | Equipos |
| ---- | --- | ------- |
| **WAN simulada** | `192.168.56.0/24` | `fw-router` `.10`, `pc-remoto` `.11` |
| **LAN** | `192.168.10.0/24` | `fw-router` `.1`, `dc01` `.10`, `srv-infra` `.11`, clientes por DHCP `.100-.200` |
| **DMZ** | `192.168.20.0/24` | `fw-router` `.1`, `srv-web` `.10` |
| **VPN** | `10.8.0.0/24` | `fw-router` `.1`, clientes remotos `.2` en adelante |

Dominio de Active Directory: **`novatech.lab`** (NetBIOS `NOVATECH`). Intranet: **`intranet.novatech.lab`**.

### Matriz de flujos (la política que debe cumplir el firewall)

| Origen → Destino | Permitido | Notas |
| ---------------- | --------- | ----- |
| **LAN → Internet** | ✅ HTTP, HTTPS, DNS, NTP | Con NAT |
| **LAN → DMZ** | ✅ **solo 443** (intranet) | Nada más |
| **DMZ → LAN** | ❌ salvo `srv-web` → `srv-infra` (logs `514/tcp`) | La DMZ **no inicia** conexiones hacia la LAN |
| **DMZ → Internet** | ✅ solo actualizaciones (80/443) | |
| **WAN → `fw-router`** | ✅ solo **UDP 51820** (VPN) | Todo lo demás, descartado |
| **VPN → LAN** | ✅ ficheros (445), DNS (53), intranet (443) | **Mínimo necesario** (`AllowedIPs`) |
| **VPN → DMZ** | ✅ 443 | |
| **Administración** (SSH/RDP) | ✅ solo desde una **IP o equipo de administración** | Nadie más |
| **srv-infra → todos** | ✅ lo necesario para monitorizar (`9100/tcp`) | |
| **Todo lo demás** | ❌ **descartado y registrado** | Política `deny` por defecto |

---

## 📋 Requisitos

Cada requisito tiene un **criterio de aceptación**: lo que debe poder **demostrarse** para darlo por cumplido.

### 🌐 Red y perímetro

| ID | Requisito | Criterio de aceptación | Nivel |
| -- | --------- | ---------------------- | ----- |
| **R1** | `fw-router` reenvía tráfico y hace NAT | Un cliente de la LAN navega a internet | **M** |
| **R2** | Zonas LAN, DMZ y VPN **segmentadas** según la matriz | **Cada fila** de la matriz de flujos tiene una prueba que la confirma (permitido **y** bloqueado) | **M** |
| **R3** | Política `deny` por defecto con **registro** de bloqueos | Un paquete bloqueado aparece en el log con origen, destino y puerto | **M** |
| **R4** | **VPN WireGuard** para acceso remoto, con `AllowedIPs` mínimos | El cliente remoto llega al servidor de ficheros e intranet, **y no** a lo demás | **M** |
| **R5** | Segmentación con **VLAN** (802.1Q) en la LAN | Dos VLAN distintas con enrutado y filtrado entre ellas | **A** |
| **R6** | **IDS** (Suricata) con al menos 3 reglas propias | Una alerta real generada y analizada en una ficha | **A** |

### 🪪 Identidad y escritorio (Windows)

| ID | Requisito | Criterio de aceptación | Nivel |
| -- | --------- | ---------------------- | ----- |
| **I1** | Dominio `novatech.lab` con **OU** por departamento | Estructura visible en `dsa.msc` y documentada | **M** |
| **I2** | Al menos 12 usuarios y los grupos de seguridad por departamento | Usuarios **creados por script desde un CSV**; en el nivel mínimo, **al menos 6 a mano** | **M / E** |
| **I3** | Clientes **unidos al dominio** | Un usuario de dominio inicia sesión en `pc-cliente01` | **M** |
| **I4** | **Al menos 3 GPO** útiles (aviso legal, unidad de red por departamento, restricción para algún grupo) | Cada GPO demostrada con `gpresult` | **M** |
| **I5** | **Contraseñas más estrictas para administradores** (PSO) | `Get-ADUserResultantPasswordPolicy` muestra la PSO | **E** |
| **I6** | **Auditoría de AD** con al menos 5 hallazgos **corregidos** | Informe antes/después | **A** |

### 🗂️ Servicios

| ID | Requisito | Criterio de aceptación | Nivel |
| -- | --------- | ---------------------- | ----- |
| **S1** | **DNS y DHCP** en `dc01`, con al menos 1 reserva | Un cliente recibe IP y resuelve `intranet.novatech.lab` | **M** |
| **S2** | **Servidor de ficheros** con carpetas por departamento y permisos **por grupos de dominio** | Un usuario accede a **su** carpeta y **no** a la de otro departamento | **M** |
| **S3** | **Intranet** con Nginx en la **DMZ**, por **HTTPS** con certificado de **tu CA** | El cliente abre `https://intranet.novatech.lab` **sin avisos** | **M** |
| **S4** | La CA se **distribuye por GPO** a los equipos del dominio | Un equipo nuevo del dominio confía en la CA sin tocar nada a mano | **E** |
| **S5** | **Alta disponibilidad** de la intranet (2 nodos + IP virtual) | Se apaga un nodo y el servicio sigue; medido | **A** |

### 🛡️ Seguridad

| ID | Requisito | Criterio de aceptación | Nivel |
| -- | --------- | ---------------------- | ----- |
| **H1** | **Hardening de los Linux** con auditoría de Lynis | El índice mejora **al menos 15 puntos** y las mejoras están justificadas | **M** |
| **H2** | **SSH solo con claves**, sin `root`, con `fail2ban` | Un intento con contraseña es rechazado; un bloqueo de `fail2ban` demostrado | **M** |
| **H3** | **Línea base de seguridad en Windows** | SMBv1 desactivado, cortafuegos activo, cuentas por defecto controladas | **M** |
| **H4** | **AppArmor** y `auditd` en el servidor web | Un perfil en `enforce` y una regla de `auditd` demostrada | **E** |

### 📈 Operación y resiliencia

| ID | Requisito | Criterio de aceptación | Nivel |
| -- | --------- | ---------------------- | ----- |
| **O1** | **Hora sincronizada** en todo el entorno | `chronyc tracking` y `w32tm /query /status` coherentes | **M** |
| **O2** | **Monitorización** con Prometheus y Grafana | Dashboard con todas las máquinas y **al menos 1 alerta** probada (**3** en el nivel estándar) | **M / E** |
| **O3** | **Logs centralizados** de los Linux | Los logs de `srv-web` y `fw-router` llegan a `srv-infra` | **M** |
| **O4** | **Backups con RPO y RTO definidos**, verificados con hash y **restauración probada** | Una restauración **real** demostrada con tiempo medido | **M** |
| **O5** | **Automatización** de la configuración Linux | Scripts documentados (**M**) / **playbook de Ansible** que reconstruye `srv-web` (**E**) / roles + Vault (**A**) | **M / E / A** |
| **O6** | **Simulacro de desastre** con medición | RTO y RPO **reales**, comparados con los objetivos | **E** |

### 📝 Documentación

| ID | Requisito | Criterio de aceptación | Nivel |
| -- | --------- | ---------------------- | ----- |
| **D1** | **Documento de arquitectura** (plantilla) | Completo, con diagrama, plan de IP y matriz de flujos | **M** |
| **D2** | **Inventario** de sistemas, cuentas de servicio y puertos | Una tabla **actualizada** | **M** |
| **D3** | **Al menos 6 runbooks** | Siguen la plantilla y **se han probado** | **M** |
| **D4** | **Plan de pruebas** con evidencias | **Todos** los criterios de aceptación probados con salida o captura | **M** |
| **D5** | **Informe de seguridad y riesgos** | Hallazgos, riesgos residuales y plan de mejora | **E** |
| **D6** | **Lecciones aprendidas** | Qué cambiarías si empezaras de nuevo | **M** |

---

## 🗓️ Fases e hitos

Los tiempos son **orientativos**. Cada fase termina con un **hito comprobable**.

| Fase | Contenido | Horas | Hito (qué debe funcionar al terminar) |
| ---- | --------- | ----- | ------------------------------------- |
| **0 · Diseño** | Arquitectura, plan de IP, matriz de flujos, plan de capacidad, backlog de tareas | 6 | Documento de arquitectura **v1** y plan de VMs |
| **1 · Red y perímetro** | `fw-router`, zonas, NAT, política y logs | 12 | Se cumple **cada fila** de la matriz de flujos |
| **2 · Identidad y servicios Windows** | AD, DNS, DHCP, GPO, ficheros, clientes | 14 | Un usuario inicia sesión, recibe GPO y accede a su carpeta |
| **3 · Servicios Linux** | Intranet en DMZ con HTTPS (CA propia), base de `srv-infra` | 10 | `https://intranet.novatech.lab` sin avisos |
| **4 · VPN y acceso remoto** | WireGuard, `AllowedIPs` mínimos | 6 | Acceso remoto limitado a lo permitido |
| **5 · Seguridad** | Hardening, `fail2ban`, línea base de Windows, (auditoría de AD, IDS, AppArmor) | 12 | Lynis y auditoría **antes/después** |
| **6 · Operación** | Hora, monitorización, logs, backups, automatización | 14 | Alerta disparada y **restauración probada** |
| **7 · Pruebas y simulacros** | Plan de pruebas, simulacro de desastre, averías a ciegas, incidente simulado | 10 | Todas las pruebas con evidencia |
| **8 · Documentación y presentación** | Documentación final, informe, demo | 8 | Repositorio y demo listos |

> 📅 **Consejo de calendario:** dedica la **mitad del tiempo** a las fases 1-4 y **la otra mitad** a 5-8. Lo que se deja para el final (seguridad, operación, documentación) es lo que más se echa de menos.

---

## 🧪 Pruebas y simulacros obligatorios

Además de probar cada requisito, deberás hacer estos **ejercicios de resistencia** y **documentar** cada resultado:

| Ejercicio | Qué se hace | Qué se mide |
| --------- | ----------- | ----------- |
| **Pruebas de aceptación** | Recorres el plan de pruebas completo | OK / KO por requisito |
| **Prueba de restauración** | Borras datos de verdad y los restauras | Tiempo y datos perdidos |
| **Averías a ciegas** | **Otra persona** (o tú, **sin mirar**) provoca **3 averías** (como en los laboratorios 10 y 11 del nivel experto) | Tiempo de diagnóstico y causa raíz |
| **Incidente simulado** | Plantas indicadores en un servidor y lo investigas (como en el laboratorio 12 del experto) | Informe de incidente |
| **Simulacro de desastre (E/A)** | Destruyes una VM y la reconstruyes | **RTO y RPO reales** |

---

## 📦 Entregables

Todo en un directorio `solucion/` dentro de esta carpeta:

```
05-proyecto-final/solucion/
├── README.md                  ← resumen, cómo está montado, cómo probarlo
├── 01-diseno/                 ← arquitectura, diagramas, plan de IP, matriz de flujos
├── 02-implementacion/         ← configuraciones SIN secretos, scripts, playbooks
├── 03-operacion/              ← runbooks, monitorización, backups, plan de recuperación
├── 04-seguridad/              ← Lynis antes/después, auditoría de AD, informe de riesgos
├── 05-pruebas/                ← plan de pruebas y evidencias (capturas, salidas)
└── 06-cierre/                 ← lecciones aprendidas, autoevaluación con la rúbrica, presentación
```

### 🚫 Qué NO subir nunca a GitHub

| Elemento | Por qué |
| -------- | ------- |
| **Claves privadas** (SSH, WireGuard, de la CA, certificados `.key`) | Cualquiera podría **hacerse pasar por tus servidores** |
| **Contraseñas**, incluidas las de DSRM y de servicios | Obvio, pero pasa constantemente |
| Archivos `.env`, `vault` sin cifrar, cualquier fichero con **tokens** | Idem |
| Imágenes de VM (`.vdi`, `.ova`) e ISO | Enormes y sin utilidad en el repo |
| **IP públicas reales**, nombres de empresas reales | Privacidad |
| Volcados completos de logs o informes de Lynis | Pueden contener **datos sensibles** de tu sistema |

En `Plantillas/` hay un **`.gitignore` sugerido**. **Revisa tu repositorio con `git status` y `git diff` antes de cada `git push`.**

---

## 🎤 Presentación (demo de 15-20 minutos)

Prepara una demo **en directo**, como si se la presentaras a la dirección de NovaTech:

| Minutos | Contenido |
| ------- | --------- |
| **3** | La arquitectura: diagrama, decisiones clave y por qué |
| **10** | **Demos cortas** (elige 5-6): inicio de sesión de dominio + GPO + carpeta por departamento · intranet por HTTPS · acceso por VPN desde fuera · el firewall **bloqueando** algo y el log correspondiente · una **alerta** de monitorización · una **restauración** de backup (o el simulacro de desastre) |
| **3** | **Seguridad**: qué mejoraste, qué riesgos **residuales** quedan |
| **2** | **Lecciones aprendidas** y qué mejorarías |

> 🎬 **Graba la demo** o ensáyala: una demo en directo puede fallar, y tener un vídeo de respaldo es profesional.

---

## 📊 Cómo se evalúa

Hay una **rúbrica de autoevaluación** (`Rubrica.md`) con seis criterios: diseño, implementación, seguridad, operación, automatización y documentación. Evalúate **con evidencias**, no con sensaciones.

---

## 🧩 Consejos

- **Empieza por el diseño, no por las máquinas.** Una hora de papel evita diez de rehacer.
- **Haz hitos pequeños y comprobables.** No avances de fase si el hito anterior no se cumple.
- **Documenta mientras construyes**, no al final. Lo que no escribes el día que lo haces, se olvida.
- **Snapshots por hito**, con nombre claro (`fase2-ok`), y una **copia de seguridad de las VMs** en otro disco.
- **Automatiza lo que repitas** más de dos veces.
- **No cambies la arquitectura a mitad** sin anotar el motivo (una "decisión de diseño" documentada).
- **Pide a alguien que lo rompa:** el mejor revisor es quien no sabe cómo lo hiciste.
- Cuando algo falle, **aplica el método de diagnóstico** del nivel experto: es parte de lo que se evalúa.

---

## 🧠 Lo que vas a demostrar

`diseño de infraestructuras` · `redes y segmentación` · `Active Directory` · `servicios Linux y Windows` · `PKI` · `VPN` · `hardening y auditoría` · `monitorización y logs` · `backups y recuperación` · `automatización` · `documentación profesional`
