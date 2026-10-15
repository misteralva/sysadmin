# 🗂️ Laboratorio 10 · Usuarios locales y carpetas compartidas

**Nivel:** 🟢 Principiante · **Sistema:** 🪟 Windows Server · **Tiempo estimado:** 75 minutos

---

## 🎯 Escenario

La empresa necesita un **servidor de archivos** sencillo. Los empleados de cada departamento deben acceder a su carpeta, y solo a la suya:

| Empleado | Departamento | Qué necesita |
| -------- | ------------ | ------------ |
| `ana` | Sistemas | Leer y escribir en `Sistemas`; leer y escribir en `Comun` |
| `luis` | Sistemas | Igual que `ana` |
| `marta` | Redes | **No** puede entrar en `Sistemas`; solo **leer** en `Comun` |

Es el mismo escenario que en los laboratorios 03 y 04 de Linux, pero ahora con las herramientas de Windows. Compáralos al terminar.

> 💾 Haz un **snapshot** antes de empezar. Los permisos mal puestos pueden bloquearte el acceso a tus propias carpetas.

---

## 📖 Conceptos nuevos

### Usuarios y grupos locales

Un servidor Windows que **no** está en un dominio guarda sus propios usuarios y grupos: son las cuentas **locales**. Funciona igual que en Linux: se crean usuarios, se agrupan, y se dan permisos a los grupos.

Grupos que ya existen y debes conocer:

| Grupo | Para qué sirve |
| ----- | -------------- |
| `Administrators` | Pueden hacer cualquier cosa en el equipo (como `sudo` en Linux) |
| `Users` | Usuarios normales |
| `Remote Desktop Users` | Pueden conectarse por escritorio remoto |

### Recurso compartido (*share*)

Una carpeta **compartida** es una carpeta accesible desde la red mediante el protocolo **SMB**. Se accede con una ruta como:

```
\\SRV-WIN01\Empresa
```

(el nombre del servidor y el del recurso compartido).

### Las dos capas de permisos

Aquí hay una diferencia importante con Linux. Un archivo compartido pasa por **dos controles**:

| Capa | Qué controla | Analogía |
| ---- | ------------ | -------- |
| **Permisos del recurso compartido** | Quién puede entrar **por la red** | La puerta del edificio |
| **Permisos NTFS** | Quién puede hacer qué con cada carpeta y archivo | La puerta de cada despacho |

Si ambas capas dicen cosas distintas, el acceso real es el **más restrictivo** de las dos. Por eso la práctica habitual es: dejar el recurso compartido abierto a los usuarios autenticados (por ejemplo, **Cambiar**) y controlar el detalle con **NTFS**.

### Permisos NTFS básicos

| Permiso | Qué permite |
| ------- | ----------- |
| **Control total** | Todo, incluido cambiar los permisos |
| **Modificar** | Leer, escribir, crear y borrar |
| **Lectura y ejecución** | Leer y ejecutar |
| **Lectura** | Solo leer |
| **Escritura** | Crear y escribir (sin leer) |

Cosas que debes saber:

- Los permisos se pueden asignar a **usuarios** o a **grupos** (mejor a grupos, como en Linux).
- **Herencia:** por defecto, una subcarpeta hereda los permisos de la carpeta padre.
- **Denegar** (*Deny*) gana siempre sobre **Permitir**. Úsalo con muchísimo cuidado: es fácil bloquearse.

### Comandos de este laboratorio

| Comando | Qué hace |
| ------- | -------- |
| `whoami` | Muestra con qué usuario estás trabajando |
| `net user` | Lista los usuarios locales |
| `net localgroup` | Lista los grupos locales (`net localgroup Sistemas` muestra los miembros de uno) |
| `Get-SmbShare` | Lista los recursos compartidos del servidor (en PowerShell) |
| `icacls "C:\ruta"` | Muestra los permisos NTFS de una carpeta. Las siglas son: `F` = Control total, `M` = Modificar, `RX` = Lectura y ejecución, `R` = Lectura, `W` = Escritura |
| `runas /user:SRV-WIN01\ana cmd` | Abre una ventana de comandos **como si fueras** ese usuario (equivale a `sudo -u` en Linux) |
| `lusrmgr.msc` | Abre la herramienta gráfica de **usuarios y grupos locales** (se ejecuta desde *Ejecutar* o desde el buscador) |

---

## 🛠️ Qué debes hacer

### Parte A · Usuarios y grupos

1. Abre **Usuarios y grupos locales** (`lusrmgr.msc`).
2. **Crea los grupos** `Sistemas` y `Redes`.
3. **Crea los usuarios** `ana`, `luis` y `marta`, con una contraseña compleja. Desmarca la opción de que cambien la contraseña en el primer inicio de sesión (así será más fácil probar).
4. **Añádelos** a sus grupos: `ana` y `luis` a `Sistemas`, `marta` a `Redes`.
5. Comprueba la pertenencia con `net localgroup Sistemas` en PowerShell.

