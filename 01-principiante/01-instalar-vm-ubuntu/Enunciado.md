# 🖥️ Laboratorio 01 · Instalar una VM con Ubuntu Server y usar snapshots

**Nivel:** 🟢 Principiante · **Sistema:** 🐧 Linux · **Tiempo estimado:** 45-60 minutos

---

## 🎯 Escenario

Eres el nuevo administrador de sistemas de una pequeña empresa. Antes de tocar ningún servidor real, tu jefe te pide que montes un **servidor de pruebas** en el que puedas practicar sin miedo a romper nada.

Tu primera misión es crear ese servidor como **máquina virtual**, comprobar que funciona y aprender a "guardar la partida" para poder volver atrás si algo sale mal.

---

## 📖 Conceptos nuevos

### Virtualización y máquina virtual (VM)

Una **máquina virtual** es un ordenador que vive *dentro* de otro ordenador. Piensa en un simulador de vuelo: parece una cabina real, pero todo ocurre dentro de tu PC. La VM tiene su propia "CPU", memoria y disco, aunque en realidad los toma prestados de tu equipo real.

El programa que crea y gestiona las VMs se llama **hipervisor**. En este laboratorio usarás **VirtualBox**. A tu ordenador real se le llama **anfitrión** (*host*) y a la VM, **invitado** (*guest*).

### Imagen ISO

Un archivo **ISO** es una copia completa de un disco de instalación, metida en un solo archivo. Es como el DVD de instalación de un sistema operativo, pero en formato digital. VirtualBox lo "inserta" en la VM como si fuera un disco real.

### Ubuntu Server frente a Ubuntu Desktop

**Ubuntu Desktop** trae entorno gráfico (ventanas, ratón). **Ubuntu Server** no: solo tiene terminal. Los servidores reales casi nunca tienen pantalla ni escritorio, porque eso gasta recursos y añade superficie de ataque. Por eso practicarás con Server.

### Recursos de la VM

Al crear la VM decides cuánta **CPU**, **RAM** y **disco** le das. Ten en cuenta que esos recursos los pierde tu equipo mientras la VM está encendida.

### Modos de red en VirtualBox

El adaptador de red de la VM puede funcionar de varias formas. Por ahora solo necesitas conocer dos:

| Modo | Qué hace |
| ---- | -------- |
| **NAT** | La VM sale a internet a través de tu equipo, pero desde fuera nadie puede conectarse a ella. Es el modo por defecto y el más seguro para empezar. |
| **Adaptador puente** (*bridged*) | La VM se conecta a tu red local como si fuera un equipo más, con su propia IP. |

### Snapshot (instantánea)

Un **snapshot** es un punto de guardado, como en un videojuego. Guarda el estado exacto de la VM en un momento concreto. Si más tarde algo se rompe, puedes **restaurar** el snapshot y la VM vuelve a ser exactamente como era.

Esta es la herramienta más importante para practicar sysadmin: te permite equivocarte sin consecuencias.

---

## 🛠️ Qué debes hacer

1. **Descarga** la ISO de Ubuntu Server (versión LTS) desde la web oficial de Ubuntu.
2. **Crea una VM** en VirtualBox con estas características mínimas:
   - Nombre: `ubuntu-server-lab`
   - Memoria RAM: 2 GB
   - CPU: 2 núcleos
   - Disco virtual: 25 GB
   - Red: modo NAT
3. **Instala Ubuntu Server** dentro de la VM:
   - Crea un usuario administrador (elige tú el nombre y una contraseña que recuerdes).
   - Pon a la máquina un nombre de host reconocible.
   - Cuando el instalador te pregunte por paquetes opcionales, **no marques nada** por ahora.
4. **Reinicia** la VM, inicia sesión y **comprueba** que todo funciona (ver siguiente sección).
5. **Crea un snapshot** llamado `instalacion-limpia`.
6. **Prueba que el snapshot funciona**: haz un cambio en la VM, restaura el snapshot y demuestra que el cambio ha desaparecido.

