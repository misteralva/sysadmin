# ⚡ Laboratorio 14 · PowerShell: Active Directory y administración remota

**Nivel:** 🟠 Avanzado · **Sistema:** 🪟 Windows Server y cliente · **Tiempo estimado:** 120 minutos

---

## 🎯 Escenario

El departamento de RR. HH. te envía un archivo con **40 empleados nuevos** que hay que dar de alta en el dominio. Después, el director quiere un **informe semanal**: cuentas inactivas, cuentas bloqueadas y usuarios sin grupo. Y, además, necesitas ejecutar comandos en los equipos **sin sentarte delante de ellos**.

Hacerlo a mano sería tedioso y propenso a errores. Tu misión es **automatizarlo todo con PowerShell**, usando lo aprendido en el nivel 1 (laboratorio 11) y en los laboratorios de Active Directory.

> 💾 **Snapshot** de `dc01` y `pc-cliente01`. Necesitas el dominio, las OU y los grupos del nivel intermedio, con `dc01` y el cliente encendidos.

---

## 📖 Conceptos nuevos

### Del comando suelto al script

Un **script** de PowerShell (`.ps1`) es una secuencia de comandos guardada en un archivo, a la que puedes añadir **variables, bucles, condiciones y funciones**. Es lo mismo que hiciste en Bash, con una sintaxis distinta y trabajando con **objetos**.

### Estructuras que necesitas

| Estructura | Sintaxis | Para qué |
| ---------- | -------- | -------- |
| Variable | `$nombre = "Ana"` | Guardar datos |
| Bucle `foreach` | `foreach ($u in $usuarios) { ... }` | Repetir para cada elemento |
| Condición | `if ($x -gt 5) { ... } else { ... }` | Decidir |
| Función | `function Nuevo-Alta { param($Nombre) ... }` | Reutilizar código |
| Gestión de errores | `try { ... } catch { ... }` | Capturar fallos sin que el script se rompa |
| Parámetro de error | `-ErrorAction Stop` | Hace que un error se convierta en excepción capturable por `catch` |

### Importar datos de un CSV

`Import-Csv archivo.csv` lee un archivo CSV y lo convierte en **objetos**, uno por fila, con una **propiedad por columna**. Ejemplo de CSV (`empleados.csv`):

```csv
Nombre,Apellido,Usuario,Departamento
Carlos,Ruiz,cruiz,Sistemas
Lucía,Pérez,lperez,Redes
```

Después puedes recorrerlo con `foreach` y usar `$u.Nombre`, `$u.Usuario`...

### Cmdlets de Active Directory más útiles

| Cmdlet | Qué hace |
| ------ | -------- |
| `New-ADUser` | Crea un usuario |
| `Get-ADUser -Filter *` / `-Identity` | Consulta usuarios |
| `Set-ADUser` / `Disable-ADAccount` / `Enable-ADAccount` | Modifica / desactiva / activa |
| `Unlock-ADAccount` | Desbloquea una cuenta bloqueada |
| `Search-ADAccount -LockedOut` | Busca cuentas **bloqueadas** |
| `Search-ADAccount -AccountInactive -TimeSpan 90.00:00:00` | Cuentas **sin actividad en 90 días** |
| `Add-ADGroupMember` | Añade a un grupo |
| `Set-ADAccountPassword -Reset` | Restablece una contraseña |

### Contraseñas iniciales: no las pongas en el script

Un script con contraseñas **en claro** es un riesgo grave: queda en el disco, en el historial y en el repositorio. Alternativas: **generar una contraseña aleatoria por usuario** y exigir **cambiarla en el primer inicio** (`-ChangePasswordAtLogon $true`), o pedirla de forma interactiva con `Read-Host -AsSecureString`.

### Administración remota con PowerShell Remoting

PowerShell puede ejecutar comandos **en otros equipos** por red mediante **WinRM**:

| Cmdlet | Qué hace |
| ------ | -------- |
| `Enable-PSRemoting -Force` | **Habilita** el remoto en un equipo (se ejecuta en el equipo destino) |
| `Test-WSMan NOMBRE` | Comprueba si un equipo acepta conexiones remotas |
| `Invoke-Command -ComputerName PC -ScriptBlock { ... }` | Ejecuta un bloque de código en uno o **varios** equipos |
| `Enter-PSSession -ComputerName PC` | Abre una **sesión interactiva** remota (sales con `Exit-PSSession`) |

### Otras herramientas útiles

| Cmdlet | Qué hace |
| ------ | -------- |
| `Start-Transcript -Path log.txt` / `Stop-Transcript` | Guarda **todo** lo que ocurre en la consola en un archivo (auditoría) |
| `ConvertTo-Html` / `Export-Csv` | Genera informes |
| `Register-ScheduledTask` | Programa un script (como `cron` en Linux) |
| `Get-Date -Format "yyyy-MM-dd"` | Fecha para nombres de archivo |

---

## 🛠️ Qué debes hacer

### Parte A · Alta masiva de usuarios

1. Crea un CSV `empleados.csv` con **10 empleados ficticios** (nombre, apellido, usuario, departamento: `Sistemas` o `Redes`).
2. Escribe un script `Alta-Usuarios.ps1` que, para **cada fila**:
   - cree el usuario en la **OU del departamento** correspondiente (`OU=Sistemas,OU=Empresa,DC=empresa,DC=local`),
   - le genere una **contraseña aleatoria** (con letras, números y símbolos) y obligue a **cambiarla** al primer inicio,
   - lo añada al **grupo** `GG_Sistemas` o `GG_Redes`,
   - **registre** en un archivo la línea "creado/error" para cada usuario (sin guardar las contraseñas en el log).
