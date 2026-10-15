# 🌍 Laboratorio 10 · Servidor web con Nginx

**Nivel:** 🟡 Intermedio · **Sistema:** 🐧 Linux · **Tiempo estimado:** 90 minutos

---

## 🎯 Escenario

La empresa quiere una **intranet**: una pequeña web interna para publicar avisos y documentación. Tu misión es montar un **servidor web con Nginx**, alojar **dos sitios** distintos en la misma máquina, revisar sus registros y abrir solo el puerto necesario en el firewall.

> 📌 Necesitas la red solo-anfitrión (laboratorio 03) para verlo desde tu navegador, y el firewall del laboratorio 07 ya activo (tendrás que abrirle el puerto 80).

---

## 📖 Conceptos nuevos

### Cómo funciona una web

1. Tu navegador (el **cliente**) pide una página por **HTTP** (puerto **80**) o **HTTPS** (puerto **443**).
2. El **servidor web** (aquí, Nginx) busca el archivo pedido y lo devuelve.
3. El navegador lo muestra.

Los códigos de respuesta que debes conocer: `200` (OK), `301`/`302` (redirección), `403` (prohibido), `404` (no encontrado), `500` (error del servidor).

### Nginx

Nginx (se pronuncia "engine-x") es uno de los servidores web más usados del mundo: ligero y rápido. También se usa como **proxy inverso** y balanceador (en niveles avanzados).

### Estructura de configuración (en Ubuntu)

| Ruta | Para qué sirve |
| ---- | -------------- |
| `/etc/nginx/nginx.conf` | Configuración **general** |
| `/etc/nginx/sites-available/` | Configuraciones de sitios **disponibles** (aunque no estén activas) |
| `/etc/nginx/sites-enabled/` | Sitios **activos**: son **enlaces simbólicos** a `sites-available` |
| `/var/www/` | Dónde suelen vivir los archivos de las webs |
| `/var/log/nginx/access.log` | Registro de **cada petición** recibida |
| `/var/log/nginx/error.log` | Registro de **errores** |

Para activar un sitio, creas un enlace en `sites-enabled` que apunte a su archivo de `sites-available`; para desactivarlo, borras el enlace (no el archivo).

### Bloques `server` (hosts virtuales)

Con **un solo Nginx** puedes servir **varias webs** distintas. Cada una se define con un bloque `server`. Nginx decide cuál usar según el **puerto** y el **nombre** que pide el navegador (`server_name`). Plantilla (completa las `???`):

```nginx
server {
    listen ???;
    server_name ???;

    root ???;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

| Directiva | Qué hace |
| --------- | -------- |
| `listen` | Puerto en el que escucha |
| `server_name` | Nombre de dominio que atiende este bloque |
| `root` | Carpeta con los archivos de la web |
| `index` | Archivo que se sirve al pedir una carpeta |
| `try_files` | Busca el archivo pedido; si no existe, devuelve 404 |

### Comandos de este laboratorio

| Comando | Qué hace |
| ------- | -------- |
| `sudo apt install nginx` | Instala Nginx |
| `sudo nginx -t` | **Comprueba** que la configuración es válida (úsalo siempre antes de recargar) |
| `sudo systemctl reload nginx` | Aplica cambios **sin cortar** conexiones activas |
| `sudo systemctl restart nginx` | Reinicia el servicio |
| `curl -I http://localhost` | Pide solo las cabeceras HTTP (`-I`), para ver el código de respuesta |
| `curl -H "Host: nombre" http://localhost` | Hace una petición **fingiendo pedir un nombre concreto** (para probar `server_name` sin DNS) |
| `sudo ln -s origen enlace` | Crea un enlace simbólico |
| `sudo tail -f /var/log/nginx/access.log` | Sigue las peticiones en vivo |
| `sudo ufw allow from 192.168.56.0/24 to any port 80 proto tcp` | Abre el puerto 80 solo para tu red de laboratorio |
| `sudo chown -R www-data:www-data /var/www/empresa` | Da la carpeta al usuario con el que corre Nginx |

---

## 🛠️ Qué debes hacer

### Parte A · Instalar y comprobar

1. Instala Nginx y comprueba que el servicio está **activo** y **habilitado**.
2. Comprueba con `ss -tuln` que escucha en el **puerto 80**.
3. Prueba desde la propia VM con `curl -I http://localhost`. ¿Qué código devuelve?
4. Desde **tu navegador**, intenta abrir `http://192.168.56.10`. Si el firewall está activo, **fallará**: razona por qué y **abre el puerto 80** solo para tu red de laboratorio. Vuelve a probar.

