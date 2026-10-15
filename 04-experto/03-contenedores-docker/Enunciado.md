# 🐳 Laboratorio 03 · Contenedores con Docker

**Nivel:** 🔴 Experto · **Sistema:** 🐧 Linux · **Tiempo estimado:** 120 minutos

---

## 🎯 Escenario

Los desarrolladores de la empresa dicen siempre: *"en mi máquina funciona"*. Cada servidor tiene versiones distintas de librerías y configuraciones, y desplegar una aplicación es una lotería. Los **contenedores** resuelven esto: la aplicación viaja **empaquetada con todo lo que necesita**, y se ejecuta igual en cualquier sitio.

Tu misión es aprender a **construir, ejecutar y proteger** contenedores con **Docker**, y entender sus **riesgos de seguridad**, especialmente cómo interactúa con tu firewall.

> 💾 **Snapshot.** VM con **2 GB de RAM** o más. Necesitas la VM con internet (adaptador NAT) para descargar imágenes.

---

## 📖 Conceptos nuevos

### Contenedor frente a máquina virtual

| | **Máquina virtual** | **Contenedor** |
| - | ------------------- | -------------- |
| Qué incluye | Un **sistema operativo completo** con su kernel | Solo la **aplicación y sus dependencias**; **comparte el kernel** del anfitrión |
| Arranque | Decenas de segundos | **Milisegundos** |
| Tamaño | Gigabytes | Megabytes |
| Aislamiento | **Fuerte** (hardware virtual) | **Más débil** (aislamiento por el kernel: *namespaces* y *cgroups*) |

> **Analogía:** una VM es un **edificio entero** con sus propias tuberías. Un contenedor es un **apartamento** dentro de un edificio compartido: más ligero y barato, pero los vecinos comparten cimientos.

### Piezas de Docker

| Pieza | Qué es |
| ----- | ------ |
| **Imagen** | La "plantilla" inmutable: aplicación + dependencias. Se construye en **capas** |
| **Contenedor** | Una **instancia en ejecución** de una imagen |
| **Registro** (*registry*) | Almacén de imágenes (Docker Hub es el público) |
| **Dockerfile** | Receta de texto para **construir** una imagen |
| **Volumen** | Almacenamiento que **sobrevive** al contenedor (los datos del contenedor son efímeros) |
| **Red** | Los contenedores pueden comunicarse por **nombre** dentro de una red de Docker |
| **Docker Compose** | Describe **varios contenedores** juntos en un archivo YAML |

### Comandos de este laboratorio

| Comando | Qué hace |
| ------- | -------- |
| `docker run -d --name web -p 8080:80 nginx` | Crea y arranca un contenedor (`-d` en segundo plano, `--name` nombre, `-p 8080:80` publica el puerto 80 del contenedor en el **8080** del anfitrión) |
| `docker ps` / `docker ps -a` | Contenedores en marcha / todos |
| `docker logs -f web` | Sigue los logs del contenedor |
| `docker exec -it web sh` | Abre una **shell dentro** del contenedor |
| `docker stop web` / `docker rm web` | Para / elimina |
| `docker images` / `docker rmi imagen` | Lista / borra imágenes |
| `docker build -t miapp:1.0 .` | **Construye** una imagen con el `Dockerfile` de la carpeta actual |
| `docker volume create datos` / `docker volume ls` | Gestiona volúmenes |
| `docker network create mired` | Crea una red |
| `docker inspect web` | Todos los detalles de un contenedor, en JSON |
| `docker stats` | Consumo de recursos en vivo |
| `docker compose up -d` / `down` | Levanta / baja todo lo que define `compose.yaml` |
| `docker run --memory 256m --cpus 0.5 ...` | Limita recursos |

### Dockerfile: esqueleto

Completa las `???` (una app Python sencilla):

```dockerfile
FROM ???
WORKDIR /app
COPY ??? /app/
USER ???              # ¡no ejecutar como root!
EXPOSE 8000
CMD ["python3", "-m", "http.server", "8000"]
```

### ⚠️ Seguridad: lo que debes saber

| Riesgo | Detalle |
| ------ | ------- |
| **El grupo `docker` es equivalente a `root`** | Quien puede ejecutar `docker` puede montar `/` del anfitrión en un contenedor y hacer lo que quiera. **No añadas a cualquiera a ese grupo** |
| **Docker salta tu firewall `ufw`** | Docker crea **sus propias reglas** de red y **publica los puertos aunque `ufw` los bloquee**. Es una sorpresa muy común |
| **Imágenes no confiables** | Una imagen de origen desconocido puede traer malware. Usa **imágenes oficiales**, con versión fijada (no `latest`) |
| **Ejecutar como root dentro del contenedor** | Si alguien escapa del contenedor, lo hace con privilegios. Usa `USER` no privilegiado |
| **Montar `/var/run/docker.sock`** | Da **control total** del anfitrión al contenedor |

---

## 🛠️ Qué debes hacer

### Parte A · Instalar y primeros contenedores