3. Usa `try/catch` para que, si un usuario ya existe, el script **siga con el siguiente** y anote el error.
4. Ejecútalo y **comprueba** que se han creado los 10. **Vuélvelo a ejecutar**: ¿qué ocurre? ¿Es idempotente? (Mejora el script para que compruebe si el usuario existe antes de crearlo.)

### Parte B · Informe de auditoría

5. Escribe `Informe-AD.ps1` que genere un **informe** con:
   - usuarios **desactivados**,
   - cuentas **bloqueadas**,
   - cuentas **inactivas** (usa un periodo corto en el laboratorio, por ejemplo 1 día, para que haya resultados),
   - usuarios que **no pertenecen a ningún grupo** de la empresa.
6. Exporta cada apartado a un **CSV**, y genera además un resumen en **HTML** con la fecha en el nombre del archivo.
7. Para probarlo: desactiva a un usuario, bloquea otro (intentando contraseñas erróneas en el cliente) y comprueba que salen en el informe.

### Parte C · Remoting

8. En `pc-cliente01`, habilita el remoto con `Enable-PSRemoting -Force`. Comprueba desde `dc01` con `Test-WSMan`.
9. Desde `dc01`, ejecuta **un comando remoto** con `Invoke-Command` para obtener el **nombre del equipo**, el **espacio libre en `C:`** y los **cinco procesos que más memoria usan** del cliente.
10. Abre una **sesión interactiva** remota con `Enter-PSSession`, explora, y cierra la sesión.
11. Ejecuta el mismo comando en **varios equipos a la vez** (`-ComputerName DC01,PC-CLIENTE01`) y observa cómo cada resultado indica de qué equipo viene (`PSComputerName`).

### Parte D · Automatizar

12. **Programa** `Informe-AD.ps1` con una **tarea programada** (`Register-ScheduledTask`) que se ejecute **cada semana**, o cada hora para probarlo, y guarde el informe en una carpeta.
13. Envuelve tus scripts con `Start-Transcript` / `Stop-Transcript` para tener un registro de **qué se hizo y cuándo**.

---

## ✅ Comprobación

| Prueba | Qué debes ver |
| ------ | ------------- |
| `Get-ADUser -Filter * -SearchBase "OU=Sistemas,OU=Empresa,DC=empresa,DC=local" \| Measure-Object` | El número de usuarios creados en esa OU |
| `Get-ADGroupMember GG_Redes` | Los usuarios de Redes del CSV |
| Segunda ejecución del alta | **Sin duplicados** y errores controlados en el log |
| Informe HTML y CSV | Archivos con la fecha en el nombre y los usuarios problemáticos |
| `Invoke-Command -ComputerName PC-CLIENTE01 -ScriptBlock { hostname }` | `PC-CLIENTE01` |
| `Get-ScheduledTask -TaskName "Informe-AD"` | La tarea, con su programación |
| Log de transcripción | Los comandos ejecutados |

---

## 🧩 Pistas

- Para una **contraseña aleatoria**, mira `[System.Web.Security.Membership]::GeneratePassword(14,3)` o genera una combinando caracteres con `Get-Random`. Conviértela a `SecureString` con `ConvertTo-SecureString -AsPlainText -Force`.
- `Get-ADUser -Filter "SamAccountName -eq '$usuario'"` te dice si un usuario existe (en un `-Filter` se usa la sintaxis de AD, no la de PowerShell).
- Si `Invoke-Command` falla con "Access denied" o "WinRM no puede procesar", revisa que has ejecutado `Enable-PSRemoting` **en el destino** y que el cortafuegos de Windows lo permite.
- Un `try/catch` solo captura **errores terminantes**: añade `-ErrorAction Stop` al cmdlet para que sus errores lo sean.
- En scripts, evita `Write-Host` para datos: devuelve **objetos** y deja que el pipeline decida cómo mostrarlos o exportarlos.
- Si no puedes ejecutar tus `.ps1`, revisa la **política de ejecución** (`Get-ExecutionPolicy`; laboratorio 11 del nivel 1).

---

## 🚀 Retos extra

1. Haz que el script de alta **envíe las contraseñas** de forma segura al responsable (investiga cómo; aquí basta con **razonar** qué sería seguro y qué no).
2. Crea una **función reutilizable** `New-EmpleadoAD` con parámetros validados (`[Parameter(Mandatory)]`, `[ValidateSet('Sistemas','Redes')]`).
3. Desactiva automáticamente las cuentas **inactivas más de 90 días** y muévelas a una OU `Desactivados` (con confirmación previa mediante `-WhatIf`, que **simula** sin ejecutar).
4. Usa `-WhatIf` en todos tus cmdlets destructivos mientras pruebas.
5. Conecta con el nivel 1: escribe una **tabla de equivalencias** entre tu script de Bash que crea usuarios (si lo has hecho) y este, y reflexiona sobre las diferencias entre texto y objetos.

---

## 📝 Qué documentar en tu `Solucion.md`

- Tus scripts, **comentados línea a línea**.
- El resultado de la segunda ejecución del alta y cómo la hiciste idempotente.
- Un ejemplo del informe HTML.
- Con tus palabras: qué es **WinRM**, qué riesgos tiene dejarlo habilitado y cómo se limitaría.

---

## 🧠 Lo que has practicado

`scripts .ps1` · `Import-Csv` · `foreach / if / try-catch` · `cmdlets de Active Directory` · `contraseñas seguras` · `PowerShell Remoting (WinRM)` · `Invoke-Command / Enter-PSSession` · `informes HTML/CSV` · `tareas programadas` · `Start-Transcript`
