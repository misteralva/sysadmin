# 🖥️ Sysadmin: Laboratorios prácticos de cero a experto

🌐 [English](README.md) | **Español**

Un repositorio de **laboratorios prácticos para aprender administración de sistemas** (Linux y Windows Server), con un objetivo claro: construir la base necesaria para trabajar en **redes y ciberseguridad**.

Cada laboratorio incluye su propio enunciado educativo (que explica los conceptos nuevos) y mi solución documentada. La dificultad aumenta poco a poco: **principiante → intermedio → avanzado → experto**, y termina con un **proyecto final**.

> 📚 Es el tercer repositorio de la serie, después de [Bash](https://github.com/misteralva/Bash) y [Python](https://github.com/misteralva/python). Mismo formato, distinto objetivo: allí escribo **scripts**, aquí **configuro, rompo y reparo sistemas reales**.

---

## 🎯 ¿Para quién es este repositorio?

- Para mí, como diario de aprendizaje.
- Como portfolio de trabajo real y documentado.
- Para cualquiera que quiera aprender sysadmin **desde cero**, practicando en sus propias máquinas virtuales.

---

## 📁 Estructura

```
sysadmin/
├── README.md
├── README.es.md
├── 01-principiante/
│   ├── 01-instalar-vm-ubuntu/
│   │   ├── Enunciado.md
│   │   └── Solucion.md
│   └── 02-.../
├── 02-intermedio/
├── 03-avanzado/
├── 04-experto/
└── 05-proyecto-final/
```

Cada carpeta de laboratorio contiene:

| Archivo        | Contenido                                                                                              |
| -------------- | ------------------------------------------------------------------------------------------------------ |
| `Enunciado.md` | Escenario, conceptos nuevos explicados, qué hacer, cómo comprobarlo, pistas y retos extra              |
| `Solucion.md`  | Mi solución: comandos con la explicación de cada uno y un apartado de "Qué salió mal"                  |

---

## 🧪 Entorno de laboratorio

Todos los laboratorios se hacen en **máquinas virtuales** propias, nunca en un sistema real o compartido.

| Herramienta | Uso |
| ----------- | --- |
| **VirtualBox** | Hipervisor para ejecutar las máquinas virtuales |
| **Ubuntu Server (LTS)** | Sistema Linux principal para los laboratorios 🐧 |
| **Windows Server (versión de evaluación)** | Sistema para los laboratorios 🪟 |

**Regla de oro:** haz un **snapshot** antes de cualquier laboratorio arriesgado (discos, red, firewall). Si algo se rompe, vuelves atrás en segundos.

---

## 🗺️ Ruta de aprendizaje

**Leyenda:** 🐧 Linux · 🪟 Windows Server · ✅ hecho · ⏳ pendiente · 🔜 por detallar

### 🟢 Nivel 1 · Principiante

| #  | Laboratorio                                                                                              | Conceptos                                               | SO | Estado |
| -- | -------------------------------------------------------------------------------------------------------- | ------------------------------------------------------- | -- | ------ |
| 01 | [Instalar una VM con Ubuntu Server y usar snapshots](01-principiante/01-instalar-vm-ubuntu/Enunciado.md) | virtualización, ISO, recursos, snapshots                | 🐧 | ⏳     |
| 02 | [Moverse por el sistema de ficheros](01-principiante/02-moverse-por-el-sistema-de-ficheros/Enunciado.md)                                                                       | `pwd`, `cd`, `ls`, `mkdir`, jerarquía de directorios    | 🐧 | ⏳     |
| 03 | [Usuarios y grupos](01-principiante/03-usuarios-y-grupos/Enunciado.md)                                                                                        | `useradd`, `usermod`, `groups`, `/etc/passwd`           | 🐧 | ⏳     |
| 04 | [Permisos de ficheros](01-principiante/04-permisos-de-ficheros/Enunciado.md)                                                                                     | `chmod`, `chown`, `rwx`, `umask`                        | 🐧 | ⏳     |
| 05 | [Instalar y gestionar paquetes](01-principiante/05-paquetes-apt/Enunciado.md)                                                                            | `apt`, repositorios, actualizaciones                    | 🐧 | ⏳     |
| 06 | [Procesos](01-principiante/06-procesos/Enunciado.md)                                                                                                 | `ps`, `top`, `kill`, señales                            | 🐧 | ⏳     |
| 07 | [Comandos de red básicos](01-principiante/07-comandos-de-red-basicos/Enunciado.md)                                                                                  | `ip`, `ping`, `ss`, resolución DNS                      | 🐧 | ⏳     |
| 08 | [Acceso remoto con SSH](01-principiante/08-acceso-remoto-ssh/Enunciado.md)                                                                                    | `ssh`, `scp`, claves de host                            | 🐧 | ⏳     |
| 09 | [Instalar Windows Server en una VM](01-principiante/09-instalar-windows-server/Enunciado.md)                                                                        | instalación, edición de evaluación, snapshots           | 🪟 | ⏳     |
| 10 | [Usuarios locales y carpetas compartidas](01-principiante/10-usuarios-locales-y-carpetas-compartidas/Enunciado.md)                                                                  | usuarios locales, recursos SMB, permisos NTFS           | 🪟 | ⏳     |
| 11 | [Primeros pasos con PowerShell](01-principiante/11-primeros-pasos-powershell/Enunciado.md)                                                                            | cmdlets, pipeline, `Get-Help`                           | 🪟 | ⏳     |

### 🟡 Nivel 2 · Intermedio

> 🧪 Antes de empezar, lee la [guía del entorno de laboratorio](02-intermedio/README.md): explica qué máquinas virtuales y redes necesitas.

| #  | Laboratorio                                        | Conceptos                                          | SO | Estado |
| -- | -------------------------------------------------- | -------------------------------------------------- | -- | ------ |
| 01 | [Servicios con `systemd`](02-intermedio/01-servicios-systemd/Enunciado.md)                            | units, `systemctl`, enable/disable                 | 🐧 | ⏳     |
| 02 | [Tareas programadas: `cron` y timers](02-intermedio/02-cron-y-timers/Enunciado.md)                | sintaxis de crontab, timers de `systemd`           | 🐧 | ⏳     |
| 03 | [Dirección IP estática](02-intermedio/03-ip-estatica-netplan/Enunciado.md)                              | `netplan`, puerta de enlace, DNS                   | 🐧 | ⏳     |
| 04 | [Discos, particiones y `fstab`](02-intermedio/04-discos-particiones-fstab/Enunciado.md)                      | `lsblk`, `fdisk`, sistemas de ficheros, montaje    | 🐧 | ⏳     |
| 05 | [LVM](02-intermedio/05-lvm/Enunciado.md)                                                | PV, VG, LV, ampliar volúmenes                      | 🐧 | ⏳     |
| 06 | [Logs con `journalctl`](02-intermedio/06-logs-journalctl/Enunciado.md)                              | `/var/log`, filtrar por servicio y fecha           | 🐧 | ⏳     |
| 07 | [Firewall con `ufw`](02-intermedio/07-firewall-ufw/Enunciado.md)                                 | reglas, políticas por defecto, puertos             | 🐧 | ⏳     |
| 08 | [Autenticación SSH con claves](02-intermedio/08-ssh-con-claves/Enunciado.md)                       | claves pública/privada, `authorized_keys`          | 🐧 | ⏳     |
| 09 | [Backups con `tar` y `rsync`](02-intermedio/09-backups-tar-rsync/Enunciado.md)                        | copias completas e incrementales, restaurar        | 🐧 | ⏳     |
| 10 | [Servidor web con Nginx](02-intermedio/10-servidor-web-nginx/Enunciado.md)                             | instalación, sitios, configuración básica          | 🐧 | ⏳     |
| 11 | [Active Directory: crear el dominio](02-intermedio/11-active-directory-crear-dominio/Enunciado.md)                 | AD DS, controlador de dominio, DNS                 | 🪟 | ⏳     |
| 12 | [Unir equipos al dominio, OUs y usuarios](02-intermedio/12-unir-equipos-dominio-ous-usuarios/Enunciado.md)            | unión al dominio, unidades organizativas           | 🪟 | ⏳     |
| 13 | [DNS y DHCP en Windows Server](02-intermedio/13-dns-y-dhcp-windows-server/Enunciado.md)                       | ámbitos, concesiones, registros DNS                | 🪟 | ⏳     |
| 14 | [GPO básicas](02-intermedio/14-gpo-basicas/Enunciado.md)                                        | directivas de grupo, vinculación, herencia         | 🪟 | ⏳     |

### 🟠 Nivel 3 · Avanzado

> 🧪 Antes de empezar, lee la [guía del entorno de laboratorio](03-avanzado/README.md): explica las máquinas virtuales y redes adicionales que necesitas. Los laboratorios de seguridad de este nivel son **defensivos** (configurar, endurecer, vigilar); la práctica ofensiva corresponde a un repositorio de ciberseguridad aparte.

| #  | Laboratorio | Conceptos | SO | Estado |
| -- | ----------- | --------- | -- | ------ |
| 01 | [RAID con `mdadm`](03-avanzado/01-raid-mdadm/Enunciado.md) | niveles RAID, array degradado, reconstrucción | 🐧 | ⏳ |
| 02 | [Compartir ficheros con NFS y Samba](03-avanzado/02-nfs-y-samba/Enunciado.md) | `/etc/exports`, `root_squash`, `smb.conf` | 🐧 | ⏳ |
| 03 | [Firewall avanzado con `nftables`](03-avanzado/03-firewall-nftables/Enunciado.md) | tablas, cadenas, reglas, filtrado con estado | 🐧 | ⏳ |
| 04 | [Protección automática con `fail2ban`](03-avanzado/04-fail2ban/Enunciado.md) | jails, filtros, bloqueo dinámico | 🐧 | ⏳ |
| 05 | [Hardening y auditoría con Lynis](03-avanzado/05-hardening-y-auditoria-lynis/Enunciado.md) | superficie de ataque, `sysctl`, `auditd`, SSH | 🐧 | ⏳ |
| 06 | [Control de acceso obligatorio con AppArmor](03-avanzado/06-apparmor/Enunciado.md) | DAC y MAC, perfiles, enforce/complain | 🐧 | ⏳ |
| 07 | [OpenLDAP: autenticación centralizada](03-avanzado/07-openldap-autenticacion-centralizada/Enunciado.md) | LDAP, LDIF, NSS y PAM | 🐧 | ⏳ |
| 08 | [Monitorización con Prometheus y Grafana](03-avanzado/08-monitorizacion-prometheus-grafana/Enunciado.md) | métricas, exporters, PromQL, alertas | 🐧 | ⏳ |
| 09 | [Introducción a Ansible](03-avanzado/09-ansible-introduccion/Enunciado.md) | inventario, playbooks, idempotencia | 🐧 | ⏳ |
| 10 | [Linux como router y NAT](03-avanzado/10-linux-router-y-nat/Enunciado.md) | `ip_forward`, masquerade, política de reenvío | 🐧 | ⏳ |
| 11 | [VLANs en Linux](03-avanzado/11-vlans-en-linux/Enunciado.md) | 802.1Q, trunk/access, router-on-a-stick | 🐧 | ⏳ |
| 12 | [VPN con WireGuard](03-avanzado/12-vpn-wireguard/Enunciado.md) | claves, peers, `AllowedIPs`, túnel cifrado | 🐧 | ⏳ |
| 13 | [GPO avanzadas y contraseñas de grano fino](03-avanzado/13-gpo-avanzadas/Enunciado.md) | filtrado de seguridad, WMI, loopback, PSO | 🪟 | ⏳ |
| 14 | [PowerShell: Active Directory y remoting](03-avanzado/14-powershell-active-directory-y-remoting/Enunciado.md) | scripts, importar CSV, WinRM, informes | 🪟 | ⏳ |
| 15 | [Backup y restauración de Active Directory](03-avanzado/15-backup-y-restauracion-de-ad/Enunciado.md) | papelera de AD, estado del sistema, DSRM | 🪟 | ⏳ |

### 🔴 Nivel 4 · Experto

> 🧪 Lee antes la [guía del nivel experto](04-experto/README.md): aquí recibes un **escenario y un objetivo**, con menos pistas, y cada laboratorio pide un **entregable profesional** (runbook, informe, plan). Los laboratorios de seguridad siguen siendo **defensivos**.

| #  | Laboratorio | Conceptos | SO | Estado |
| -- | ----------- | --------- | -- | ------ |
| 01 | [Recuperar un sistema que no arranca](04-experto/01-recuperar-sistema-que-no-arranca/Enunciado.md) | proceso de arranque, GRUB, modo rescate, `chroot` | 🐧 | ⏳ |
| 02 | [Rendimiento y cuellos de botella](04-experto/02-rendimiento-y-cuellos-de-botella/Enunciado.md) | método USE, `vmstat`, `iostat`, cgroups, OOM killer | 🐧 | ⏳ |
| 03 | [Contenedores con Docker](04-experto/03-contenedores-docker/Enunciado.md) | imágenes, volúmenes, Compose, seguridad de contenedores | 🐧 | ⏳ |
| 04 | [Logs centralizados con `rsyslog`](04-experto/04-logs-centralizados-rsyslog/Enunciado.md) | logs remotos, plantillas, hora sincronizada, integridad | 🐧 | ⏳ |
| 05 | [PKI y certificados](04-experto/05-pki-y-certificados/Enunciado.md) | CA propia, CSR, SAN, HTTPS, cadena de confianza | 🐧 | ⏳ |
| 06 | [IDS con Suricata](04-experto/06-ids-suricata/Enunciado.md) | reglas, firmas, alertas, `eve.json` | 🐧 | ⏳ |
| 07 | [Alta disponibilidad: HAProxy y keepalived](04-experto/07-alta-disponibilidad-haproxy-keepalived/Enunciado.md) | balanceo, VRRP, IP virtual, failover | 🐧 | ⏳ |
| 08 | [Simulacro de desastre](04-experto/08-simulacro-de-desastre/Enunciado.md) | RPO/RTO, runbooks, reconstrucción desde cero | 🐧 | ⏳ |
| 09 | [Auditoría de seguridad de Active Directory](04-experto/09-auditoria-de-seguridad-de-active-directory/Enunciado.md) | cuentas privilegiadas, `krbtgt`, delegación, remediación | 🪟 | ⏳ |
| 10 | [Break/fix en Linux](04-experto/10-break-fix-linux/Enunciado.md) | método de diagnóstico, causa raíz, runbooks | 🐧 | ⏳ |
| 11 | [Break/fix en Windows y red](04-experto/11-break-fix-windows-y-red/Enunciado.md) | DNS, hora/Kerberos, GPO, DHCP, VLAN | 🪟 | ⏳ |
| 12 | [Respuesta a incidentes](04-experto/12-respuesta-a-incidentes/Enunciado.md) | evidencias, persistencia, contención, informe | 🐧 | ⏳ |

### 🏁 Nivel 5 · Proyecto final

El broche final: diseñar, construir, proteger, documentar y **demostrar** la infraestructura completa de una empresa ficticia, **NovaTech Sistemas S.L.**, usando todo lo de los cuatro niveles. No hay pasos guiados: recibes un escenario, requisitos con criterios de aceptación, una arquitectura de referencia y unos entregables. Tres niveles de ambición (mínimo, estándar y avanzado) permiten ajustarlo a tu tiempo y a tu equipo.

| Documento | Qué es |
| --------- | ------ |
| [Enunciado del proyecto](05-proyecto-final/Enunciado.md) | Escenario, arquitectura, requisitos, fases, pruebas y entregables |
| [Rúbrica de autoevaluación](05-proyecto-final/Rubrica.md) | Seis criterios, 100 puntos, basada en evidencias |
| [Plantillas](05-proyecto-final/Plantillas/) | Documento de arquitectura, runbook, plan de pruebas, informe de seguridad y un `.gitignore` sugerido |

---

## 🧭 Cómo usar este repositorio para aprender

1. Lee el `Enunciado.md` del laboratorio.
2. Intenta resolverlo **por tu cuenta**, usando los conceptos y las pistas.
3. Compruébalo con el apartado de **Comprobación** del enunciado.
4. Prueba los retos extra.
5. Compara con `Solucion.md` solo al final.

> Romper cosas y arreglarlas es la mejor parte de aprender. Para eso existen los snapshots. 💪

---

## ⚠️ Nota de seguridad

Todo esto se practica **solo en mis propias máquinas virtuales**, en redes aisladas. Nunca pruebes ejercicios de firewall, ataque o hardening en sistemas que no sean tuyos o sobre los que no tengas permiso explícito.

---

## 📄 Licencia

Este repositorio se publica bajo la [licencia Creative Commons Reconocimiento 4.0 Internacional (CC BY 4.0)](LICENSE). Puedes compartirlo y adaptarlo, incluso con fines comerciales, **siempre que reconozcas la autoría**, enlaces la licencia e indiques los cambios.

**Cómo dar el reconocimiento:**

> Basado en "sysadmin" de David ([@misteralva](https://github.com/misteralva)), <https://github.com/misteralva/sysadmin>, bajo licencia CC BY 4.0. Cambios: *(describe lo que has cambiado)*.

---

## 👤 Autor

**David** · [@misteralva](https://github.com/misteralva) · [misteralva.github.io](https://misteralva.github.io)

Estudiante de ASIR interesado en redes, ciberseguridad y automatización.
