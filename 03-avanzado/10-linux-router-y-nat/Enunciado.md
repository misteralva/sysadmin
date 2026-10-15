# 🔀 Laboratorio 10 · Linux como router y NAT

**Nivel:** 🟠 Avanzado · **Sistema:** 🐧 Linux · **Tiempo estimado:** 120 minutos

---

## 🎯 Escenario

La empresa tiene **dos redes internas**: la de **Sistemas** (`192.168.20.0/24`) y la de **Invitados** (`192.168.30.0/24`). Quieren que:

- Cada red tenga **salida a internet**.
- Sistemas **pueda** llegar a Invitados, pero **Invitados no pueda** llegar a Sistemas.

No hay presupuesto para un router profesional, así que vas a **convertir un servidor Linux en un router con firewall y NAT**. Es justo lo que hacen por dentro muchos routers domésticos.

> 💾 **Snapshot** de todas las VMs. Necesitas tres máquinas pequeñas (clones, 1 GB de RAM): `router-lab`, `cliente-a` y `cliente-b` (ver `03-avanzado/README.md`).

---

## 📖 Conceptos nuevos

### Qué hace un router

Un **router** conecta **redes distintas** y **reenvía** (*forward*) los paquetes de una a otra según su **tabla de rutas**. Un equipo normal descarta lo que no es para él; un router lo reenvía.

> **Analogía:** una **oficina de correos** que recibe cartas de un barrio y las manda al barrio correcto, o a otra ciudad si el destino no es local.

### La topología del laboratorio

```
                   internet
                      │
                  (NAT, VirtualBox)
                      │ enp0s3
              ┌───────┴────────┐
              │   router-lab   │
              └─┬────────────┬─┘
        enp0s8  │            │  enp0s9
   192.168.20.1 │            │ 192.168.30.1
   red interna  │            │  red interna
     "lan-a"    │            │    "lan-b"
          ┌─────┴────┐  ┌────┴─────┐
          │ cliente-a│  │ cliente-b│
          │ .20.10   │  │ .30.10   │
          └──────────┘  └──────────┘
```

(Los nombres de interfaz `enp0s3`, `enp0s8`, `enp0s9` pueden variar: compruébalos con `ip -br a`.)

### Reenvío de paquetes en Linux

Por seguridad, Linux **no reenvía** paquetes entre interfaces por defecto. Hay que activarlo con el parámetro del kernel **`net.ipv4.ip_forward`**.

### Puerta de enlace de los clientes

Cada cliente necesita que su **puerta de enlace** sea la IP del router **en su red** (`cliente-a` → `192.168.20.1`; `cliente-b` → `192.168.30.1`). Así sabe adónde mandar lo que no es de su red.

### NAT y *masquerade*

Las redes `192.168.x.x` son **privadas**: no se enrutan por internet. Para que salgan, el router hace **NAT** (*Network Address Translation*): **sustituye la IP privada** del paquete por la **suya** al salir, y deshace el cambio con la respuesta. La variante **masquerade** usa automáticamente la IP actual de la interfaz de salida.

> **Analogía:** la **centralita** de una empresa: desde fuera todas las llamadas salen con el mismo número, y la centralita sabe a qué extensión devolver cada respuesta.

### Reglas en `nftables`

Necesitas dos tablas: una de **filtrado** (la cadena `forward`) y otra de **NAT** (la cadena `postrouting`). Esqueleto (completa las `???`):

```nft
table inet filtro {
    chain forward {
        type filter hook forward priority 0; policy ???;
        ct state established,related accept
        ???                        # Sistemas (enp0s8) puede ir a todas partes
        ???                        # Invitados (enp0s9) solo puede salir por internet (enp0s3)
    }
}

table ip nat {
    chain postrouting {
        type nat hook postrouting priority 100;
        ???                        # masquerade por la interfaz de salida
    }
}
```

### Comandos de este laboratorio

| Comando | Qué hace |
| ------- | -------- |
| `sysctl net.ipv4.ip_forward` | Consulta si el reenvío está activo (`0` = no, `1` = sí) |
| `sudo sysctl -w net.ipv4.ip_forward=1` | Lo activa **ahora** (se pierde al reiniciar) |
| `/etc/sysctl.d/99-routing.conf` | Archivo donde lo haces **permanente** |
| `ip r` | Tabla de rutas |
| `ip route get IP` | Dice **por dónde** saldría un paquete hacia esa IP |
| `sudo tcpdump -ni enp0s8 icmp` | **Captura** el tráfico ICMP (ping) que pasa por esa interfaz (`-n` sin resolver nombres, `-i` interfaz) |
| `traceroute -n IP` | Muestra los saltos hasta un destino (`-n` = números) |
| `sudo nft list ruleset` | Muestra las reglas activas |

---

## 🛠️ Qué debes hacer

### Parte A · Montar la topología

