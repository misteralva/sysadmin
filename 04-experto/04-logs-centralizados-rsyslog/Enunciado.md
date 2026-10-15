# 🗄️ Laboratorio 04 · Logs centralizados con `rsyslog`

**Nivel:** 🔴 Experto · **Sistema:** 🐧 Linux · **Tiempo estimado:** 100 minutos

---

## 🎯 Escenario

Con 20 servidores, revisar los logs **uno por uno** es inviable. Peor aún: si un atacante entra en un servidor, **lo primero que hace es borrar sus logs**. Si los logs solo viven en la máquina comprometida, **desaparece la evidencia**.

La solución es **centralizarlos**: cada servidor envía sus registros **en tiempo real** a un servidor aparte, donde se guardan y se pueden analizar de forma conjunta. Tu misión es montar un **servidor de logs** y conectarle a un cliente.

> 💾 **Snapshot** de las dos VMs. **Servidor de logs:** `ubuntu-server-lab` (`192.168.56.10`). **Cliente:** `ubuntu-cliente` (`192.168.56.11`).

---

## 📖 Conceptos nuevos

### Por qué centralizar

| Motivo | Detalle |
| ------ | ------- |
| **Búsqueda conjunta** | Buscar "intentos fallidos" en **todos** los servidores a la vez |
| **Integridad** | Si el cliente es comprometido, los logs **ya están a salvo** en otro sitio |
| **Correlación** | Reconstruir un incidente que afectó a varias máquinas |
| **Cumplimiento** | Muchas normativas exigen conservar logs |

### `rsyslog`

**`rsyslog`** es el servicio clásico de `syslog` en Linux. Puede **recibir** logs por red (módulos `imtcp`/`imudp`) y **enviarlos** a otro servidor. En un cliente, una línea como esta reenvía **todo** (`*.*`) a un servidor:

```
*.* @@192.168.56.10:514
```

(`@@` = por **TCP**, más fiable; `@` = por UDP.)

### Plantillas: un archivo por equipo

Un servidor puede **separar los logs de cada cliente** con una **plantilla**. Esqueleto del servidor (completa las `???`):

```
module(load="imtcp")
input(type="imtcp" port="???")

template(name="PorHost" type="string"
         string="/var/log/remoto/%HOSTNAME%/%PROGRAMNAME%.log")

if $fromhost-ip startswith '192.168.56.' then {
    action(type="omfile" dynaFile="PorHost")
    stop
}
```

### La hora importa

Para **correlacionar** eventos entre máquinas, **todos los relojes deben estar sincronizados**. Una diferencia de segundos ya complica una investigación. Se usa **NTP** (con `chrony` o `systemd-timesyncd`).

### Seguridad de `syslog`

El protocolo clásico es **texto plano y sin autenticación**: cualquiera en la red podría **enviar logs falsos** o **leer** los reales. Por eso en producción se usa **TLS** (puerto 6514) y se limita por firewall quién puede enviar. En este laboratorio empezarás sin TLS y lo añadirás en un reto.

### Rotación en el servidor

Los logs centralizados crecen **muy rápido**. Hay que configurar `logrotate` también para `/var/log/remoto/`.

### Comandos y archivos de este laboratorio

| Comando / archivo | Qué hace |
| ----------------- | -------- |
| `/etc/rsyslog.conf` y `/etc/rsyslog.d/*.conf` | Configuración de `rsyslog` |
| `sudo rsyslogd -N1` | **Valida** la configuración (comprueba la sintaxis) |
| `sudo systemctl restart rsyslog` | Reinicia el servicio |
| `logger -n IP -P 514 -T "mensaje"` | Envía un mensaje de prueba **directamente** a un servidor (`-T` = TCP) |
| `logger "mensaje"` | Envía un mensaje al `syslog` **local** |
| `timedatectl` | Hora, zona horaria y si hay sincronización |
| `sudo apt install chrony` / `chronyc tracking` | Cliente NTP y su estado |
| `sudo ufw allow from 192.168.56.0/24 to any port 514 proto tcp` | Abre el puerto 514 **solo para tu red** |
| `ss -tln \| grep 514` | Comprueba que el servidor **escucha** |
| `sudo tail -f /var/log/remoto/*/*.log` | Sigue los logs recibidos en vivo |

---

## 🛠️ Qué debes hacer

### Parte A · Preparar

1. Comprueba que **ambas VMs tienen la hora sincronizada** (`timedatectl`; instala `chrony` si hace falta) y que están en la **misma zona horaria**.
2. Pon **nombres de host distintos** a cada VM (`servidor-lab` y `cliente-lab`).

### Parte B · El servidor de logs

