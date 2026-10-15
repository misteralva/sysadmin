# 🧰 Laboratorio 13 · GPO avanzadas y contraseñas por grupo

**Nivel:** 🟠 Avanzado · **Sistema:** 🪟 Windows Server y cliente · **Tiempo estimado:** 100 minutos

---

## 🎯 Escenario

Con las GPO básicas ya puedes configurar equipos y usuarios por OU. Pero las empresas reales tienen necesidades más finas:

- Una política que se aplique solo a **un grupo concreto**, sin tener que mover usuarios entre OU.
- **Contraseñas más exigentes** para los administradores que para los empleados normales.
- Configuraciones que dependan de **la máquina** (por ejemplo, solo en portátiles).
- Un **recurso compartido** que cambie según el usuario, pero **siempre** en equipos de uso común.

Tu misión es dominar las herramientas que resuelven todo esto.

> 💾 **Snapshot** de `dc01` y `pc-cliente01`. Necesitas el dominio, las OU, los grupos `GG_Sistemas` y `GG_Redes` y las GPO básicas de los laboratorios 11 a 14 del nivel intermedio.

---

## 📖 Conceptos nuevos

### Filtrado de seguridad (*Security Filtering*)

Por defecto, una GPO se aplica a **todos los usuarios y equipos autenticados** (`Authenticated Users`) de la OU donde está vinculada. El **filtrado de seguridad** restringe esa aplicación a **grupos concretos**.

> **Analogía:** el aviso del tablón se entrega a todo el edificio; con el filtrado, **solo a quienes figuren en una lista**.

Detalle importante: para que una GPO se aplique, el usuario o equipo necesita permisos de **Leer** *y* **Aplicar directiva de grupo** sobre ella.

### Filtros WMI

Un **filtro WMI** hace que una GPO se aplique **solo si se cumple una condición del equipo**, consultada en lenguaje WQL. Ejemplo: solo en máquinas con Windows 11, o solo si el equipo tiene batería (portátil). Lo hace flexible pero **más lento**, así que se usa con moderación.

### Procesamiento de bucle invertido (*Loopback*)

Normalmente, la configuración de **usuario** de una GPO depende de dónde está el **usuario**. En **equipos compartidos** (aulas, quioscos, salas de reuniones) interesa lo contrario: que los ajustes dependan del **equipo**, sea quien sea el usuario que entre. Eso es el **loopback**, con dos modos:

| Modo | Qué hace |
| ---- | -------- |
| **Reemplazar** | Aplica **solo** la configuración de usuario de las GPO del **equipo** |
| **Combinar** | Aplica la del usuario **y además** la del equipo; si chocan, gana la del equipo |

### Directivas de contraseña de grano fino (PSO)

La **política de contraseñas del dominio** (en la *Default Domain Policy*) es **única** para todo el dominio: no se puede tener una distinta por OU. Para dar políticas distintas a grupos distintos se usan las **PSO** (*Password Settings Objects*, "política de contraseñas de grano fino"), que se **aplican a usuarios o grupos de seguridad**.

| Ejemplo | Empleados | Administradores |
| ------- | --------- | --------------- |
| Longitud mínima | 8 | 14 |
| Intentos fallidos antes de bloqueo | 10 | 3 |

Si un usuario tiene **varias** PSO aplicables, gana la de **menor valor de precedencia** (*precedence*).

### Herencia: Enforced y Block Inheritance

Ya los viste: **Block Inheritance** impide que una OU reciba GPO de las superiores; **Enforced** obliga a que una GPO se aplique **aunque se bloquee la herencia**. Si hay choque, **Enforced gana**.

### Herramientas y comandos de este laboratorio

| Herramienta / comando | Qué hace |
| --------------------- | -------- |
| `gpmc.msc` | Consola de GPO (filtrado de seguridad, filtros WMI, enforced...) |
| `dsac.exe` | **Centro de administración de Active Directory**: aquí se crean las PSO |
| `Get-ADFineGrainedPasswordPolicy -Filter *` | Lista las PSO |
| `New-ADFineGrainedPasswordPolicy -Name "PSO-Admins" -Precedence 10 -MinPasswordLength 14 ...` | Crea una PSO |
| `Add-ADFineGrainedPasswordPolicySubject -Identity "PSO-Admins" -Subjects "GG_Admins"` | Aplica la PSO a un grupo |
| `Get-ADUserResultantPasswordPolicy -Identity ana` | Muestra la PSO **que de verdad se aplica** a un usuario |
| `gpresult /r` / `gpresult /h informe.html` | Qué GPO se han aplicado (en el cliente) |
| `Backup-GPO -All -Path C:\BackupGPO` | Copia de seguridad de todas las GPO |
| `Restore-GPO -Name "nombre" -Path C:\BackupGPO` | Restaura una GPO |
| `Get-GPResultantSetOfPolicy` / **Modelado de directivas de grupo** | Simula qué GPO se aplicarían |

---

## 🛠️ Qué debes hacer

### Parte A · Filtrado de seguridad

1. Crea un grupo de seguridad global `GG_Admins` y añade a `ana`.
2. Crea una GPO `GPO-Herramientas-Admins` vinculada a la OU `Empresa`, que **oculte un aviso** o aplique un ajuste visible y fácil de comprobar (por ejemplo, un **fondo de pantalla** o mostrar **extensiones de archivo**).
3. En **Ámbito → Filtrado de seguridad**, **quita** `Authenticated Users` y **añade** solo `GG_Admins`.
4. **Importante:** en la pestaña **Delegación**, comprueba que `Authenticated Users` sigue teniendo permiso de **lectura** (si no, la directiva falla; este es un fallo clásico).
5. Comprueba en el cliente, con `ana` y con `marta`, que **solo `ana`** recibe la configuración (`gpresult /r`).

