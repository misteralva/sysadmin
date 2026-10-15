# 🏷️ Laboratorio 11 · VLANs en Linux

**Nivel:** 🟠 Avanzado · **Sistema:** 🐧 Linux · **Tiempo estimado:** 120 minutos

---

## 🎯 Escenario

En el laboratorio anterior separaste dos redes usando **dos cables** (dos interfaces) en el router. Pero en una oficina real, un mismo cable y un mismo switch tienen que servir a **varios departamentos** que no deben verse entre sí: Sistemas, Contabilidad, Invitados...

La solución son las **VLAN**: redes **virtuales** separadas que comparten la misma infraestructura física. Tu misión es montar dos VLAN sobre **un único enlace**, y un router que las conecte por ese mismo cable (*router-on-a-stick*).

> 💾 **Snapshot** de todas las VMs. Usarás `router-lab`, `cliente-a` y `cliente-b` del laboratorio anterior, pero con una nueva red interna llamada `trunk` (ver más abajo). **Apaga** las demás redes de esas VMs para no confundirte.

---

## 📖 Conceptos nuevos

### Qué es una VLAN

Una **VLAN** (*Virtual LAN*) divide **una red física en varias redes lógicas** aisladas entre sí. Equipos de VLAN distintas **no se ven**, aunque estén enchufados al mismo switch.

> **Analogía:** un **edificio de oficinas con una sola escalera** y dos empresas distintas en pisos separados. Comparten estructura, pero cada empresa solo tiene llave de su planta.

Ventajas: **seguridad** (aislas departamentos), **orden** (menos tráfico mezclado) y **flexibilidad** (agrupas por función, no por cable).

### El etiquetado 802.1Q

Para que varias VLAN viajen por **un mismo cable**, cada trama lleva una **etiqueta** (*tag*) con el **número de VLAN** (de 1 a 4094). Es el estándar **IEEE 802.1Q**. El destinatario mira la etiqueta y sabe a qué VLAN pertenece.

### Puertos *access* y *trunk* (en un switch real)

| Tipo de puerto | Qué hace |
| -------------- | -------- |
| **Access** | Pertenece a **una sola VLAN**; los equipos finales no ven las etiquetas |
| **Trunk** | Transporta **varias VLAN** con etiquetas (entre switches, o hacia un router) |

> En un switch Cisco serían los comandos `switchport mode access` / `switchport mode trunk`. En este laboratorio no hay switch gestionable: cada VM **etiqueta ella misma** sus tramas.

### Router-on-a-stick

Para que dos VLAN se comuniquen se necesita un **router** (son redes distintas). Con *router-on-a-stick*, el router tiene **una sola interfaz física** (el trunk) y **una subinterfaz por VLAN** (`enp0s8.10`, `enp0s8.20`), cada una con la IP de puerta de enlace de su VLAN.

### Plan del laboratorio

| VLAN | Red | Equipo | IP |
| ---- | --- | ------ | -- |
| **10** (Sistemas) | `10.10.10.0/24` | router (subinterfaz) | `10.10.10.1` |
| | | `cliente-a` | `10.10.10.10` |
| **20** (Invitados) | `10.10.20.0/24` | router (subinterfaz) | `10.10.20.1` |
| | | `cliente-b` | `10.10.20.10` |

Las **tres VMs** se conectan con **una sola** interfaz a la red interna `trunk` de VirtualBox. Aviso: VirtualBox **no entiende de VLAN** (la red interna se comporta como un cable compartido que deja pasar las etiquetas); el aislamiento aquí lo logras con **tu configuración**, no con el "switch".

### Comandos de este laboratorio

| Comando | Qué hace |
| ------- | -------- |
| `sudo ip link add link enp0s8 name enp0s8.10 type vlan id 10` | Crea una **subinterfaz VLAN** (`link` = interfaz física, `name` = nombre, `id` = número de VLAN) |
| `sudo ip addr add 10.10.10.1/24 dev enp0s8.10` | Le asigna una IP |
| `sudo ip link set enp0s8.10 up` | La enciende |
| `ip -d link show enp0s8.10` | Muestra el detalle, incluido `vlan id` |
| `sudo ip link del enp0s8.10` | La borra |
| `sudo tcpdump -ni enp0s8 -e vlan` | Captura tráfico **mostrando la etiqueta VLAN** (`-e` muestra la cabecera de enlace) |
| `sudo sysctl -w net.ipv4.ip_forward=1` | Activa el reenvío (como en el laboratorio 10) |

Los cambios con `ip` **se pierden al reiniciar**. Para hacerlos permanentes, `netplan` tiene una sección `vlans:` (reto 1).

---

## 🛠️ Qué debes hacer

### Parte A · VLAN y aislamiento

