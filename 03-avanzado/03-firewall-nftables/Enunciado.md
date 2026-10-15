# 🧱 Laboratorio 03 · Firewall avanzado con `nftables`

**Nivel:** 🟠 Avanzado · **Sistema:** 🐧 Linux · **Tiempo estimado:** 90 minutos

---

## 🎯 Escenario

En el nivel intermedio usaste `ufw`, una capa **sencilla** sobre el firewall de Linux. Está bien para empezar, pero en una infraestructura seria necesitas **más control**: reglas con límites de velocidad, registros, conjuntos de IP y, más adelante, enrutado y NAT (laboratorio 10).

Tu misión es **escribir tu propio firewall desde cero con `nftables`**, la herramienta nativa y moderna de Linux, y entender qué hay "debajo" de `ufw`.

> ⚠️ **Riesgo de quedarte sin acceso.** Haz **snapshot**, ten la **consola de VirtualBox** abierta y usa el truco de seguridad que se explica abajo (un "botón de pánico" automático).

---

## 📖 Conceptos nuevos

### El motor de filtrado de Linux

El kernel de Linux filtra el tráfico con **`netfilter`**. Se configura con:

| Herramienta | Qué es |
| ----------- | ------ |
| `iptables` | La herramienta **clásica** (todavía muy extendida) |
| **`nftables`** (`nft`) | La **sucesora**, más limpia y potente |
| `ufw` | Una interfaz **simplificada** que genera reglas por debajo |

### Tablas, cadenas y reglas

`nftables` se organiza en tres niveles:

| Elemento | Qué es | Analogía |
| -------- | ------ | -------- |
| **Tabla** | Contenedor de cadenas, asociado a una familia (`inet` = IPv4 e IPv6) | Un archivador |
| **Cadena** | Lista ordenada de reglas, enganchada a un **punto del recorrido del paquete** | Una carpeta con instrucciones |
| **Regla** | Condición + acción (`accept`, `drop`, `reject`...) | Una instrucción |

### Los tres "puntos de control" (*hooks*)

| Hook | Cuándo actúa |
| ---- | ------------ |
| `input` | Paquetes que **entran al propio servidor** |
| `forward` | Paquetes que **pasan de largo** (cuando hace de router; laboratorio 10) |
| `output` | Paquetes que **salen del servidor** |

### Estado de la conexión (*stateful*)

El firewall recuerda las conexiones. Una regla clave es dejar pasar el tráfico de **respuesta** a conexiones que ya has permitido: `ct state established,related accept`. Sin ella, no podrías ni recibir la respuesta de un `ping` que has enviado tú.

### `drop` frente a `reject`

- `drop`: descarta el paquete **sin contestar** (el atacante no sabe si hay algo).
- `reject`: descarta y **avisa** al origen (más amable; útil en redes internas).

### Esqueleto del archivo de configuración

Completa los huecos:

```nft
#!/usr/sbin/nft -f
flush ruleset

table inet filtro {
    chain input {
        type filter hook input priority 0; policy ???;

        ct state ??? accept          # respuestas a conexiones ya establecidas
        iif "lo" accept              # tráfico interno de la propia máquina
        ???                          # ping (ICMP), con límite de velocidad
        ???                          # SSH solo desde 192.168.56.0/24
        ???                          # registro de lo que se descarta
    }
}
```

### Comandos de este laboratorio

| Comando | Qué hace |
| ------- | -------- |
| `sudo nft list ruleset` | Muestra **todas** las reglas activas |
| `sudo nft -a list ruleset` | Igual, con el **identificador** (*handle*) de cada regla |
| `sudo nft -c -f archivo` | **Comprueba** la sintaxis de un archivo sin aplicarlo (`-c` = *check*) |
| `sudo nft -f archivo` | Carga un archivo de reglas |
| `sudo nft delete rule inet filtro input handle N` | Borra una regla concreta |
| `sudo systemctl enable --now nftables` | Aplica `/etc/nftables.conf` al arrancar |
| `sudo ufw disable` | Desactiva `ufw` (para que no se mezclen las dos herramientas) |
| `sudo systemd-run --on-active=120 --unit=panico nft flush ruleset` | **Botón de pánico**: dentro de 120 s **borra todas las reglas** automáticamente |
| `sudo systemctl stop panico.timer` | **Cancela** el botón de pánico (cuando ya has comprobado que todo va bien) |
| `sudo journalctl -k \| grep "nft-"` | Mira los paquetes descartados que has registrado |

> 🛟 **El botón de pánico** es la red de seguridad: antes de cargar unas reglas que pueden dejarte fuera, programas que, a los 2 minutos, se vacíe el firewall. Si todo va bien, cancelas el temporizador. Si te bloqueas, esperas dos minutos y recuperas el acceso.

