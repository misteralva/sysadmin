# 📘 Runbook · [Nombre del procedimiento]

> **Plantilla.** Un runbook se escribe para que **otra persona**, **a las 3:00 de la madrugada y bajo presión**, pueda seguirlo **sin pensar** ni conocer el sistema. Si un paso requiere "saber lo que haces", falta detalle.

| Campo | Valor |
| ----- | ----- |
| **Versión** | [1.0] |
| **Autor** | [tu nombre] |
| **Última revisión** | [AAAA-MM-DD] |
| **Última vez que se PROBÓ** | [AAAA-MM-DD] *(un runbook no probado es una suposición)* |
| **Sistema(s) afectados** | [`dc01`, `srv-web`...] |
| **Duración estimada** | [X minutos] |
| **Nivel de riesgo** | [Bajo / Medio / Alto] |

---

## 1 · Objetivo

*Una frase: qué consigue este procedimiento.*

[...]

## 2 · Cuándo usarlo

*Los síntomas o situaciones en los que aplicar este runbook.*

- [Síntoma o situación 1]
- [Síntoma o situación 2]

## 3 · Cuándo NO usarlo

*Casos en los que hay que parar y consultar, o usar otro runbook.*

- [...]

## 4 · Prerrequisitos

- [ ] [Acceso necesario: por ejemplo, "consola de VirtualBox o SSH a `fw-router`"]
- [ ] [Credenciales necesarias y **dónde están** (no las escribas aquí)]
- [ ] [Backup disponible y **verificado**]
- [ ] [Herramientas necesarias]

## 5 · Impacto

*Qué usuarios o servicios se ven afectados mientras se ejecuta, y durante cuánto.*

[...]

## 6 · Procedimiento

*Cada paso: **qué hacer**, **el comando exacto** y **qué resultado esperas**.*

| # | Acción | Comando | Resultado esperado | Si no sale así |
| - | ------ | ------- | ------------------ | -------------- |
| 1 | [...] | `[comando]` | [...] | [qué hacer / a quién avisar] |
| 2 | [...] | `[comando]` | [...] | |
| 3 | [...] | `[comando]` | [...] | |

## 7 · Verificación

*Cómo confirmas que el procedimiento ha funcionado, con pruebas **concretas**.*

- [ ] [Comprobación 1 y resultado esperado]
- [ ] [Comprobación 2 y resultado esperado]

## 8 · Marcha atrás (*rollback*)

*Cómo deshacer los cambios si algo sale mal.*

1. [...]
2. [...]

## 9 · Riesgos y cuidados

- ⚠️ [Qué puede salir mal y cómo evitarlo]

## 10 · Prevención

*Qué evitaría tener que ejecutar este runbook (monitorización, alerta, cambio de diseño).*

[...]

## 11 · Relacionados

- [Otros runbooks, documentos o laboratorios relacionados]

## 12 · Historial

| Fecha | Cambio | Quién | Resultado de la prueba |
| ----- | ------ | ----- | ---------------------- |
| | Creación | | |
