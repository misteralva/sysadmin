# 📜 Laboratorio 05 · PKI y certificados: tu propia autoridad de certificación

**Nivel:** 🔴 Experto · **Sistema:** 🐧 Linux (con verificación desde Windows) · **Tiempo estimado:** 120 minutos

---

## 🎯 Escenario

Los servicios internos de la empresa (la intranet, el servidor de logs, el LDAP) viajan **sin cifrar**: cualquiera en la red puede leer lo que pasa. Para cifrarlos hace falta **HTTPS/TLS**, y para eso hacen falta **certificados**.

No vas a pedir un certificado a una entidad pública (es una red interna que internet no ve). Vas a hacer lo que hacen las empresas: **montar tu propia autoridad de certificación (CA)**, emitir certificados con ella y conseguir que **los equipos de la empresa confíen** en esa CA. Verás por qué el navegador mostraba avisos de "no seguro" en el laboratorio 10 del nivel intermedio y cómo se eliminan **correctamente**.

> 💾 **Snapshot** de las dos VMs. La **CA** y el **servidor web** vivirán en `ubuntu-server-lab`; el **cliente** que debe confiar será `ubuntu-cliente` (y, para el reto, tu Windows).

---

## 📖 Conceptos nuevos

### Qué problema resuelve un certificado

Cuando te conectas a `https://intranet.empresa.lab`, el cifrado se establece **antes** de enviar datos. Pero cifrar no basta: ¿cómo sabes que estás hablando **con el servidor real** y no con un impostor? Un **certificado** es un documento digital que dice: *"esta clave pública pertenece a `intranet.empresa.lab`"*, **firmado por alguien en quien confías**.

> **Analogía:** un **DNI**. Es una tarjeta con tus datos, **emitida y firmada por una autoridad** (la policía) en la que todos confían. Si el DNI lo hubieras hecho tú en casa, nadie te creería.

### Cadena de confianza

| Pieza | Qué es |
| ----- | ------ |
| **CA raíz** (*Certificate Authority*) | La autoridad de la que se parte. Su certificado está **autofirmado** y se instala como **de confianza** en los equipos |
| **Certificado de servidor** | Identifica a un servicio; lo **firma la CA** |
| **Clave privada** | Pertenece **a cada entidad** (CA y servidor). **Nunca se comparte** |
| **CSR** (*Certificate Signing Request*) | Una **petición de firma**: el servidor genera su clave, y manda a la CA una petición con su clave **pública** y su nombre |
| **SAN** (*Subject Alternative Name*) | Lista de nombres/IP para los que el certificado es válido. **Los navegadores modernos ignoran el campo CN y exigen SAN** |

Flujo: el servidor crea su clave y un CSR → la CA lo **firma** → el servidor instala el certificado firmado → el cliente, que **confía en la CA**, valida la firma y por tanto confía en el servidor.

### Cosas que caducan

Los certificados tienen **fecha de caducidad**. Un certificado caducado = servicio roto. Los certificados públicos duran menos de un año (y bajando). En una empresa, **vigilar caducidades** es parte del trabajo.

### Herramientas con `openssl`

| Comando | Qué hace |
| ------- | -------- |
| `openssl genrsa -out clave.key 4096` | Genera una **clave privada RSA** (`-out` archivo, `4096` bits) |
| `openssl req -x509 -new -key ca.key -sha256 -days 3650 -out ca.crt -subj "/CN=CA Empresa"` | Crea el **certificado autofirmado** de la CA (`-x509` = certificado en lugar de CSR, `-days` validez, `-subj` datos del sujeto) |
| `openssl req -new -key servidor.key -out servidor.csr -subj "/CN=intranet.empresa.lab"` | Crea un **CSR** |
| `openssl x509 -req -in servidor.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out servidor.crt -days 365 -sha256 -extfile ext.cnf` | **Firma** el CSR con la CA (`-extfile` añade extensiones como el SAN) |
| `openssl x509 -in servidor.crt -text -noout` | **Lee** un certificado en texto |
| `openssl verify -CAfile ca.crt servidor.crt` | Comprueba que un certificado **lo firmó** esa CA |
| `openssl x509 -in servidor.crt -noout -enddate` | Fecha de **caducidad** |
| `openssl s_client -connect IP:443 -servername nombre` | Se **conecta** a un servidor TLS y enseña la cadena |
| `curl --cacert ca.crt https://nombre` | Hace una petición **confiando solo** en esa CA |
| `sudo update-ca-certificates` | En Linux, **añade** tu CA a las de confianza del sistema (tras copiar el `.crt` a `/usr/local/share/ca-certificates/`) |

Archivo de extensiones `ext.cnf` (completa las `???`):

```
authorityKeyIdentifier = keyid,issuer
basicConstraints = CA:FALSE
keyUsage = digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1 = ???
IP.1  = ???
```

> 🚨 **La clave privada de la CA es el tesoro.** Quien la tenga puede emitir certificados "válidos" para **cualquier** nombre. En empresas reales vive en un equipo **aislado** (a menudo apagado y sin red) o en un **HSM**. En el laboratorio, protégela con permisos `600`.

---

## 🛠️ Qué debes hacer

### Parte A · Tu CA

1. Crea una carpeta de trabajo `~/pki` con subcarpeta `ca` y protégela (`chmod 700`).
2. Genera la **clave privada de la CA** (4096 bits) y protege el archivo (`chmod 600`).
3. Crea el **certificado raíz autofirmado** de la CA, válido 10 años, con un nombre claro (`CN=CA Empresa Lab`).
4. Lee el certificado con `openssl x509 -text`: localiza **emisor**, **sujeto** (¿por qué son iguales?), **validez** y `CA:TRUE`.

