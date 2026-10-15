# 🔎 Laboratorio 09 · Auditoría de seguridad de Active Directory

**Nivel:** 🔴 Experto · **Sistema:** 🪟 Windows Server · **Tiempo estimado:** 120 minutos

---

## 🎯 Escenario

La empresa te pide una **revisión de seguridad** del directorio antes de una auditoría externa. En Active Directory, un par de malas costumbres acumuladas con los años (cuentas que nunca caducan, administradores de más, servicios con privilegios excesivos) bastan para que **un único fallo permita controlar todo el dominio**.

Tu misión es hacer una **auditoría defensiva** de tu propio dominio `empresa.local`: **encontrar** los puntos débiles con comandos de PowerShell, **clasificarlos por riesgo**, **corregir** los más graves y **repetir** la auditoría para demostrar la mejora.

> 💾 **Snapshot** de `dc01` y `pc-cliente01`. Todo se hace **sobre tu dominio de laboratorio**, con **solo lectura** en la fase de auditoría. **Nunca** audites un dominio ajeno sin autorización escrita.

---

## 📖 Conceptos nuevos

### Por qué se audita

El dominio es un "todo o nada": quien consigue control de **una cuenta con privilegios altos**, controla toda la red. Una auditoría busca **reducir ese riesgo** comprobando lo que realmente tienes configurado, no lo que crees tener.

### Qué se mira (áreas de revisión)

| Área | Qué se comprueba | Por qué importa |
| ---- | ---------------- | --------------- |
| **Cuentas privilegiadas** | Quién está en `Domain Admins`, `Enterprise Admins`, `Schema Admins` y `Administrators` | **Cuantos menos, mejor**: es el principio de **mínimo privilegio** |
| **Contraseñas** | Cuentas con `PasswordNeverExpires`, contraseñas muy antiguas, política del dominio | Una contraseña que nunca cambia, si se filtra, vale **para siempre** |
| **Cuentas inactivas o sobrantes** | Usuarios y equipos sin actividad desde hace meses; empleados que se fueron | Cuentas olvidadas = **puertas sin vigilar** |
| **Cuentas de servicio** | Usuarios con **SPN** (nombre de servicio), sus contraseñas y permisos | Suelen tener contraseñas débiles y privilegios de más |
| **La cuenta `krbtgt`** | Fecha de su último cambio de contraseña | Es la cuenta que sostiene **Kerberos**; si se compromete y nunca se cambia, el daño dura |
| **Delegación** | Equipos o cuentas con *delegación sin restricciones* | Permiten **suplantar** a otros usuarios en la red |
| **Protocolos antiguos** | SMBv1, NTLM, LDAP sin firma | Débiles y con vulnerabilidades conocidas |
| **Cuentas por defecto** | `Administrator`, `Guest` | `Guest` debe estar **desactivada**; `Administrator`, **controlada** |
| **Protecciones del directorio** | Papelera de AD, protección contra eliminación, copias de seguridad | Capacidad de **recuperarse** (avanzado, 15) |

### Severidad y priorización

No todo se arregla a la vez. Cada hallazgo se clasifica por **riesgo** = **impacto × facilidad de explotación**:

| Severidad | Criterio |
| --------- | -------- |
| 🔴 **Crítica** | Compromiso del dominio muy probable y de gran impacto |
| 🟠 **Alta** | Riesgo serio, pero necesita condiciones adicionales |
| 🟡 **Media** | Debilidad que **facilita** otros ataques |
| 🟢 **Baja** | Buena práctica incumplida, riesgo limitado |

### Cmdlets y comandos de este laboratorio

| Comando | Qué hace |
| ------- | -------- |
| `Get-ADGroupMember "Domain Admins" -Recursive \| Select-Object Name, objectClass` | Miembros (incluso los anidados) de un grupo privilegiado |
| `Get-ADUser -Filter * -Properties PasswordNeverExpires, PasswordLastSet, LastLogonDate, Enabled` | Datos de seguridad de las cuentas |
| `Get-ADUser -Filter 'PasswordNeverExpires -eq $true'` | Cuentas cuya contraseña **nunca caduca** |
| `Search-ADAccount -AccountInactive -TimeSpan 90.00:00:00 -UsersOnly` | Cuentas **inactivas** más de 90 días |
| `Get-ADUser -Filter 'ServicePrincipalName -like "*"' -Properties ServicePrincipalName` | Cuentas con **SPN** (de servicio) |
| `Get-ADUser krbtgt -Properties PasswordLastSet` | Edad de la contraseña de `krbtgt` |
| `Get-ADComputer -Filter 'TrustedForDelegation -eq $true'` | Equipos con **delegación sin restricciones** (los DC aparecerán: es lo esperado) |
| `Get-ADDefaultDomainPasswordPolicy` | Política de contraseñas del dominio |
| `Get-SmbServerConfiguration \| Select-Object EnableSMB1Protocol` | Si **SMBv1** está activo en un servidor |
| `Get-ADOptionalFeature -Filter 'Name -like "Recycle*"'` | Estado de la papelera de AD |
| `Get-ADUser Guest \| Select-Object Enabled` | Si la cuenta `Guest` está activa |
| `Get-ADOrganizationalUnit -Filter * -Properties ProtectedFromAccidentalDeletion` | OU protegidas |

