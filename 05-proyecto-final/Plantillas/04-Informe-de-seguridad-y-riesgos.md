# 🛡️ Informe de seguridad y riesgos · NovaTech Sistemas

> **Plantilla.** Este informe es lo que leería la dirección (y un auditor). Debe ser **honesto**: un informe que dice "todo está perfecto" no es creíble. Lo valioso es **qué riesgos hay, cuáles has reducido y cuáles aceptas conscientemente**.

| Campo | Valor |
| ----- | ----- |
| **Versión** | [1.0] |
| **Fecha** | [AAAA-MM-DD] |
| **Autor** | [tu nombre] |
| **Alcance** | [sistemas y zonas revisados] |

---

## 1 · Resumen ejecutivo

*Cinco líneas, sin tecnicismos, para alguien no técnico: cómo está la seguridad, qué se ha mejorado, qué riesgos importantes quedan y qué se recomienda.*

[...]

## 2 · Alcance y metodología

- **Qué se revisó:** [red, Linux, Windows/AD, servicios, operación]
- **Cómo:** [Lynis, auditoría de AD con PowerShell, revisión de la matriz de flujos, pruebas de aceptación...]
- **Qué quedó fuera:** [...]

## 3 · Mejoras de seguridad realizadas

| Área | Antes | Después | Evidencia |
| ---- | ----- | ------- | --------- |
| Hardening Linux (índice Lynis) | [XX] | [YY] | [ruta] |
| SSH | [contraseña permitida] | [solo claves, sin root] | |
| Segmentación de red | | | |
| Contraseñas / PSO | | | |
| Línea base de Windows | | | |
| Backups y recuperación | | | |

## 4 · Hallazgos

| # | Hallazgo | Evidencia | Severidad | Estado |
| - | -------- | --------- | --------- | ------ |
| H-01 | | | 🔴 / 🟠 / 🟡 / 🟢 | Corregido / Pendiente / Aceptado |

**Criterio de severidad:** 🔴 Crítica (compromiso muy probable y de gran impacto) · 🟠 Alta · 🟡 Media · 🟢 Baja.

## 5 · Mapa de riesgos residuales

Para cada riesgo que **queda**, valora **probabilidad** e **impacto** (1 = bajo, 3 = alto). **Nivel = probabilidad × impacto.**

| Riesgo | Probabilidad (1-3) | Impacto (1-3) | Nivel | Tratamiento |
| ------ | ------------------ | ------------- | ----- | ----------- |
| [Ej.: no hay segundo controlador de dominio] | | | | Mitigar / Aceptar / Transferir / Evitar |

| Nivel | Lectura |
| ----- | ------- |
| 7-9 | Alto: actuar de inmediato |
| 4-6 | Medio: planificar mejora |
| 1-3 | Bajo: vigilar |

## 6 · Riesgos aceptados conscientemente

*Cada riesgo que decides **no** corregir, con quién lo acepta, por qué y hasta cuándo.*

| Riesgo | Motivo de aceptación | Quién lo acepta | Revisar el |
| ------ | -------------------- | --------------- | ---------- |
| | | | |

## 7 · Cobertura de defensa en profundidad

| Capa | Controles implementados | Carencias |
| ---- | ----------------------- | --------- |
| Perímetro | | |
| Red interna | | |
| Equipos | | |
| Aplicación | | |
| Datos | | |
| Identidad | | |
| Detección y respuesta | | |
| Recuperación | | |

## 8 · Plan de mejora

| Prioridad | Acción | Esfuerzo | Responsable | Plazo |
| --------- | ------ | -------- | ----------- | ----- |
| 1 | | | | |

## 9 · Conclusión

*Una valoración honesta: ¿está la empresa lista para la auditoría? ¿Qué falta para estarlo?*

[...]

## 10 · Anexos

- Informes de Lynis (**antes y después**, sin datos sensibles)
- Informe de auditoría de AD
- Matriz de flujos final
- Registro de simulacros e incidentes