### Parte B · Tu primer sitio

5. Crea el directorio `/var/www/empresa` y dentro un `index.html` sencillo (con un título que diga "Intranet de la empresa").
6. Dale propiedad a `www-data` con `chown`.
7. Crea `/etc/nginx/sites-available/empresa` con un bloque `server` (usa la plantilla) para `empresa.lab` en el puerto 80.
8. **Actívalo** con un enlace simbólico en `sites-enabled`.
9. **Desactiva el sitio por defecto** (el enlace `default` de `sites-enabled`).
10. Comprueba la configuración (`nginx -t`) y **recarga**.
11. Prueba con `curl -H "Host: empresa.lab" http://localhost`.

### Parte C · Segundo sitio en otro puerto

12. Crea `/var/www/docs` con otro `index.html` ("Documentación técnica").
13. Crea un segundo bloque `server` que sirva ese sitio en el **puerto 8081**.
14. Abre el puerto 8081 en el firewall (solo para tu red de laboratorio) y comprueba que ves **las dos webs** desde tu navegador.

### Parte D · Nombres y logs

15. En **tu ordenador**, edita el archivo `hosts` para que `empresa.lab` apunte a `192.168.56.10`:
    - Windows: `C:\Windows\System32\drivers\etc\hosts` (como administrador)
    - Linux/macOS: `/etc/hosts`
16. Abre `http://empresa.lab` en el navegador.
17. Mira `access.log` en vivo mientras recargas la página: identifica la **IP**, la **ruta pedida** y el **código** de cada petición.
18. Pide una URL que **no exista** y localiza el 404 en el log.
19. Crea una **página de error 404 personalizada** con la directiva `error_page`.

---

## ✅ Comprobación

| Prueba | Qué debes ver |
| ------ | ------------- |
| `systemctl is-active nginx` / `is-enabled nginx` | `active` y `enabled` |
| `sudo nginx -t` | `syntax is ok` y `test is successful` |
| `curl -I -H "Host: empresa.lab" http://localhost` | `HTTP/1.1 200 OK` |
| Navegador → `http://empresa.lab` | "Intranet de la empresa" |
| Navegador → `http://192.168.56.10:8081` | "Documentación técnica" |
| `sudo tail -n 3 /var/log/nginx/access.log` | Tus peticiones, con código 200 y 404 |
| `ls -l /etc/nginx/sites-enabled` | Enlaces a tus dos sitios (y sin `default`) |

---

## 🧩 Pistas

- Si `nginx -t` da error, **lee la línea y el archivo** que indica: casi siempre es un `;` o una `}` que falta.
- Si ves la página por defecto de Nginx en vez de la tuya, ¿has quitado el sitio `default`? ¿Has recargado?
- Si te sale **403 Forbidden**, revisa los permisos: `www-data` debe poder **leer** los archivos y **entrar** (`x`) en todas las carpetas del camino.
- Si el navegador no llega a la VM, comprueba la IP, el firewall y que Nginx escucha en el puerto correcto.
- Para escribir `index.html` rápido: `echo "<h1>Intranet de la empresa</h1>" | sudo tee /var/www/empresa/index.html` (`tee` escribe lo que recibe en el archivo y necesita `sudo` aquí; una redirección `>` con `sudo` no sirve, porque la hace tu shell, no `sudo`).

---

## 🚀 Retos extra

1. **HTTPS:** genera un **certificado autofirmado** con `openssl` y sirve la web por el puerto 443. Observa el aviso del navegador: ¿por qué aparece? (Adelanta la PKI del nivel experto.)
2. Oculta la versión de Nginx con `server_tokens off;` y comprueba el efecto con `curl -I`.
3. Redirige todo el tráfico HTTP a HTTPS con una respuesta `301`.
4. Protege una carpeta con **autenticación básica** (`auth_basic` y `htpasswd`).
5. Haz una prueba de carga sencilla con `ab` (paquete `apache2-utils`) y observa cómo se comporta el servidor. Hazlo solo contra tu propia VM.

---

## 📝 Qué documentar en tu `Solucion.md`

- Tus dos bloques `server` completos, explicados línea a línea.
- Capturas de las dos webs funcionando.
- Un fragmento de `access.log` y cómo lo interpretas.
- Una explicación de por qué se usan enlaces simbólicos en `sites-enabled`.

---

## 🧠 Lo que has practicado

`HTTP y códigos de respuesta` · `Nginx` · `sites-available / sites-enabled` · `bloques server` · `nginx -t` · `hosts virtuales` · `access.log / error.log` · `permisos de www-data` · `firewall + servicio web`
