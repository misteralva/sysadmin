# 🔥 Laboratorio 08 · Simulacro de desastre (*game day*)

**Nivel:** 🔴 Experto · **Sistema:** 🐧 Linux · **Tiempo estimado:** 150 minutos

---

## 🎯 Escenario

Son las 3:00 de la madrugada. Una avería eléctrica ha destruido el servidor de la intranet: **el disco no se recupera y la máquina no volverá a arrancar**. Hay que **reconstruir el servicio en una máquina nueva** con lo que tienes: los **backups** (laboratorio 09 del nivel intermedio), tu **automatización con Ansible** (laboratorio 09 del avanzado) y tu **documentación**.

Muchas empresas descubren **demasiado tarde** que sus copias de seguridad no sirven, o que nadie sabe **en qué orden** hay que reconstruir las cosas. Tu misión es **ensayar el desastre** antes de que ocurra, **medir cuánto se tarda** y escribir el **plan de recuperación**.

> 💾 Esta vez **no uses solo el snapshot**: el objetivo es **reconstruir una VM desde cero**. Necesitarás una ISO de Ubuntu Server y, como "salvavidas", **un snapshot** del estado inicial por si te pierdes.

---

## 📖 Conceptos nuevos

### Dos números que mandan: RPO y RTO

| Sigla | Pregunta | Ejemplo |
| ----- | -------- | ------- |
| **RPO** (*Recovery Point Objective*) | ¿**Cuántos datos** puedo permitirme perder? (tiempo desde la última copia) | "Como mucho, 4 horas de datos" |
| **RTO** (*Recovery Time Objective*) | ¿**Cuánto tiempo** puede estar el servicio caído? | "Debe volver en menos de 2 horas" |

> **Analogía:** tras un incendio en una tienda, el **RPO** es "cuántas ventas del día perdemos si el registro se quema"; el **RTO** es "cuánto tardamos en volver a abrir".

El RPO dicta **cada cuánto haces backups**; el RTO dicta **cuánta automatización y ensayo** necesitas.

### Plan de recuperación ante desastres (DRP) y runbook

Un **DRP** es el documento que dice **qué hacer** cuando todo falla. Se compone de **runbooks**: procedimientos **paso a paso** que cualquiera del equipo pueda seguir **a las 3:00 de la madrugada, sin pensar**, aunque no sea el que lo montó.

### *Game day* (día de simulacro)

Un **simulacro** consiste en **provocar un fallo controlado** y **ver si el equipo y los procedimientos aguantan**. Los backups no probados y los runbooks no ensayados son **suposiciones, no garantías**.

### Las preguntas de un buen simulacro

- ¿Teníamos **todo** lo necesario (claves, contraseñas, configuraciones, ISO)? ¿O estaba **solo en la máquina que murió**?
- ¿El backup **se restaura de verdad**? ¿Con qué rapidez?
- ¿En qué **orden** hay que reconstruir?
- ¿Cuánto tiempo **real** tardamos? (Compáralo con el RTO.)
- ¿Cuántos datos perdimos? (Compáralo con el RPO.)

### Herramientas que reutilizas