---

## ✅ Comprobación

Dentro de la VM, ejecuta estos comandos. Cada uno te dice algo distinto sobre el sistema:

| Comando | Qué hace | Qué debes ver |
| ------- | -------- | ------------- |
| `hostnamectl` | Muestra el nombre del equipo, el sistema operativo y el kernel | El nombre de host que elegiste y "Ubuntu" |
| `ip -br a` | Lista las interfaces de red de forma resumida (`ip` gestiona la red, `-br` es *brief* = breve, `a` es *address* = direcciones) | Una interfaz con una IP asignada |
| `df -h` | Muestra el espacio de disco usado y libre (`-h` = *human-readable*: en GB y MB en lugar de bytes) | Un disco de unos 25 GB |
| `free -h` | Muestra la memoria RAM total, usada y libre | Unos 2 GB de memoria total |
| `ping -c 2 8.8.8.8` | Envía 2 paquetes (`-c 2` = *count*) a un servidor de internet para probar la conexión | Respuestas sin pérdida de paquetes |
| `ping -c 2 google.com` | Lo mismo, pero con un nombre en lugar de una IP: comprueba además que el **DNS** funciona | Respuestas sin pérdida de paquetes |

> 💡 Si el primer `ping` funciona y el segundo no, el problema está en el DNS, no en la conexión. Fíjate en esa diferencia: es una de las primeras cosas que se comprueban al diagnosticar problemas de red.

**Comprobación del snapshot:**

1. Después de crear el snapshot, dentro de la VM crea un archivo vacío con `touch ~/fichero-de-prueba` (`touch` crea un archivo vacío; `~` es tu carpeta personal).
2. Comprueba que existe con `ls ~` (`ls` lista el contenido de una carpeta).
3. Apaga la VM y **restaura** el snapshot `instalacion-limpia` desde VirtualBox.
4. Vuelve a encender la VM y ejecuta otra vez `ls ~`. El archivo **no debe aparecer**.

> 💡 No hace falta romper nada de verdad para entender cómo funcionan los snapshots. Con un archivo de prueba se ve igual de claro.

---

## 🧩 Pistas

- En VirtualBox, busca en la configuración de la VM la sección de **Almacenamiento** para "insertar" la ISO en la unidad óptica virtual.
- Si el instalador te deja elegir entre varias opciones de disco, la opción por defecto (usar todo el disco) es la correcta para este laboratorio.
- Los snapshots se gestionan desde la herramienta de **Instantáneas** de la VM en VirtualBox.
- Si no sabes qué hace un comando, la mayoría tienen ayuda integrada: prueba `comando --help` o `man comando`.

---

## 🚀 Retos extra

1. **Cambia el nombre del equipo** desde la terminal, sin reinstalar. Pista: investiga el comando `hostnamectl` y su opción `set-hostname`. Necesitarás `sudo`, que significa "ejecuta este comando con privilegios de administrador".
2. **Crea un segundo snapshot** después de cambiar el nombre. ¿Puedes volver al primero? ¿Qué pasa con el segundo?
3. **Clona la VM** y busca la diferencia entre un clon *vinculado* y un clon *completo*. ¿Cuándo usarías cada uno?
4. **Exporta la VM** como archivo `.ova`. ¿Para qué serviría en un entorno real?

---

## 📝 Qué documentar en tu `Solucion.md`

- Los valores que elegiste al crear la VM y por qué.
- Cualquier problema que encontraste y cómo lo resolviste (apartado **"Qué salió mal"**).
- Los comandos que usaste, cada uno con una explicación de lo que hace.
- Una captura de la salida de los comandos de comprobación.

---

## 🧠 Lo que has practicado

`virtualización` · `ISO` · `recursos de VM` · `modos de red` · `snapshots` · `hostnamectl` · `ip` · `df` · `free` · `ping` · `touch` · `ls`