---

## 🛠️ Qué debes hacer

### Parte A · Preparar el terreno ("ensuciar" el laboratorio)

Un dominio de laboratorio es demasiado limpio para auditarlo. **Crea defectos realistas** para tener algo que encontrar:

1. Crea una cuenta `svc-backup` con la contraseña **que nunca caduca** y añádela (error típico) a un grupo con **privilegios elevados**.
2. Crea un usuario `exempleado` que **no se haya usado** (cuenta activa sin actividad) y otro, `invitado2`, **en un grupo que no le corresponde**.
3. Crea una cuenta de servicio con un **SPN** (investiga `setspn -S` o `Set-ADUser -ServicePrincipalNames`).
4. Deja `Guest` **activada** o la contraseña de algún usuario como **muy antigua** (si puedes simularlo).
5. Haz **snapshot** ahora: es tu **"estado sucio"** de referencia.

### Parte B · Auditar (solo lectura)

6. Ejecuta **cada comprobación** de la tabla y **guarda la salida** en archivos (`Export-Csv`/`Out-File`) con fecha.
7. Escribe un script `Audit-AD.ps1` que **automatice** todas las comprobaciones y genere un **informe único** en HTML o CSV.
8. Revisa los resultados e **identifica los hallazgos**: algunos serán **los que has creado a propósito**, y otros pueden ser **sorpresas reales** de tu dominio.

### Parte C · Clasificar

9. Rellena una **tabla de hallazgos** (plantilla abajo) con **cada problema**: **descripción, evidencia, severidad, riesgo y recomendación**.
10. **Prioriza**: ordena por severidad y decide **qué corregir primero**.

### Parte D · Corregir y volver a auditar

11. **Corrige**, al menos, **cinco hallazgos** (por ejemplo: quitar al servicio de un grupo privilegiado, activar caducidad de contraseña o rotarla, desactivar cuentas inactivas, desactivar `Guest`, restringir miembros de `Domain Admins`...).
12. **Cambia con cuidado** y **verifica** que nada se rompe (los servicios siguen funcionando).
13. **Vuelve a ejecutar** tu script de auditoría y **compara** el "antes" con el "después".
14. Redacta el **informe ejecutivo** (una página) y el **plan de remediación** para los hallazgos pendientes.

---

## ✅ Comprobación

| Prueba | Qué debes ver |
| ------ | ------------- |
| Salida de las comprobaciones | Guardada y con fecha, para **cada área** |
| `Audit-AD.ps1` | Genera un informe con **todas** las áreas |
| Tabla de hallazgos | **Mínimo diez** filas, con severidad y recomendación |
| Tras corregir | El script muestra **menos hallazgos** que antes |
| Servicios | Siguen funcionando (comprobado) |
| Informe ejecutivo | Entendible por **alguien no técnico** |

---

## 🧩 Pistas

- Los **controladores de dominio** aparecen en la lista de "delegación sin restricciones" **por diseño**; el hallazgo es cualquier **otro** equipo ahí.
- Para encontrar usuarios **anidados** en grupos privilegiados, usa siempre `-Recursive`.
- **No corrijas** sin comprobar antes **qué usa esa cuenta**: quitar privilegios a una cuenta de servicio puede **romper un servicio**. Primero entiende, luego cambia.
- Si una contraseña de servicio se rota, **hay que actualizarla** en el servicio que la usa.
- `PasswordLastSet` y `LastLogonDate` son las propiedades clave para la **antigüedad**.
- Un **hallazgo sin evidencia** no sirve: guarda siempre la **salida del comando** como prueba.

---

## 🚀 Retos extra

1. Instala y ejecuta una herramienta gratuita de evaluación de AD (por ejemplo **PingCastle**, **solo en tu dominio de laboratorio**) y **compara** su informe con el tuyo: ¿qué detecta que tu script no?
2. Investiga **LAPS** (*Local Administrator Password Solution*): ¿qué problema resuelve (la misma contraseña de administrador local en todos los equipos) y cómo se despliega?
3. Investiga el **modelo de administración por niveles** (*tiering*): ¿por qué un administrador de dominio no debería iniciar sesión en equipos de usuario?
4. Activa y revisa la **auditoría de seguridad de eventos** (inicios de sesión fallidos, cambios en grupos privilegiados, eventos `4720`, `4728`, `4732`) y búscalos en el **Visor de eventos**. Conecta con el laboratorio 04 (logs centralizados).
5. Investiga la **política de contraseñas de grano fino** (avanzado, 13) como forma de dar **requisitos más fuertes** a las cuentas privilegiadas.

---

## 📝 Qué documentar en tu `Solucion.md`

**Plantilla de tabla de hallazgos:**

| # | Hallazgo | Evidencia (comando y salida) | Severidad | Riesgo (por qué importa) | Recomendación | Estado |
| - | -------- | ---------------------------- | --------- | ------------------------ | ------------- | ------ |

Además: tu script `Audit-AD.ps1` comentado, la comparación **antes/después** y el **informe ejecutivo**.

---

## 🧠 Lo que has practicado

`auditoría de AD` · `mínimo privilegio` · `cuentas privilegiadas y de servicio` · `krbtgt` · `delegación` · `política de contraseñas` · `severidad y priorización` · `remediación` · `informe ejecutivo`
