# 📈 Laboratorio 08 · Monitorización con Prometheus y Grafana

**Nivel:** 🟠 Avanzado · **Sistema:** 🐧 Linux · **Tiempo estimado:** 100 minutos

---

## 🎯 Escenario

Hasta ahora te enterabas de los problemas **cuando alguien te avisaba** ("el servidor va lento"). Un sysadmin profesional quiere enterarse **antes** que los usuarios: ver gráficas del uso de CPU, memoria y disco, y recibir una **alerta** cuando algo se sale de lo normal.

Tu misión es montar un sistema de **monitorización** con **Prometheus** (que recoge las métricas) y **Grafana** (que las dibuja), vigilando tus dos servidores Ubuntu.

> 💾 **Snapshot** de las dos VMs. Servidor de monitorización: `ubuntu-server-lab` (`192.168.56.10`, mejor con 2 GB de RAM). Servidor vigilado adicional: `ubuntu-cliente` (`192.168.56.11`).

---

## 📖 Conceptos nuevos

### Monitorización: el cuadro de mandos

Piensa en el **cuadro de mandos de un coche**: velocidad, combustible, temperatura. Te avisa **antes** de que el motor se funda. La monitorización es lo mismo para un servidor: medir continuamente y alertar si algo va mal.

### Métricas

Una **métrica** es un número que cambia con el tiempo: porcentaje de CPU, memoria libre, bytes enviados por la red... Se guardan junto con su hora, formando una **serie temporal**.

### La arquitectura

| Pieza | Qué hace |
| ----- | -------- |
| **Exporter** (`node_exporter`) | Un pequeño programa **instalado en cada servidor vigilado** que expone sus métricas por HTTP (puerto **9100**) |
| **Prometheus** (puerto **9090**) | **Va a buscar** (*scrape*) las métricas a cada exporter cada pocos segundos y las **almacena**. Es un modelo **pull** |
| **Grafana** (puerto **3000**) | **Dibuja** las métricas en paneles y dashboards, y gestiona alertas visuales |

```
 servidor 1: node_exporter ─┐
                            ├──► Prometheus ───► Grafana ───► tu navegador
 servidor 2: node_exporter ─┘     (guarda)       (dibuja)
```

### Targets y jobs

En la configuración de Prometheus defines **jobs** (grupos de servidores) y sus **targets** (las direcciones `IP:puerto` de cada exporter). El estado de cada uno se mide con una métrica especial: **`up`** (1 = responde, 0 = caído).

### PromQL: el lenguaje de consulta

Algunas consultas básicas (pruébalas en la interfaz de Prometheus):

| Consulta | Qué muestra |
| -------- | ----------- |
| `up` | Qué targets responden |
| `node_memory_MemAvailable_bytes` | Memoria disponible |
| `rate(node_cpu_seconds_total{mode="idle"}[5m])` | Ritmo de tiempo ocioso de CPU en los últimos 5 min |
| `100 - avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100` | **% de CPU usada** |

(`rate()` calcula cuánto **crece por segundo** un contador en una ventana de tiempo.)

### Comandos y archivos de este laboratorio

| Comando / archivo | Qué hace |
| ----------------- | -------- |
| `sudo apt install prometheus prometheus-node-exporter` | Instala Prometheus y el exporter desde los repositorios de Ubuntu |
| `curl -s localhost:9100/metrics \| head` | Muestra las métricas **crudas** que expone un exporter |
| `/etc/prometheus/prometheus.yml` | Configuración de Prometheus (jobs y targets) |
| `promtool check config /etc/prometheus/prometheus.yml` | **Valida** la configuración antes de recargar |
| `sudo systemctl restart prometheus` | Reinicia Prometheus |
| `http://IP:9090` | Interfaz web de Prometheus (`Status → Targets`) |
| `http://IP:3000` | Interfaz web de Grafana (usuario inicial `admin`, que te pide cambiar la contraseña) |
| `yes > /dev/null &` | Genera carga de CPU (nivel 1, laboratorio 06). Recuerda **matarlo** después |
| `sudo ufw allow from 192.168.56.0/24 to any port 9100 proto tcp` | Abre el puerto de un exporter solo para tu red de laboratorio |

Plantilla del bloque de Prometheus que debes completar:

```yaml
scrape_configs:
  - job_name: '???'
    static_configs:
      - targets: ['???:9100', '???:9100']
```

---

## 🛠️ Qué debes hacer

### Parte A · Las métricas

1. En **los dos servidores**, instala `prometheus-node-exporter` y comprueba con `curl` que expone métricas en el puerto 9100.
2. Abre el puerto 9100 en el firewall de **ambos**, solo desde tu red de laboratorio.
3. Mira las métricas crudas: busca las de **CPU**, **memoria** y **disco**.

