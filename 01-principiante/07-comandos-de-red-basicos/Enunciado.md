# 🌐 Laboratorio 07 · Comandos de red básicos

**Nivel:** 🟢 Principiante · **Sistema:** 🐧 Linux · **Tiempo estimado:** 60 minutos

---

## 🎯 Escenario

Un usuario te escribe: *"No tengo internet en el servidor"*. Es el aviso más vago del mundo, y el 90 % de los problemas de red se resuelven con un método: **comprobar capa por capa, de dentro hacia fuera**.

En este laboratorio aprenderás ese método sobre tu propia VM, y al final lo practicarás **rompiendo la red a propósito** y reparándola.

---

## 📖 Conceptos nuevos

### Los cinco datos de una configuración de red

Piensa en el sistema de correos:

| Concepto | Qué es | Analogía |
| -------- | ------ | -------- |
| **Dirección IP** | Identifica a tu equipo en la red | Tu dirección postal |
| **Máscara de red** | Indica qué direcciones están en tu "barrio" (tu red local) | El código postal |
| **Puerta de enlace** (*gateway*) | El router por el que sales a otras redes | La oficina de correos del barrio |
| **DNS** | Traduce nombres (`google.com`) a IPs | La agenda de contactos |
| **Puerto** | Número que identifica **qué servicio** dentro de un equipo atiende la conexión | El número de puerta o piso |

Ejemplos de puertos conocidos: `22` (SSH), `80` (web HTTP), `443` (web HTTPS), `53` (DNS).

### Interfaces y la dirección especial `127.0.0.1`

Una **interfaz de red** es la "tarjeta de red" (real o virtual) del equipo. Además de la tuya (`enp0s3` o similar en VirtualBox), existe `lo`, la interfaz **loopback**, con la IP `127.0.0.1`: es el equipo hablando consigo mismo.

### Diagnóstico por capas

Cuando algo falla, avanza en este orden. El **primer paso que falla** te dice dónde está el problema:

1. ¿Está la **interfaz** activa?
2. ¿Tiene **IP**?
3. ¿Llego a la **puerta de enlace**?
4. ¿Llego a **internet por IP**? (`8.8.8.8`)
5. ¿Funciona el **DNS**? (`google.com`)

### Comandos de este laboratorio

| Comando | Qué hace |
| ------- | -------- |
| `ip -br a` | Resumen de interfaces y direcciones IP |
| `ip a` | Detalle completo de interfaces (incluye máscara, en formato `/24`) |
| `ip r` | Muestra la **tabla de rutas**; la línea `default via …` es tu puerta de enlace |
| `ip link set dev INTERFAZ down` / `up` | Apaga o enciende una interfaz |
| `ping -c 3 destino` | Envía 3 paquetes para probar si hay respuesta |
| `ss -tuln` | Lista puertos a la escucha (`t`=TCP, `u`=UDP, `l`=*listening*, `n`=números en vez de nombres) |
| `sudo ss -tulnp` | Lo mismo, añadiendo `p` para ver **qué proceso** usa cada puerto (necesita `sudo`) |
| `resolvectl status` | Muestra los servidores DNS que usa el sistema |
| `dig nombre` | Consulta DNS y muestra la respuesta detallada |
| `traceroute destino` | Muestra los "saltos" (routers) hasta llegar a un destino |
| `curl -I https://dominio` | Pide solo las cabeceras HTTP de una web (`-I`) para comprobar que responde |

> 💡 `dig` y `traceroute` pueden no estar instalados. Instálalos con `apt` (laboratorio 05): los paquetes se llaman `bind9-dnsutils` y `traceroute`.

---

## 🛠️ Qué debes hacer

### Parte A · Conocer tu red

