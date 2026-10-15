# 📂 Laboratorio 02 · Moverse por el sistema de ficheros

**Nivel:** 🟢 Principiante · **Sistema:** 🐧 Linux · **Tiempo estimado:** 45 minutos

---

## 🎯 Escenario

Te acaban de dar acceso a un servidor Linux que no conoces. No hay entorno gráfico, solo una terminal. Antes de tocar nada, tu jefe te pide que lo **explores**, entiendas cómo está organizado y prepares una estructura de carpetas para los tres departamentos de la empresa.

---

## 📖 Conceptos nuevos

### Todo parte de una raíz

En Linux no hay "discos C: y D:". Todo cuelga de un único punto, la **raíz**, que se escribe `/`. Imagina un árbol invertido (o un edificio): la raíz es la planta baja y de ella salen todas las carpetas.

### Los directorios principales

| Directorio | Para qué sirve |
| ---------- | -------------- |
| `/home` | Las carpetas personales de los usuarios |
| `/etc` | Los archivos de **configuración** del sistema y de los servicios |
| `/var` | Datos que **cambian** con el tiempo: logs, colas, cachés |
| `/tmp` | Archivos temporales (se borran al reiniciar) |
| `/usr` | Programas y librerías instalados |
| `/bin`, `/sbin` | Programas básicos del sistema |
| `/dev` | Los **dispositivos** (discos, terminales…) representados como archivos |
| `/proc` | Información en vivo del sistema y de los procesos |

> 💡 En Linux "casi todo es un archivo": hasta un disco duro o un proceso se representan como un archivo dentro de un directorio.

### Rutas absolutas y relativas

- **Absoluta:** empieza por `/` y funciona desde cualquier sitio. Es como dar una dirección completa: *"Calle Mayor 5, 2º B, Barcelona"*.
- **Relativa:** parte de donde estás ahora. Es como decir *"sube un piso y gira a la derecha"*.

Atajos útiles:

| Símbolo | Significa |
| ------- | --------- |
| `.` | El directorio actual |
| `..` | El directorio padre (el de arriba) |
| `~` | Tu carpeta personal |
| `/` | La raíz |

### Comandos de este laboratorio

| Comando | Qué hace |
| ------- | -------- |
| `pwd` | Muestra dónde estás (*print working directory*) |
| `cd ruta` | Cambia de directorio (*change directory*). Sin ruta, te lleva a `~` |
| `ls` | Lista el contenido. Con `-l` da detalles, con `-a` muestra los ocultos (empiezan por `.`), con `-h` pone los tamaños legibles |
| `mkdir nombre` | Crea un directorio. Con `-p` crea también los directorios intermedios |
| `touch fichero` | Crea un archivo vacío (o actualiza su fecha si ya existe) |
| `cp origen destino` | Copia (con `-r` para copiar directorios completos) |
| `mv origen destino` | Mueve **o renombra** |
| `rm fichero` | Borra un archivo (con `-r` borra directorios y su contenido) |
| `cat fichero` | Muestra el contenido de un archivo |
| `less fichero` | Muestra un archivo página a página (sales con `q`) |

> ⚠️ **Cuidado con `rm`:** en Linux no hay papelera. Lo que borras, desaparece. Y nunca ejecutes `rm -r` sobre rutas que no tengas clarísimas, mucho menos sobre `/`.

---

## 🛠️ Qué debes hacer

1. **Averigua dónde estás** al iniciar sesión y vuelve siempre a tu carpeta personal cuando te pierdas.
2. **Explora** los directorios `/etc`, `/var/log`, `/home`, `/tmp`, `/usr/bin` y `/dev`. En tu `Solucion.md`, escribe con tus palabras qué has encontrado en cada uno (no copies la tabla de arriba).
3. **Crea esta estructura** dentro de tu carpeta personal:

```
empresa/
├── administracion/
│   ├── documentos/
│   └── copias/
├── sistemas/
│   ├── documentos/
│   └── copias/
└── redes/
    ├── documentos/
    └── copias/
```

4. Crea el archivo vacío `inventario.txt` dentro de `empresa/sistemas/documentos/`.
5. Escribe algo dentro de ese archivo (¿cómo se puede escribir en un archivo sin abrir un editor? Investiga la redirección con `>` y el comando `echo`).
6. **Copia** el archivo a la carpeta `copias` del mismo departamento.
7. **Renombra** la copia a `inventario-backup.txt`.
8. **Mueve** el archivo original a `empresa/redes/documentos/`.
9. Desde `empresa/redes/`, llega a `empresa/sistemas/copias/` usando **solo rutas relativas**. Después, hazlo desde `/tmp` usando **solo rutas absolutas**.
10. **Borra** la carpeta `empresa/administracion` entera.

---

## ✅ Comprobación

| Comando | Qué hace | Qué debes ver |
| ------- | -------- | ------------- |
| `ls -R ~/empresa` | Lista todo recursivamente (`-R` = *recursive*) | Solo `sistemas` y `redes`, con sus subcarpetas |
| `cat ~/empresa/redes/documentos/inventario.txt` | Muestra el contenido del archivo movido | El texto que escribiste |
| `ls ~/empresa/sistemas/copias` | Lista las copias | `inventario-backup.txt` |
| `ls ~/empresa/sistemas/documentos` | Lista los documentos | Vacía (moviste el original) |

---

## 🧩 Pistas

- Si no sabes cuántas veces escribir `cd ..`, recuerda que puedes encadenar: `cd ../..`.
- El tabulador (`Tab`) autocompleta rutas. Úsalo siempre: ahorra errores y tiempo.
- La flecha hacia arriba recupera comandos anteriores, y `history` los lista todos.
- Con `ls -l`, la primera letra de cada línea indica el tipo: `-` archivo, `d` directorio, `l` enlace.

---

## 🚀 Retos extra

1. Crea toda la estructura del paso 3 con **un único comando `mkdir`**. Pista: investiga la expansión de llaves `{a,b,c}` en Bash.
2. Ejecuta `ls -l /` y busca directorios con una `l` al principio. ¿A dónde apunta `/bin`? ¿Qué es un enlace simbólico?
3. Usa `find ~/empresa -name "*.txt"` para buscar archivos por nombre. ¿Qué hace `find` y por qué es más potente que `ls`?
4. Usa `ls -lah /var/log` y ordena por tamaño con `ls -lahS` (`-S` ordena de mayor a menor). ¿Cuál es el log más grande?

---

## 📝 Qué documentar en tu `Solucion.md`

- Para qué sirve cada directorio explorado, con tus palabras.
- Los comandos usados, cada uno con su explicación.
- Qué errores te salieron (¿rutas mal escritas?, ¿permisos denegados?) y cómo los resolviste.

---

## 🧠 Lo que has practicado

`jerarquía de directorios` · `rutas absolutas y relativas` · `pwd` · `cd` · `ls` · `mkdir` · `touch` · `cp` · `mv` · `rm` · `cat` · `less` · `echo` · `>`
