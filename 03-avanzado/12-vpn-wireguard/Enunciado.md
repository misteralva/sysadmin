# 🔐 Laboratorio 12 · VPN con WireGuard

**Nivel:** 🟠 Avanzado · **Sistema:** 🐧 Linux · **Tiempo estimado:** 100 minutos

---

## 🎯 Escenario

Un empleado trabaja desde casa y necesita acceder a los recursos internos de la empresa. Si los servicios internos se **abren a internet**, cualquiera puede intentar atacarlos. La solución estándar es una **VPN**: un **túnel cifrado** entre el portátil del empleado y la red de la empresa. Desde fuera, solo se ve **un puerto** y el tráfico es ilegible.

Tu misión es montar una VPN con **WireGuard**, la tecnología moderna, rápida y sencilla, entre tus dos servidores.

> 💾 **Snapshot** de las dos VMs. **Servidor VPN** (la "empresa"): `ubuntu-server-lab` (`192.168.56.10`). **Cliente VPN** (el "empleado en casa"): `ubuntu-cliente` (`192.168.56.11`).

---

## 📖 Conceptos nuevos

### Qué es una VPN

Una **VPN** (*Virtual Private Network*) crea un **túnel cifrado** sobre una red no confiable (como internet). Para los programas, es como si los dos extremos estuvieran en **la misma red privada**.

> **Analogía:** un **tubo opaco** que atraviesa una ciudad. La gente de fuera ve pasar el tubo, pero **no puede ver ni tocar lo que viaja dentro**.

### Por qué WireGuard

| Característica | WireGuard |
| -------------- | --------- |
| **Código** | Muy pequeño (unas pocas miles de líneas): más fácil de auditar |
| **Rendimiento** | Muy rápido, está en el kernel de Linux |
| **Configuración** | Sencilla, un archivo corto |
| **Criptografía** | Moderna y fija, sin elegir algoritmos (menos errores posibles) |
| **Protocolo** | UDP, normalmente puerto **51820** |

### Cómo funciona: claves y *peers*

Cada extremo (*peer*) tiene un **par de claves**: una **privada** (secreta, **nunca** sale de esa máquina) y una **pública** (se la das al otro extremo). Es la misma idea que en SSH con claves (laboratorio 08 del nivel intermedio).

| Dato | Qué es |
| ---- | ------ |
| **Interfaz** `wg0` | La "tarjeta de red virtual" del túnel |
| **Dirección del túnel** | IP privada dentro del túnel (aquí `10.8.0.0/24`: servidor `10.8.0.1`, cliente `10.8.0.2`) |
| **`AllowedIPs`** | Qué direcciones se pueden **enviar por el túnel hacia ese peer**; y también, qué IP de origen se **aceptan** de él. Es **enrutado y filtro a la vez** |
| **`Endpoint`** | IP y puerto **reales** por los que se llega al peer |
| **`PersistentKeepalive`** | Envía un paquete cada N segundos para mantener el túnel abierto detrás de un NAT |

> 🚨 **La clave privada es secreta**: no la pegues en ninguna conversación, no la subas a GitHub, no la dejes con permisos abiertos. Si se filtra, el túnel deja de ser seguro.

### Esqueleto de los archivos de configuración

Servidor, `/etc/wireguard/wg0.conf` (completa las `???`):

```ini
[Interface]
Address = 10.8.0.1/24
ListenPort = ???
PrivateKey = ???            # la privada DEL SERVIDOR

[Peer]
PublicKey = ???             # la pública DEL CLIENTE
AllowedIPs = 10.8.0.2/32
```

Cliente:

```ini
[Interface]
Address = 10.8.0.2/24
PrivateKey = ???            # la privada DEL CLIENTE

[Peer]
PublicKey = ???             # la pública DEL SERVIDOR
Endpoint = ???:51820
AllowedIPs = 10.8.0.1/32
PersistentKeepalive = 25
```

### Comandos de este laboratorio

| Comando | Qué hace |
| ------- | -------- |
| `sudo apt install wireguard` | Instala WireGuard |
| `umask 077` | Hace que los archivos nuevos que crees **solo** los pueda leer su propietario (importante para claves) |
| `wg genkey \| tee privada.key \| wg pubkey > publica.key` | Genera la clave **privada**, la guarda y calcula su **pública** (`tee` guarda y deja pasar la salida) |
| `sudo wg-quick up wg0` / `down wg0` | **Levanta** / **baja** el túnel a partir de `/etc/wireguard/wg0.conf` |
| `sudo wg show` | Estado del túnel: peers, último *handshake*, datos enviados y recibidos |
| `sudo systemctl enable --now wg-quick@wg0` | Lo arranca **al iniciar** el sistema |
| `ip a show wg0` | La interfaz virtual y su IP |
| `sudo ufw allow from 192.168.56.0/24 to any port 51820 proto udp` | Abre el puerto UDP de WireGuard |
| `sudo tcpdump -ni enp0s8 udp port 51820` | Captura el tráfico **cifrado** del túnel |

---

## 🛠️ Qué debes hacer

### Parte A · Claves

