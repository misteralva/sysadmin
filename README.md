# 🖥️ Sysadmin: Hands-on Labs from Zero to Expert

🌐 **English** | [Español](README.es.md)

A repository of **hands-on labs to learn systems administration** (Linux and Windows Server), with a clear goal: to build the foundations needed to work in **networking and cybersecurity**.

Each lab comes with its own educational statement (which explains the new concepts) and my documented solution. Difficulty increases gradually: **beginner → intermediate → advanced → expert**, and it ends with a **final project**.

> 📚 This is the third repository of the series, after [Bash](https://github.com/misteralva/Bash) and [Python](https://github.com/misteralva/python). Same format, different goal: there I write **scripts**, here I **configure, break and fix real systems**.
>
> ℹ️ The statements (`Enunciado.md`) and solutions (`Solucion.md`) are currently written in Spanish.

---

## 🎯 Who is this repository for?

- For me, as a learning journal.
- As a portfolio of real, documented work.
- For anyone who wants to learn sysadmin **from zero**, practicing on their own virtual machines.

---

## 📁 Structure

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

Each lab folder contains:

| File           | Content                                                                                           |
| -------------- | ------------------------------------------------------------------------------------------------- |
| `Enunciado.md` | Scenario, new concepts explained, what to do, how to verify it, hints and extra challenges        |
| `Solucion.md`  | My solution: commands with an explanation of each one, and a "What went wrong" section            |

---

## 🧪 Lab environment

All labs are done on **virtual machines** that I own, never on a real or shared system.

| Tool | Use |
| ---- | --- |
| **VirtualBox** | Hypervisor to run the virtual machines |
| **Ubuntu Server (LTS)** | Main Linux system for the 🐧 labs |
| **Windows Server (evaluation version)** | System for the 🪟 labs |

**Golden rule:** take a **snapshot** before any risky lab (disks, network, firewall). If something breaks, you go back in seconds.

---

## 🗺️ Learning path

**Legend:** 🐧 Linux · 🪟 Windows Server · ✅ done · ⏳ pending · 🔜 to be detailed

### 🟢 Level 1 · Beginner

| #  | Lab                                                                                                    | Concepts                                              | OS | Status |
| -- | ------------------------------------------------------------------------------------------------------ | ----------------------------------------------------- | -- | ------ |
| 01 | [Install an Ubuntu Server VM and use snapshots](01-principiante/01-instalar-vm-ubuntu/Enunciado.md)    | virtualization, ISO, resources, snapshots             | 🐧 | ⏳     |
| 02 | [Move around the file system](01-principiante/02-moverse-por-el-sistema-de-ficheros/Enunciado.md)                                                                            | `pwd`, `cd`, `ls`, `mkdir`, file system hierarchy     | 🐧 | ⏳     |
| 03 | [Users and groups](01-principiante/03-usuarios-y-grupos/Enunciado.md)                                                                                       | `useradd`, `usermod`, `groups`, `/etc/passwd`         | 🐧 | ⏳     |
| 04 | [File permissions](01-principiante/04-permisos-de-ficheros/Enunciado.md)                                                                                       | `chmod`, `chown`, `rwx`, `umask`                      | 🐧 | ⏳     |
| 05 | [Install and manage packages](01-principiante/05-paquetes-apt/Enunciado.md)                                                                            | `apt`, repositories, updates                          | 🐧 | ⏳     |
| 06 | [Processes](01-principiante/06-procesos/Enunciado.md)                                                                                              | `ps`, `top`, `kill`, signals                          | 🐧 | ⏳     |
| 07 | [Basic network commands](01-principiante/07-comandos-de-red-basicos/Enunciado.md)                                                                                 | `ip`, `ping`, `ss`, DNS resolution                    | 🐧 | ⏳     |
| 08 | [Remote access with SSH](01-principiante/08-acceso-remoto-ssh/Enunciado.md)                                                                                 | `ssh`, `scp`, host keys                               | 🐧 | ⏳     |
| 09 | [Install Windows Server in a VM](01-principiante/09-instalar-windows-server/Enunciado.md)                                                                         | installation, evaluation edition, snapshots           | 🪟 | ⏳     |
| 10 | [Local users and shared folders](01-principiante/10-usuarios-locales-y-carpetas-compartidas/Enunciado.md)                                                                         | local users, SMB shares, NTFS permissions             | 🪟 | ⏳     |
| 11 | [First steps with PowerShell](01-principiante/11-primeros-pasos-powershell/Enunciado.md)                                                                            | cmdlets, pipeline, `Get-Help`                         | 🪟 | ⏳     |

### 🟡 Level 2 · Intermediate

> 🧪 Before starting, read the [lab environment guide](02-intermedio/README.md): it explains which virtual machines and networks you need.

| #  | Lab                                                | Concepts                                       | OS | Status |
| -- | -------------------------------------------------- | ---------------------------------------------- | -- | ------ |
| 01 | [Services with `systemd`](02-intermedio/01-servicios-systemd/Enunciado.md)                            | units, `systemctl`, enable/disable             | 🐧 | ⏳     |
| 02 | [Scheduled tasks: `cron` and timers](02-intermedio/02-cron-y-timers/Enunciado.md)                 | crontab syntax, `systemd` timers               | 🐧 | ⏳     |
| 03 | [Static IP address](02-intermedio/03-ip-estatica-netplan/Enunciado.md)                                  | `netplan`, gateway, DNS                        | 🐧 | ⏳     |
| 04 | [Disks, partitions and `fstab`](02-intermedio/04-discos-particiones-fstab/Enunciado.md)                      | `lsblk`, `fdisk`, file systems, mounting       | 🐧 | ⏳     |
| 05 | [LVM](02-intermedio/05-lvm/Enunciado.md)                                                | PV, VG, LV, extending volumes                  | 🐧 | ⏳     |
| 06 | [Logs with `journalctl`](02-intermedio/06-logs-journalctl/Enunciado.md)                             | `/var/log`, filtering by service and time      | 🐧 | ⏳     |
| 07 | [Firewall with `ufw`](02-intermedio/07-firewall-ufw/Enunciado.md)                                | rules, default policies, ports                 | 🐧 | ⏳     |
| 08 | [SSH key authentication](02-intermedio/08-ssh-con-claves/Enunciado.md)                             | public/private keys, `authorized_keys`         | 🐧 | ⏳     |
| 09 | [Backups with `tar` and `rsync`](02-intermedio/09-backups-tar-rsync/Enunciado.md)                     | full and incremental backups, restoring        | 🐧 | ⏳     |
| 10 | [Web server with Nginx](02-intermedio/10-servidor-web-nginx/Enunciado.md)                              | installation, sites, basic configuration       | 🐧 | ⏳     |
| 11 | [Active Directory: create the domain](02-intermedio/11-active-directory-crear-dominio/Enunciado.md)                | AD DS, domain controller, DNS                  | 🪟 | ⏳     |
| 12 | [Join machines to the domain, OUs and users](02-intermedio/12-unir-equipos-dominio-ous-usuarios/Enunciado.md)         | domain join, organizational units              | 🪟 | ⏳     |
| 13 | [DNS and DHCP on Windows Server](02-intermedio/13-dns-y-dhcp-windows-server/Enunciado.md)                     | scopes, leases, DNS records                    | 🪟 | ⏳     |
| 14 | [Basic GPOs](02-intermedio/14-gpo-basicas/Enunciado.md)                                         | group policies, linking, inheritance           | 🪟 | ⏳     |

### 🟠 Level 3 · Advanced

> 🧪 Before starting, read the [lab environment guide](03-avanzado/README.md): it explains the extra virtual machines and networks you need. Security labs here are **defensive** (configure, harden, monitor); offensive practice belongs in a separate cybersecurity repository.

| #  | Lab | Concepts | OS | Status |
| -- | --- | -------- | -- | ------ |
| 01 | [RAID with `mdadm`](03-avanzado/01-raid-mdadm/Enunciado.md) | RAID levels, degraded arrays, rebuild | 🐧 | ⏳ |
| 02 | [File sharing with NFS and Samba](03-avanzado/02-nfs-y-samba/Enunciado.md) | `/etc/exports`, `root_squash`, `smb.conf` | 🐧 | ⏳ |
| 03 | [Advanced firewall with `nftables`](03-avanzado/03-firewall-nftables/Enunciado.md) | tables, chains, rules, stateful filtering | 🐧 | ⏳ |
| 04 | [Automatic protection with `fail2ban`](03-avanzado/04-fail2ban/Enunciado.md) | jails, filters, dynamic banning | 🐧 | ⏳ |
| 05 | [Hardening and auditing with Lynis](03-avanzado/05-hardening-y-auditoria-lynis/Enunciado.md) | attack surface, `sysctl`, `auditd`, SSH | 🐧 | ⏳ |
| 06 | [Mandatory access control with AppArmor](03-avanzado/06-apparmor/Enunciado.md) | DAC vs MAC, profiles, enforce/complain | 🐧 | ⏳ |
| 07 | [OpenLDAP: centralized authentication](03-avanzado/07-openldap-autenticacion-centralizada/Enunciado.md) | LDAP, LDIF, NSS and PAM | 🐧 | ⏳ |
| 08 | [Monitoring with Prometheus and Grafana](03-avanzado/08-monitorizacion-prometheus-grafana/Enunciado.md) | metrics, exporters, PromQL, alerts | 🐧 | ⏳ |
| 09 | [Introduction to Ansible](03-avanzado/09-ansible-introduccion/Enunciado.md) | inventory, playbooks, idempotence | 🐧 | ⏳ |
| 10 | [Linux as a router and NAT](03-avanzado/10-linux-router-y-nat/Enunciado.md) | `ip_forward`, masquerade, forwarding policy | 🐧 | ⏳ |
| 11 | [VLANs on Linux](03-avanzado/11-vlans-en-linux/Enunciado.md) | 802.1Q, trunk/access, router-on-a-stick | 🐧 | ⏳ |
| 12 | [VPN with WireGuard](03-avanzado/12-vpn-wireguard/Enunciado.md) | keys, peers, `AllowedIPs`, encrypted tunnel | 🐧 | ⏳ |
| 13 | [Advanced GPOs and fine-grained passwords](03-avanzado/13-gpo-avanzadas/Enunciado.md) | security filtering, WMI, loopback, PSO | 🪟 | ⏳ |
| 14 | [PowerShell: Active Directory and remoting](03-avanzado/14-powershell-active-directory-y-remoting/Enunciado.md) | scripts, CSV import, WinRM, reports | 🪟 | ⏳ |
| 15 | [Active Directory backup and restore](03-avanzado/15-backup-y-restauracion-de-ad/Enunciado.md) | Recycle Bin, System State, DSRM | 🪟 | ⏳ |

### 🔴 Level 4 · Expert

> 🧪 Read the [expert level guide](04-experto/README.md) first: here you get a **scenario and a goal**, less guidance, and each lab asks for a **professional deliverable** (runbook, report, plan). Security labs remain **defensive**.

| #  | Lab | Concepts | OS | Status |
| -- | --- | -------- | -- | ------ |
| 01 | [Recover a system that won't boot](04-experto/01-recuperar-sistema-que-no-arranca/Enunciado.md) | boot process, GRUB, rescue mode, `chroot` | 🐧 | ⏳ |
| 02 | [Performance and bottlenecks](04-experto/02-rendimiento-y-cuellos-de-botella/Enunciado.md) | USE method, `vmstat`, `iostat`, cgroups, OOM killer | 🐧 | ⏳ |
| 03 | [Containers with Docker](04-experto/03-contenedores-docker/Enunciado.md) | images, volumes, Compose, container security | 🐧 | ⏳ |
| 04 | [Centralized logs with `rsyslog`](04-experto/04-logs-centralizados-rsyslog/Enunciado.md) | remote logging, templates, time sync, integrity | 🐧 | ⏳ |
| 05 | [PKI and certificates](04-experto/05-pki-y-certificados/Enunciado.md) | own CA, CSR, SAN, HTTPS, trust chain | 🐧 | ⏳ |
| 06 | [IDS with Suricata](04-experto/06-ids-suricata/Enunciado.md) | rules, signatures, alerts, `eve.json` | 🐧 | ⏳ |
| 07 | [High availability: HAProxy and keepalived](04-experto/07-alta-disponibilidad-haproxy-keepalived/Enunciado.md) | load balancing, VRRP, virtual IP, failover | 🐧 | ⏳ |
| 08 | [Disaster recovery drill](04-experto/08-simulacro-de-desastre/Enunciado.md) | RPO/RTO, runbooks, rebuild from scratch | 🐧 | ⏳ |
| 09 | [Active Directory security audit](04-experto/09-auditoria-de-seguridad-de-active-directory/Enunciado.md) | privileged accounts, `krbtgt`, delegation, remediation | 🪟 | ⏳ |
| 10 | [Break/fix on Linux](04-experto/10-break-fix-linux/Enunciado.md) | diagnostic method, root cause, runbooks | 🐧 | ⏳ |
| 11 | [Break/fix on Windows and network](04-experto/11-break-fix-windows-y-red/Enunciado.md) | DNS, time/Kerberos, GPO, DHCP, VLAN | 🪟 | ⏳ |
| 12 | [Incident response](04-experto/12-respuesta-a-incidentes/Enunciado.md) | evidence, persistence, containment, report | 🐧 | ⏳ |

### 🏁 Level 5 · Final project

The capstone: design, build, secure, document and **demonstrate** the full infrastructure of a fictional company, **NovaTech Sistemas S.L.**, using everything from the four levels. There are no guided steps: you get a scenario, requirements with acceptance criteria, a reference architecture and deliverables. Three ambition levels (minimum, standard, advanced) let you scale it to your time and hardware.

| Document | What it is |
| -------- | ---------- |
| [Project brief](05-proyecto-final/Enunciado.md) | Scenario, architecture, requirements, phases, tests and deliverables |
| [Self-assessment rubric](05-proyecto-final/Rubrica.md) | Six criteria, 100 points, evidence-based |
| [Templates](05-proyecto-final/Plantillas/) | Architecture document, runbook, test plan, security report and a suggested `.gitignore` |

---

## 🧭 How to use this repository to learn

1. Read the lab's `Enunciado.md`.
2. Try to solve it **on your own**, using the concepts and hints.
3. Check it with the **Verification** section of the statement.
4. Try the extra challenges.
5. Compare with `Solucion.md` only at the end.

> Breaking things and fixing them is the best part of learning. Snapshots exist for that. 💪

---

## ⚠️ Safety note

Everything here is practiced **only on my own virtual machines**, in isolated networks. Never test firewall, attack or hardening exercises on systems you do not own or do not have explicit permission to touch.

---

## 📄 License

This repository is licensed under the [Creative Commons Attribution 4.0 International License (CC BY 4.0)](LICENSE). You can share and adapt it, even commercially, **as long as you give appropriate credit**, link to the license and indicate any changes.

**How to credit:**

> Based on "sysadmin" by David ([@misteralva](https://github.com/misteralva)), <https://github.com/misteralva/sysadmin>, licensed under CC BY 4.0. Changes: *(describe what you changed)*.

---

## 👤 Author

**David** · [@misteralva](https://github.com/misteralva) · [misteralva.github.io](https://misteralva.github.io)

ASIR student interested in networking, cybersecurity and automation.
