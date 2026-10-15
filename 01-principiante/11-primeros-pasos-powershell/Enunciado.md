# ⚡ Laboratorio 11 · Primeros pasos con PowerShell

**Nivel:** 🟢 Principiante · **Sistema:** 🪟 Windows Server · **Tiempo estimado:** 75 minutos

---

## 🎯 Escenario

Hasta ahora has gestionado Windows Server con el ratón. Es cómodo para una vez, pero en una empresa tendrás que crear 50 usuarios, comprobar 20 servidores o generar un informe cada semana. Para eso existe **PowerShell**: la herramienta de línea de comandos y automatización de Windows.

Tu misión es empezar a hacer con comandos lo que ya sabes hacer con el ratón, y generar un pequeño informe del servidor.

> 💾 Haz un **snapshot** antes de empezar. Vas a crear usuarios y modificar cosas con comandos.

---

## 📖 Conceptos nuevos

### Qué es PowerShell

Es una consola como `cmd` (el Símbolo del sistema de toda la vida), pero mucho más potente. Se abre desde el menú Inicio. Para tareas de administración, ábrela **como administrador** (clic derecho → *Ejecutar como administrador*).

### Cmdlets: verbo-sustantivo

Los comandos de PowerShell se llaman **cmdlets** y siguen siempre el patrón **`Verbo-Sustantivo`**:

| Cmdlet | Qué significa |
| ------ | ------------- |
| `Get-Process` | **Obtener** los procesos |
| `Get-Service` | **Obtener** los servicios |
| `New-LocalUser` | **Crear** un usuario local |
| `Remove-Item` | **Eliminar** un elemento |

Una vez aprendes los verbos (`Get`, `Set`, `New`, `Remove`, `Start`, `Stop`…), puedes adivinar cmdlets que nunca has visto.

### PowerShell trabaja con objetos, no con texto

En Bash (laboratorio 06) un comando devuelve **texto** que el siguiente tiene que interpretar. En PowerShell, un comando devuelve **objetos**: cosas con propiedades (nombre, tamaño, estado…).

> **Analogía:** en Bash pasas una **nota escrita a mano** al siguiente comando, y este tiene que descifrarla. En PowerShell pasas un **paquete etiquetado**, y el siguiente sabe exactamente qué contiene cada compartimento.

Por eso se puede **filtrar y ordenar por propiedades** con mucha facilidad.

### El pipeline `|`

Igual que en Linux, la tubería (`|`) pasa la salida de un comando como entrada del siguiente. Es la herramienta más importante de PowerShell:

```
Get-Process | Sort-Object CPU -Descending | Select-Object -First 5
```

Se lee así: *obtén los procesos → ordénalos por CPU de mayor a menor → quédate con los 5 primeros.*

### Cmdlets de este laboratorio

**Para aprender y explorar:**

| Cmdlet | Qué hace |
| ------ | -------- |
| `Get-Help cmdlet` | Muestra la ayuda de un cmdlet (`-Examples` añade ejemplos) |
| `Get-Command *palabra*` | Busca cmdlets por parte del nombre |
| `Get-Member` | Muestra las **propiedades y métodos** de un objeto (úsalo tras un `\|`) |

**Archivos (equivalentes a `ls`, `cd`, `cp`... de Linux):**

| Cmdlet | Equivale a | Qué hace |
| ------ | ---------- | -------- |
| `Get-ChildItem` | `ls` | Lista el contenido de una carpeta |
| `Set-Location` | `cd` | Cambia de carpeta |
| `New-Item` | `touch` / `mkdir` | Crea archivos o carpetas (`-ItemType Directory` para carpetas) |
| `Copy-Item` | `cp` | Copia |
| `Move-Item` | `mv` | Mueve o renombra |
| `Remove-Item` | `rm` | Borra |
| `Get-Content` | `cat` | Muestra el contenido de un archivo |

> 💡 PowerShell incluye **alias** para que Linux y cmd te resulten familiares: `ls`, `cd`, `cat`, `cp` funcionan. Pero en scripts se recomienda usar el nombre completo.

**Sistema:**

| Cmdlet | Qué hace |
| ------ | -------- |
| `Get-Process` / `Stop-Process` | Ver / terminar procesos |
| `Get-Service` / `Start-Service` / `Stop-Service` | Ver / iniciar / detener servicios |
| `Get-LocalUser` / `New-LocalUser` | Ver / crear usuarios locales |
| `Get-LocalGroup` / `Add-LocalGroupMember` | Ver grupos / añadir miembros |

**Manipular resultados:**

| Cmdlet | Qué hace |
| ------ | -------- |
| `Select-Object` | Elige propiedades (columnas) o los primeros/últimos N elementos |
| `Where-Object` | **Filtra** elementos por una condición |
| `Sort-Object` | **Ordena** por una propiedad |
| `Measure-Object` | Cuenta, suma o calcula medias |
| `Export-Csv` | Guarda el resultado en un archivo CSV |

### Variables

Empiezan por `$`: `$nombre = "Ana"`. Para pedir un dato al usuario: `Read-Host "Pregunta"`; para una contraseña, `Read-Host "Contraseña" -AsSecureString` (la escribes sin que se vea y queda guardada de forma segura).

### Operadores de comparación

En PowerShell **no se usa `>` ni `==`** para comparar. Se usan:

| Operador | Significa |
| -------- | --------- |
| `-eq` | Igual a |
| `-ne` | Distinto de |
| `-gt` / `-lt` | Mayor que / Menor que |
| `-like` | Coincide con un patrón (con `*`) |