1. Crea los clones `router-lab`, `cliente-a` y `cliente-b` (1 GB de RAM) con **nombres de host distintos**.
2. Configura las **redes de VirtualBox**:
   - `router-lab`: Adaptador 1 **NAT**, Adaptador 2 **red interna `lan-a`**, Adaptador 3 **red interna `lan-b`**.
   - `cliente-a`: un solo adaptador en `lan-a`.
   - `cliente-b`: un solo adaptador en `lan-b`.
3. Con **netplan**, asigna IPs fijas: router `192.168.20.1/24` y `192.168.30.1/24`; `cliente-a` `192.168.20.10/24` con puerta de enlace `192.168.20.1`; `cliente-b` `192.168.30.10/24` con puerta de enlace `192.168.30.1`. Pon un DNS público (por ejemplo `8.8.8.8`) en los clientes.

### Parte B · Sin reenvío

4. Desde `cliente-a`, haz `ping` al router (`192.168.20.1`): debe responder. Haz `ping` a `cliente-b`: **no** debe llegar. Haz `ping` a `8.8.8.8`: **no**.
5. En el router, comprueba que `ip_forward` vale **0** y razona qué significa.

### Parte C · Activar el router

6. **Activa** el reenvío y haz que **persista** tras reiniciar.
7. Vuelve a probar `ping` desde `cliente-a` a `cliente-b`. Con `tcpdump` en el router, **observa** el paquete pasando por las dos interfaces.
8. Prueba `ping 8.8.8.8` desde `cliente-a`: la petición sale pero **no vuelve**. Razona por qué (pista: ¿conoce internet la IP `192.168.20.10`?).

### Parte D · NAT y política

9. Escribe tu configuración de `nftables`: tabla `nat` con **masquerade** por la interfaz de salida, y tabla `filtro` con la cadena `forward`.
10. **Comprueba la sintaxis** (`nft -c`), cárgala y haz que **persista** (`/etc/nftables.conf` y el servicio `nftables`).
11. Comprueba que `cliente-a` y `cliente-b` **navegan** (`ping 8.8.8.8` y `curl` a un sitio).
12. Aplica la política: **Sistemas → Invitados: permitido; Invitados → Sistemas: bloqueado** (las respuestas a conexiones iniciadas desde Sistemas sí deben pasar). Compruébalo con `ping`.
13. **Registra** los intentos bloqueados con `log` en la cadena `forward` y localízalos en el journal.

---

## ✅ Comprobación

| Prueba | Qué debes ver |
| ------ | ------------- |
| `sysctl net.ipv4.ip_forward` | `= 1`, también tras reiniciar |
| `ip route get 8.8.8.8` (cliente-a) | `via 192.168.20.1` |
| `ping 192.168.30.10` desde `cliente-a` | **Responde** |
| `ping 192.168.20.10` desde `cliente-b` | **Sin respuesta** |
| `ping 8.8.8.8` desde ambos | Responde (NAT funciona) |
| `traceroute -n 8.8.8.8` (cliente-a) | El primer salto es `192.168.20.1` |
| `sudo journalctl -k \| grep "forward-drop"` (router) | Los intentos de Invitados → Sistemas |

---

## 🧩 Pistas

- Si un ping llega al destino pero **no vuelve**, el problema casi siempre es la **puerta de enlace** del destino o la falta de NAT.
- Con `tcpdump` en cada interfaz del router ves **dónde se pierde** el paquete: es la herramienta de diagnóstico de redes por excelencia.
- Si olvidas `ct state established,related accept`, bloquearás **hasta las respuestas** de las conexiones permitidas.
- `iifname "enp0s9" oifname "enp0s8" drop` descarta paquetes que **entran por** Invitados y **salen por** Sistemas.
- Si cambias reglas y algo deja de ir, `sudo nft list ruleset` te enseña el estado **real**, no el que creías.
- Recuerda: NAT solo en el router; los clientes **no** hacen NAT.

---

## 🚀 Retos extra

1. Instala un servidor **DHCP** (`isc-dhcp-server` o `dnsmasq`) en el router para que los clientes reciban IP automáticamente, como hiciste en Windows.
2. Añade **reenvío de puertos** (DNAT): que `http://IP-del-router:8080` llegue a un servidor web de `cliente-a`.
3. Añade una **ruta estática** hacia una tercera red inventada y compara con `ip route get`.
4. Haz que el router **bloquee** el acceso de Invitados a un puerto concreto (por ejemplo SSH) **hacia internet**.
5. Investiga qué cambia con **IPv6**: ¿por qué no se necesita NAT en IPv6?

---

## 📝 Qué documentar en tu `Solucion.md`

- Un esquema de la red con **todas** las IP y las interfaces.
- Tu `nftables.conf`, explicado.
- Qué pasa con `ip_forward` a 0 y a 1, con tus capturas de `tcpdump`.
- Con tus palabras: **NAT**, **forward** y **puerta de enlace**, y cómo se relacionan.

---

## 🧠 Lo que has practicado

`routing` · `ip_forward` · `tabla de rutas` · `puerta de enlace` · `NAT / masquerade` · `cadena forward` · `segmentación con firewall` · `tcpdump` · `traceroute`
