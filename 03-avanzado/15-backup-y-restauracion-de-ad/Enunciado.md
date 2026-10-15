# 🛟 Laboratorio 15 · Copia de seguridad y restauración de Active Directory

**Nivel:** 🟠 Avanzado · **Sistema:** 🪟 Windows Server · **Tiempo estimado:** 120 minutos

---

## 🎯 Escenario

Un martes por la mañana, un compañero de soporte, con prisa, **borra por error la OU de Sistemas completa**, con todos sus usuarios y grupos. Nadie puede iniciar sesión. El director pregunta: *"¿cuánto tardamos en recuperarlo?"*

El directorio es el **corazón** de una red Windows: si se pierde o se corrompe, la empresa se para. Tu misión es **prepararte antes de que ocurra**: proteger los objetos, hacer copias de seguridad del directorio y practicar la **recuperación** (de un objeto suelto y de un borrado masivo).

> ⚠️ **Haz snapshot de `dc01` antes de cada parte** (en este laboratorio, lo vas a necesitar). Y recuerda: **en una empresa real nunca se restaura un controlador de dominio con snapshots de máquina virtual** (puede causar daños graves de replicación, *USN rollback*, que investigaste en el laboratorio 11 del nivel intermedio). Aquí es un laboratorio con un único DC y no hay replicación que romper.

---

## 📖 Conceptos nuevos

### Tres líneas de defensa

| Línea | Qué protege | Herramienta |
| ----- | ----------- | ----------- |
| **1. Prevención** | Evitar el borrado accidental | Proteger OU y objetos contra eliminación |
| **2. Recuperación rápida** | Deshacer un borrado reciente | **Papelera de reciclaje de AD** (*AD Recycle Bin*) |
| **3. Recuperación total** | Fallo grave, corrupción o borrado masivo | **Copia de seguridad del estado del sistema** y restauración |

### Protección contra eliminación accidental

Cada objeto de AD tiene una casilla **"Proteger objeto contra eliminación accidental"**. Si está marcada, no se puede borrar **sin desmarcarla antes**. Las OU nuevas creadas con la consola ya suelen traerla marcada.

### La papelera de reciclaje de AD

Por defecto, cuando se borra un objeto de AD, queda como un esqueleto casi vacío (*tombstone*), con casi todos sus atributos **perdidos**: **no se puede recuperar bien**. Si activas la **papelera de reciclaje de AD**, los objetos borrados se conservan **con todos sus atributos** (incluida su pertenencia a grupos) durante un tiempo, y se pueden **restaurar** completos.

> ⚠️ **Activar la papelera es irreversible**: una vez activada, no se puede desactivar. Pero es una buena práctica y apenas tiene coste.

### Copia de seguridad del estado del sistema

El **estado del sistema** (*System State*) de un controlador de dominio incluye la **base de datos de AD** (`ntds.dit`), `SYSVOL`, el registro, y otros componentes críticos. Se copia con **Windows Server Backup** (`wbadmin`). Con eso, puedes **reconstruir** el directorio incluso desde cero.

### Los dos tipos de restauración

| Tipo | Qué hace | Cuándo |
| ---- | -------- | ------ |
| **No autoritativa** | Restaura la copia; el DC se **pone al día** con los demás DC (replicación) | Fallo del propio DC |
| **Autoritativa** | Restaura objetos concretos y **marca esa versión como "la buena"**, de forma que **sobrescribe** a los demás DC | Un borrado accidental que **ya se ha replicado** a todos |

En un entorno con un solo DC, como el tuyo, la distinción es **conceptual** (no hay con quién replicar), pero conviene entenderla. Se hace desde el **modo de restauración de servicios de directorio (DSRM)**, con la contraseña que definiste al crear el dominio.

### Comandos de este laboratorio

| Comando | Qué hace |
| ------- | -------- |
| `Get-ADOrganizationalUnit -Filter * -Properties ProtectedFromAccidentalDeletion` | Muestra qué OU están protegidas |
| `Set-ADObject -Identity "..." -ProtectedFromAccidentalDeletion $true` | Activa la protección |
| `Enable-ADOptionalFeature 'Recycle Bin Feature' -Scope ForestOrConfigurationSet -Target empresa.local` | **Activa** la papelera (¡irreversible!) |
| `Get-ADObject -Filter 'isDeleted -eq $true' -IncludeDeletedObjects` | Lista los objetos **eliminados** recuperables |
| `Restore-ADObject -Identity <GUID o DN>` | **Restaura** un objeto de la papelera |
| `Install-WindowsFeature Windows-Server-Backup` | Instala la herramienta de copias |
| `wbadmin start systemstatebackup -backuptarget:E: -quiet` | Hace una copia del **estado del sistema** en la unidad `E:` |
| `wbadmin get versions` | Lista las copias existentes |
| `bcdedit /set safeboot dsrepair` | Prepara el arranque en **modo DSRM** (y `bcdedit /deletevalue safeboot` lo revierte) |
| `ntdsutil` | Herramienta de mantenimiento de AD (restauración autoritativa) |

---

## 🛠️ Qué debes hacer

### Parte A · Prevención

1. Haz **snapshot**. En `dc01`, comprueba con PowerShell **qué OU tienen la protección** activada.
2. Activa la protección en `Empresa`, `Sistemas`, `Redes` y `Equipos`.
3. Intenta **eliminar** la OU `Redes` desde `dsa.msc` y observa el mensaje de error. (Si no puedes verla, activa en el menú *Ver → Características avanzadas*.)

### Parte B · La papelera de reciclaje