1. Averigua y **anota** en tu `Solucion.md`: nombre de tu interfaz, IP, máscara, puerta de enlace y servidor(es) DNS.
2. Comprueba cuántas interfaces tiene tu VM y para qué sirve cada una.
3. Haz `ping` a tu **propia IP**, a `127.0.0.1`, a la **puerta de enlace** y a `8.8.8.8`.
4. Haz `ping` a `google.com` y observa la IP a la que se resuelve.
5. Consulta el DNS con `dig` y localiza en la respuesta la IP y el tiempo de consulta.
6. Con `traceroute 8.8.8.8` cuenta cuántos saltos hay y cuáles no responden.

### Parte B · Qué hay escuchando

7. Lista los puertos que tiene abiertos tu VM, y **qué proceso** escucha en cada uno. ¿Hay alguno que no esperabas?

### Parte C · Break/fix (mini)

> 💾 **Haz un snapshot ahora.** Usa la **consola de VirtualBox** (no SSH), porque vas a cortar la red.

8. **Rompe la red**: apaga la interfaz con `ip link set … down`.
9. **Diagnostica** con el método de las cinco capas. ¿En qué paso falla por primera vez?
10. **Repárala** y verifica que vuelve a funcionar.
11. Segunda avería (opcional si te sientes cómodo): sigue con la red activa, pero **borra la puerta de enlace** (pista: `sudo ip route del default`). Haz el diagnóstico otra vez. ¿Qué capa falla ahora? ¿Qué sigue funcionando? (Si no lo arreglas, **reinicia la VM** o restaura el snapshot.)

---

## ✅ Comprobación

| Prueba | Qué debes ver |
| ------ | ------------- |
| `ping -c 2 127.0.0.1` | Responde siempre (si no, algo muy grave pasa en el sistema) |
| `ping -c 2 <puerta de enlace>` | Responde: llegas a tu router |
| `ping -c 2 8.8.8.8` | Responde: tienes salida a internet |
| `ping -c 2 google.com` | Responde: el DNS funciona |
| `ss -tuln` | Al menos algún puerto a la escucha; reconoces los que ves |
| Tras la avería | Con la interfaz apagada, **ningún** ping externo funciona; tras `up`, vuelven todos |

---

## 🧩 Pistas

- En la salida de `ip a`, `inet 10.0.2.15/24` significa IP `10.0.2.15` con máscara `/24` (= `255.255.255.0`).
- Si el `ping` a `8.8.8.8` funciona pero el de `google.com` no, el problema es el **DNS**, no la conexión.
- Si el `ping` a la puerta de enlace falla, el problema está **entre tu máquina y el router**.
- Para interrumpir un `ping` sin `-c`, pulsa `Ctrl+C`.
- Las VMs en modo NAT de VirtualBox suelen tener la IP `10.0.2.15` y la puerta de enlace `10.0.2.2`: es normal.

---

## 🚀 Retos extra

1. Averigua tu **IP pública** con `curl ifconfig.me`. ¿Por qué es distinta de la IP de la VM?
2. Usa `ip -s link` para ver **estadísticas** de paquetes enviados, recibidos y errores.
3. Con `ss -s` obtén un **resumen** de conexiones. ¿Cuántas están establecidas?
4. Levanta un servidor web mínimo con `python3 -m http.server 8000` en una terminal, y desde otra comprueba con `ss` y `curl` que responde. ¿Qué puerto aparece y qué proceso lo usa?
5. Investiga la diferencia entre **TCP** y **UDP**. ¿Cuál usa `ping`? (Pista: no usa ninguno de los dos.)

---

## 📝 Qué documentar en tu `Solucion.md`

- La tabla con tu configuración de red (interfaz, IP, máscara, gateway, DNS).
- El resultado de cada prueba de conectividad.
- Tu **método de diagnóstico** por capas, con las capturas de la avería.
- Qué puertos están a la escucha y qué los usa.

---

## 🧠 Lo que has practicado

`IP / máscara / gateway / DNS / puertos` · `ip a / ip r / ip link` · `ping` · `ss` · `dig` · `traceroute` · `curl` · `diagnóstico por capas` · `break/fix`
