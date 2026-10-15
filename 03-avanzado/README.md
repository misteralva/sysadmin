# 🟠 Nivel 3 · Avanzado: entorno de laboratorio

En este nivel pasas de **administrar máquinas** a **diseñar y proteger infraestructuras**: varias máquinas, varias redes, automatización y seguridad. Esta guía resume qué necesitas montar. Parte de lo que ya tienes del nivel intermedio (lee antes `02-intermedio/README.md`).

---

## 🖥️ Máquinas

| VM | Sistema | Red | Se usa en |
| -- | ------- | --- | --------- |
| `ubuntu-server-lab` | Ubuntu Server (la de siempre) | NAT + solo-anfitrión `192.168.56.10` | Casi todos los laboratorios Linux |
| `ubuntu-cliente` | **Clon** de la anterior | NAT + solo-anfitrión `192.168.56.11` | 02, 04, 07, 08, 09, 12 |
| `router-lab` | Clon, 1 GB RAM | NAT + 2 redes internas | 10, 11 |
| `cliente-a`, `cliente-b` | Clones, 1 GB RAM | 1 red interna cada uno | 10, 11 |
| `dc01` y `pc-cliente01` | Los del nivel intermedio | Red interna `lan-empresa` | 13, 14, 15 |

> 💡 Usa **clones vinculados** (*linked clones*) para ahorrar disco: comparten el disco base y solo guardan las diferencias. Al clonar, elige siempre **"Generar nuevas direcciones MAC"** y cambia el **nombre de host** y, si hace falta, el **machine-id** (`sudo rm /etc/machine-id` y `sudo systemd-machine-id-setup`) para que no haya duplicados.

**Reparto de RAM aproximado** (para no ahogar tu equipo): un Ubuntu Server cómodo con 1-2 GB, un Windows Server con 4 GB. Enciende **solo lo que cada laboratorio pida**.

---

## 🧭 Orden recomendado

Los laboratorios de Linux son bastante independientes, pero conviene este orden:

1. **Almacenamiento y ficheros:** 01 (RAID) → 02 (NFS y Samba)
2. **Seguridad del servidor:** 03 (nftables) → 04 (fail2ban) → 05 (hardening) → 06 (AppArmor)
3. **Servicios de infraestructura:** 07 (LDAP) → 08 (monitorización) → 09 (Ansible)
4. **Redes:** 10 (router y NAT) → 11 (VLANs) → 12 (VPN)
5. **Windows Server:** 13 → 14 → 15 (necesitan el dominio del nivel intermedio)

---

## 🛡️ Enfoque de seguridad

Los laboratorios de seguridad de este nivel son **defensivos**: configurar, endurecer, vigilar y detectar. El lado ofensivo (probar ataques) encaja mejor en un repositorio de ciberseguridad aparte, con sus propias reglas y entornos aislados. Aquí, cuando veas ataques, serán **solo los intentos de acceso fallidos** que necesitas para comprobar tus defensas, siempre entre tus propias VMs.

---

## 💾 Reglas de oro

1. **Snapshot antes de cada laboratorio**, y más aún antes de firewall (03), hardening (05), AppArmor (06) y la restauración de AD (15).
2. Mantén siempre una **segunda sesión abierta** (o la consola de VirtualBox) cuando toques red, firewall o SSH.
3. Las **claves privadas, contraseñas y configuraciones** de tu laboratorio **no se suben a GitHub**.
4. Todo se practica **únicamente en tus VMs y redes aisladas**.