4. **Activa** la papelera de reciclaje de AD (acuérdate de que es irreversible: tu snapshot es la red de seguridad).
5. Crea un usuario de prueba `borrame` en `Sistemas` y asígnalo a `GG_Sistemas`.
6. **Bórralo.** Comprueba con `Get-ADObject -IncludeDeletedObjects` que aparece como **objeto eliminado**.
7. **Restáuralo** con `Restore-ADObject` y comprueba que **conserva su grupo y atributos**.
8. Haz el mismo ejercicio con una **OU de pruebas** que contenga varios usuarios: bórrala (¡recuerda desproteger antes!) y restaura **la OU y su contenido**, en el orden correcto (primero el contenedor, luego los hijos).

### Parte C · Copia del estado del sistema

9. **Apaga `dc01`**, añade un **disco virtual nuevo de 20 GB** y enciéndelo. Inicializa y formatea el disco en **`E:`** (gestión de discos).
10. Instala **Windows Server Backup** y haz una **copia del estado del sistema** en `E:`. Puede tardar varios minutos.
11. Comprueba con `wbadmin get versions` que existe la copia y anota su **identificador de versión**.

### Parte D · Desastre y restauración total

12. **Haz snapshot.** Ahora simula **el desastre**: desprotege y **elimina la OU `Sistemas` entera** con todos sus usuarios (con la papelera activa, esto ya sería recuperable; para este ejercicio, **desactiva la papelera conceptualmente**: usa el snapshot anterior a su activación si quieres practicar la restauración "dura", o limítate a razonar el procedimiento).
13. **Intenta iniciar sesión** como `ana` en el cliente: no podrás.
14. **Recupera** el directorio desde la copia:
    - Arranca `dc01` en **modo DSRM** (`bcdedit`, como `Administrator` local con la contraseña de DSRM).
    - Restaura la copia con `wbadmin start systemstaterecovery`.
    - Reinicia en modo normal y comprueba que `Sistemas` y sus usuarios han vuelto.
    *(Este paso es largo y delicado; si no te sale, anota **dónde** te atascas: es parte del aprendizaje. Con el snapshot puedes empezar de nuevo.)*

---

## ✅ Comprobación

| Prueba | Qué debes ver |
| ------ | ------------- |
| `Get-ADOrganizationalUnit -Filter * -Properties ProtectedFromAccidentalDeletion \| Select-Object Name, ProtectedFromAccidentalDeletion` | `True` en todas |
| Borrar `Redes` | Error: acceso denegado por la protección |
| `Get-ADOptionalFeature -Filter 'Name -like "Recycle*"' \| Select-Object Name, EnabledScopes` | La papelera con un ámbito en `EnabledScopes` |
| `Get-ADObject -Filter 'isDeleted -eq $true' -IncludeDeletedObjects \| Select-Object Name` | El usuario borrado (antes de restaurar) |
| `Get-ADUser borrame -Properties MemberOf` tras restaurar | Con su grupo `GG_Sistemas` |
| `wbadmin get versions` | Una versión con "SystemState" |
| Tras la restauración | `Sistemas` y sus usuarios de vuelta; `ana` entra |

---

## 🧩 Pistas

- Para borrar una OU protegida hay que **desproteger primero** (propiedades del objeto → pestaña *Objeto* → desmarcar la casilla) y, si tiene hijos, también ellos.
- Al restaurar jerarquías, el orden importa: **primero el padre**, luego los hijos. Si no, `Restore-ADObject` da error de "no se encuentra el contenedor padre".
- Si `wbadmin` dice que el destino no es válido, comprueba que `E:` es un **volumen distinto** del sistema y que está **formateado**.
- La contraseña de **DSRM** es la que definiste al promover `dc01` (laboratorio 11 del nivel intermedio). Es la que **no** tiene que ver con tu contraseña de dominio.
- Si el arranque en DSRM queda "atascado", recuerda que `bcdedit /deletevalue safeboot` lo revierte, y que tu **snapshot** te salva.
- No olvides **desactivar el modo seguro** (`safeboot`) tras restaurar.

---

## 🚀 Retos extra

1. Investiga la **restauración autoritativa** con `ntdsutil` (`authoritative restore`, `restore subtree`): escribe los **comandos paso a paso** de cómo recuperarías una OU con su contenido y por qué hace falta cuando hay **varios** DC.
2. Programa la copia del estado del sistema con una **tarea programada** diaria y razona cuánta retención necesitarías.
3. Exporta la **estructura de OU y usuarios** a CSV con PowerShell (laboratorio 14) como **copia lógica adicional**: ¿qué ventajas e inconvenientes tiene frente al backup del estado del sistema?
4. Investiga qué es un **segundo controlador de dominio**, por qué es la **mejor protección** y cuándo un backup deja de ser suficiente. (Hazlo como ejercicio teórico o monta un segundo DC si tu equipo aguanta.)
5. Aplica la **regla 3-2-1** (laboratorio 09 del nivel intermedio) al directorio: ¿dónde guardarías las copias de AD?

---

## 📝 Qué documentar en tu `Solucion.md`

- Un **plan de recuperación ante desastres** (DRP) de una página para AD: qué se copia, cada cuánto, dónde, y **los pasos exactos** para recuperar, con el **tiempo estimado**.
- Capturas de la papelera de reciclaje, de la copia y de la restauración.
- Con tus palabras: **restauración autoritativa frente a no autoritativa**.
- Un resumen de **qué aprendiste** del desastre simulado y qué harías distinto.

---

## 🧠 Lo que has practicado

`protección contra eliminación` · `papelera de reciclaje de AD` · `Restore-ADObject` · `Windows Server Backup` · `copia del estado del sistema` · `DSRM` · `restauración autoritativa y no autoritativa` · `plan de recuperación ante desastres`
