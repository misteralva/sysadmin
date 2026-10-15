# ⚖️ Laboratorio 07 · Alta disponibilidad: HAProxy y keepalived

**Nivel:** 🔴 Experto · **Sistema:** 🐧 Linux · **Tiempo estimado:** 120 minutos

---

## 🎯 Escenario

La intranet de la empresa corre en **un solo servidor**. Si se estropea, se reinicia por una actualización o hay que hacerle mantenimiento, **la intranet cae**. Un servidor único es un **punto único de fallo** (*SPOF*, *Single Point of Failure*).

Tu misión es diseñar y montar una intranet **sin punto único de fallo**: dos servidores web que **reparten la carga** y una **dirección IP virtual** que se mueve sola al servidor que siga vivo cuando el otro falla. Tendrás que **demostrar el fallo** apagando cosas.

> 💾 **Snapshot** de las dos VMs. Usarás las dos máquinas a la vez: `ubuntu-server-lab` (`192.168.56.10`) y `ubuntu-cliente` (`192.168.56.11`). Las llamaremos **nodo 1** y **nodo 2**. La **IP virtual** será `192.168.56.100`.

---

## 📖 Conceptos nuevos

### Disponibilidad y "los nueves"

La **disponibilidad** es el porcentaje de tiempo que un servicio funciona:

| Disponibilidad | Caída permitida al año | Se le llama |
| -------------- | ---------------------- | ----------- |
| 99 % | ~3,65 días | "dos nueves" |
| 99,9 % | ~8,76 horas | "tres nueves" |
| 99,99 % | ~52,6 minutos | "cuatro nueves" |
| 99,999 % | ~5,3 minutos | "cinco nueves" |

Cada "nueve" extra **cuesta mucho más**: redundancia, automatización y vigilancia.

### Dos problemas, dos herramientas

| Problema | Herramienta | Qué hace |
| -------- | ----------- | -------- |
| **Repartir** peticiones entre varios servidores y **dejar de usar** los que fallan | **HAProxy** (balanceador de carga) | Recibe las peticiones y las **reenvía** a los servidores "sanos" |
| Que **el propio balanceador** no sea un SPOF | **keepalived** (con el protocolo **VRRP**) | Mantiene una **IP virtual** que vive en **un nodo a la vez**; si ese nodo cae, **pasa al otro** en segundos |

> **Analogía:** HAProxy es la **recepcionista** que reparte las visitas entre varios despachos. keepalived es el sistema que, si la recepcionista se pone mala, **hace que su compañera ocupe el mostrador** y conteste al mismo número de teléfono.

### VRRP y la IP virtual

| Concepto | Qué es |
| -------- | ------ |
| **VIP** (*Virtual IP*) | Una IP **que no pertenece a una máquina concreta**: la "tiene" el nodo **MASTER** |
| **MASTER / BACKUP** | El nodo que **sostiene** la VIP / el que espera para sustituirlo |
| **`priority`** | Quien tiene **mayor prioridad** es el MASTER |
| **`virtual_router_id`** | Número (1-255) que identifica el **grupo VRRP**; debe **coincidir** en los dos nodos |
| **Anuncios** (*adverts*) | El MASTER **avisa** cada segundo de que sigue vivo; si el BACKUP deja de oírlo, **toma** la VIP |
| **Split-brain** | Los dos nodos creen que son MASTER (la red entre ellos falla): ambos sostienen la VIP. **Es el peor escenario**; se evita con buen diseño de red |

### Algoritmos de balanceo (HAProxy)

| Algoritmo | Cómo reparte |
| --------- | ------------ |
| `roundrobin` | Uno a uno, **por turnos** |
| `leastconn` | Al servidor con **menos conexiones** activas |
| `source` | Según la **IP del cliente** (el mismo cliente va al mismo servidor) |

### Comprobaciones de salud (*health checks*)