| Herramienta | Para qué |
| ----------- | -------- |
| `tar`, `rsync`, `sha256sum` | Hacer y **verificar** backups (nivel intermedio, 09) |
| **Ansible** | **Reconstruir** configuración (paquetes, usuarios, servicios) en minutos (avanzado, 09) |
| `cron`/**timers** | Programar los backups (nivel intermedio, 02) |
| `date`, `time` y un cronómetro | **Medir** cada fase |

---

## 🛠️ Qué debes hacer

### Parte A · Definir el servicio y los objetivos

1. Elige el **servicio a proteger**: la **intranet** con Nginx (nivel intermedio, 10), con su sitio web, algunos **datos** que cambian (por ejemplo, una carpeta de documentos) y los **usuarios** del sistema.
2. Define **por escrito** un **RPO** y un **RTO** realistas para este servicio (por ejemplo, RPO = 1 hora, RTO = 60 minutos).
3. Haz una **lista de inventario** (la "mochila de supervivencia"): qué hace falta para reconstruir (ISO, playbook de Ansible, backups, claves SSH, certificados, contraseñas...) y **dónde está cada cosa**, en un lugar **que no sea la propia VM**.

### Parte B · Preparar las defensas

4. Asegura que tienes un **playbook de Ansible** que reconstruye la configuración (paquetes, usuarios, Nginx, firewall). Si no es completo, **complétalo ahora**.
5. Configura **backups automáticos** de los datos y de las configuraciones relevantes, con **verificación** (hash) y **guardados fuera** de la VM (otra VM, tu ordenador).
6. Escribe un **primer borrador del runbook** de recuperación, con los pasos en orden.

### Parte C · El simulacro

7. Apunta la **hora de inicio** y haz un **último backup**. Modifica un dato **después** del backup (para medir la pérdida real, el RPO efectivo).
8. **Destruye** la VM: **apágala y elimínala** de VirtualBox con sus discos.
9. **Cronometra** y reconstruye **desde cero**:
   - Crea la VM nueva e instala Ubuntu Server (**mínimo**, sin pasos que no estén en tu runbook).
   - Prepara el acceso (red, SSH).
   - Aplica tu **Ansible** para reconstruir la configuración.
   - **Restaura los datos** desde el backup.
   - **Verifica** con una lista de comprobación (servicio, datos, usuarios, firewall, certificados...).
10. Apunta **la hora a la que el servicio vuelve a funcionar** y **calcula**: **RTO real**, **tiempo por fase** y **datos perdidos (RPO real)**.

### Parte D · Aprender del simulacro

11. Compara con los objetivos: ¿cumpliste el RTO y el RPO?
12. Haz una **reunión post-mortem** contigo mismo: ¿**qué falló o faltaba**? ¿Qué **dependía de tu memoria** y no estaba escrito? ¿Qué fue **más lento** de lo esperado?
13. **Mejora** el runbook y la automatización con lo aprendido, y **repite** el simulacro (o al menos las fases que fallaron) para comprobar la mejora.

---

## ✅ Comprobación

| Prueba | Qué debes ver |
| ------ | ------------- |
| Inventario de la "mochila" | Todos los elementos, **fuera** de la VM |
| Backup | Con **hash verificado** y copia fuera de la VM |
| Servicio reconstruido | La intranet responde con **los datos restaurados** |
| Lista de comprobación post-restauración | **Todos los puntos** en verde |
| Tabla de tiempos | Tiempo **por fase** y **RTO real** |
| Post-mortem | **Al menos tres** mejoras concretas |

---

## 🧩 Pistas

- Si lo que necesitas para recuperar **solo existe en la VM destruida** (una clave, un script, la contraseña), has descubierto una **dependencia circular**: es **el hallazgo más valioso** del simulacro. Anótalo.
- **Escribe el runbook como si lo fuera a seguir otra persona** que no sabe nada de tu sistema.
- Automatiza **lo que más tiempo te ha costado**, pero **no pierdas** el runbook manual: si falla la automatización, tendrás que hacerlo a mano.
- Mide **cada fase** por separado (instalación, red, Ansible, datos, verificación) para saber **dónde se va el tiempo**.
- Si tu RTO real es mucho mayor que el objetivo, tienes tres opciones: **automatizar más**, **preparar una imagen/plantilla** base, o **renegociar el objetivo** con el "negocio".

---

## 🚀 Retos extra

1. **Ten una VM plantilla** (con Ubuntu ya instalado y actualizado) y mide **cuánto baja el RTO** al partir de ella en vez de instalar desde ISO.
2. Prueba un **segundo tipo de desastre**: no se destruye la VM, sino que se **corrompen los datos** (borrado o *ransomware* simulado). ¿Sirve tu backup? ¿Tiene versiones **anteriores** a la corrupción? (Pista: copias **inmutables o con historial**.)
3. Haz el simulacro **a ciegas**: pide a otra persona que **siga tu runbook** sin ayuda, y observa **dónde se atasca**.
4. Aplica el mismo ejercicio a **Active Directory** (avanzado, 15): ¿qué sería el RTO y el RPO de un dominio?
5. Redacta un **informe para la dirección** de una página, sin tecnicismos: *"qué ocurrió, cuánto tardamos, qué perdimos, qué cuesta mejorar"*.

---

## 📝 Qué documentar en tu `Solucion.md`

- El **DRP** completo: servicio, RPO, RTO, inventario, procedimiento **paso a paso**.
- La **tabla de tiempos** del simulacro.
- El **post-mortem**: hallazgos, causas y mejoras (formato **"sin culpables"**: se analiza el **proceso**, no a la persona).
- Una reflexión: *¿qué diferencia hay entre "tener backups" y "poder recuperarse"?*

---

## 🧠 Lo que has practicado

`RPO y RTO` · `plan de recuperación (DRP)` · `runbooks` · `game day` · `reconstrucción desde cero` · `Ansible + backups` · `medición de tiempos` · `post-mortem sin culpables`