### Parte B · Filtro WMI

6. Crea un **filtro WMI** que solo se cumpla en equipos con sistema operativo **cliente** (investiga la consulta WQL sobre `Win32_OperatingSystem` y la propiedad `ProductType`).
7. Asócialo a una GPO y comprueba que **se aplica** en `pc-cliente01` y **no** en `dc01`.

### Parte C · Loopback

8. Crea una OU `Aula` y mueve a ella `pc-cliente01`.
9. Crea una GPO vinculada a `Aula` con una **configuración de usuario** (por ejemplo, restringir el Panel de control) y activa el **loopback en modo Reemplazar** (Configuración del equipo → Directivas → Plantillas administrativas → Sistema → Directiva de grupo → *Configurar el modo de procesamiento de bucle de reversa de la directiva de grupo de usuario*).
10. Inicia sesión en el cliente con **`ana`** (que normalmente no tiene la restricción): ahora **sí** la tiene, porque el equipo está en el `Aula`. Compruébalo.

### Parte D · Contraseñas de grano fino

11. Con el **Centro de administración de Active Directory**, crea dos PSO:
    - `PSO-Admins`: longitud mínima **14**, precedencia **10**, aplicada a `GG_Admins`.
    - `PSO-Empleados`: longitud mínima **8**, precedencia **20**, aplicada a `GG_Sistemas` y `GG_Redes`.
12. Comprueba con `Get-ADUserResultantPasswordPolicy` qué política se aplica a `ana` (¿pertenece a las dos?) y a `marta`.
13. Intenta **cambiar la contraseña** de `ana` a una de 10 caracteres: debe **rechazarse** (por la PSO de 14).

### Parte E · Seguridad operativa

14. Haz una **copia de seguridad de todas las GPO** con `Backup-GPO -All`.
15. **Elimina** una GPO de pruebas y **restáurala** desde el backup.
16. Usa **Modelado de directivas de grupo** (en `gpmc.msc`) para simular qué GPO se aplicarían a un usuario y equipo hipotéticos.

---

## ✅ Comprobación

| Prueba | Qué debes ver |
| ------ | ------------- |
| `gpresult /r` con `ana` | `GPO-Herramientas-Admins` en las aplicadas |
| `gpresult /r` con `marta` | La misma GPO en "**filtradas** (seguridad)" |
| GPO con filtro WMI | Se aplica en el cliente; en `dc01`, **denegada por filtro WMI** |
| `ana` en equipo del `Aula` | Tiene la restricción del Panel de control |
| `Get-ADUserResultantPasswordPolicy -Identity ana` | `PSO-Admins` (precedencia 10) |
| Cambio de contraseña a 10 caracteres | **Rechazado** |
| `Get-GPO -All` tras restaurar | Tu GPO de pruebas de vuelta |

---

## 🧩 Pistas

- Si una GPO filtrada **no se aplica a nadie**, casi siempre es la **Delegación**: falta el permiso de **lectura** a `Authenticated Users` (o a *Equipos del dominio*, si es de equipo).
- Una GPO de **usuario** filtrada por un grupo debe filtrar por el **grupo de usuarios**, no por el de equipos.
- Para el filtro WMI: `SELECT * FROM Win32_OperatingSystem WHERE ProductType = "1"` (1 = cliente, 2 = controlador de dominio, 3 = servidor).
- Tras mover un equipo de OU, **reinícialo** para que reciba las GPO nuevas.
- Las PSO se aplican a **grupos de seguridad globales o usuarios**, **no a OU**.
- Si una PSO no aparece aplicada, revisa la **precedencia** y a quién está asignada.

---

## 🚀 Retos extra

1. Haz que una GPO se aplique **solo en horario laboral** o a un grupo de **equipos** (no de usuarios) con filtrado de seguridad sobre un grupo de equipos.
2. Compara **Enforced** frente a **Block Inheritance**: crea un conflicto y averigua cuál gana.
3. **Delega** en `GG_Sistemas` el permiso para **crear GPO** (pestaña Delegación en *Objetos de directiva de grupo*), sin darles control total del dominio. ¿Por qué es mejor que añadirles al grupo de administradores del dominio?
4. Investiga las **plantillas administrativas ADMX** y el **almacén central** (`PolicyDefinitions` en SYSVOL).
5. Reflexiona: ¿cuántas GPO es **demasiado**? Revisa por qué es mala idea tener decenas de GPO pequeñas sin organización y cómo documentarlas.

---

## 📝 Qué documentar en tu `Solucion.md`

- Un esquema con las OU, las GPO y el filtrado aplicado a cada una.
- Capturas de `gpresult`, de las PSO y de `Get-ADUserResultantPasswordPolicy`.
- Con tus palabras: **cuándo usar** filtrado de seguridad, WMI, loopback o PSO.
- El fallo que tuviste (¿la delegación?) y cómo lo resolviste.

---

## 🧠 Lo que has practicado

`filtrado de seguridad` · `filtros WMI` · `loopback` · `PSO (contraseñas de grano fino)` · `precedencia` · `Enforced / Block Inheritance` · `backup y restauración de GPO` · `modelado de directivas`