### Parte B · Certificado para la intranet

5. Genera la **clave privada** del servidor y un **CSR** para `intranet.empresa.lab`.
6. Escribe el archivo de **extensiones** con SAN para el nombre **y** la IP `192.168.56.10`.
7. **Firma** el CSR con tu CA, con validez de **1 año**.
8. Comprueba con `openssl verify` que el certificado es válido para tu CA y mira su SAN.

### Parte C · HTTPS en Nginx

9. Copia el certificado y la clave a `/etc/ssl/` (clave con permisos `600`, propietario `root`) y configura en Nginx un bloque `server` con `listen 443 ssl`, `ssl_certificate` y `ssl_certificate_key` para `intranet.empresa.lab`.
10. Abre el puerto 443 en el firewall **solo para tu red**, valida con `nginx -t` y recarga.
11. Comprueba con `curl -k https://...` (el `-k` **ignora** la verificación, para ver que funciona) y **sin** `-k`: debe **fallar**, porque tu equipo aún no confía en tu CA.

### Parte D · Confiar en la CA

12. En `ubuntu-cliente`, copia **solo el certificado público** de la CA (`ca.crt`; **nunca** la clave) y haz que el sistema **confíe** en él con `update-ca-certificates`.
13. Añade a `/etc/hosts` del cliente: `192.168.56.10 intranet.empresa.lab`.
14. Prueba `curl https://intranet.empresa.lab` **sin `-k`** y comprueba que ahora **funciona**.
15. Con `openssl s_client` muestra la **cadena de certificados** y comprueba `Verify return code: 0 (ok)`.
16. **Provoca fallos** y observa los mensajes: (a) usa `https://192.168.56.10` pero con un certificado cuyo SAN **no incluye la IP**; (b) usa un nombre que no está en el SAN.

### Parte E · Caducidad y renovación

17. Crea un certificado de **1 día de validez** (o ya caducado) y comprueba qué dice el cliente.
18. Escribe un **script de Bash** que compruebe la fecha de caducidad de tus certificados y **avise si faltan menos de 30 días** (usa `openssl x509 -enddate` y `date`).

---

## ✅ Comprobación

| Prueba | Qué debes ver |
| ------ | ------------- |
| `openssl x509 -in ca.crt -noout -subject -issuer` | **Mismo** sujeto y emisor (autofirmado) |
| `openssl verify -CAfile ca.crt servidor.crt` | `servidor.crt: OK` |
| `openssl x509 -in servidor.crt -noout -ext subjectAltName` | El DNS y la IP que pusiste |
| `curl https://intranet.empresa.lab` **antes** de confiar | Error de verificación |
| La misma tras `update-ca-certificates` | Responde con normalidad |
| `openssl s_client -connect ... -servername intranet.empresa.lab` | `Verify return code: 0 (ok)` |
| Tu script de caducidad | Avisa con el certificado de corta validez |

---

## 🧩 Pistas

- Si el cliente sigue rechazando el certificado tras instalar la CA, comprueba **(1)** que el archivo acaba en `.crt` (si no, `update-ca-certificates` lo ignora), **(2)** el SAN, **(3)** que usas **el nombre** que aparece en el SAN.
- `curl -v https://...` te enseña la negociación TLS paso a paso, con el motivo exacto del fallo.
- Si Nginx no arranca tras añadir TLS, `nginx -t` y el `error.log` dicen **qué archivo** no puede leer.
- **Nunca** copies la clave privada de la CA a otro equipo, ni la subas a GitHub.
- Los navegadores y `curl` exigen que el nombre **coincida con un SAN**; el CN solo ya no vale.

---

## 🚀 Retos extra

1. **CA intermedia:** crea una CA raíz y, con ella, una **CA intermedia** que sea la que firma los certificados de servidor. ¿Por qué se hace así? (Pista: poder **guardar la raíz offline**).
2. **Revocación:** crea una **CRL** y revoca un certificado. ¿Cómo se entera el cliente?
3. **Autenticación mutua (mTLS):** emite un **certificado de cliente** y haz que Nginx **solo acepte** a quien lo presente.
4. Instala la CA en **Windows** (`certmgr.msc` → Entidades de certificación raíz de confianza) y prueba el acceso con el navegador. Con **Active Directory** y una **GPO**, ¿cómo lo distribuirías a todos los equipos del dominio a la vez? (Investiga los **Servicios de certificados de Active Directory**, AD CS.)
5. Conecta con laboratorios anteriores: añade **TLS al LDAP** (avanzado, 07) o al **servidor de logs** (experto, 04) con tus certificados.

---

## 📝 Qué documentar en tu `Solucion.md`

- Un diagrama de la **cadena de confianza** de tu laboratorio.
- Los comandos de cada paso, explicados, **sin incluir ninguna clave privada**.
- Con tus palabras: **por qué el navegador avisaba antes y ya no**, y qué tendría que hacer un atacante para suplantar a la intranet.
- Una **política de gestión de certificados** de media página: dónde se guarda la clave de la CA, cuánto duran los certificados, quién los renueva y cómo se vigila la caducidad.

---

## 🧠 Lo que has practicado

`PKI` · `CA y cadena de confianza` · `CSR` · `SAN` · `openssl` · `HTTPS en Nginx` · `confianza en el sistema` · `caducidad y renovación` · `gestión de claves`
