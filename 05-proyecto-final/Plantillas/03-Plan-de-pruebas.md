# 🧪 Plan de pruebas · NovaTech Sistemas

> **Plantilla.** Cada requisito del enunciado debe tener **al menos una prueba** que lo demuestre. Una prueba sin **evidencia** (salida de comando o captura) **no cuenta**.

| Campo | Valor |
| ----- | ----- |
| **Versión** | [1.0] |
| **Fecha de ejecución** | [AAAA-MM-DD] |
| **Ejecutado por** | [tu nombre] |
| **Entorno** | [versiones de los sistemas y de VirtualBox] |

---

## 1 · Convenciones

- **Estado:** ✅ OK · ❌ KO · ⚠️ Parcial · ⏳ Pendiente
- **Evidencia:** ruta al archivo de captura o salida (por ejemplo, `evidencias/R2-01.txt`). **Nunca incluyas secretos** en las evidencias.

## 2 · Pruebas de aceptación

### Red y perímetro

| ID | Requisito | Procedimiento | Resultado esperado | Resultado real | Evidencia | Estado |
| -- | --------- | ------------- | ------------------ | -------------- | --------- | ------ |
| P-R1 | R1 | Desde un cliente de la LAN: `ping 8.8.8.8` y `curl` a una web | Responde | | | ⏳ |
| P-R2a | R2 | LAN → DMZ por 443 | **Permitido** | | | ⏳ |
| P-R2b | R2 | LAN → DMZ por un puerto distinto (por ejemplo, 22) | **Bloqueado** | | | ⏳ |
| P-R2c | R2 | DMZ → LAN (iniciando conexión) | **Bloqueado** | | | ⏳ |
| P-R3 | R3 | Intento bloqueado: buscar la línea en el log | Aparece con origen, destino y puerto | | | ⏳ |
| P-R4a | R4 | Cliente remoto con VPN: acceso a ficheros e intranet | Funciona | | | ⏳ |
| P-R4b | R4 | Cliente remoto con VPN: acceso a un recurso **no permitido** | **Bloqueado** | | | ⏳ |

### Identidad y servicios

| ID | Requisito | Procedimiento | Resultado esperado | Resultado real | Evidencia | Estado |
| -- | --------- | ------------- | ------------------ | -------------- | --------- | ------ |
| P-I3 | I3 | Inicio de sesión de dominio en `pc-cliente01` | Entra | | | ⏳ |
| P-I4 | I4 | `gpresult /r` con cada usuario | Aparecen las GPO esperadas | | | ⏳ |
| P-S1 | S1 | Cliente nuevo por DHCP: IP y `nslookup intranet.novatech.lab` | IP del rango y nombre resuelto | | | ⏳ |
| P-S2a | S2 | Usuario de Sistemas abre su carpeta | Accede | | | ⏳ |
| P-S2b | S2 | Usuario de Redes abre la carpeta de Sistemas | **Acceso denegado** | | | ⏳ |
| P-S3 | S3 | Navegador: `https://intranet.novatech.lab` | **Sin avisos** | | | ⏳ |

### Seguridad

| ID | Requisito | Procedimiento | Resultado esperado | Resultado real | Evidencia | Estado |
| -- | --------- | ------------- | ------------------ | -------------- | --------- | ------ |
| P-H1 | H1 | `lynis audit system` antes y después | Mejora ≥ 15 puntos | | | ⏳ |
| P-H2a | H2 | `ssh` con contraseña | **Rechazado** | | | ⏳ |
| P-H2b | H2 | Varios intentos fallidos de SSH | `fail2ban` bloquea | | | ⏳ |
| P-H3 | H3 | Comprobar SMBv1, cortafuegos y cuentas | Cumple la línea base | | | ⏳ |

### Operación

| ID | Requisito | Procedimiento | Resultado esperado | Resultado real | Evidencia | Estado |
| -- | --------- | ------------- | ------------------ | -------------- | --------- | ------ |
| P-O1 | O1 | `chronyc tracking` y `w32tm /query /status` | Sincronizados | | | ⏳ |
| P-O2 | O2 | Provocar la condición de una alerta | La alerta salta (*Firing*) | | | ⏳ |
| P-O3 | O3 | `logger` en `srv-web` | El mensaje llega a `srv-infra` | | | ⏳ |
| P-O4 | O4 | **Restauración real** de un backup | Datos íntegros (hash igual) y tiempo medido | | | ⏳ |

*(Añade las filas de los requisitos E y A que hayas abordado.)*

## 3 · Simulacros

| Ejercicio | Fecha | Qué se hizo | Resultado medido | Conclusión y mejoras |
| --------- | ----- | ----------- | ---------------- | -------------------- |
| Prueba de restauración | | | Tiempo: \_\_ · Datos perdidos: \_\_ | |
| Avería a ciegas 1 | | | Diagnóstico: \_\_ min | |
| Avería a ciegas 2 | | | | |
| Avería a ciegas 3 | | | | |
| Incidente simulado | | | | |
| Simulacro de desastre (E/A) | | | **RTO real:** \_\_ · **RPO real:** \_\_ | |

## 4 · Resumen

| | Total | ✅ OK | ❌ KO | ⚠️ Parcial | ⏳ Pendiente |
| - | ----- | ----- | ----- | --------- | ------------ |
| Pruebas | | | | | |

**Pruebas fallidas o parciales (qué pasó y qué plan hay):**

- [...]