1. Instala WireGuard en **las dos** máquinas.
2. En cada una, con `umask 077` activo, genera su **par de claves** en `/etc/wireguard/`.
3. Apunta (sin pegarlas en ningún sitio público) **la clave pública** de cada una: la necesitarás en la otra.
4. Comprueba los **permisos** de las claves privadas: solo `root` debe poder leerlas.

### Parte B · Montar el túnel

5. Escribe `/etc/wireguard/wg0.conf` en el **servidor** y en el **cliente** con las plantillas. En `Endpoint` pon la IP del servidor.
6. Abre el puerto UDP **51820** en el firewall del **servidor**, solo para tu red de laboratorio.
7. Levanta el túnel en el servidor y en el cliente con `wg-quick up`.
8. Comprueba con `wg show` que aparece un **handshake** reciente y que se envían y reciben datos.
9. Haz `ping` entre las IP del **túnel** (`10.8.0.1` ↔ `10.8.0.2`).

### Parte C · Ver el cifrado

10. En el servidor, captura con `tcpdump` el tráfico **UDP 51820** mientras haces ping por el túnel. ¿Se ve algo legible?
11. Captura ahora en la interfaz **`wg0`** (`tcpdump -ni wg0`) y compara: en `wg0` ves el ping **en claro**; en la interfaz física, **solo UDP cifrado**.
12. Razona con tus palabras: ¿qué vería alguien que capture el tráfico en `enp0s8`?

### Parte D · Acceder a un servicio solo por el túnel

13. En el **servidor**, haz que Nginx (laboratorio 10 del nivel intermedio) o `python3 -m http.server` **escuche solo en la IP del túnel** (`10.8.0.1`).
14. **Bloquea** ese puerto en el firewall para la red `192.168.56.0/24` y **permítelo solo** desde la interfaz `wg0`.
15. Desde el cliente: `curl http://10.8.0.1` **funciona** con el túnel levantado. Bájalo y comprueba que **deja de funcionar**.

### Parte E · Permanencia

16. Habilita `wg-quick@wg0` en el servidor, **reinicia** y comprueba que el túnel se reconstruye.

---

## ✅ Comprobación

| Prueba | Qué debes ver |
| ------ | ------------- |
| `ls -l /etc/wireguard/` | Claves y `wg0.conf` con permisos `-rw-------` |
| `sudo wg show` | `latest handshake` de hace segundos y `transfer` con datos |
| `ping 10.8.0.1` desde el cliente | Responde |
| `tcpdump -ni enp0s8 udp port 51820` | Paquetes UDP **ilegibles** |
| `tcpdump -ni wg0` | Los pings en claro (ICMP) |
| `curl http://10.8.0.1` con túnel | Responde |
| `curl http://10.8.0.1` sin túnel | **Falla** |

---

## 🧩 Pistas

- Si no hay *handshake*, revisa **(1)** que las claves públicas están **cruzadas** (la del otro, no la tuya), **(2)** el `Endpoint`, **(3)** que el puerto UDP está abierto en el servidor.
- WireGuard **no responde** a paquetes de peers desconocidos: si algo está mal, simplemente **no pasa nada**, sin mensaje de error. Es una característica de seguridad, pero hace el diagnóstico más difícil.
- Si copias una clave y falla, comprueba que no has incluido **saltos de línea** ni espacios.
- `AllowedIPs` mal puesto es el fallo más frecuente: si un paquete no coincide con ningún `AllowedIPs`, no sale.
- No cambies el `PrivateKey` de un lado sin actualizar la `PublicKey` en el otro.

---

## 🚀 Retos extra

1. **Túnel completo** (*full tunnel*): haz que **todo** el tráfico del cliente salga por el servidor (`AllowedIPs = 0.0.0.0/0`), activando `ip_forward` y NAT en el servidor, como en el laboratorio 10. Comprueba tu IP de salida antes y después.
2. **Túnel dividido** (*split tunnel*): que solo el tráfico a una **red interna concreta** vaya por el túnel (con un segundo adaptador interno en el servidor).
3. Añade un **tercer peer** (por ejemplo, tu móvil o tu PC real con la app oficial de WireGuard) y compara cómo se configura.
4. Genera una **clave precompartida** (`wg genpsk`) y añádela para una capa adicional de seguridad.
5. Compara **WireGuard, OpenVPN e IPsec** en una tabla (complejidad, rendimiento, soporte, seguridad).

---

## 📝 Qué documentar en tu `Solucion.md`

- Un esquema del túnel con las IP reales y las del túnel.
- Los dos `wg0.conf` **sin las claves privadas** (sustitúyelas por `<CLAVE-PRIVADA>`).
- Las capturas de `tcpdump` y tu interpretación.
- Con tus palabras: ¿por qué una VPN es mejor que abrir el servicio directamente a internet?

---

## 🧠 Lo que has practicado

`VPN` · `WireGuard` · `claves pública y privada` · `peers` · `AllowedIPs` · `túnel cifrado` · `wg show / wg-quick` · `tcpdump` · `acceso interno solo por túnel`