---

## 🛠️ Qué debes hacer

### Parte A · Preparación

1. Anota las reglas actuales de `ufw` y **desactívalo** (`sudo ufw disable`).
2. Mira el estado actual de `nftables` (`nft list ruleset`) para ver qué hay.
3. Haz una copia de seguridad de `/etc/nftables.conf` (`cp ... .bak`).

### Parte B · Escribir tu firewall

4. Edita `/etc/nftables.conf` con **tu propia política**:
   - Política por defecto de `input`: **descartar**.
   - Permitir las respuestas a conexiones establecidas y el tráfico de `lo`.
   - Permitir **ping**, pero limitado a unos pocos por segundo (`limit rate`).
   - Permitir **SSH solo desde `192.168.56.0/24`**.
   - Permitir **HTTP (80)** solo desde tu ordenador (`192.168.56.1`).
   - **Registrar** (`log prefix "nft-drop: "`) lo que se descarte, con un límite de velocidad para no inundar el log.
5. **Comprueba la sintaxis** con `nft -c -f`.
6. **Activa el botón de pánico** (120 segundos).
7. Carga las reglas con `nft -f` y verifica que **sigues conectado**. Abre una **segunda** sesión SSH para comprobarlo.
8. Si todo funciona, **cancela** el botón de pánico.

### Parte C · Probar

9. Desde tu ordenador, haz `ping` a la VM y comprueba que responde.
10. Lanza un servidor de prueba en la VM (`python3 -m http.server 8080`) e intenta acceder: debe fallar y quedar **registrado**.
11. Mira el registro y localiza: IP de origen, puerto de destino y protocolo.
12. Abre el puerto 8080 **añadiendo una regla** con `nft add rule` (sin editar el archivo), comprueba que funciona y **bórrala** con su `handle`.

### Parte D · Hacerlo permanente

13. Habilita el servicio `nftables` para que cargue tus reglas al arrancar.
14. **Reinicia** la VM y comprueba que las reglas siguen aplicadas.

---

## ✅ Comprobación

| Prueba | Qué debes ver |
| ------ | ------------- |
| `sudo nft list ruleset` | Tu tabla `inet filtro`, con política `drop` en `input` |
| SSH desde tu ordenador | Funciona |
| `ping` a la VM | Responde (pero con el límite aplicado) |
| `curl` al puerto 8080 sin regla | Falla |
| `sudo journalctl -k \| grep "nft-drop"` | Líneas con `SRC=`, `DPT=8080`, `PROTO=TCP` |
| Tras `nft add rule` | El puerto 8080 responde; tras borrar la regla, vuelve a fallar |
| Tras reiniciar | `nft list ruleset` sigue mostrando tus reglas |

---

## 🧩 Pistas

- Si `nft -c` falla, **lee el número de línea** que indica: casi siempre es un `;` o una llave mal puesta.
- Para limitar el ping: `icmp type echo-request limit rate 5/second accept` (¿qué pasa con los que superen el límite? Llegan a la regla de registro/descarte).
- Para filtrar por origen: `ip saddr 192.168.56.0/24`; por puerto: `tcp dport 22`.
- Cada regla se evalúa **en orden**: la primera que coincide decide, así que las que descartan van **después** de las que permiten.
- Para los registros de descarte con límite: `limit rate 5/minute log prefix "nft-drop: "`.
- Si te quedas fuera, espera a que salte el botón de pánico, o usa la consola de VirtualBox.

---

## 🚀 Retos extra

1. Usa un **conjunto** (`set`) de IP permitidas en vez de repetir reglas, y añade una IP al conjunto **en caliente**.
2. Añade **contadores** (`counter`) a tus reglas y mide cuántos paquetes casa cada una.
3. Escribe una regla que limite las **nuevas conexiones SSH** a 5 por minuto y compárala con `ufw limit` del nivel intermedio.
4. Mira qué reglas ha generado `ufw` antes (`sudo nft list ruleset` con `ufw` activo) y compáralas con las tuyas.
5. Investiga cómo se bloquearía el **tráfico saliente** hacia un destino concreto con la cadena `output`.

---

## 📝 Qué documentar en tu `Solucion.md`

- Tu archivo `/etc/nftables.conf` completo, explicado línea a línea.
- La salida del registro con un paquete descartado, interpretada.
- Qué te aporta `nftables` frente a `ufw` y cuándo usarías cada uno.
- Cómo usaste el **botón de pánico** y por qué es buena práctica.

---

## 🧠 Lo que has practicado

`netfilter` · `nftables` · `tablas, cadenas y reglas` · `hooks input / forward / output` · `ct state` · `limit rate` · `drop frente a reject` · `registro de paquetes` · `red de seguridad ante bloqueos`