### Parte B · Prometheus

4. En `ubuntu-server-lab`, instala `prometheus`. Edita su configuración para añadir un job `servidores` con **los dos targets** (los dos exporters).
5. **Valida** el archivo con `promtool` y reinicia el servicio.
6. En la interfaz web (`:9090`), entra en **Status → Targets** y comprueba que los dos están en **UP**.
7. Ejecuta las consultas de la tabla, incluida la del **% de CPU**, y dibuja la gráfica.
8. En `ubuntu-cliente`, lanza `yes > /dev/null &` durante un minuto y observa **cómo sube la CPU** en la gráfica. **Mata el proceso** al terminar.
9. **Apaga** `ubuntu-cliente` y mira cómo `up` pasa a 0 y el target se marca como **DOWN**.

### Parte C · Grafana

10. Instala **Grafana** siguiendo la **documentación oficial** (añade su repositorio APT con su clave de firma; lee bien cada comando antes de ejecutarlo). Abre el puerto 3000 al firewall.
11. Entra en la interfaz y cambia la contraseña inicial.
12. Añade **Prometheus como fuente de datos** (`http://localhost:9090`).
13. **Importa** el dashboard comunitario **"Node Exporter Full"** (ID **1860**) y explora los paneles de CPU, memoria, disco y red de tus dos servidores.
14. Crea un **panel propio** con una consulta PromQL tuya.

### Parte D · Alertas (nivel básico)

15. Crea una **regla de alerta** en Grafana (o en Prometheus) que se dispare cuando **la CPU supere el 80 %** durante 2 minutos, o cuando un target pase a `DOWN`.
16. Provócala y comprueba que **cambia a estado "Firing"**.

---

## ✅ Comprobación

| Prueba | Qué debes ver |
| ------ | ------------- |
| `curl -s localhost:9100/metrics \| grep node_load1` | Una métrica con su valor |
| `promtool check config /etc/prometheus/prometheus.yml` | `SUCCESS` |
| Prometheus → Status → Targets | Dos targets en **UP** |
| Consulta `up` | Dos series con valor `1` |
| Con `yes` en marcha | La gráfica de CPU sube de forma clara |
| Con el cliente apagado | `up == 0` y target **DOWN** |
| Grafana → dashboard 1860 | Datos de los dos servidores |
| La alerta | Pasa a **Firing** al forzar la condición |

---

## 🧩 Pistas

- El YAML de Prometheus es estricto con la **indentación** (espacios, nunca tabuladores), como `netplan`.
- Si un target está en **DOWN**, comprueba: ¿el exporter está activo? ¿El firewall permite el 9100? ¿La IP es correcta? Prueba con `curl http://IP:9100/metrics` desde el servidor de Prometheus.
- En Grafana, al añadir la fuente de datos, pulsa **"Save & test"**: debe decirte que funciona.
- Si el dashboard sale vacío, comprueba el **rango de tiempo** (esquina superior derecha) y que el desplegable de **instancia** tiene elegida alguna.
- Los puertos **9090, 9100 y 3000** son los de siempre: si algo no responde, mira con `ss -tuln` que el servicio escucha.

---

## 🚀 Retos extra

1. Añade tu **servidor Nginx** del nivel intermedio como target usando el `nginx-prometheus-exporter` o el módulo de estado.
2. Instala **Alertmanager** y envía las alertas a un webhook de prueba.
3. Monitoriza un servicio concreto con una alerta: *"el servicio `ssh` ha dejado de estar activo"* (investiga el *collector* `systemd` de `node_exporter`).
4. Investiga **cuánto tiempo** guarda Prometheus los datos por defecto y cómo cambiarlo. ¿Cuánto espacio ocuparían 30 días de métricas de 50 servidores?
5. Explica la diferencia entre **monitorización** (métricas) y **logging** (eventos, laboratorio 06 del nivel intermedio). ¿Cuándo usarías cada una?

---

## 📝 Qué documentar en tu `Solucion.md`

- Un esquema de la arquitectura con **tus** IPs y puertos.
- Tu `prometheus.yml`, explicado.
- Capturas de la gráfica de CPU con y sin carga, y de la alerta en `Firing`.
- Con tus palabras: ¿por qué es mejor un modelo **pull** (Prometheus va a buscar) que uno **push**?

---

## 🧠 Lo que has practicado

`monitorización` · `métricas y series temporales` · `node_exporter` · `Prometheus` · `scrape / targets / jobs` · `PromQL básico` · `Grafana y dashboards` · `alertas` · `observabilidad`
