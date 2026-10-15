# ⚙️ Laboratorio 06 · Procesos

**Nivel:** 🟢 Principiante · **Sistema:** 🐧 Linux · **Tiempo estimado:** 60 minutos

---

## 🎯 Escenario

Un compañero te avisa: *"El servidor va lento"*. Antes de reiniciar nada, tienes que **encontrar qué proceso está consumiendo recursos**, entender de dónde viene y **terminarlo de forma controlada**.

Como todavía no hay un problema real, tú mismo lo provocarás en tu VM para practicar el diagnóstico.

---

## 📖 Conceptos nuevos

### Programa frente a proceso

- Un **programa** es un archivo guardado en disco: una receta de cocina.
- Un **proceso** es ese programa **en ejecución**: el cocinero cocinando la receta.

Puedes tener un solo programa y muchos procesos a la vez (varios cocineros con la misma receta).

### Identificadores y jerarquía

- Cada proceso tiene un **PID** (*Process ID*), un número único.
- Cada proceso nace de otro, su **padre**, identificado por su **PPID**. Es un árbol: todos descienden del proceso con **PID 1** (`systemd`), que arranca al iniciar el sistema.

### Estados de un proceso

| Letra | Estado |
| ----- | ------ |
| `R` | *Running*: ejecutándose o listo para ejecutarse |
| `S` | *Sleeping*: esperando algo (lo normal en casi todos) |
| `T` | *Stopped*: detenido |
| `Z` | *Zombie*: terminó pero su padre aún no lo ha "recogido" |

### Primer plano y segundo plano

Un comando en **primer plano** ocupa tu terminal hasta que acaba. Si lo lanzas con `&` al final, va a **segundo plano** y recuperas el control.

### Señales: cómo "hablar" con un proceso

Las **señales** son mensajes que se envían a un proceso. Las tres que debes conocer:

| Señal | Número | Qué significa | Analogía |
| ----- | ------ | ------------- | -------- |
| `SIGTERM` | 15 | "Por favor, termina" (el proceso puede cerrar con calma) | Pedirle al cocinero que apague el fuego y recoja |
| `SIGKILL` | 9 | "Muere ya" (el sistema lo elimina sin dejar que se despida) | Cortar la luz de la cocina |
| `SIGHUP` | 1 | "Se cerró la terminal" (muchos servicios lo usan para recargar su configuración) | Avisar de un cambio |

> ⚠️ **Siempre prueba primero con `SIGTERM`.** `SIGKILL` no deja que el programa guarde datos ni cierre archivos, y puede dejar cosas a medias. Úsalo solo si el proceso no responde.

### Comandos de este laboratorio

| Comando | Qué hace |
| ------- | -------- |
| `ps aux` | Lista **todos** los procesos con usuario, uso de CPU/memoria y comando (`a`=de todos los usuarios, `u`=formato detallado, `x`=incluye los sin terminal) |
| `ps -eo pid,ppid,stat,comm` | Lista procesos eligiendo columnas: PID, PPID, estado y nombre |
| `top` | Monitor en vivo ordenado por consumo de CPU (sales con `q`) |
| `pgrep -a nombre` | Busca procesos por nombre y muestra su PID y comando |
| `kill PID` | Envía `SIGTERM` al proceso (`kill -9 PID` envía `SIGKILL`) |
| `pkill nombre` | Igual que `kill`, pero buscando por nombre |
| `sleep N` | No hace nada durante N segundos (útil para practicar) |
| `jobs`, `fg`, `bg` | Gestionan tareas de la terminal actual (listar, traer a primer plano, mandar a segundo plano) |
| `nice` / `renice` | Cambian la **prioridad** de un proceso (de −20, máxima, a 19, mínima) |
| `uptime` | Tiempo encendido y carga media del sistema |

También verás dos símbolos que se usarán mucho:

- `|` (tubería, *pipe*): pasa la salida de un comando como entrada del siguiente. Ejemplo: `ps aux | grep sleep`.
- `> /dev/null`: envía la salida de un comando a "la nada" (`/dev/null` descarta todo lo que recibe).

---

## 🛠️ Qué debes hacer

1. **Explora** los procesos: ejecuta `ps aux` y localiza el proceso con PID 1. ¿Cuál es? ¿Quién es su padre?
2. Abre `top`, identifica en la cabecera la **carga media** y los procesos que más CPU gastan, y sal.
3. **Lanza** un proceso en segundo plano: `sleep 600 &`. Anota su **PID**.
4. Localízalo con `ps` y con `pgrep`. Comprueba su estado y su PPID.
5. **Termínalo** de forma amable con `SIGTERM`. Verifica que ha desaparecido.
6. Lanza otro `sleep 600 &` y esta vez termínalo con `SIGKILL`. ¿Notas alguna diferencia visible?
7. **Provoca "lentitud"**: lanza `yes > /dev/null &`. (`yes` escribe la letra "y" sin parar; al redirigirla a `/dev/null` solo consume CPU.)
8. Con `top`, **encuentra** ese proceso, comprueba su consumo de CPU y su PID.
9. **Baja su prioridad** con `renice` y observa el efecto. Después **elimínalo**.
10. Lanza un comando largo **en primer plano** (por ejemplo `sleep 300`), **suspéndelo** con `Ctrl+Z`, mira `jobs`, y vuélvelo a segundo plano con `bg`. Termina con `kill`.

---

## ✅ Comprobación

| Comando | Qué hace | Qué debes ver |
| ------- | -------- | ------------- |
| `pgrep -a sleep` | Busca procesos `sleep` | Nada tras terminarlos |
| `ps aux \| grep yes` | Busca `yes` (verás también el propio `grep`) | Nada relevante tras matarlo |
| `ps -eo pid,ppid,stat,comm \| head` | Primeras líneas del árbol | `systemd` con PID 1 |
| `uptime` | Carga del sistema | La carga baja tras eliminar `yes` |

---

## 🧩 Pistas

- `$!` guarda el PID del último proceso lanzado en segundo plano: `echo $!` justo tras lanzarlo.
- En `top`, pulsa `P` para ordenar por CPU, `M` por memoria, `k` para enviar una señal a un PID y `q` para salir.
- `kill -l` lista todas las señales disponibles.
- Para bajar la prioridad de un proceso en marcha: `renice -n 10 -p PID` (un número `nice` más alto significa **menos** prioridad).

---

## 🚀 Retos extra

1. Instala `htop` (lo viste en el laboratorio 05) y compara con `top`. ¿Qué ventajas ves?
2. Usa `pstree` para ver el árbol de procesos. ¿Quién cuelga de `sshd` o de `systemd`?
3. Lanza un proceso con `nohup comando &` y cierra la terminal. ¿Qué cambia respecto a lanzarlo sin `nohup`?
4. Investiga el **estado zombi**: ¿por qué aparece y por qué no se puede "matar" un zombi?
5. Averigua cuánta memoria usa cada proceso y ordena de mayor a menor con `ps aux --sort=-%mem | head`.

---

## 📝 Qué documentar en tu `Solucion.md`

- El PID, estado y PPID de cada proceso que lances.
- Qué diferencia observaste entre `SIGTERM` y `SIGKILL`.
- Capturas de `top` antes y después de matar `yes`.
- Un pequeño "protocolo de diagnóstico" con tus palabras: qué mirarías primero si un servidor real va lento.

---

## 🧠 Lo que has practicado

`PID/PPID` · `estados` · `ps` · `top` · `pgrep` · `kill/pkill` · `señales` · `& / jobs / fg / bg` · `nice/renice` · `pipes` · `/dev/null`