HAProxy **sondea** a los servidores (`check` en la configuración): si uno no responde, lo **saca del reparto** hasta que se recupere. Así los usuarios no ven errores.

### Esqueletos de configuración

**HAProxy** (`/etc/haproxy/haproxy.cfg`, añade al final; completa las `???`):

```
frontend intranet
    bind ???:80
    default_backend webs

backend webs
    balance roundrobin
    option httpchk GET /
    server web1 192.168.56.10:8080 check
    server web2 192.168.56.11:8080 ???
```

**keepalived** (`/etc/keepalived/keepalived.conf`, un nodo MASTER y otro BACKUP):

```
vrrp_instance VI_1 {
    state ???                     # MASTER en un nodo, BACKUP en el otro
    interface enp0s8
    virtual_router_id 51
    priority ???                  # p. ej. 150 el MASTER, 100 el BACKUP
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass ???             # igual en ambos
    }
    virtual_ipaddress {
        192.168.56.100/24
    }
}
```

### Detalle importante: *bind* a una IP que aún no tienes

El nodo BACKUP **no tiene la VIP** hasta que toma el relevo, así que HAProxy no podría escuchar en ella. Solución: permitir `bind` a IP no locales con el parámetro del kernel `net.ipv4.ip_nonlocal_bind=1`.

### Comandos de este laboratorio

| Comando | Qué hace |
| ------- | -------- |
| `sudo apt install haproxy keepalived nginx` | Instala los componentes |
| `sudo haproxy -c -f /etc/haproxy/haproxy.cfg` | **Valida** la configuración de HAProxy |
| `sudo systemctl restart haproxy keepalived` | Reinicia los servicios |
| `ip -br a show enp0s8` | Muestra **quién tiene la VIP** en cada momento |
| `sudo journalctl -u keepalived -f` | Sigue los cambios de estado (MASTER/BACKUP) |
| `curl -s http://192.168.56.100` | Pide la web **a través de la VIP** |
| `for i in $(seq 1 6); do curl -s http://192.168.56.100; done` | Hace 6 peticiones seguidas para ver el **reparto** |
| `echo "show stat" \| sudo socat stdio /run/haproxy/admin.sock` | Estadísticas de HAProxy (requiere `socat`) |
| `sudo sysctl -w net.ipv4.ip_nonlocal_bind=1` | Permite escuchar en IP que no tienes todavía |

---

## 🛠️ Qué debes hacer

### Parte A · Los dos servidores web

1. En **cada nodo**, configura **Nginx escuchando en el puerto 8080** (no en el 80: ese será el de HAProxy), con una página que muestre **qué nodo** es ("Web 1" / "Web 2").
2. Comprueba cada una por separado con `curl`.

### Parte B · El balanceador

3. En **cada nodo**, instala HAProxy y configúralo para **balancear entre los dos Nginx** (puerto 80) con `roundrobin` y **comprobaciones de salud**.
4. **Valida** la configuración y comprueba, con la bucle `for`, que las peticiones **alternan** entre "Web 1" y "Web 2".

### Parte C · La IP virtual

5. Activa `ip_nonlocal_bind` y haz que sea **permanente**.
6. Configura keepalived: **nodo 1 como MASTER** (prioridad mayor) y **nodo 2 como BACKUP**. Mismo `virtual_router_id` y contraseña.
7. Abre en el firewall lo necesario para **VRRP** (protocolo IP 112) entre los nodos, y el puerto 80 y 8080 para tu red. (Investiga cómo se permite el protocolo 112 en `ufw`/`nftables`.)
8. Comprueba **quién tiene la VIP** (`ip -br a`) y que `curl http://192.168.56.100` funciona.

### Parte D · Demostrar la alta disponibilidad (el "juego de la guerra")

Haz cada prueba y **cronometra** lo que tarda el servicio en recuperarse. Mantén un `curl` en bucle (con `sleep 1`) desde tu anfitrión o desde otro equipo.

