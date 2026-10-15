# 👁️ Laboratorio 06 · Detección de intrusiones con Suricata

**Nivel:** 🔴 Experto · **Sistema:** 🐧 Linux · **Tiempo estimado:** 120 minutos

---

## 🎯 Escenario

El firewall decide **qué entra y qué no**, pero no mira **qué contiene** lo que deja pasar. Si alguien está **explorando** tu red buscando puertos abiertos, o un equipo infectado envía tráfico sospechoso hacia fuera, el firewall puede dejarlo pasar sin enterarse.

Tu misión es instalar un **IDS** (*Intrusion Detection System*): un "vigilante de seguridad" que **observa el tráfico de red** y **avisa** cuando reconoce un patrón sospechoso. Usarás **Suricata**, una herramienta profesional y de código abierto.

> 💾 **Snapshot** de las dos VMs. **Sensor (IDS):** `ubuntu-server-lab` (`192.168.56.10`). **Fuente de tráfico de prueba:** `ubuntu-cliente` (`192.168.56.11`), **solo contra tus propias máquinas**. No escanees ni pruebes nada contra equipos que no sean tuyos.

---

## 📖 Conceptos nuevos

### IDS frente a IPS frente a firewall

| Herramienta | Qué hace | Analogía |
| ----------- | -------- | -------- |
| **Firewall** | **Permite o bloquea** por IP, puerto y protocolo | El **portero** que mira la lista de invitados |
| **IDS** | **Observa y avisa** (no bloquea) | La **cámara de seguridad** con alarma |
| **IPS** | Observa **y bloquea** automáticamente | La cámara conectada a una **puerta que se cierra sola** |

Se complementan: el firewall reduce el ruido, el IDS ve lo que el firewall deja pasar.

### Detección por firmas

Suricata compara cada paquete con una lista de **reglas** (firmas) que describen patrones conocidos de actividad sospechosa. Si coincide, genera una **alerta**.

### Anatomía de una regla

```
alert tcp any any -> $HOME_NET 22 (msg:"Intento de conexion SSH"; flow:to_server; sid:1000001; rev:1;)
```

| Parte | Significado |
| ----- | ----------- |
| `alert` | **Acción**: generar una alerta (`drop` bloquearía en modo IPS) |
| `tcp` | Protocolo |
| `any any` | IP y puerto de **origen** (cualquiera) |
| `->` | Dirección del tráfico |
| `$HOME_NET 22` | Destino: **tu red** y el **puerto 22** |
| `msg:"..."` | Texto de la alerta |
| `flow:to_server` | Solo paquetes **hacia el servidor** |
| `sid:1000001` | **Identificador único** (usa números **a partir de 1 000 000** para tus reglas) |
| `rev:1` | Revisión de la regla |

Más opciones útiles: `content:"texto";` (busca un texto en el contenido), `http.uri; content:"/admin";` (en la URL de una petición HTTP), `threshold:...;` (limita la frecuencia de alertas).

### Dónde mira y qué guarda

| Archivo / elemento | Qué es |
| ------------------ | ------ |
| `/etc/suricata/suricata.yaml` | Configuración principal: `HOME_NET`, la interfaz a vigilar (`af-packet`)... |
| `/var/lib/suricata/rules/suricata.rules` | Reglas descargadas de **Emerging Threats Open** (gratuitas) |
| `/var/lib/suricata/rules/local.rules` o tu propio archivo | **Tus reglas** (el nombre lo referencias en `rule-files:`) |
| `/var/log/suricata/fast.log` | Una línea por alerta, fácil de leer |
| `/var/log/suricata/eve.json` | **Todos** los eventos en **JSON** (para analizar con herramientas) |

### Comandos de este laboratorio

| Comando | Qué hace |
| ------- | -------- |
| `sudo apt install suricata jq nmap` | Instala el IDS, `jq` (para leer JSON) y `nmap` (en el cliente, como generador de tráfico de prueba) |
| `sudo suricata-update` | Descarga y actualiza las reglas oficiales |
| `sudo suricata -T -c /etc/suricata/suricata.yaml` | **Valida** la configuración y las reglas (`-T` = *test*) |
| `sudo systemctl restart suricata` | Reinicia el IDS |
| `sudo tail -f /var/log/suricata/fast.log` | Sigue las alertas en vivo |
| `sudo jq 'select(.event_type=="alert")' /var/log/suricata/eve.json` | Filtra solo los eventos de alerta |
| `sudo jq -r 'select(.event_type=="alert") \| .alert.signature' /var/log/suricata/eve.json \| sort \| uniq -c \| sort -rn` | **Ranking** de alertas por tipo |
| `nmap -sS IP` (en el cliente) | Escaneo de puertos **hacia tu propio servidor** |
| `curl http://IP/admin` | Petición web de prueba |

---

## 🛠️ Qué debes hacer

### Parte A · Instalar y configurar

