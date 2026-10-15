# 📋 Laboratorio 14 · Directivas de grupo (GPO) básicas

**Nivel:** 🟡 Intermedio · **Sistema:** 🪟 Windows Server y cliente · **Tiempo estimado:** 90 minutos

---

## 🎯 Escenario

Tienes una empresa con decenas de equipos. Quieres que **todos muestren un aviso legal al iniciar sesión**, que el personal de **Sistemas tenga una unidad de red** con su carpeta compartida, y que el de **Redes no pueda abrir el Panel de control**. Hacerlo equipo por equipo sería imposible.

La herramienta de Windows para esto es la **directiva de grupo** (*Group Policy*, **GPO**): configuras una vez, en el servidor, y se aplica sola a todos los usuarios y equipos que correspondan.

> 💾 Snapshot de `dc01` y del cliente. Necesitas el dominio y las OU de los laboratorios 11 y 12, con `dc01` y `pc-cliente01` encendidos y el equipo cliente dentro de la OU `Equipos`.

---

## 📖 Conceptos nuevos

### Qué es una GPO

Una **GPO** (*Group Policy Object*) es un **conjunto de configuraciones** que se aplican automáticamente a usuarios y equipos del dominio. Analogía: es el **reglamento interno** de la empresa, que se entrega solo a quien corresponde, sin pasar por cada mesa.

### Dos mitades: Equipo y Usuario

| Configuración | Se aplica a... | Ejemplos |
| ------------- | -------------- | -------- |
| **Configuración del equipo** | El **ordenador**, haga login quien haga | Aviso legal al inicio, firewall, scripts de arranque |
| **Configuración de usuario** | La **persona**, esté en el equipo que esté | Fondo de escritorio, unidades de red, restricciones |

> ⚠️ Una GPO con **configuración de usuario** solo se aplica si el **usuario** está en (o por debajo de) la OU a la que está vinculada. Una con **configuración de equipo**, si lo está el **equipo**. Es la causa nº 1 de "mi GPO no funciona".

### Vincular

Una GPO **no hace nada** hasta que se **vincula** (*link*) a un sitio, un dominio o una **OU**. Puedes vincular la misma GPO a varias OU.

### Orden de aplicación: LSDOU

Cuando varias GPO coinciden, se aplican en este orden. **La última en aplicarse gana** si hay conflicto:

1. **L**ocal (la política del propio equipo)
2. **S**ite (sitio de AD)
3. **D**ominio
4. **OU** (de la más general a la más específica)

La **herencia** hace que las OU hijas reciban las GPO de sus padres. Se puede **bloquear** (*Block Inheritance*) o **forzar** (*Enforced*).

### Buenas prácticas

- **No edites** la *Default Domain Policy* para cosas pequeñas: crea **una GPO nueva por cada propósito**, con un nombre claro.
- Una GPO por función (`GPO-Aviso-Legal`, `GPO-Unidad-Sistemas`) es más fácil de mantener y de diagnosticar.
- Antes de aplicar a todos, **prueba en una OU de pruebas**.

### Comandos y herramientas de este laboratorio

| Herramienta | Qué hace |
| ----------- | -------- |
| `gpmc.msc` | **Administración de directivas de grupo**: aquí se crean, vinculan y editan las GPO (en el DC) |
| `gpupdate /force` | En el cliente: **aplica ya** las directivas (si no, tardan hasta 90 minutos) |
| `gpresult /r` | Muestra qué GPO **se han aplicado** al usuario y al equipo |
| `gpresult /h informe.html` | Genera un informe HTML detallado con el resultado |
| `Get-GPO -All` | (PowerShell, en el DC) Lista las GPO |
| `Backup-GPO -Name "nombre" -Path "C:\Backups"` | Hace una **copia de seguridad** de una GPO |
| `whoami /groups` | Grupos del usuario actual |

---

## 🛠️ Qué debes hacer

### Parte A · GPO de equipo: aviso legal

1. En `dc01`, abre `gpmc.msc` y localiza tu dominio y las OU.
2. **Crea una GPO** llamada `GPO-Aviso-Legal` y **vincúlala a la OU `Equipos`**.
3. Edítala: en *Configuración del equipo → Directivas → Configuración de Windows → Configuración de seguridad → Directivas locales → Opciones de seguridad*, busca las dos opciones de **"Inicio de sesión interactivo: título del mensaje / texto del mensaje para los usuarios que intentan iniciar una sesión"** y escribe un aviso (por ejemplo, "Equipo de la empresa: uso solo para fines profesionales").
4. En el **cliente**, ejecuta `gpupdate /force`, **reinicia** (las directivas de equipo se aplican al arrancar) y comprueba que **aparece el aviso** en la pantalla de inicio de sesión.

