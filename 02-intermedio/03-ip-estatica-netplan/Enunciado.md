# 🌐 Laboratorio 03 · Dirección IP estática con `netplan`

**Nivel:** 🟡 Intermedio · **Sistema:** 🐧 Linux · **Tiempo estimado:** 60 minutos

---

## 🎯 Escenario

Un servidor debe tener **siempre la misma dirección IP**: si cambia cada vez que reinicia, nadie podría encontrarlo. En un servidor real se configura una IP **estática** (fija).

Además, vas a conectar tu VM a una red por la que **tu propio ordenador pueda hablar con ella**, sin reenvíos de puertos. Esa red será la base de los laboratorios de SSH con claves, firewall y Nginx.

> 💾 Haz un **snapshot** antes de empezar. Un error de red puede dejarte sin acceso remoto (pero siempre tienes la consola de VirtualBox).

---

## 📖 Conceptos nuevos

### IP dinámica frente a IP estática

| Tipo | Quién la decide | Para qué sirve |
| ---- | --------------- | -------------- |
| **Dinámica (DHCP)** | Un servidor DHCP te la presta durante un tiempo | Equipos de usuario |
| **Estática** | Tú la escribes a mano | Servidores, routers, impresoras |

### Máscara en formato CIDR

La máscara se escribe como una barra y un número: `/24`. Indica **cuántos bits** de la IP identifican la **red** y cuántos al **equipo**.

| CIDR | Máscara | Equipos posibles | Ejemplo |
| ---- | ------- | ---------------- | ------- |
| `/24` | `255.255.255.0` | 254 | `192.168.56.0` a `192.168.56.255` |
| `/16` | `255.255.0.0` | 65 534 | `10.1.0.0` a `10.1.255.255` |
| `/30` | `255.255.255.252` | 2 | Enlaces punto a punto |

### Qué necesita una configuración estática

Ya conoces las piezas del laboratorio 07 del nivel 1: **IP + máscara + puerta de enlace + DNS**. Con una red de solo-anfitrión no hay salida a internet, así que ese adaptador **no necesita puerta de enlace** (¡y no debe tenerla!: dos rutas por defecto dan problemas).

### `netplan`: la configuración de red de Ubuntu

Ubuntu configura la red con archivos **YAML** en `/etc/netplan/`. YAML es muy estricto con la **indentación**: se usan **espacios, nunca tabuladores**, y cada nivel se indenta igual.

Estructura (completa las `???`):

```yaml
network:
  version: 2
  ethernets:
    ???:               # nombre de la interfaz
      dhcp4: ???
      addresses:
        - ???
```

Los archivos se leen en orden alfabético y los posteriores completan o sobreescriben a los anteriores. **No toques** el archivo que ya existe (suele llamarse `50-cloud-init.yaml` y puede ser regenerado por `cloud-init`): crea uno **nuevo** para el segundo adaptador.

### Comandos de este laboratorio

| Comando | Qué hace |
| ------- | -------- |
| `ip -br a` / `ip r` | Direcciones y rutas (nivel 1) |
| `resolvectl status` | Servidores DNS en uso |
| `sudo netplan generate` | **Comprueba** que el archivo YAML es válido, sin aplicarlo |
| `sudo netplan try` | **Aplica** la configuración y la **revierte sola** a los 120 s si no la confirmas. Es la forma segura |
| `sudo netplan apply` | Aplica la configuración definitivamente |
| `ls -l /etc/netplan/` | Lista los archivos de netplan |
| `ping -c 2 IP` | Prueba de conectividad |

---

## 🛠️ Qué debes hacer

### Parte A · Añadir el segundo adaptador

1. **Apaga** la VM.
2. En VirtualBox, activa el **Adaptador 2** de la VM y elige **Red solo-anfitrión** (*Host-Only*). Si no existe ninguna, créala desde las preferencias de red de VirtualBox. Deja el Adaptador 1 como NAT.
3. **Enciende** la VM y mira con `ip -br a` qué interfaz nueva aparece y si tiene IP.

### Parte B · IP fija

4. Anota el **nombre** de la interfaz nueva (suele ser `enp0s8`) y la red del anfitrión (por defecto, `192.168.56.0/24`).
5. Crea el archivo `/etc/netplan/60-hostonly.yaml` con la IP fija **`192.168.56.10/24`**, sin DHCP y sin puerta de enlace.
6. Cambia los permisos del archivo a `600` (solo el propietario): `netplan` avisa si es legible por otros.
7. **Valida** el archivo con `netplan generate` y aplícalo de forma segura con `netplan try`.
8. Comprueba que la interfaz tiene la IP correcta y que la NAT sigue funcionando (`ping -c 2 8.8.8.8`).

### Parte C · Probar desde tu ordenador

9. Desde tu ordenador real, haz `ping` a `192.168.56.10`.
10. Conéctate por **SSH** a `192.168.56.10` (sin reenvío de puertos: ya no hace falta).
11. Reinicia la VM y comprueba que la IP **sigue siendo la misma**.

### Parte D · Un fallo a propósito

12. Edita el archivo y **rompe** la indentación (por ejemplo, pon un tabulador o quita dos espacios). Ejecuta `netplan generate` y lee el error. ¿Qué te dice? Arréglalo.

---

## ✅ Comprobación

| Prueba | Qué debes ver |
| ------ | ------------- |
| `ip -br a` | `enp0s8` con `192.168.56.10/24` y la NAT con su IP |
| `ip r` | **Una sola** ruta `default` (la de la NAT) |
| `ping -c 2 8.8.8.8` | Responde (internet sigue funcionando) |
| `ping 192.168.56.10` desde tu equipo | Responde |
| `ssh usuario@192.168.56.10` desde tu equipo | Entras en la VM |
| Tras reiniciar | La misma IP |

---

## 🧩 Pistas

- El nombre de la interfaz va **sin guiones ni comillas**, tal como lo muestra `ip -br a`.
- `addresses:` es una **lista**: cada elemento empieza por `- ` (guion y espacio) y lleva la **máscara en CIDR**.
- Si `netplan try` te dice que el archivo es demasiado permisivo, ejecuta `chmod 600` sobre él.
- Si no ves la red solo-anfitrión desde tu equipo, comprueba en VirtualBox que el adaptador está activo y que su red tiene una IP del anfitrión (`192.168.56.1` suele ser tu ordenador).
- Si `netplan try` no te deja confirmar y se revierte, ¡es justo para lo que sirve! Revisa el archivo y repite.

---

## 🚀 Retos extra

1. Añade una **segunda IP** a la misma interfaz (`192.168.56.11/24`). ¿Para qué sirve?
2. Añade `nameservers` con direcciones DNS al archivo. ¿En qué se nota con `resolvectl status`?
3. Añade una entrada a `/etc/hosts` para que `servidor-lab` resuelva a `192.168.56.10` **solo en la VM**. Y en tu ordenador, ¿cómo lo harías?
4. Investiga cómo se configuraría una **IP estática con puerta de enlace** (`routes:` en netplan) y por qué aquí no la usamos.
5. Averigua por qué `netplan` y `NetworkManager` son dos formas distintas de hacer lo mismo.

---

## 📝 Qué documentar en tu `Solucion.md`

- Tu archivo YAML completo, explicado línea a línea.
- El error que obtuviste al romper la indentación y cómo lo resolviste.
- Un esquema de las dos redes de tu VM y para qué sirve cada una.

---

## 🧠 Lo que has practicado

`IP estática` · `CIDR` · `YAML` · `netplan generate / try / apply` · `red solo-anfitrión` · `varios adaptadores` · `una sola ruta por defecto`