1. Instala Suricata en el servidor y localiza su configuración. Ajusta en `suricata.yaml` **la interfaz** que debe vigilar (`enp0s8`, la red de laboratorio) y `HOME_NET` (`192.168.56.0/24`).
2. Descarga las reglas con `suricata-update` y **cuenta** cuántas hay (`wc -l`).
3. **Valida** la configuración con `suricata -T` y arranca el servicio. Comprueba que `fast.log` y `eve.json` se crean.

### Parte B · Tus propias reglas

4. Crea un archivo de reglas propio y regístralo en `suricata.yaml`.
5. Escribe una regla que **alerte de cada `ping`** (ICMP *echo request*) hacia tu red, con un mensaje claro. Pista: `alert icmp any any -> $HOME_NET any (itype:8; ...)`.
6. Escribe una regla que **alerte cuando alguien pide la URL `/admin`** en tu servidor web (Nginx del nivel intermedio).
7. Escribe una regla que **alerte cuando alguien se conecte al puerto 22** desde fuera de tu red de confianza (por ejemplo, desde una IP que no sea tu anfitrión).
8. **Valida**, recarga y **prueba cada una** desde `ubuntu-cliente`. Comprueba las alertas en `fast.log`.

### Parte C · Detectar un escaneo

9. Desde `ubuntu-cliente`, lanza un **escaneo de puertos básico** hacia el servidor (`nmap -sS`, **solo tu servidor**).
10. En el servidor, localiza **qué alertas** ha generado Suricata (de las reglas oficiales y/o de las tuyas), con qué **prioridad** y desde qué IP.
11. Genera un **ranking** de alertas por firma con `jq`.

### Parte D · Análisis y respuesta

12. Redacta una **ficha de alerta** (plantilla abajo) con **una** alerta real de tu laboratorio.
13. Cruza la alerta con otros registros: ¿qué ves en `auth.log`, en el log de Nginx y en tu **servidor de logs** (laboratorio 04)? Ten en cuenta la **línea temporal**.
14. Decide la **respuesta**: ¿bloquearías la IP (laboratorio 03, `fail2ban` del avanzado)? ¿Cuándo sería **falso positivo**?

---

## ✅ Comprobación

| Prueba | Qué debes ver |
| ------ | ------------- |
| `sudo suricata -T -c /etc/suricata/suricata.yaml` | `Configuration provided was successfully loaded` |
| `systemctl is-active suricata` | `active` |
| Un `ping` desde el cliente | Tu alerta ICMP en `fast.log` |
| `curl http://IP/admin` | Tu alerta de `/admin` |
| Escaneo con `nmap` | Varias alertas con la IP del cliente como origen |
| Ranking con `jq` | Las firmas ordenadas por número de apariciones |
| Tu ficha de alerta | Completa y coherente con los logs |

---

## 🧩 Pistas

- Si Suricata no genera alertas, **comprueba la interfaz**: tiene que ser por la que **pasa el tráfico** de prueba.
- En el **`fast.log`** cada línea trae: fecha, `[**] [sid] mensaje [**]`, prioridad e IP origen → destino. Aprende a leerla.
- Si tu regla no salta, valida con `suricata -T` y comprueba el `sid`, las **comas, puntos y comas y paréntesis**.
- Para el tráfico HTTP, Suricata solo ve el contenido **si no va cifrado**: con HTTPS (laboratorio 05) **no podrá ver la URL**. Es una limitación real de los IDS de red.
- Si el IDS consume mucha CPU, es normal en una VM pequeña con todas las reglas cargadas.
- Un **falso positivo** es una alerta de algo que **no** era un ataque; un **falso negativo**, un ataque que **no** se detectó.

---

## 🚀 Retos extra

1. Cambia la regla de `/admin` para que **solo** salte si hay **más de 3 peticiones en 60 segundos** (`threshold` o `detection_filter`).
2. Investiga el **modo IPS** (`nfqueue`) y qué cambia (y qué riesgos tiene) al pasar de `alert` a `drop`.
3. Escribe un script que lea `eve.json` y envíe a tu **servidor de logs** un resumen diario de alertas.
4. Compara **Suricata, Snort y Zeek**: ¿en qué se diferencian (firmas, análisis de comportamiento)?
5. Piensa: ¿**qué no ve** un IDS de red? (Pista: tráfico cifrado, ataques en el propio equipo, usuarios internos con credenciales válidas...) Relaciónalo con **defensa en profundidad**.

---

## 📝 Qué documentar en tu `Solucion.md`

**Plantilla de ficha de alerta:**

| Campo | Contenido |
| ----- | --------- |
| Fecha y hora | |
| Firma (sid y mensaje) | |
| IP origen → destino | |
| Evidencia (log, línea) | |
| Otros registros que lo corroboran | |
| ¿Falso positivo? Razonamiento | |
| Acción recomendada | |

Además: tus reglas, **una a una, explicadas**, y una reflexión sobre **qué detecta y qué no** un IDS.

---

## 🧠 Lo que has practicado

`IDS frente a IPS frente a firewall` · `Suricata` · `reglas y firmas` · `suricata-update` · `fast.log y eve.json` · `jq` · `falsos positivos` · `análisis de alertas` · `defensa en profundidad`
