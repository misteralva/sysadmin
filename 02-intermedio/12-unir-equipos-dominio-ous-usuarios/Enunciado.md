# 👤 Laboratorio 12 · Unir equipos al dominio, OUs y usuarios

**Nivel:** 🟡 Intermedio · **Sistema:** 🪟 Windows Server y cliente · **Tiempo estimado:** 90 minutos

---

## 🎯 Escenario

El dominio `empresa.local` ya existe, pero está **vacío**. Es hora de poblarlo: crear la **estructura de la empresa**, dar de alta a los empleados y conseguir que un **equipo cliente** inicie sesión con un usuario del dominio.

Es el mismo escenario de siempre: `ana` y `luis` (Sistemas) y `marta` (Redes). Pero ahora se crean **una sola vez**, en el directorio, y sirven en **todos** los equipos.

> 💾 Snapshot de `dc01` y del cliente antes de empezar. Necesitas haber hecho el laboratorio 11 y tener **`dc01` encendido** durante todo el laboratorio.

---

## 📖 Conceptos nuevos

### Unidades organizativas (OU)

Una **OU** (*Organizational Unit*) es una **carpeta dentro del directorio** donde agrupas usuarios, grupos y equipos. Sirve para **organizar** y, sobre todo, para **aplicar políticas distintas** a cada grupo (lo verás en el laboratorio 14) y delegar la administración.

> **Analogía:** las OU son los **departamentos** del organigrama. Dentro del departamento de Sistemas están las personas y los equipos de Sistemas.

Estructura que vas a crear:

```
empresa.local
└── Empresa  (OU)
    ├── Sistemas  (OU)  → ana, luis
    ├── Redes     (OU)  → marta
    └── Equipos   (OU)  → ordenadores cliente
```

### OU frente a grupo: no es lo mismo

| | **OU** | **Grupo** |
| - | ------ | --------- |
| Para qué sirve | **Organizar** y aplicar **políticas** (GPO) | **Dar permisos** a recursos (carpetas, impresoras) |
| Un usuario puede estar en... | **Una sola** OU | **Varios** grupos |

### Qué es unir un equipo al dominio

Un equipo que **se une al dominio** deja de validar los usuarios por sí mismo: **pregunta al controlador de dominio**. A partir de ahí, cualquier usuario del dominio puede iniciar sesión en él (si se le permite).

Requisito esencial: el cliente debe poder **resolver el DNS del dominio**. Por eso se configura su DNS apuntando al **controlador de dominio**.

### Comandos de este laboratorio

| Comando | Qué hace |
| ------- | -------- |
| `New-ADOrganizationalUnit -Name "Empresa" -Path "DC=empresa,DC=local"` | Crea una OU (`-Path` es la ruta LDAP donde se crea) |
| `New-ADGroup -Name "GG_Sistemas" -GroupScope Global -GroupCategory Security -Path "OU=..."` | Crea un grupo de seguridad global |
| `New-ADUser -Name "ana" -SamAccountName ana -AccountPassword (Read-Host -AsSecureString) -Enabled $true -Path "OU=..."` | Crea un usuario |
| `Add-ADGroupMember -Identity GG_Sistemas -Members ana,luis` | Añade usuarios a un grupo |
| `Get-ADUser -Filter *` / `Get-ADGroupMember GG_Sistemas` | Lista usuarios / miembros de un grupo |
| `Move-ADObject -Identity "..." -TargetPath "OU=..."` | Mueve un objeto a otra OU |
| `Add-Computer -DomainName empresa.local -Credential EMPRESA\Administrator -Restart` | **Une** un equipo al dominio (en el cliente) |
| `whoami /groups` | Muestra los grupos del usuario actual |
| `dsa.msc` | Interfaz gráfica de usuarios y equipos |

---

## 🛠️ Qué debes hacer

### Parte A · Estructura en el dominio (en `dc01`)

1. Con **`dsa.msc`** (gráfico), crea la OU `Empresa` y dentro las OU `Sistemas`, `Redes` y `Equipos`.
2. Crea los usuarios **`ana`** y **`luis`** en la OU `Sistemas`, y **`marta`** en `Redes`. Contraseñas complejas. Desmarca "cambiar contraseña en el próximo inicio" para poder probar cómodamente.
3. Crea los **grupos de seguridad globales** `GG_Sistemas` y `GG_Redes` (en las OU correspondientes) y añade a cada usuario a su grupo.
4. Comprueba todo con PowerShell: `Get-ADUser -Filter *` y `Get-ADGroupMember`.

### Parte B · Preparar el cliente