### Parte B · GPO de usuario: restringir al personal de Redes

5. **Crea** `GPO-Sin-PanelControl` y **vincúlala a la OU `Redes`**.
6. Edítala: en *Configuración de usuario → Directivas → Plantillas administrativas → Panel de control*, activa **"Prohibir el acceso al Panel de control y a Configuración del PC"**.
7. En el cliente, **cierra sesión** y entra como `EMPRESA\marta`. Ejecuta `gpupdate /force` y comprueba que **no puede abrir** el Panel de control.
8. Entra ahora como `EMPRESA\ana` y comprueba que **sí puede**: la política **no** le afecta, porque no está en la OU `Redes`.

### Parte C · Unidad de red para Sistemas

9. En `dc01` (o en el servidor de archivos), **comparte** una carpeta `C:\Empresa\Sistemas` con el recurso compartido `Sistemas` y da permisos al grupo `GG_Sistemas` (repite lo que aprendiste en el nivel 1, ahora con los **grupos del dominio**).
10. **Crea** `GPO-Unidad-Sistemas` y vincúlala a la OU `Sistemas`.
11. En *Configuración de usuario → Preferencias → Configuración de Windows → Asignaciones de unidades*, crea una nueva unidad `S:` que apunte a `\\DC01\Sistemas`.
12. Entra como `ana` en el cliente, ejecuta `gpupdate /force` y comprueba que aparece la unidad **S:**.

### Parte D · Diagnóstico

13. En el cliente, con `ana`, ejecuta `gpresult /r` y localiza las **GPO aplicadas**. ¿Aparece la de `Redes`? ¿Por qué no?
14. Genera el informe HTML con `gpresult /h` y ábrelo en el navegador.
15. En `dc01`, **haz una copia de seguridad** de una de tus GPO con `Backup-GPO`.

---

## ✅ Comprobación

| Prueba | Qué debes ver |
| ------ | ------------- |
| Pantalla de inicio de sesión del cliente | Tu aviso legal |
| `marta` → Panel de control | Mensaje de **restricción** y no se abre |
| `ana` → Panel de control | Se abre con normalidad |
| `ana` → Explorador | Unidad `S:` apuntando a `\\DC01\Sistemas` |
| `gpresult /r` (como `ana`) | `GPO-Unidad-Sistemas` en las aplicadas; **sin** `GPO-Sin-PanelControl` |
| `Get-GPO -All \| Select-Object DisplayName` | Tus tres GPO |

---

## 🧩 Pistas

- Si una GPO "no se aplica", comprueba **en este orden**: ¿está vinculada a la OU correcta? ¿El usuario/equipo **está dentro** de esa OU? ¿Has hecho `gpupdate /force` (y reiniciado o cerrado sesión)? ¿La configuración es de **equipo** o de **usuario**?
- Las directivas de **equipo** se aplican al **arrancar**; las de **usuario**, al **iniciar sesión**.
- Si el cliente no ve las GPO, comprueba el **DNS** (debe apuntar al DC) y que `PC-CLIENTE01` está en la OU `Equipos`.
- Para ver rápido qué ha pasado: `gpresult /r /scope user` (solo usuario) o `/scope computer`.
- Si ves los archivos de la GPO, están en la carpeta `SYSVOL` del DC, la que viste en el laboratorio 11.

---

## 🚀 Retos extra

1. **Filtrado de seguridad:** haz que una GPO vinculada a la OU `Empresa` solo se aplique al grupo `GG_Sistemas`.
2. Prueba **Block Inheritance** y **Enforced**: ¿cuál gana si chocan?
3. Crea una GPO de **política de contraseñas** (longitud mínima 10, complejidad) en el **dominio**. Averigua por qué las políticas de contraseñas del dominio **solo** se aplican a nivel de dominio y no a una OU.
4. Despliega un **fondo de pantalla** corporativo (necesitarás una imagen en un recurso compartido accesible).
5. Prueba a **deshabilitar** una GPO (o la mitad de usuario) y mide cómo mejora el tiempo de inicio de sesión. ¿Por qué tener una sola GPO enorme es mala idea?

---

## 📝 Qué documentar en tu `Solucion.md`

- Un esquema de tus OU con las GPO vinculadas a cada una.
- Capturas de cada comprobación y de `gpresult`.
- La explicación del orden **LSDOU** con un ejemplo de **tu** laboratorio.
- Qué fallos tuviste y cuál era la causa real.

---

## 🧠 Lo que has practicado

`GPO` · `configuración de equipo / usuario` · `vincular a OU` · `LSDOU, herencia` · `gpmc.msc` · `gpupdate` · `gpresult` · `preferencias (unidades de red)` · `copia de seguridad de GPO`
