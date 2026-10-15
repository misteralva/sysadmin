# 🏢 Laboratorio 11 · Active Directory: crear el dominio

**Nivel:** 🟡 Intermedio · **Sistema:** 🪟 Windows Server · **Tiempo estimado:** 90 minutos

---

## 🎯 Escenario

En el laboratorio 10 del nivel 1 creaste usuarios **locales** en un servidor. Funciona para una máquina, pero si la empresa tiene 50 equipos, tendrías que crear a `ana` en cada uno de ellos, con la misma contraseña, y cambiarla 50 veces cuando caduque.

La solución es un **directorio central**: un único sitio donde viven todos los usuarios y equipos de la empresa. En Windows, eso es **Active Directory**. Tu misión es convertir un servidor en el **controlador de dominio** de la empresa.

> ⚠️ **Haz un snapshot ANTES y otro DESPUÉS.** Promover un servidor a controlador de dominio es un cambio profundo y difícilmente reversible. Y lee primero el README de `02-intermedio` para montar las redes.

---

## 📖 Conceptos nuevos

### Qué es Active Directory

**Active Directory** (AD) es el **directorio** de Microsoft: una base de datos centralizada de **usuarios, grupos, equipos y políticas**. Analogía: es la **secretaría de la empresa**: un único registro donde constan todos los empleados y qué pueden hacer, en vez de cada despacho llevando su propia lista.

(Es el equivalente en el mundo Windows a lo que hiciste con OpenLDAP; de hecho, AD usa **LDAP** por debajo.)

### Vocabulario clave

| Término | Qué es |
| ------- | ------ |
| **Dominio** | El conjunto de usuarios, equipos y recursos que comparten un mismo directorio. Tiene un nombre como `empresa.local` |
| **Controlador de dominio** (DC) | El **servidor** que guarda el directorio y valida los inicios de sesión |
| **AD DS** | *Active Directory Domain Services*: el **rol** de Windows Server que lo hace posible |
| **Bosque** (*forest*) | El contenedor más grande: agrupa uno o varios dominios |
| **FQDN** | Nombre completo, como `dc01.empresa.local` |
| **Nombre NetBIOS** | Nombre corto del dominio, como `EMPRESA` (por el que se inicia sesión: `EMPRESA\ana`) |
| **DSRM** | *Directory Services Restore Mode*: modo de recuperación del DC, con su propia contraseña |
| **SYSVOL** | Carpeta compartida del DC donde se guardan scripts y políticas (GPO) |

### El DNS es imprescindible

AD **depende totalmente de DNS**: los equipos encuentran al controlador de dominio **consultando registros DNS**. Por eso, al crear el dominio, el servidor instala también el rol **DNS** y se configura **a sí mismo** como servidor DNS.

### Requisitos antes de promover

1. **IP estática** (un DC no puede cambiar de IP).
2. **Nombre de equipo definitivo** (`DC01`): cambiarlo después es complicado.
3. **DNS apuntando a sí mismo** (la propia IP del servidor).
4. Contraseña de `Administrator` **compleja**.

> 💡 En un entorno real el dominio no se llamaría `empresa.local` (`.local` se desaconseja porque choca con otros servicios). Lo recomendable es un **subdominio de un dominio que posees**, como `ad.tuempresa.com`. Para el laboratorio, `empresa.local` está bien.

### Comandos de este laboratorio

| Comando (PowerShell como administrador) | Qué hace |
| --------------------------------------- | -------- |
| `Rename-Computer -NewName DC01 -Restart` | Renombra el equipo y reinicia |
| `New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.10.10 -PrefixLength 24` | Asigna una IP fija (`-PrefixLength 24` es la máscara `/24`) |
| `Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 192.168.10.10` | Fija el servidor DNS |
| `Get-NetAdapter` | Lista los adaptadores y sus nombres (`InterfaceAlias`) |
| `Install-WindowsFeature AD-Domain-Services -IncludeManagementTools` | Instala el rol AD DS **y** sus herramientas de gestión |
| `Install-ADDSForest -DomainName "empresa.local" -DomainNetbiosName "EMPRESA"` | **Crea** el bosque y el dominio (promueve el servidor) |
| `Get-ADDomain` / `Get-ADForest` | Muestra información del dominio / bosque |
| `dcdiag /q` | Diagnóstico del controlador de dominio (`/q` = solo muestra los errores) |
| `Resolve-DnsName empresa.local` | Resuelve un nombre por DNS |
| `dsa.msc` | Abre **Usuarios y equipos de Active Directory** |

