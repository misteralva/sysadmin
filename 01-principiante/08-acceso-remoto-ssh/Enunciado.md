# 🔑 Laboratorio 08 · Acceso remoto con SSH

**Nivel:** 🟢 Principiante · **Sistema:** 🐧 Linux · **Tiempo estimado:** 60 minutos

---

## 🎯 Escenario

Los servidores reales casi nunca tienen una pantalla delante: están en un rack, en otra sala o en otra ciudad. Un administrador los gestiona **a distancia**.

Hasta ahora has trabajado desde la consola de VirtualBox. Hoy vas a conectarte a tu servidor **desde tu propio ordenador**, como lo harías en un entorno real, y a copiar archivos entre los dos.

> 💾 Haz un **snapshot** antes de empezar. En el reto extra tocarás la configuración de SSH, y un error ahí puede dejarte sin acceso.

---

## 📖 Conceptos nuevos

### Qué es SSH

**SSH** (*Secure Shell*) es una forma de abrir una terminal en otro equipo **a través de la red, con la conexión cifrada**. Es como un teléfono directo y privado con el servidor: lo que escribes viaja protegido, a diferencia de protocolos antiguos como `telnet`, que enviaban todo en texto plano.

### Cliente y servidor

- El **servidor** (`sshd`, el *daemon* de SSH) corre en la máquina a la que te conectas y **escucha en el puerto 22**.
- El **cliente** (`ssh`) es el programa que usas desde tu equipo.

Tu VM necesita tener instalado el servidor. Tu ordenador solo necesita el cliente, que ya incluyen Windows, macOS y Linux.

### La huella del servidor (*fingerprint*)

La **primera vez** que te conectas, SSH te enseña la "huella digital" del servidor y te pregunta si confías en él. Si aceptas, la guarda en `~/.ssh/known_hosts`. En las siguientes conexiones, si la huella **cambia**, SSH te avisa con un mensaje de peligro: puede significar que alguien se está haciendo pasar por tu servidor.

### Reenvío de puertos en VirtualBox (NAT)

Con red NAT, tu VM **no es accesible desde fuera**. Para conectarte necesitas una **regla de reenvío de puertos**, que dice:

> *"Todo lo que llegue al puerto 2222 de mi ordenador, mándalo al puerto 22 de la VM."*

Así, conectándote a `127.0.0.1:2222` (tu propio equipo), llegas a la VM.

### Comandos de este laboratorio

| Comando | Qué hace |
| ------- | -------- |
| `sudo systemctl status ssh` | Muestra si el servicio SSH está activo (`systemctl` gestiona los servicios; lo verás a fondo en el nivel intermedio) |
| `ssh usuario@host` | Conecta con un servidor |
| `ssh -p 2222 usuario@127.0.0.1` | Igual, pero por un puerto distinto del 22 (`-p` = puerto, en minúscula) |
| `ssh usuario@host 'comando'` | Ejecuta **un solo comando** en el servidor y vuelve |
| `scp -P 2222 archivo usuario@host:ruta` | Copia un archivo **hacia** el servidor (`-P` = puerto, en **mayúscula** en `scp`) |
| `scp -P 2222 usuario@host:ruta archivo` | Copia un archivo **desde** el servidor |
| `exit` | Cierra la sesión SSH |
| `who` | Lista las sesiones abiertas en el servidor |
| `sudo tail -n 20 /var/log/auth.log` | Muestra las últimas 20 líneas del registro de autenticación |

---

## 🛠️ Qué debes hacer

