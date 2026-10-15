# 🏗️ Documento de arquitectura · NovaTech Sistemas

> **Plantilla.** Sustituye los textos entre `[corchetes]` y borra las instrucciones en cursiva. Mantén este documento **actualizado** durante todo el proyecto: la documentación que no coincide con la realidad es peor que ninguna.

| Campo | Valor |
| ----- | ----- |
| **Versión** | [1.0] |
| **Fecha** | [AAAA-MM-DD] |
| **Autor** | [tu nombre] |
| **Estado** | [Borrador / Revisado / Vigente] |

---

## 1 · Resumen

*Cuatro o cinco líneas, sin tecnicismos: qué infraestructura es, a quién sirve y qué problema resuelve.*

[...]

## 2 · Objetivos y alcance

**Objetivos:**

- [Objetivo 1]
- [Objetivo 2]

**Dentro del alcance:** [...]

**Fuera del alcance:** [...] *(declarar lo que no haces es tan importante como lo que haces)*

## 3 · Diagrama de la arquitectura

*Inserta el diagrama (imagen o texto). Debe coincidir con lo construido.*

```
[diagrama]
```

## 4 · Inventario de sistemas

| Sistema | Rol | SO | IP | Zona | RAM / CPU | Notas |
| ------- | --- | -- | -- | ---- | --------- | ----- |
| `fw-router` | Firewall, router, VPN | Ubuntu | | | | |
| `dc01` | AD, DNS, DHCP, ficheros | Windows Server | | | | |
| `srv-web` | Intranet | Ubuntu | | | | |
| `srv-infra` | Logs, monitorización, backups | Ubuntu | | | | |
| `pc-cliente01` | Cliente de dominio | Windows | | | | |

## 5 · Plan de direccionamiento

| Zona | Red | Puerta de enlace | DNS | Rango DHCP |
| ---- | --- | ---------------- | --- | ---------- |
| LAN | | | | |
| DMZ | | | | |
| VPN | | | | |
| WAN simulada | | | | |

## 6 · Matriz de flujos

| Origen → Destino | Protocolo y puerto | Permitido | Justificación | Regla que lo implementa |
| ---------------- | ------------------ | --------- | ------------- | ----------------------- |
| LAN → Internet | | | | |
| LAN → DMZ | | | | |
| DMZ → LAN | | | | |
| WAN → fw-router | | | | |
| VPN → LAN | | | | |

## 7 · Servicios y puertos

| Servicio | Sistema | Puerto/protocolo | Accesible desde | Autenticación |
| -------- | ------- | ---------------- | --------------- | ------------- |
| | | | | |

## 8 · Decisiones de diseño

*Una ficha por decisión importante.*

### Decisión 1: [título]

- **Contexto:** [qué problema o necesidad había]
- **Decisión:** [qué elegiste]
- **Alternativas consideradas:** [qué otras opciones había y por qué no]
- **Consecuencias:** [ventajas y costes de esta elección]

## 9 · Seguridad del diseño

- **Principios aplicados:** [mínimo privilegio, defensa en profundidad, segmentación...]
- **Superficie expuesta:** [qué es visible desde fuera]
- **Gestión de secretos:** [dónde y cómo se guardan claves y contraseñas]

## 10 · Resiliencia y recuperación

| Servicio | Punto único de fallo (SPOF) | RPO | RTO | Mecanismo de recuperación |
| -------- | --------------------------- | --- | --- | ------------------------- |
| | | | | |

## 11 · Supuestos y limitaciones

- [Supuesto o limitación 1: por ejemplo, "es un entorno de laboratorio sin internet real"]

## 12 · Historial de cambios

| Versión | Fecha | Cambio | Motivo |
| ------- | ----- | ------ | ------ |
| 1.0 | | Versión inicial | |