---

## 🛠️ Qué debes hacer

### Parte A · Preparar la máquina

1. **Clona** la VM de Windows Server del laboratorio 09 (clon **completo**) y llámala `dc01`. Elige *Generar nuevas direcciones MAC*.
2. En `dc01`, deja **un solo adaptador** de red: **Red interna**, con nombre `lan-empresa`.
3. Inicia la VM y comprueba el nombre del adaptador con `Get-NetAdapter`.
4. **Renombra** el equipo a `DC01` (se reinicia).
5. Asigna la IP fija **`192.168.10.10/24`**, sin puerta de enlace, y pon como **DNS la propia IP** (`192.168.10.10`).
6. Comprueba la configuración con `ipconfig /all`.
7. **Snapshot** `antes-de-promover`.

### Parte B · Instalar y promover

8. Instala el rol **AD DS** con sus herramientas (por PowerShell o desde el Administrador del servidor).
9. **Promueve** el servidor creando un **bosque nuevo** con el dominio `empresa.local` y NetBIOS `EMPRESA`. Define la contraseña de **DSRM** (guárdala) y acepta los avisos de DNS.
10. Espera al reinicio automático.
11. Inicia sesión como **`EMPRESA\Administrator`** (fíjate en el nuevo formato de nombre de usuario).
12. **Snapshot** `dominio-creado`.

### Parte C · Explorar

13. Abre **Usuarios y equipos de Active Directory** y explora los contenedores: `Users`, `Computers`, `Domain Controllers`.
14. Abre el **Administrador de DNS** y localiza la zona `empresa.local` y sus registros.
15. Comprueba el estado del dominio con los comandos de PowerShell y con `dcdiag`.
16. Mira la carpeta compartida `SYSVOL`: ¿qué hay dentro?

---

## ✅ Comprobación

| Prueba | Qué debes ver |
| ------ | ------------- |
| `hostname` | `DC01` |
| `ipconfig /all` | IP `192.168.10.10`, DNS `192.168.10.10` y sufijo `empresa.local` |
| `Get-ADDomain \| Select-Object Name, DNSRoot, NetBIOSName` | `empresa`, `empresa.local`, `EMPRESA` |
| `Resolve-DnsName dc01.empresa.local` | Resuelve a `192.168.10.10` |
| `dcdiag /q` | Sin errores (alguno puede ser un aviso al principio: léelo) |
| `whoami` | `empresa\administrator` |
| `dsa.msc` | Se abre y muestra el dominio `empresa.local` |

---

## 🧩 Pistas

- Si el asistente te dice que no puede crear la delegación DNS, es **normal en un laboratorio**: se ignora el aviso.
- Si la promoción falla por la contraseña, recuerda que debe ser **compleja**.
- Si algo falla con el DNS tras el reinicio, espera un par de minutos: el servicio tarda en arrancar y registrar sus entradas.
- Los comandos de PowerShell largos se pueden escribir en varias líneas terminando cada una con el acento grave `` ` ``; o escríbelos en una sola línea.
- Si te equivocas con la IP, `Remove-NetIPAddress` quita la que sobra.

---

## 🚀 Retos extra

1. Mira qué **registros SRV** crea AD en la zona `_msdcs` y `_tcp`. ¿Para qué los usa un cliente?
2. Averigua por qué **no** se hacen snapshots de un controlador de dominio **en producción** (investiga *USN rollback*) y por qué en este laboratorio sí puedes.
3. Añade un **segundo adaptador NAT** a `dc01` y observa los problemas de DNS que aparecen en un DC con dos tarjetas (*multihomed*). Quítalo después.
4. Abre `ntdsutil` y localiza dónde guarda el DC la base de datos (`ntds.dit`).
5. Compara AD con tu práctica de **OpenLDAP**: ¿qué papel tiene cada pieza en ambos?

---

## 📝 Qué documentar en tu `Solucion.md`

- Un esquema de tu red de laboratorio con IPs.
- Los comandos de preparación y promoción, explicados.
- Capturas de `dsa.msc`, del Administrador de DNS y de `Get-ADDomain`.
- Una explicación con tus palabras de **por qué** AD necesita DNS.

---

## 🧠 Lo que has practicado

`Active Directory` · `dominio / bosque / controlador de dominio` · `AD DS` · `DNS integrado` · `FQDN / NetBIOS` · `DSRM` · `IP fija en Windows` · `Install-ADDSForest` · `dcdiag`