1. **Instala** el servidor SSH en tu VM (el paquete se llama `openssh-server`). Si lo marcaste durante la instalación, comprueba que ya está.
2. **Comprueba** que el servicio está activo y que el puerto 22 está a la escucha (repasa `ss` del laboratorio 07).
3. En VirtualBox, **crea la regla de reenvío de puertos** en la configuración de red de la VM: puerto anfitrión `2222`, puerto invitado `22`, IP `127.0.0.1` en el anfitrión.
4. Desde **tu ordenador** (no desde la VM), **conéctate** por SSH a tu servidor. Acepta la huella la primera vez.
5. Ya dentro, ejecuta `hostname`, `whoami` y `who`. ¿Aparece tu sesión?
6. Sal con `exit` y **ejecuta un solo comando remoto** sin abrir sesión interactiva.
7. **Copia** un archivo desde tu ordenador al servidor, y otro del servidor a tu ordenador. Comprueba que han llegado.
8. En el servidor, **consulta el registro** `/var/log/auth.log` y localiza tus conexiones.
9. En tu ordenador, abre `~/.ssh/known_hosts` y localiza la huella que guardaste.
10. **Prueba un fallo controlado**: conéctate con una contraseña incorrecta. Mira cómo se refleja en `auth.log`.

---

## ✅ Comprobación

| Prueba | Qué debes ver |
| ------ | ------------- |
| `sudo systemctl status ssh` | `active (running)` |
| `ss -tln \| grep :22` | Una línea con el puerto 22 a la escucha |
| `ssh -p 2222 usuario@127.0.0.1 'hostname'` | El nombre de tu VM, sin entrar en ella |
| `who` (dentro de la VM, con una sesión SSH abierta) | Tu usuario con un terminal tipo `pts/0` |
| `sudo grep sshd /var/log/auth.log \| tail` | Líneas de "Accepted password" y, tras el fallo, "Failed password" |
| El archivo copiado | Aparece en el destino con el mismo contenido (comprueba con `cat`) |

---

## 🧩 Pistas

- En **Windows** puedes usar PowerShell o el Símbolo del sistema: el cliente `ssh` viene instalado en versiones recientes. En macOS y Linux, la terminal habitual.
- Si la conexión da *"Connection refused"*, revisa que el servicio esté activo. Si da *"Connection timed out"*, revisa la regla de reenvío de puertos.
- Si cambias el servidor y reinstalas la VM, SSH te avisará de que la huella ha cambiado. No ignores ese aviso en entornos reales; en tu laboratorio, puedes eliminar la entrada vieja de `known_hosts`.
- El nombre del servicio en Ubuntu es `ssh` (en otras distribuciones se llama `sshd`).

---

## 🚀 Retos extra

1. **Cambia el puerto** de SSH (archivo `/etc/ssh/sshd_config`, directiva `Port`). Después reinicia el servicio. ¿Qué debes cambiar también en VirtualBox y en el comando de conexión? *(Haz snapshot antes: si te equivocas, perderás el acceso.)*
2. **Prohíbe el acceso directo de `root`** con la directiva `PermitRootLogin no`. ¿Por qué es una buena práctica?
3. Crea un archivo `~/.ssh/config` en tu ordenador con un **alias**, de modo que puedas conectarte con solo `ssh lab`.
4. Investiga qué es un **túnel SSH** (`ssh -L`) y qué problema resuelve.
5. Averigua por qué dejar SSH con contraseña expuesto a internet es peligroso: busca cuántos intentos de acceso fallidos recibe un servidor público en solo un día. *(En el laboratorio intermedio 08 aprenderás a protegerlo con **claves**.)*

---

## 📝 Qué documentar en tu `Solucion.md`

- Capturas de la configuración de reenvío de puertos y de la sesión SSH.
- Los comandos de conexión y de copia, con su explicación.
- Qué ves en `auth.log` en una conexión correcta y en una fallida.
- Una reflexión: ¿por qué SSH es mejor que `telnet`? ¿Qué riesgos tiene dejarlo abierto con contraseña?

---

## 🧠 Lo que has practicado

`SSH cliente/servidor` · `puerto 22` · `fingerprint / known_hosts` · `reenvío de puertos NAT` · `ssh` · `scp` · `systemctl status` · `auth.log`