(Si has hecho el repositorio de Bash, verás que te suenan: `-eq`, `-lt`, `-gt`.)

### Política de ejecución

Por seguridad, Windows controla si se pueden ejecutar scripts `.ps1`. `Get-ExecutionPolicy` te dice el nivel actual. Más adelante, para tus propios scripts, tendrás que revisarlo.

---

## 🛠️ Qué debes hacer

### Parte A · Explorar

1. Abre PowerShell **como administrador**.
2. Usa `Get-Help` para ver la ayuda de `Get-Process` con ejemplos. Después, con `Get-Command`, busca todos los cmdlets que contengan la palabra `Service`.
3. Ejecuta `Get-Process | Get-Member` y localiza dos propiedades del objeto (pista: busca `CPU` y `Id`).

### Parte B · Archivos

4. Crea en `C:\` esta estructura con **solo cmdlets** (usa `New-Item`):

```
C:\Informes\
├── Servicios\
└── Usuarios\
```

5. Crea el archivo `prueba.txt` dentro de `C:\Informes`, escribe algo con `Set-Content` o `Add-Content`, cópialo, renómbralo y bórralo.

### Parte C · Informes con el pipeline

6. Muestra los **5 procesos que más memoria consumen** (propiedad `WorkingSet`), con solo las columnas `Name`, `Id` y `WorkingSet`.
7. Cuenta cuántos **servicios** están en estado `Running`.
8. Muestra los servicios **detenidos** cuyo nombre empiece por `W`. (Pista: `Where-Object` con `-eq` y `-like`.)
9. Exporta la lista de servicios (nombre, estado) a `C:\Informes\Servicios\servicios.csv` con `Export-Csv -NoTypeInformation` (esa opción evita una línea de cabecera de tipo que no te interesa).
10. Abre el CSV con `Get-Content` para comprobar que se ha escrito bien.

### Parte D · Usuarios con PowerShell

11. Crea un usuario nuevo llamado `pablo` con `New-LocalUser`. Necesitarás una contraseña **segura**: pídela con `Read-Host -AsSecureString` y guárdala en una variable.
12. Añade a `pablo` al grupo `Sistemas` (el del laboratorio 10) con `Add-LocalGroupMember`.
13. Comprueba con `Get-LocalGroupMember -Group "Sistemas"` que aparece.
14. Elimina a `pablo` con `Remove-LocalUser`.

---

## ✅ Comprobación

| Comando | Qué debes ver |
| ------- | ------------- |
| `Test-Path C:\Informes\Servicios` | `True` (`Test-Path` dice si una ruta existe) |
| `Get-ChildItem C:\Informes\Servicios` | El archivo `servicios.csv` |
| `Get-Content C:\Informes\Servicios\servicios.csv -TotalCount 5` | Las 5 primeras líneas del CSV |
| `Get-LocalGroupMember -Group "Sistemas"` | `ana`, `luis` (y `pablo` mientras exista) |
| `Get-LocalUser \| Select-Object Name, Enabled` | Tus usuarios con su estado |

---

## 🧩 Pistas

- Si no recuerdas el nombre de un cmdlet, **no lo busques en internet**: prueba `Get-Command *LocalUser*` o `Get-Command -Verb New`.
- `Where-Object` acepta una forma corta: `Where-Object Status -eq 'Running'`. O la larga con llaves: `Where-Object { $_.Status -eq 'Running' }` (`$_` significa "el elemento actual").
- Si un comando te da un error en rojo, **léelo completo**: PowerShell suele decir exactamente qué falta o dónde falló.
- El tabulador autocompleta cmdlets, parámetros y rutas. Úsalo siempre.
- Para ver **todas** las propiedades de un objeto: `... | Format-List *`.
- `Get-Service` devuelve el estado en una propiedad llamada `Status`; `Get-Process` guarda la memoria en `WorkingSet`.

---

## 🚀 Retos extra

1. **Haz un script.** Guarda en `C:\Informes\informe.ps1` los comandos de la Parte C para que generen un informe de servicios y procesos con la fecha en el nombre del archivo (pista: `Get-Date -Format "yyyy-MM-dd"`). Comprueba la política de ejecución para poder lanzarlo.
2. **Crea varios usuarios de golpe** a partir de una lista (`$nombres = "u1","u2","u3"`) con un bucle `foreach`. Esto conecta con el bucle `for` de Bash y de Python.
3. Investiga `Get-WinEvent -LogName System -MaxEvents 10` y consigue **los 10 últimos eventos** del sistema. Es el equivalente a `journalctl` de Linux.
4. Comprueba la conectividad con `Test-NetConnection 8.8.8.8 -Port 53`. Es una mezcla de `ping` y de comprobar un puerto, algo muy útil en diagnóstico.
5. Haz una **tabla de equivalencias** Bash ↔ PowerShell con al menos 10 comandos y súbela a tu `Solucion.md`.

---

## 📝 Qué documentar en tu `Solucion.md`

- Cada comando que usaste, explicado por partes (cmdlet, parámetros y qué hace el pipeline).
- Tu tabla de equivalencias entre Bash y PowerShell.
- Qué diferencia has notado entre trabajar con **texto** (Bash) y con **objetos** (PowerShell).
- El apartado **"Qué salió mal"**.

---

## 🧠 Lo que has practicado

`cmdlets Verbo-Sustantivo` · `objetos y pipeline` · `Get-Help / Get-Command / Get-Member` · `Where-Object / Select-Object / Sort-Object` · `Export-Csv` · `New-LocalUser` · `variables y SecureString` · `operadores -eq / -like` · `ExecutionPolicy`
