# 🪟 Laboratorio 09 · Instalar Windows Server en una VM

**Nivel:** 🟢 Principiante · **Sistema:** 🪟 Windows Server · **Tiempo estimado:** 60-90 minutos

---

## 🎯 Escenario

La empresa también tiene servicios que corren sobre **Windows Server**: carpetas compartidas, usuarios, y más adelante el directorio de la empresa (Active Directory). Para practicar sin riesgos, necesitas un segundo servidor de pruebas, esta vez con Windows.

Tu misión es montarlo en una máquina virtual, dejarlo con un nombre reconocible, comprobar que funciona y guardar un snapshot, exactamente como hiciste con Ubuntu en el laboratorio 01.

---

## 📖 Conceptos nuevos

### Windows Server frente a Windows de escritorio

Windows Server es el Windows pensado para **ofrecer servicios** a otros equipos: archivos, usuarios, correo, web, bases de datos… No está hecho para que alguien navegue o juegue en él, sino para estar encendido y estable, y para administrarse a distancia.

### Ediciones y licencia de evaluación

Microsoft ofrece una **versión de evaluación** gratuita, con todas las funciones, durante un período limitado (normalmente 180 días). Es perfecta para laboratorios: es la misma versión que usarías en producción, pero sin licencia comercial.

> 📌 Descárgala siempre desde la web oficial de Microsoft (su **Evaluation Center**). Nunca desde sitios de terceros: una ISO modificada es una forma clásica de meter malware.

### Dos formas de instalarlo

| Opción | Qué es |
| ------ | ------ |
| **Desktop Experience** (con escritorio) | Tiene entorno gráfico con ventanas, como un Windows normal. Es la más cómoda para aprender |
| **Server Core** (sin escritorio) | Solo línea de comandos. Gasta menos y tiene menos superficie de ataque, pero necesita más soltura |

En este laboratorio instalarás **Desktop Experience**. Server Core es un reto extra.

### Administrador del servidor (*Server Manager*)

Es la herramienta gráfica que se abre al iniciar sesión. Desde aquí se instalan **roles** (qué servicio ofrece el servidor: ficheros, DNS, DHCP…) y **características** (funciones adicionales). Será tu "centro de mando" en los próximos laboratorios.

### La cuenta `Administrator`

Windows Server se instala con una cuenta de administrador llamada **`Administrator`** (la equivalente de `root` en Linux). Te pedirá una contraseña **compleja**: mínimo 8 caracteres y mezcla de mayúsculas, minúsculas, números o símbolos.

### Nombre del equipo

Por defecto, Windows pone un nombre aleatorio (`WIN-X7K2…`). En un entorno real, los servidores tienen **nombres claros y siguiendo un criterio**, como `SRV-WIN01`.

### Comandos de este laboratorio

Se ejecutan en **PowerShell** (búscalo en el menú Inicio; ábrelo como administrador):

| Comando | Qué hace |
| ------- | -------- |
| `hostname` | Muestra el nombre del equipo |
| `Get-ComputerInfo \| Select-Object OsName, OsVersion` | Muestra el sistema operativo y su versión (`Get-ComputerInfo` recoge los datos; `\|` pasa el resultado al siguiente comando; `Select-Object` se queda solo con las columnas que pides) |
| `ipconfig` | Muestra la configuración de red (IP, máscara, puerta de enlace) |
| `ipconfig /all` | Igual, con todos los detalles (incluidos los servidores DNS) |
| `ping -n 2 8.8.8.8` | Envía 2 paquetes a internet (en Windows, `-n` es el número de paquetes; en Linux era `-c`) |
| `Rename-Computer -NewName "NOMBRE" -Restart` | Cambia el nombre del equipo y reinicia |
| `Get-PSDrive -PSProvider FileSystem` | Muestra las unidades de disco y su espacio libre |

---

## 🛠️ Qué debes hacer

1. **Descarga** la ISO de Windows Server (versión de evaluación) desde la web oficial de Microsoft.
2. **Crea una VM** en VirtualBox:
   - Nombre: `windows-server-lab`
   - Tipo: *Microsoft Windows*, versión *Windows Server (64-bit)* (o la más cercana que ofrezca tu VirtualBox)
   - Memoria RAM: 4 GB
   - CPU: 2 núcleos
   - Disco virtual: 50 GB
   - Red: modo NAT
