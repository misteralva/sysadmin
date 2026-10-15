# 🔐 Laboratorio 04 · Permisos de ficheros

**Nivel:** 🟢 Principiante · **Sistema:** 🐧 Linux · **Tiempo estimado:** 60 minutos

---

## 🎯 Escenario

El departamento de Sistemas guarda un informe en el servidor, y quieres que:

- **`ana`** (la autora) pueda leerlo y modificarlo.
- Los demás miembros del grupo **`sistemas`** puedan **solo leerlo**.
- **Nadie más** (por ejemplo `marta`, de Redes) pueda ni abrirlo.

Además, hay un script de copia de seguridad que solo debe poder **ejecutar** el grupo `sistemas`.

> 📌 Usa los usuarios y grupos del laboratorio 03. Si restauraste un snapshot anterior, créalos de nuevo. Y recuerda: `luis` fue eliminado, así que **vuelve a crearlo** en el grupo `sistemas`.

---

## 📖 Conceptos nuevos

### Tres tipos de personas, tres tipos de permiso

Imagina un despacho en un edificio. Hay tres categorías de personas:

| Categoría | Quién es |
| --------- | -------- |
| **u** (*user*) | El **propietario** del archivo |
| **g** (*group*) | Los miembros del **grupo** propietario |
| **o** (*others*) | **Todos los demás** |

Y tres cosas que se pueden hacer:

| Permiso | En un archivo | En un directorio |
| ------- | ------------- | ---------------- |
| **r** (*read*) | Leerlo | Ver qué contiene (`ls`) |
| **w** (*write*) | Modificarlo | Crear y borrar archivos dentro |
| **x** (*execute*) | Ejecutarlo como programa | **Entrar** en él (`cd`) |

### Cómo leer `ls -l`

```
-rwxr-x--- 1 ana sistemas 1024 oct  9 10:00 informe.txt
│└┬┘└┬┘└┬┘   │   │
│ │  │  │    │   └─ grupo propietario
│ │  │  │    └───── usuario propietario
│ │  │  └────────── permisos de los demás (o)
│ │  └───────────── permisos del grupo (g)
│ └──────────────── permisos del propietario (u)
└────────────────── tipo (- archivo, d directorio, l enlace)
```

### Permisos en número (notación octal)

Cada permiso vale un número, y se suman:

| r | w | x |
| - | - | - |
| 4 | 2 | 1 |

Así, `rwx` = 4+2+1 = **7**, `r-x` = 4+0+1 = **5**, `r--` = **4**, `---` = **0**.
Con tres cifras (propietario, grupo, otros): `750` = `rwxr-x---`.

### Comandos de este laboratorio

| Comando | Qué hace |
| ------- | -------- |
| `ls -l` / `ls -ld directorio` | Muestra permisos (con `-d` muestra el propio directorio, no su contenido) |
| `chmod modo archivo` | Cambia permisos. Admite notación octal (`chmod 640 f`) o simbólica (`chmod g+w f`, `chmod o-r f`) |
| `chown usuario:grupo archivo` | Cambia propietario y grupo |
| `chgrp grupo archivo` | Cambia solo el grupo |
| `umask` | Muestra los permisos que se **quitan** por defecto a los archivos nuevos |
| `sudo -u usuario comando` | Ejecuta un comando **como si fueras ese usuario** (ideal para probar permisos) |

> ⚠️ **Nunca** uses `chmod 777` "para que funcione": da permisos totales a todo el mundo y es un agujero de seguridad. Y ten mucho cuidado con `-R` (recursivo): un `chmod -R` o `chown -R` mal dirigido puede dejar el sistema inservible.

---

## 🛠️ Qué debes hacer

1. Con el usuario `ana`, crea una carpeta `~/informes` y dentro el archivo `informe.txt` con algún texto.
2. Cambia el **grupo propietario** de `informe.txt` a `sistemas`.
3. Ajusta los permisos para que:
   - `ana` pueda leer y escribir,
   - el grupo `sistemas` solo pueda leer,
   - los demás no tengan ningún permiso.

   Hazlo primero en notación **octal** y luego repítelo en notación **simbólica**.
4. Crea un script `backup.sh` con el contenido que quieras (por ejemplo, un `echo "copia hecha"`) y dale permisos para que `ana` pueda **leer, escribir y ejecutar**, el grupo `sistemas` **leer y ejecutar**, y los demás nada.
5. **Ejecuta** el script. ¿Qué pasa si le quitas el permiso de ejecución antes de lanzarlo?
6. Comprueba qué pasa con una **carpeta**: quítale a `~/informes` el permiso `x` para el grupo y trata de entrar con `luis`. Después devuélvelo.
7. Consulta tu `umask` y crea un archivo nuevo. ¿Qué permisos tiene por defecto? ¿Cómo se relaciona con la `umask`?

---

## ✅ Comprobación

Prueba los permisos **como otro usuario** con `sudo -u`:

| Comando | Qué hace | Qué debes ver |
| ------- | -------- | ------------- |
| `ls -l ~/informes` | Muestra permisos y propietarios | `-rw-r-----` para el informe, `ana sistemas` |
| `sudo -u luis cat /home/ana/informes/informe.txt` | `luis` intenta leer | Debe **funcionar** (es de `sistemas`) |
| `sudo -u luis sh -c 'echo x >> /home/ana/informes/informe.txt'` | `luis` intenta escribir | **Permission denied** |
| `sudo -u marta cat /home/ana/informes/informe.txt` | `marta` intenta leer | **Permission denied** |

> 💡 Si `luis` y `marta` no pueden ni entrar en `/home/ana`, es porque la carpeta personal de `ana` tiene sus propios permisos. Piensa por qué, y qué cambio **mínimo** necesitas hacer (o si es mejor usar otra ubicación, como `/srv`).

---

## 🧩 Pistas

- Para leer un archivo hace falta `r` en el archivo y `x` en **todas las carpetas** del camino hasta él.
- Octal: para `rw-r-----` calcula cada grupo de tres por separado: `rw-` = 6, `r--` = 4, `---` = 0.
- `chmod u+x,g+x,o-rwx fichero` combina varios cambios en un solo comando.
- Con `chown ana:sistemas fichero` cambias propietario y grupo de una vez.

---

## 🚀 Retos extra

1. **Bit setgid en un directorio:** crea `/srv/proyecto` con grupo `sistemas` y aplica `chmod g+s`. Después crea un archivo dentro y observa qué grupo recibe. ¿Para qué sirve en una carpeta compartida?
2. **Sticky bit:** ejecuta `ls -ld /tmp` y fíjate en la `t` final. ¿Por qué en `/tmp` cualquiera puede crear archivos pero no puede borrar los de otros?
3. Cambia tu `umask` para que los archivos nuevos queden sin permisos para "otros". Comprueba el resultado.
4. Investiga las **ACL** (`getfacl` / `setfacl`): ¿qué problema resuelven que los permisos clásicos no pueden resolver? (Pista: ¿cómo darías a un segundo grupo permisos distintos sobre el mismo archivo?)

---

## 📝 Qué documentar en tu `Solucion.md`

- Los comandos, con la traducción octal ↔ simbólica de cada permiso que apliques.
- La salida de `ls -l` antes y después.
- Los resultados de las pruebas con `sudo -u`.
- La respuesta razonada a la pregunta de la carpeta personal de `ana`.

---

## 🧠 Lo que has practicado

`rwx` · `propietario/grupo/otros` · `notación octal y simbólica` · `chmod` · `chown` · `chgrp` · `umask` · `permisos en directorios` · `sudo -u`