1. **Instala Docker** siguiendo la **documentación oficial** de Docker para Ubuntu (lee cada comando, sobre todo el que añade el repositorio). Comprueba con `docker run hello-world`.
2. Lanza un contenedor **nginx** publicando el puerto 8080. Comprueba desde la VM y desde tu navegador (`http://192.168.56.10:8080`).
3. Abre una shell **dentro** del contenedor y averigua: qué usuario eres, qué procesos hay y cuántos son (¿pocos o muchos? ¿por qué?).
4. **Para y elimina** el contenedor. ¿Qué pasa con los datos que hubieras creado dentro?

### Parte B · Datos persistentes

5. Lanza un contenedor de una base de datos ligera (por ejemplo `redis` o `mariadb`, con una **versión fijada**) con un **volumen**. Guarda un dato.
6. **Destruye y vuelve a crear** el contenedor con el mismo volumen: comprueba que el dato **sigue ahí**.
7. Repite **sin** volumen y comprueba que **desaparece**.

### Parte C · Tu propia imagen

8. Crea una carpeta con un `index.html` y un `Dockerfile` que sirva ese contenido (con `nginx` o con Python, como en la plantilla).
9. **Construye** la imagen, ejecútala y comprueba que funciona.
10. Haz que el contenedor se ejecute con un **usuario sin privilegios** y compruébalo con `docker exec ... id`.
11. Aplica **límites** de memoria y CPU y compruébalos con `docker stats`.

### Parte D · Varios contenedores con Compose

12. Escribe un `compose.yaml` con **dos servicios**: tu web y una base de datos o un segundo servicio, en una **red común**. Comprueba que se resuelven **por nombre** (`ping nombre-del-servicio` dentro de un contenedor).
13. Levanta y baja todo con un solo comando.

### Parte E · La sorpresa del firewall

14. **Activa `ufw`** (política `deny incoming`; permite SSH) y **publica un contenedor** en el puerto 8080 **sin** abrir ese puerto en `ufw`.
15. Desde tu ordenador, intenta acceder a `http://192.168.56.10:8080`: ¿funciona? **Documenta** lo que observas y busca por qué (`sudo iptables -S`/`sudo nft list ruleset`).
16. Investiga **cómo solucionarlo**: publicar el puerto **solo en `127.0.0.1`** (`-p 127.0.0.1:8080:80`) o gestionar las reglas en la cadena `DOCKER-USER`.

---

## ✅ Comprobación

| Prueba | Qué debes ver |
| ------ | ------------- |
| `docker run hello-world` | Mensaje de bienvenida |
| `curl -I http://localhost:8080` | `200 OK` |
| Dato en volumen tras recrear | **Persiste** |
| Dato sin volumen tras recrear | **Se pierde** |
| `docker exec nombre id` | UID distinto de 0 (usuario sin privilegios) |
| `docker stats` | Límite de memoria y CPU aplicado |
| Servicios de Compose | Se hacen `ping` por nombre |
| Prueba del firewall | El puerto **responde aunque `ufw` lo bloquee** (y has aplicado la corrección) |

---

## 🧩 Pistas

- Si `docker` da *permission denied*, usa `sudo` (**no** añadas tu usuario al grupo `docker` sin entender qué implica).
- Una imagen y un contenedor **no** son lo mismo: puedes borrar el contenedor y conservar la imagen.
- En Dockerfile, cada instrucción crea una **capa**: pon lo que menos cambia **primero** para aprovechar la caché.
- `docker logs` es lo primero que miras cuando un contenedor "muere" nada más arrancar; y `docker ps -a` te dice con qué código de salida.
- El nombre de un servicio de Compose es su **nombre DNS** dentro de la red.

---

## 🚀 Retos extra

1. Escanea tus imágenes en busca de **vulnerabilidades conocidas** con `trivy` y razona qué harías con los resultados.
2. Reduce el tamaño de tu imagen con una base **`alpine`** o *multi-stage build* y compara.
3. Ejecuta un contenedor con **`--read-only`**, **`--cap-drop ALL`** y **`--security-opt no-new-privileges`**. ¿Qué dejas de poder hacer y por qué es más seguro?
4. Investiga qué es un **registro privado** y por qué una empresa lo querría.
5. Compara con **Podman** (sin demonio y sin necesidad de `root`): ¿qué problema de seguridad de Docker resuelve?

---

## 📝 Qué documentar en tu `Solucion.md`

- Tu `Dockerfile` y `compose.yaml` explicados línea a línea.
- Una **tabla comparativa** contenedor frente a VM con **tu** experiencia (tiempo de arranque, tamaño, memoria).
- La explicación de **por qué Docker saltó tu firewall** y cómo lo corregiste.
- Una **lista de buenas prácticas de seguridad** para contenedores, con tus palabras.

---

## 🧠 Lo que has practicado

`contenedores` · `imágenes y capas` · `Dockerfile` · `volúmenes` · `redes de Docker` · `Docker Compose` · `límites de recursos` · `seguridad de contenedores` · `Docker y firewall`
