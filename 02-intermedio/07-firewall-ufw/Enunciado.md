# 🔥 Laboratorio 07 · Firewall con `ufw`

**Nivel:** 🟡 Intermedio · **Sistema:** 🐧 Linux · **Tiempo estimado:** 75 minutos

---

## 🎯 Escenario

Tu servidor tiene varios servicios escuchando (los viste con `ss` en el nivel 1), pero **no todos deben ser visibles desde fuera**. Un buen servidor sigue la regla de oro: *"todo cerrado, y se abre solo lo imprescindible"*.

Tu misión es configurar el **cortafuegos** (firewall) para que solo se pueda entrar por **SSH** y solo desde tu red de laboratorio, y abrir después un servicio de prueba de forma controlada.

> ⚠️ **Este laboratorio puede dejarte sin acceso SSH.** Haz un **snapshot** antes y trabaja siempre con la **consola de VirtualBox** abierta como plan B. Necesitas haber hecho el laboratorio 03 (red solo-anfitrión).

---

## 📖 Conceptos nuevos

### Qué es un firewall

Un **firewall** es el **portero** del servidor: mira cada conexión que intenta entrar o salir y decide si la deja pasar o la bloquea, según unas **reglas**. Sin firewall, cualquier servicio que escuche en un puerto es accesible por cualquiera que llegue a la máquina.

### `ufw`: *Uncomplicated Firewall*

En Linux, el filtrado de paquetes lo hace el kernel (con `netfilter`), y se configura con herramientas como `iptables` o `nftables` (nivel avanzado). **`ufw`** es una capa **sencilla** por encima, ideal para empezar.

### Políticas por defecto

Lo que ocurre con una conexión que **no coincide con ninguna regla**:

| Política | Recomendada | Significa |
| -------- | ----------- | --------- |
| `incoming` | **deny** | Todo lo que **entra** se bloquea, salvo lo que permitas |
| `outgoing` | **allow** | Todo lo que **sale** se permite |

> 🧠 Es un enfoque de **lista blanca**: lo que no se permite expresamente, está prohibido.

### Orden de las reglas

Las reglas se evalúan **de arriba abajo** y gana la **primera que coincide**. Una regla de `deny` puesta antes puede "tapar" un `allow` posterior.

### Comandos de este laboratorio

| Comando | Qué hace |
| ------- | -------- |
| `sudo ufw status verbose` | Estado, políticas por defecto y reglas |
| `sudo ufw status numbered` | Reglas **numeradas** (para poder borrarlas) |
| `sudo ufw default deny incoming` | Política por defecto: bloquear lo que entra |
| `sudo ufw default allow outgoing` | Política por defecto: permitir lo que sale |
| `sudo ufw allow 22/tcp` | Permite el puerto 22 TCP desde cualquier origen |
| `sudo ufw allow from 192.168.56.0/24 to any port 22 proto tcp` | Permite SSH **solo desde esa red** |
| `sudo ufw deny 8080/tcp` | Bloquea un puerto |
| `sudo ufw delete N` | Borra la regla número N |
| `sudo ufw limit ssh` | Permite SSH **limitando** los intentos repetidos (frena la fuerza bruta) |
| `sudo ufw logging on` | Activa el registro de conexiones bloqueadas (`/var/log/ufw.log`) |
| `sudo ufw enable` / `disable` | Activa / desactiva el firewall |
| `sudo ufw reset` | Borra todas las reglas y lo desactiva |
| `python3 -m http.server 8080` | Lanza un servidor web mínimo en el puerto 8080 (servicio de prueba) |
| `ss -tuln` | Puertos a la escucha (nivel 1) |

> 🚨 **Regla que no puedes olvidar:** antes de `ufw enable`, **permite SSH**. Si activas el firewall con política `deny` sin permitir el puerto 22, te quedas fuera de tu propia máquina (por SSH; la consola de VirtualBox siempre funciona).

---

## 🛠️ Qué debes hacer

### Parte A · Punto de partida

1. Comprueba el estado de `ufw` (debería estar inactivo).
2. Con `ss -tuln`, apunta qué puertos tiene abiertos tu servidor.
3. Desde **tu ordenador**, comprueba que puedes conectarte por SSH a `192.168.56.10`.

