# 📦 Laboratorio 05 · Instalar y gestionar paquetes con `apt`

**Nivel:** 🟢 Principiante · **Sistema:** 🐧 Linux · **Tiempo estimado:** 45 minutos

---

## 🎯 Escenario

Tu servidor recién instalado está desactualizado y le faltan herramientas básicas de administración. Tienes que:

1. **Actualizarlo** a las últimas versiones.
2. **Instalar** un pequeño kit de herramientas: `tree`, `htop`, `curl` y `ncdu`.
3. Aprender a averiguar **qué ha instalado** cada paquete y cómo **eliminarlo limpiamente**.

> 💾 Haz un **snapshot** antes de actualizar el sistema. Una actualización grande puede cambiar muchas cosas, y así podrás volver atrás.

---

## 📖 Conceptos nuevos

### Paquetes, repositorios y gestor de paquetes

En Linux no se descarga un `.exe` de una web. Se usa un **gestor de paquetes**:

- Un **paquete** es un programa empaquetado con la información de lo que necesita para funcionar (sus **dependencias**).
- Un **repositorio** es un servidor con miles de paquetes, firmados y revisados. Es como la tienda de aplicaciones del sistema.
- El **gestor de paquetes** (`apt` en Ubuntu y Debian) consulta los repositorios, descarga el paquete **y todas sus dependencias**, y lo instala.

Ventaja de seguridad: los paquetes de los repositorios oficiales están **firmados**, y se actualizan todos desde un único sitio.

### `update` no es `upgrade`

Es la confusión más común:

| Comando | Qué hace | Analogía |
| ------- | -------- | -------- |
| `apt update` | **Actualiza la lista** de paquetes disponibles. No instala nada | Leer el catálogo nuevo de la tienda |
| `apt upgrade` | **Instala** las versiones nuevas de lo que ya tienes | Comprar lo que ha salido nuevo |

Siempre se ejecuta primero `update` y luego `upgrade`.

### Comandos de este laboratorio

| Comando | Qué hace |
| ------- | -------- |
| `sudo apt update` | Refresca el catálogo de paquetes |
| `sudo apt upgrade` | Instala las actualizaciones disponibles |
| `apt search texto` | Busca paquetes por nombre o descripción |
| `apt show paquete` | Muestra información de un paquete (versión, tamaño, dependencias) |
| `sudo apt install paquete` | Instala un paquete (y sus dependencias) |
| `sudo apt remove paquete` | Desinstala el paquete pero **conserva** sus archivos de configuración |
| `sudo apt purge paquete` | Desinstala el paquete **y** sus archivos de configuración |
| `sudo apt autoremove` | Elimina dependencias que ya nadie necesita |
| `apt list --installed` | Lista los paquetes instalados |
| `dpkg -L paquete` | Lista **los archivos** que instaló un paquete |
| `dpkg -S /ruta/archivo` | Dice **a qué paquete** pertenece un archivo |
| `apt policy paquete` | Muestra de qué repositorio viene y qué versión instalaría |
| `which comando` | Muestra dónde está el ejecutable de un comando |

> 💡 `dpkg` es la herramienta de bajo nivel que instala un paquete individual. `apt` trabaja por encima: resuelve dependencias y descarga desde repositorios.

---

## 🛠️ Qué debes hacer

1. **Actualiza** el catálogo y después instala las actualizaciones pendientes. Anota cuántos paquetes se actualizan.
2. **Busca** un paquete que muestre el árbol de directorios (pista: el comando se llama `tree`) y **mira su información** antes de instalarlo.
3. **Instala** `tree`, `htop`, `curl` y `ncdu` (puedes hacerlo en un solo comando).
4. Comprueba **dónde** quedó instalado el ejecutable de cada uno y qué **archivos** instaló el paquete `tree`.
5. Averigua **a qué paquete pertenece** el archivo `/bin/ls`.
6. **Desinstala** `ncdu` conservando su configuración. Después **purga** `htop`. ¿Qué diferencia observas?
7. Limpia con `autoremove` y anota qué elimina.
8. Mira **qué repositorios** usa tu sistema (los archivos de configuración están en `/etc/apt/`).

---

## ✅ Comprobación

| Comando | Qué hace | Qué debes ver |
| ------- | -------- | ------------- |
| `which tree` | Ruta del ejecutable | Algo como `/usr/bin/tree` |
| `tree --version` | Versión instalada | Un número de versión |
| `apt list --installed 2>/dev/null \| grep -E "^(tree\|curl)/"` | Lista instalados filtrando por nombre (`\|` encadena comandos, `grep` filtra, `2>/dev/null` oculta avisos) | `tree` y `curl` aparecen |
| `which htop` | Tras purgar `htop` | No devuelve nada |
| `dpkg -l ncdu` | Estado de `ncdu` tras `remove` | Estado `rc` (*removed, config files remain*) |

---

## 🧩 Pistas

- Si `apt install` te pregunta `¿Desea continuar? [S/n]`, lee la lista de paquetes antes de aceptar: es una buena costumbre.
- Con `-y` se acepta sin preguntar, pero **no lo uses** hasta que entiendas qué va a hacer cada comando.
- Si `apt` dice "Could not get lock", otro proceso está usando el gestor (a menudo una actualización automática). Espera un minuto y repite.
- Para simular una instalación sin ejecutarla, añade `-s` (*simulate*): `apt install -s paquete`.

---

## 🚀 Retos extra

1. **Simula** la instalación de un paquete grande (`apt install -s apache2`) y cuenta cuántas dependencias arrastra.
2. Descarga un paquete **sin instalarlo** con `apt download tree` y examina qué contiene un archivo `.deb`.
3. Averigua para qué sirve el paquete `unattended-upgrades` y por qué es una buena práctica de seguridad en servidores.
4. Compara `apt list --upgradable` antes y después de `apt update`. ¿Qué cambia y por qué?
5. Busca qué diferencia hay entre `apt` y `apt-get`.

---

## 📝 Qué documentar en tu `Solucion.md`

- Qué paquetes se actualizaron y cuántos.
- La diferencia entre `remove` y `purge` con la evidencia que viste.
- A qué paquete pertenece `/bin/ls` y cómo lo descubriste.
- Qué repositorios usa tu sistema.

---

## 🧠 Lo que has practicado

`repositorios` · `dependencias` · `apt update/upgrade` · `apt search/show/install/remove/purge` · `autoremove` · `dpkg -L / -S` · `which`