3. **Instala** Windows Server:
   - Elige la edición **con Desktop Experience** (la que aparece con "(Desktop Experience)" en su nombre).
   - Cuando pregunte por el tipo de instalación, elige la **personalizada** (instalación nueva) y usa el disco entero.
   - Define la contraseña del `Administrator`.
4. **Inicia sesión.** En VirtualBox no puedes pulsar `Ctrl+Alt+Supr` directamente: usa el menú *Máquina → Insertar Ctrl-Alt-Supr* (o la tecla *Host* + `Supr`).
5. Cuando se abra el **Administrador del servidor**, familiarízate con él: localiza dónde se ve el nombre del equipo y dónde se añadirían roles.
6. **Cambia el nombre** del equipo a `SRV-WIN01`. Hazlo primero desde la interfaz gráfica; en el reto extra lo harás con PowerShell.
7. Comprueba que el sistema tiene **red e internet**.
8. Ajusta la **zona horaria** y el idioma del teclado si hace falta.
9. **Crea un snapshot** llamado `instalacion-limpia`.
10. Prueba el snapshot igual que en el laboratorio 01: crea un archivo en el escritorio, restaura el snapshot y comprueba que el archivo ya no está.

---

## ✅ Comprobación

En PowerShell:

| Comando | Qué debes ver |
| ------- | ------------- |
| `hostname` | `SRV-WIN01` |
| `Get-ComputerInfo \| Select-Object OsName, OsVersion` | "Microsoft Windows Server" y su versión |
| `ipconfig` | Una IP asignada (en NAT suele ser `10.0.2.15`) y la puerta de enlace |
| `ping -n 2 8.8.8.8` | Respuestas sin pérdida de paquetes |
| `ping -n 2 google.com` | Respuestas (comprueba también el DNS) |
| `Get-PSDrive -PSProvider FileSystem` | La unidad `C:` con unos 50 GB en total |

Comprobación del snapshot: el archivo del escritorio **desaparece** tras restaurarlo.

---

## 🧩 Pistas

- La instalación puede tardar y reiniciar varias veces. Es normal.
- Si el instalador no detecta el disco, revisa que la VM tenga el disco virtual creado y conectado.
- Si no puedes escribir el símbolo `\` o `|` por el idioma del teclado, cambia el **idioma de entrada** (esquina inferior derecha) a *Español (España)*.
- Si PowerShell te dice que no tiene permisos, ciérralo y ábrelo con clic derecho → *Ejecutar como administrador*.
- Puedes comparar con el laboratorio 01: `ipconfig` ↔ `ip a`, `hostname` ↔ `hostnamectl`, `ping -n` ↔ `ping -c`.

---

## 🚀 Retos extra

1. **Cambia el nombre con PowerShell** en vez de con el ratón, usando `Rename-Computer`. Necesitarás reiniciar.
2. Instala las **Guest Additions** de VirtualBox (menú *Dispositivos*) para tener mejor resolución y portapapeles compartido.
3. Crea una segunda VM instalando la variante **Server Core**. ¿Qué ves al iniciar sesión? Investiga la herramienta `sconfig`.
4. Compara el consumo de memoria de Desktop Experience y Server Core una vez arrancados (`Get-Process` o el Administrador de tareas). ¿Por qué es relevante?
5. Averigua cuántos días de evaluación te quedan (pista: busca el comando `slmgr /dlv`).

---

## 📝 Qué documentar en tu `Solucion.md`

- Los valores de la VM y por qué los elegiste.
- Las diferencias que notes con la instalación de Ubuntu (tiempo, recursos, pasos).
- Capturas del Administrador del servidor y de la salida de los comandos de comprobación.
- El apartado **"Qué salió mal"**.

---

## 🧠 Lo que has practicado

`Windows Server` · `edición de evaluación` · `Desktop Experience / Server Core` · `Administrador del servidor` · `cuenta Administrator` · `nombre de equipo` · `PowerShell básico` · `ipconfig` · `snapshots`
