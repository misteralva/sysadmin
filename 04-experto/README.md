# 🔴 Nivel 4 · Experto: entorno y forma de trabajar

En los niveles anteriores seguías **pasos guiados**. En el nivel experto el trabajo se parece más al **real**: se te da un **escenario y un objetivo**, y **tú decides cómo llegar**. Los enunciados dan menos pistas, y cada laboratorio pide un **entregable profesional** (un runbook, un informe, un plan).

---

## 🧭 Qué cambia respecto al nivel avanzado

| Antes | Ahora |
| ----- | ----- |
| Pasos numerados con los comandos casi dados | **Objetivos y criterios de éxito**; elige tú las herramientas |
| Se explica cada comando nuevo | Se explican los **conceptos**; los comandos que ya conoces no se repiten |
| El fallo es una excepción | **Romper cosas y arreglarlas** es parte del trabajo (laboratorios 01, 10, 11, 12) |
| Documentas lo que hiciste | Documentas como un **profesional**: runbooks, informes de incidente, planes de recuperación |

---

## 🖥️ Máquinas

Reutilizas las de los niveles anteriores:

| VM | Se usa en |
| -- | --------- |
| `ubuntu-server-lab` (`192.168.56.10`) | Casi todos los laboratorios de Linux |
| `ubuntu-cliente` (`192.168.56.11`) | 03, 04, 05, 06, 07, 08, 12 |
| `router-lab`, `cliente-a`, `cliente-b` | 11 (fallos de red) |
| `dc01` y `pc-cliente01` | 09, 11 |

Para los laboratorios 01 y 08 necesitarás además una **ISO de Ubuntu Server** (para arrancar en modo *live* y para reconstruir una VM).

---

## 🧰 Cómo trabajar en este nivel

1. **Snapshot antes de todo.** En este nivel provocas fallos graves a propósito.
2. **Cronometra.** Anota cuánto tardas en cada diagnóstico: en la vida real, el tiempo importa.
3. **Método antes que prisa.** Observar → hipótesis → comprobar → arreglar → verificar → documentar.
4. **No resuelvas "a lo bruto".** Reiniciar o restaurar el snapshot sin entender la causa no cuenta.
5. **Todo se documenta.** Un buen sysadmin deja escrito qué pasó, por qué y cómo evitarlo.

---

## 🛡️ Ética y alcance

Todo lo que hagas aquí es **sobre tus propias VMs, en redes aisladas**. Los laboratorios de seguridad son **defensivos**: detectar, investigar, auditar y recuperar. No uses estas técnicas, ni las de diagnóstico, sobre sistemas que no sean tuyos o sobre los que no tengas **autorización expresa**.

---

## 🧭 Orden recomendado

1. **Recuperación:** 01 (arranque) → 02 (rendimiento)
2. **Servicios modernos:** 03 (Docker) → 04 (logs centralizados) → 05 (PKI)
3. **Detección y resiliencia:** 06 (IDS) → 07 (alta disponibilidad) → 08 (simulacro de desastre)
4. **Windows:** 09 (auditoría de AD)
5. **Juegos de guerra:** 10 (break/fix Linux) → 11 (break/fix Windows y red) → 12 (respuesta a incidentes)

Los tres últimos son los más completos: **conviene hacerlos al final**, porque reutilizan lo de todos los demás.