### Parte B · Carpetas

6. Crea esta estructura:

```
C:\Empresa\
├── Sistemas\
└── Comun\
```

### Parte C · Permisos NTFS

7. En `C:\Empresa\Sistemas`, abre *Propiedades → Seguridad → Opciones avanzadas*.
8. **Deshabilita la herencia** y convierte los permisos heredados en explícitos (para poder quitar permisos que vienen de `C:\`).
9. Deja únicamente:
   - `Administrators` y `SYSTEM`: Control total.
   - `Sistemas`: Modificar.

   Quita a `Users` y a cualquier otro grupo.
10. En `C:\Empresa\Comun`, haz lo mismo y deja:
    - `Administrators` y `SYSTEM`: Control total.
    - `Sistemas`: Modificar.
    - `Redes`: Lectura y ejecución.

### Parte D · Recurso compartido

11. Comparte `C:\Empresa` con el nombre `Empresa`. En los **permisos del recurso compartido**, deja a `Authenticated Users` (Usuarios autenticados) con **Cambiar**, y a `Administrators` con **Control total**. El detalle fino ya lo controla NTFS.
12. Comprueba con `Get-SmbShare` que aparece el recurso.

### Parte E · Probar

13. Prueba los permisos con `runas`, como `ana` y como `marta`:
    - Abre una ventana como cada uno y prueba a **listar** y **crear** archivos en `Sistemas` y en `Comun`.
14. En las propiedades de `Sistemas`, ve a *Opciones avanzadas → Acceso efectivo* y comprueba los permisos que el sistema calcula para `marta`.

---

## ✅ Comprobación

| Prueba | Qué debes ver |
| ------ | ------------- |
| `net localgroup Sistemas` | `ana` y `luis` |
| `Get-SmbShare` | Un recurso llamado `Empresa` |
| `icacls "C:\Empresa\Sistemas"` | `Sistemas:(OI)(CI)(M)`, y **sin** `Users` |
| `ana` en `Sistemas` | Puede listar y crear archivos |
| `marta` en `Sistemas` | **Acceso denegado** |
| `marta` en `Comun` | Puede leer, pero al crear un archivo: **acceso denegado** |
| `ana` en `Comun` | Puede leer y crear |

Ejemplo de prueba como `marta`:

```
runas /user:SRV-WIN01\marta cmd
```

(te pedirá su contraseña y abrirá una consola; allí ejecuta `dir C:\Empresa\Sistemas` y `echo prueba > C:\Empresa\Comun\test.txt`).

---

## 🧩 Pistas

- Si al quitar `Users` de la carpeta te dice que no puedes porque son heredados, es que no has deshabilitado la herencia (paso 8).
- Si te bloqueas tú mismo, entra con `Administrator`: los administradores pueden **tomar posesión** de una carpeta desde *Opciones avanzadas → Propietario*.
- Si `runas` no funciona, comprueba que escribes bien el nombre completo `SRV-WIN01\marta` (con el nombre real de tu servidor).
- Para ver el nombre de tu equipo: `hostname`.
- Si la consola de `runas` se cierra al instante, revisa que la contraseña sea correcta.

---

## 🚀 Retos extra

1. Repite la **Parte A** usando solo PowerShell. Pistas: `New-LocalUser`, `New-LocalGroup`, `Add-LocalGroupMember` (los usarás en el laboratorio 11).
2. Aplica permisos con **`icacls`** en vez de la interfaz. Por ejemplo, `icacls "C:\Empresa\Comun" /grant "Redes:(OI)(CI)RX"` (`/grant` da permisos; `(OI)(CI)` hace que se hereden a archivos y subcarpetas; `RX` = Lectura y ejecución). Estudia qué hace cada parte.
3. **Accede al recurso compartido desde tu ordenador real.** En NAT no se puede: necesitarás un adaptador **Host-Only** o **Puente** en la VM. ¿Qué ruta usarías? (pista: `\\IP-del-servidor\Empresa`).
4. Crea una carpeta oculta compartida con un `$` al final del nombre (por ejemplo `Admin$`). ¿Para qué se usan?
5. Compara con Linux: ¿qué equivale al grupo `sudo`? ¿Qué diferencia hay entre un permiso NTFS y los `rwx`? ¿Y la doble capa del recurso compartido, tiene equivalente en Samba?

---

## 📝 Qué documentar en tu `Solucion.md`

- Capturas de usuarios, grupos y permisos.
- La salida de `icacls` antes y después de deshabilitar la herencia.
- Los resultados de las pruebas con `runas`.
- Una tabla comparativa **Linux ↔ Windows** de lo que has visto (usuarios, grupos, permisos, compartir).

---

## 🧠 Lo que has practicado

`usuarios y grupos locales` · `SMB` · `recurso compartido` · `permisos NTFS` · `herencia` · `acceso efectivo` · `icacls` · `runas` · `net user / net localgroup`