1. Cambia el adaptador de las tres VMs a una **red interna llamada `trunk`** (y desactiva los otros adaptadores que usaste en el laboratorio 10).
2. En `cliente-a`, crea una **subinterfaz VLAN 10** con IP `10.10.10.10/24`. En `cliente-b`, una **VLAN 20** con `10.10.20.10/24`. (La interfaz física solo se enciende, sin IP.)
3. En el `router-lab`, crea las **dos subinterfaces** (`.10` y `.20`) con `10.10.10.1/24` y `10.10.20.1/24`.
4. Comprueba que `cliente-a` hace ping a `10.10.10.1`, y `cliente-b` a `10.10.20.1`.
5. Comprueba que `cliente-a` **no** llega a `cliente-b` todavía (¿por qué? Pista: no tienen puerta de enlace ni reenvío).

### Parte B · Router-on-a-stick

6. Configura la **puerta de enlace** de cada cliente: `10.10.10.1` y `10.10.20.1` respectivamente (`ip route add default via ...`).
7. **Activa el reenvío** en el router.
8. Comprueba el ping **entre VLAN** (`cliente-a` → `10.10.20.10`).
9. Haz `traceroute -n 10.10.20.10` desde `cliente-a` y comprueba que el primer salto es el router.

### Parte C · Ver las etiquetas

10. En el router, captura con `tcpdump -ni enp0s8 -e vlan` mientras haces ping entre las VLAN. Localiza **la etiqueta** (`vlan 10` y `vlan 20`) en cada paquete.
11. Compara con una captura **sin** el filtro `vlan`: ¿qué ves?

### Parte D · Política entre VLAN

12. Con `nftables` en el router (laboratorio 03 y 10), **bloquea el tráfico de la VLAN 20 hacia la VLAN 10**, dejando pasar el tráfico de **Sistemas hacia Invitados** y las respuestas. Usa los nombres de las **subinterfaces** (`iifname "enp0s8.20" oifname "enp0s8.10" drop`).
13. Comprueba con `ping` que la regla funciona en **un solo sentido**.

---

## ✅ Comprobación

| Prueba | Qué debes ver |
| ------ | ------------- |
| `ip -d link show enp0s8.10` | Línea con `vlan protocol 802.1Q id 10` |
| `ping 10.10.10.1` desde `cliente-a` | Responde |
| Antes del reenvío: `ping 10.10.20.10` desde `cliente-a` | Sin respuesta |
| Con reenvío: el mismo ping | Responde |
| `traceroute -n 10.10.20.10` | Primer salto `10.10.10.1` |
| `tcpdump -ni enp0s8 -e vlan` | Tramas con `vlan 10` y `vlan 20` |
| Con la política | Invitados → Sistemas **bloqueado**; Sistemas → Invitados, **permitido** |

---

## 🧩 Pistas

- Si ves `RTNETLINK answers: File exists` al crear una ruta o IP, ya existe: bórrala antes (`ip addr del`, `ip route del`).
- El **nombre de la interfaz física** (aquí `enp0s8`) puede ser otro: míralo con `ip -br a`.
- Las subinterfaces VLAN **necesitan que la interfaz física esté encendida**: `ip link set enp0s8 up`.
- Si el ping no pasa, usa `tcpdump` **en cada VM**: te dirá dónde se pierde. Es el equivalente a un "modo depuración" de una red.
- Si cambias nombres de VLAN a un número distinto en un extremo y otro, **no se verán**: el número tiene que coincidir.

---

## 🚀 Retos extra

1. Haz la configuración **permanente** con `netplan` (sección `vlans:` dentro de la interfaz) y reinicia para comprobarlo.
2. Añade una **tercera VLAN** (30, "Contabilidad") y ponla **sin acceso a ninguna otra**.
3. Instala un servidor **DHCP** distinto por VLAN en el router, como en el laboratorio 13 del nivel intermedio.
4. Mira cómo se haría lo mismo en un **switch Cisco** o en **Packet Tracer**: escribe los comandos equivalentes de `vlan 10`, `switchport mode access/trunk` y `interface g0/0.10` + `encapsulation dot1Q 10`.
5. Investiga la **VLAN nativa** y por qué una mala configuración puede ser un riesgo de seguridad (*VLAN hopping*). Es un concepto, no una práctica.

---

## 📝 Qué documentar en tu `Solucion.md`

- Un esquema del trunk con las dos VLAN y sus IP.
- Capturas de `tcpdump` con las etiquetas **802.1Q**.
- Con tus palabras: **VLAN, access, trunk y router-on-a-stick**.
- La tabla de **equivalencias** con los comandos de un switch/router Cisco.

---

## 🧠 Lo que has practicado

`VLAN` · `802.1Q` · `access y trunk` · `subinterfaces` · `router-on-a-stick` · `segmentación` · `filtrado entre VLAN` · `tcpdump con etiquetas`