### Parte B · Configurar el firewall

4. Fija las **políticas por defecto**: bloquear lo que entra y permitir lo que sale.
5. **Antes de activar nada**, crea la regla que permita SSH **solo desde la red `192.168.56.0/24`**.
6. Activa el firewall y comprueba con `status verbose` que todo está como esperas.
7. **Sin cerrar la sesión SSH actual**, abre una **segunda** conexión desde tu ordenador y comprueba que sigue funcionando.

### Parte C · Servicio de prueba

8. En la VM, lanza un servidor web de prueba en el puerto **8080**: `python3 -m http.server 8080`.
9. Desde tu ordenador, intenta acceder a `http://192.168.56.10:8080` (navegador o `curl`). Debe fallar: está **bloqueado** por el firewall.
10. Mira en el log del firewall que ha bloqueado esos intentos (activa `logging` si hace falta).
11. **Permite** el puerto 8080, pero **solo desde tu ordenador** (`192.168.56.1` suele ser tu anfitrión), y comprueba que ahora sí funciona.
12. Lista las reglas **numeradas** y **borra** la del puerto 8080. Comprueba que vuelve a estar bloqueado.

### Parte D · Reforzar SSH

13. Sustituye la regla de SSH por `limit` y razona qué hace.
14. Prueba a intentar 7-8 conexiones SSH rápidas con contraseña incorrecta. ¿Qué observas?
15. Revisa el estado final y para el servidor de prueba.

---

## ✅ Comprobación

| Prueba | Qué debes ver |
| ------ | ------------- |
| `sudo ufw status verbose` | `Status: active`, `deny (incoming)`, `allow (outgoing)` y tus reglas |
| SSH desde tu ordenador | Funciona |
| `curl http://192.168.56.10:8080` **antes** de permitirlo | Se queda esperando o falla |
| `curl http://192.168.56.10:8080` **después** de permitirlo | Devuelve el listado de archivos |
| `sudo tail -n 10 /var/log/ufw.log` | Líneas con `[UFW BLOCK]` e IP de origen y puerto de destino |
| `sudo ufw status numbered` | Las reglas con su número |

Desde Windows, para probar un puerto, usa PowerShell: `Test-NetConnection 192.168.56.10 -Port 8080` (nivel 1, laboratorio 11): te dice `TcpTestSucceeded : True/False`.

---

## 🧩 Pistas

- Si pierdes el acceso por SSH: entra por la **consola de VirtualBox**, ejecuta `sudo ufw disable` y revisa tus reglas.
- El orden importa: **permitir SSH antes de `enable`**.
- Para permitir desde una sola IP: `allow from IP to any port N`.
- Si `curl` no responde, comprueba que el servidor Python **sigue corriendo** en la VM (`ss -tuln`).
- `ufw` escribe los bloqueos en `/var/log/ufw.log` y también en el journal (`journalctl -k | grep UFW`).

---

## 🚀 Retos extra

1. Haz que el firewall **bloquee también la salida** hacia un destino concreto (por ejemplo, `ufw deny out to 8.8.8.8`) y comprueba el efecto con `ping`. Después quítalo.
2. Explora los **perfiles de aplicación** (`ufw app list`) y úsalos para permitir `OpenSSH`.
3. Mira cómo `ufw` traduce tus reglas con `sudo iptables -S` o `sudo nft list ruleset`. ¿Reconoces tus reglas?
4. Escanea **tu propia VM** con `nmap` desde tu ordenador (si lo instalas) y compara lo que ve **antes y después** del firewall. *(Solo sobre tus propias máquinas.)*
5. Razona en una frase: ¿por qué un firewall **no sustituye** a mantener los servicios actualizados y bien configurados?

---

## 📝 Qué documentar en tu `Solucion.md`

- La lista final de reglas y la razón de cada una.
- La salida del log del firewall con un bloqueo real (ofusca IPs que no sean tuyas).
- Qué pasó al saltarte el orden (si lo probaste) y cómo lo resolviste.

---

## 🧠 Lo que has practicado

`firewall` · `ufw` · `política deny/allow` · `lista blanca` · `reglas por puerto y origen` · `orden de reglas` · `ufw limit` · `logs del firewall`