3. Configura `rsyslog` en el servidor para **recibir por TCP** en el puerto 514 y guardar **un archivo por equipo y programa** bajo `/var/log/remoto/`.
4. **Valida** la configuración con `rsyslogd -N1`, reinicia, y comprueba con `ss` que escucha.
5. Abre el puerto 514 en el firewall **solo para tu red de laboratorio**.

### Parte C · El cliente

6. En el cliente, configura el reenvío de **todos los logs** al servidor por TCP.
7. Envía un mensaje con `logger "prueba desde el cliente"` y compruébalo **en el servidor**.
8. En el cliente, haz **un intento de SSH fallido** a sí mismo (o desde el servidor) y comprueba que los registros de **autenticación del cliente** (`sshd`, `sudo`) **aparecen en el servidor**.
9. Genera un evento de `sudo` en el cliente y búscalo en el servidor.

### Parte D · Análisis conjunto

10. Genera eventos en **ambas** máquinas (por ejemplo, fallos de SSH hacia cada una) y **busca en el servidor central**: ¿cuántos intentos fallidos hubo **en total**, y por **máquina**? (`grep -r`, `sort | uniq -c`.)
11. Reconstruye **una línea temporal** de lo ocurrido: ¿quién hizo qué y en qué orden, en las dos máquinas?

### Parte E · Operación

12. Configura **`logrotate`** para `/var/log/remoto/*/*.log`: rotación diaria, conservar 14 copias, comprimir.
13. **Demuestra la ventaja de la centralización:** en el cliente, **borra** `/var/log/auth.log` (como lo haría un intruso) y comprueba que el servidor **conserva** los registros.
14. Provoca un **fallo**: apaga el servidor de logs, genera eventos en el cliente y vuelve a encenderlo. ¿Se pierden los mensajes? (Investiga las **colas** de `rsyslog` con `action.resumeRetryCount` y colas en disco: reto 2.)

---

## ✅ Comprobación

| Prueba | Qué debes ver |
| ------ | ------------- |
| `sudo rsyslogd -N1` | `rsyslogd: End of config validation run. Bye.` sin errores |
| `ss -tln \| grep 514` | `rsyslogd` escuchando |
| `ls /var/log/remoto/` | Una carpeta por cada equipo cliente |
| Mensaje de `logger` | Aparece en el servidor con el nombre de host del cliente |
| Fallos de SSH del cliente | Visibles en `/var/log/remoto/cliente-lab/sshd.log` |
| Tras borrar `auth.log` en el cliente | Los eventos **siguen en el servidor** |
| `timedatectl` | `System clock synchronized: yes` en ambas |

---

## 🧩 Pistas

- Si el servidor no recibe nada, comprueba **por orden**: ¿escucha? (`ss`), ¿el firewall deja pasar? (`ufw status`), ¿el cliente apunta bien? (IP y puerto), ¿la configuración valida? (`rsyslogd -N1`).
- Las plantillas con `%HOSTNAME%` usan el nombre **que dice el cliente en el mensaje**: por eso el nombre de host distinto es importante.
- Si ves **dos entradas** por mensaje, probablemente el servidor también guarda lo remoto en `/var/log/syslog`: la regla `stop` evita eso.
- Para ver qué hace `rsyslog` por dentro: `sudo journalctl -u rsyslog -n 30`.
- Si el cliente muestra *"connection refused"*, mira las **colas** y el estado del servicio en el servidor.

---

## 🚀 Retos extra

1. **Cifra el tráfico con TLS** (`gtls` en `rsyslog` con certificados de tu **propia CA**, como en el laboratorio siguiente). ¿Qué ataque evitas?
2. Configura una **cola en disco** en el cliente para que **no se pierda ningún mensaje** si el servidor cae, y compruébalo.
3. Envía también los logs de **Nginx** (`imfile`) y busca **peticiones 404 masivas** en el servidor central.
4. Investiga **Windows Event Forwarding** o el agente de logs de Windows para enviar eventos de `dc01` a tu servidor Linux.
5. Compara con soluciones más completas (**Graylog, Loki, ELK/OpenSearch**): ¿qué añaden a `rsyslog`? ¿Cuándo te quedarías corto con `rsyslog`?

---

## 📝 Qué documentar en tu `Solucion.md`

- Un diagrama del flujo de logs con IP y puertos.
- Tus configuraciones de servidor y cliente, explicadas.
- La **línea temporal** del evento que reconstruiste con los dos orígenes.
- Una reflexión: *¿qué garantiza y qué no garantiza este montaje frente a un atacante?* (Pista: ¿qué pasa si compromete **también** el servidor de logs?)

---

## 🧠 Lo que has practicado

`logging centralizado` · `rsyslog servidor y cliente` · `plantillas` · `sincronización horaria` · `logrotate` · `integridad de evidencias` · `correlación de eventos` · `syslog y su seguridad`