5. Crea la VM `pc-cliente01` (Windows edición **Pro/Enterprise** de evaluación, o un Windows Server clonado como servidor miembro), con **un solo adaptador en la red interna `lan-empresa`**.
6. Dale la IP **fija `192.168.10.20/24`** y como **DNS** el controlador de dominio: `192.168.10.10`.
7. Comprueba desde el cliente: `ping 192.168.10.10` y `nslookup dc01.empresa.local`. **Si el DNS no resuelve, no sigas.**

### Parte C · Unir al dominio

8. **Une** el equipo al dominio `empresa.local` (gráficamente desde *Propiedades del sistema*, o con `Add-Computer`). Usa las credenciales de `EMPRESA\Administrator`.
9. Reinicia, y en la pantalla de inicio de sesión elige **otro usuario** y escribe `EMPRESA\ana`.
10. Una vez dentro, comprueba con `whoami` y `whoami /groups` quién eres y a qué grupos perteneces.
11. En `dc01`, localiza el **objeto equipo** `PC-CLIENTE01` (estará en el contenedor `Computers` por defecto) y **muévelo** a la OU `Empresa\Equipos`.

### Parte D · Gestión diaria

12. Desde `dc01`, **restablece la contraseña** de `marta` y **desactiva** la cuenta de `luis`. Comprueba que `luis` ya no puede iniciar sesión en el cliente.
13. **Reactívala** y comprueba que vuelve a entrar.
14. Con PowerShell, crea un **cuarto usuario** `pablo` en `Sistemas` y quítalo del grupo y del dominio cuando termines (`Remove-ADUser`).

---

## ✅ Comprobación

| Prueba | Qué debes ver |
| ------ | ------------- |
| `Get-ADUser -Filter * \| Select-Object Name` | `ana`, `luis`, `marta`, además de `Administrator`, `Guest` y `krbtgt` |
| `Get-ADGroupMember GG_Sistemas` | `ana` y `luis` |
| En el cliente, `nslookup dc01.empresa.local` | `192.168.10.10` |
| `whoami` (cliente, sesión de ana) | `empresa\ana` |
| `Get-ADComputer PC-CLIENTE01` | El equipo, ya en la OU `Equipos` |
| Con `luis` desactivado | El cliente rechaza su inicio de sesión |

---

## 🧩 Pistas

- Si el cliente no encuentra el dominio al unirlo, **es casi siempre un problema de DNS**: comprueba que apunta a `192.168.10.10` y no a otro.
- Un fallo clásico: usar `ana` en vez de `EMPRESA\ana` (o `ana@empresa.local`) y que el equipo busque en el sistema local.
- Si no puedes elegir otro usuario en la pantalla de inicio, pulsa en **"Otro usuario"**.
- En Windows de escritorio, **Home no puede unirse a un dominio**. Revisa la edición.
- Los nombres LDAP se leen de derecha a izquierda: `OU=Sistemas,OU=Empresa,DC=empresa,DC=local` es el camino hasta la OU `Sistemas`.

---

## 🚀 Retos extra

1. **Crea los usuarios en masa**: escribe un CSV con 10 usuarios y un script de PowerShell que los dé de alta con un bucle (`Import-Csv` + `foreach`). Conecta con tus ejercicios de Bash y Python.
2. **Repite el laboratorio 10 del nivel 1** (carpetas compartidas), pero con **los grupos del dominio** (`GG_Sistemas`, `GG_Redes`) en vez de usuarios locales. Esta es la razón de ser de AD: ¿qué ventaja ves?
3. Averigua para qué sirve la estrategia **AGDLP** (Account → Global → Domain Local → Permission).
4. Permite que `ana` inicie sesión en el DC y comprueba por qué normalmente **no** se permite a los usuarios normales entrar directamente en un controlador de dominio.
5. Compara este laboratorio con tu práctica de **OpenLDAP**: ¿qué equivale a una OU en LDAP? ¿Y a un grupo?

---

## 📝 Qué documentar en tu `Solucion.md`

- El árbol de OU que creaste, con una captura de `dsa.msc`.
- Los comandos de PowerShell usados, explicados.
- La salida de `whoami /groups` del cliente.
- Una explicación de la diferencia entre OU y grupo con **tus palabras y un ejemplo**.

---

## 🧠 Lo que has practicado

`OU` · `grupos de seguridad` · `usuarios del dominio` · `unir un equipo al dominio` · `DNS del cliente` · `inicio de sesión de dominio (EMPRESA\usuario)` · `New-ADUser / New-ADGroup` · `gestión de cuentas`