9. **Fallo de un servidor web:** para Nginx en el nodo 2. ¿Siguen respondiendo los clientes? ¿Qué hizo HAProxy? Vuélvelo a arrancar y observa cuándo vuelve al reparto.
10. **Fallo del balanceador:** para HAProxy en el nodo MASTER. ¿Qué pasa con la VIP? (Ahora keepalived no comprueba HAProxy: ¿es un problema? Es una carencia que resolverás en el reto 1.)
11. **Fallo total de un nodo:** **apaga** el nodo MASTER con `sudo poweroff`. Comprueba **cuánto tarda** en moverse la VIP y que el servicio **sigue**.
12. **Recuperación:** vuelve a encender el nodo. ¿Recupera la VIP? ¿Debería? (Investiga `nopreempt`.)

---

## ✅ Comprobación

| Prueba | Qué debes ver |
| ------ | ------------- |
| `sudo haproxy -c -f /etc/haproxy/haproxy.cfg` | `Configuration file is valid` |
| Bucle de 6 `curl` a la VIP | Respuestas **alternadas** "Web 1" / "Web 2" |
| `ip -br a show enp0s8` | La VIP `192.168.56.100` **solo en el nodo MASTER** |
| Parar un Nginx | El servicio **sigue**, y solo responde el otro |
| Apagar el MASTER | La VIP **aparece en el BACKUP** en pocos segundos y el servicio sigue |
| Tu tabla de tiempos | Con **tiempo de recuperación medido** para cada fallo |

---

## 🧩 Pistas

- Si ambos nodos tienen la VIP a la vez, es **split-brain**: revisa que `virtual_router_id` coincide, que el protocolo VRRP **no está bloqueado** por el firewall y que ambos usan la misma `interface`.
- Si HAProxy no arranca en el BACKUP, es casi seguro `ip_nonlocal_bind`.
- Durante el cambio de VIP puede haber **uno o dos segundos** con errores: es normal y es parte de lo que mides.
- Los clientes de tu red de laboratorio aprenden la nueva MAC de la VIP gracias a un mensaje que keepalived envía al cambiar (*gratuitous ARP*). Si el cambio "no llega", repite el `curl` unos segundos.
- Para ver qué hace HAProxy: `sudo journalctl -u haproxy -f`.

---

## 🚀 Retos extra

1. **Que keepalived vigile HAProxy:** añade un `vrrp_script` con `chk_haproxy` (`killall -0 haproxy` o `pidof haproxy`), con `track_script`, para que, si HAProxy cae, **la VIP se mueva** al otro nodo (repite la prueba 10).
2. Activa la **página de estadísticas** de HAProxy y obsérvala mientras provocas fallos.
3. Añade **persistencia de sesión** (cookie o `source`) y razona **cuándo** hace falta.
4. Calcula la **disponibilidad teórica** de tu sistema: si cada servidor tiene un 99 % de disponibilidad, ¿cuál es la del conjunto en paralelo? (Pista: la probabilidad de que **fallen los dos** a la vez.)
5. Investiga qué sucede con los **datos** (una base de datos, archivos subidos): **balancear web es fácil, replicar datos no**. ¿Qué soluciones existen (replicación, almacenamiento compartido)?

---

## 📝 Qué documentar en tu `Solucion.md`

- Un **diagrama** con VIP, nodos, HAProxy y Nginx.
- Tus configuraciones, explicadas.
- Una **tabla de fallos**: qué falló, qué pasó, cuánto tardó la recuperación, si el usuario lo notó y qué mejorarías.
- Con tus palabras: **SPOF**, y **qué otros puntos únicos de fallo** quedan en tu diseño (¿la red? ¿el switch? ¿la electricidad? ¿tú?).

---

## 🧠 Lo que has practicado

`alta disponibilidad` · `SPOF` · `balanceo de carga` · `health checks` · `HAProxy` · `VRRP y keepalived` · `IP virtual` · `failover` · `split-brain` · `disponibilidad y los nueves`
