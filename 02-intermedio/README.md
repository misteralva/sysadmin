# 🟡 Nivel 2 · Intermedio: entorno de laboratorio

En el nivel intermedio ya no basta con una sola máquina: vas a necesitar **varias VMs conectadas entre sí**. Esta guía describe qué montar **una vez** y lo reutilizarás en todos los laboratorios.

---

## 🖥️ Máquinas

| VM | Sistema | Se usa en | Recursos mínimos |
| -- | ------- | --------- | ---------------- |
| `ubuntu-server-lab` | Ubuntu Server (la del nivel 1) | Laboratorios 01-10 | 2 GB RAM, 2 CPU |
| `dc01` | Windows Server (clon de la del laboratorio 09) | Laboratorios 11-14 | 4 GB RAM, 2 CPU |
| `pc-cliente01` | Windows de escritorio, edición Pro/Enterprise (evaluación) | Laboratorios 12-14 | 4 GB RAM, 2 CPU |

> ⚠️ **Windows Home no puede unirse a un dominio.** Necesitas una edición Pro, Enterprise o Education. Si tu equipo no aguanta tres VMs a la vez, puedes usar como cliente **otro Windows Server** (clónalo del laboratorio 09): es un "servidor miembro" y sirve igual para practicar.

Enciende solo las VMs que necesites en cada laboratorio.

---

## 🌐 Redes

VirtualBox permite varios tipos de red. Usarás tres:

| Tipo | Qué hace | Uso |
| ---- | -------- | --- |
| **NAT** | La VM sale a internet; nadie entra | Actualizaciones y descargas |
| **Red solo-anfitrión** (*Host-Only*) | Conecta tu ordenador real con las VMs, sin internet | Acceder desde tu equipo (SSH, navegador) |
| **Red interna** (*Internal Network*) | Una red privada **solo entre VMs** | El "mundo" de tu empresa ficticia (dominio, DHCP...) |

### Linux (`ubuntu-server-lab`)

| Adaptador | Tipo | IP |
| --------- | ---- | -- |
| Adaptador 1 | NAT | Automática (DHCP) |
| Adaptador 2 | Red solo-anfitrión | Fija, `192.168.56.10/24` (laboratorio 03) |

### Windows (`dc01` y `pc-cliente01`)

| VM | Adaptador | Tipo | IP |
| -- | --------- | ---- | -- |
| `dc01` | Adaptador 1 | Red interna, nombre `lan-empresa` | Fija, `192.168.10.10/24` |
| `pc-cliente01` | Adaptador 1 | Red interna, nombre `lan-empresa` | `192.168.10.20` fija (lab. 12), luego por DHCP (lab. 13) |

El controlador de dominio usa **un solo adaptador** a propósito: los DC con varias tarjetas de red dan problemas de DNS (verás por qué en un reto del laboratorio 11).

---

## 💾 Reglas de oro

1. **Snapshot** antes de cada laboratorio. En Windows, sobre todo antes del 11.
2. Apunta las **IPs y contraseñas** de tu laboratorio en un archivo (no lo subas a GitHub).
3. Haz una pausa y **lee los errores completos**: casi siempre dicen qué falla.
4. Todo se practica en **tus propias VMs** y en redes aisladas.

---

## 🗺️ Orden recomendado

Los laboratorios 01-10 (Linux) son prácticamente independientes del 11-14 (Windows), pero dentro de cada bloque hay dependencias:

- **03** (IP fija) antes del **07** (firewall), **08** (SSH con claves) y **10** (Nginx), porque usan la red solo-anfitrión.
- **04** (discos) antes del **05** (LVM) y del **09** (backups).
- **11** (dominio) → **12** (unir equipos) → **13** (DNS/DHCP) → **14** (GPO), en ese orden.
