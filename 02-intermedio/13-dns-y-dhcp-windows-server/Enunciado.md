# 📡 Laboratorio 13 · DNS y DHCP en Windows Server

**Nivel:** 🟡 Intermedio · **Sistema:** 🪟 Windows Server · **Tiempo estimado:** 90 minutos

---

## 🎯 Escenario

Hasta ahora has puesto la IP del cliente **a mano**. En una empresa con 50 equipos eso es inviable. Tu misión: que los equipos reciban su configuración de red **automáticamente** (con **DHCP**) y que los servidores de la empresa tengan **nombres propios** (con **DNS**), como `intranet.empresa.local`.

> 💾 Snapshot de `dc01` y del cliente antes de empezar. Necesitas `dc01` con el dominio del laboratorio 11 y el cliente del laboratorio 12.

---

## 📖 Conceptos nuevos

### DHCP: el "recepcionista" de la red

**DHCP** (*Dynamic Host Configuration Protocol*) entrega automáticamente a cada equipo que se conecta: **IP, máscara, puerta de enlace y DNS**. Analogía: el recepcionista de un hotel que, al llegar, te asigna una **habitación** (IP) por un tiempo y te dice dónde está el comedor (DNS).

El proceso, conocido como **DORA**:

| Paso | Mensaje | Qué ocurre |
| ---- | ------- | ---------- |
| 1 | **D**iscover | El cliente grita: *"¿hay algún servidor DHCP?"* |
| 2 | **O**ffer | El servidor ofrece una IP |
| 3 | **R**equest | El cliente la pide formalmente |
| 4 | **A**cknowledge | El servidor la confirma |

### Elementos de un servidor DHCP

| Término | Qué es |
| ------- | ------ |
| **Ámbito** (*scope*) | El rango de IPs que se pueden repartir (`192.168.10.100` a `192.168.10.200`) |
| **Exclusión** | IPs del rango que **no** se reparten |
| **Concesión** (*lease*) | El "alquiler" temporal de una IP; al caducar, se renueva |
| **Reserva** | Asigna **siempre la misma IP** a un equipo concreto (por su MAC) |
| **Opciones** | Datos extra que se entregan: `003` puerta de enlace, `006` servidores DNS, `015` sufijo DNS |
| **Autorización** | En un dominio, el servidor DHCP debe **estar autorizado en AD** para repartir IPs (evita servidores "pirata") |

### DNS: la agenda de la red

Ya lo conoces del nivel 1. En Windows Server, el DNS del dominio guarda **zonas** con **registros**:

| Registro | Qué hace | Ejemplo |
| -------- | -------- | ------- |
| **A** | Nombre → IPv4 | `intranet` → `192.168.10.10` |
| **CNAME** | Un **alias** de otro nombre | `www` → `intranet` |
| **PTR** | **Inverso**: IP → nombre | `192.168.10.10` → `dc01` |

Las zonas pueden ser **directas** (nombre → IP) o **inversas** (IP → nombre). La zona `empresa.local` ya existe desde el laboratorio 11.

### Comandos de este laboratorio

| Comando | Qué hace |
| ------- | -------- |
| `Install-WindowsFeature DHCP -IncludeManagementTools` | Instala el rol DHCP |
| `Add-DhcpServerInDC -DnsName dc01.empresa.local -IPAddress 192.168.10.10` | **Autoriza** el servidor DHCP en Active Directory |
| `Add-DhcpServerv4Scope -Name "LAN" -StartRange 192.168.10.100 -EndRange 192.168.10.200 -SubnetMask 255.255.255.0` | Crea un ámbito |
| `Set-DhcpServerv4OptionValue -DnsServer 192.168.10.10 -DnsDomain empresa.local` | Fija las opciones 006 y 015 |
| `Add-DhcpServerv4Reservation -ScopeId 192.168.10.0 -IPAddress 192.168.10.50 -ClientId MAC -Name nombre` | Crea una reserva |
| `Get-DhcpServerv4Lease -ScopeId 192.168.10.0` | Lista las concesiones activas |
| `Add-DnsServerResourceRecordA -Name intranet -ZoneName empresa.local -IPv4Address 192.168.10.10` | Crea un registro A |
| `Add-DnsServerResourceRecordCName -Name www -HostNameAlias intranet.empresa.local -ZoneName empresa.local` | Crea un alias |
| `Resolve-DnsName intranet.empresa.local` / `nslookup nombre` | Prueba la resolución |
| `ipconfig /release` y `ipconfig /renew` | En el cliente: **suelta** y **vuelve a pedir** la IP |
| `ipconfig /flushdns` | Vacía la **caché DNS** del cliente |
| `ipconfig /all` | Muestra IP, DNS, servidor DHCP y fechas de la concesión |

---

## 🛠️ Qué debes hacer

### Parte A · DHCP

1. En `dc01`, instala el rol **DHCP** y finaliza la configuración posterior (la **autorización** en AD).
2. Crea un **ámbito** con el rango `192.168.10.100-200`, máscara `/24` y una **duración de concesión** de 8 horas.
3. Configura las **opciones**: servidor DNS `192.168.10.10` y sufijo DNS `empresa.local`.
4. Crea una **exclusión** (por ejemplo, `192.168.10.100-110`) y razona para qué sirve.
5. **Activa** el ámbito.
6. En el cliente, cambia la configuración de red a **obtener IP automáticamente** (DHCP).
7. Ejecuta `ipconfig /release` y `ipconfig /renew`, y comprueba con `ipconfig /all`: IP, servidor DHCP y fecha de concesión.
8. En `dc01`, mira la **concesión** en la consola DHCP o con `Get-DhcpServerv4Lease`.
9. Crea una **reserva** para el cliente con la IP `192.168.10.50`. Renueva la IP en el cliente y comprueba que recibe esa.

### Parte B · DNS

10. En el **Administrador de DNS**, crea un registro **A**: `intranet.empresa.local` → `192.168.10.10`.
11. Crea un **alias (CNAME)**: `www.empresa.local` → `intranet.empresa.local`.
12. Desde el **cliente**, comprueba con `nslookup` que resuelve `intranet` y `www`. Si no, `ipconfig /flushdns`.
13. Crea una **zona de búsqueda inversa** para `192.168.10.0/24` y un registro **PTR** para el servidor. Comprueba `nslookup 192.168.10.10`.
14. Observa en los registros del DNS cómo el cliente **se registró solo** con su nombre (los equipos del dominio actualizan su propio registro).

---

## ✅ Comprobación

| Prueba | Qué debes ver |
| ------ | ------------- |
| `Get-DhcpServerv4Scope` | Tu ámbito, estado `Active` |
| `ipconfig /all` (cliente) | `DHCP habilitado: sí`, IP del rango, `Servidor DHCP: 192.168.10.10` |
| `Get-DhcpServerv4Lease -ScopeId 192.168.10.0` | El cliente con su IP y su MAC |
| Tras la reserva | El cliente recibe `192.168.10.50` |
| `nslookup intranet.empresa.local` (cliente) | `192.168.10.10` |
| `nslookup www.empresa.local` | Alias de `intranet`, con su IP |
| `nslookup 192.168.10.10` | Un nombre (por el registro PTR) |

---

## 🧩 Pistas

- Si el cliente no recibe IP, mira dos cosas: que el ámbito esté **activado** y que el servidor DHCP esté **autorizado** (en la consola aparece con una flecha verde).
- El cliente debe estar en la **misma red** (`lan-empresa`): no hay router que reenvíe peticiones DHCP.
- Para la reserva necesitas la **dirección MAC** del cliente: `ipconfig /all` (campo "Dirección física"), con guiones en vez de dos puntos.
- Si un cambio en DNS no se refleja en el cliente, casi siempre es **caché**: `ipconfig /flushdns`.
- El nombre en `Add-DhcpServerv4Reservation` va con la MAC **sin guiones** o con ellos según el cmdlet; si te da error, prueba con el otro formato.

---

## 🚀 Retos extra

1. Añade un **reenviador** (*forwarder*) en el DNS (`8.8.8.8`) para que el servidor resuelva nombres de internet. Necesitará salida a internet (¿qué tendrías que añadir a `dc01`? Acuérdate del reto 3 del laboratorio 11).
2. Captura con `Wireshark` (si lo tienes) el proceso **DORA**: localiza los cuatro paquetes.
3. Activa el **registro de auditoría** de DHCP y busca en él la concesión del cliente.
4. Explica cómo un **agente de retransmisión DHCP** (*relay*) permitiría que **un solo servidor** repartiera IPs a **varias redes**.
5. Investiga **DHCP failover**: ¿qué pasa en tu empresa si `dc01` se apaga y no hay segundo servidor DHCP?

---

## 📝 Qué documentar en tu `Solucion.md`

- Los parámetros de tu ámbito (rango, exclusión, concesión, opciones) y **por qué** los elegiste.
- Un esquema del proceso **DORA** con tus palabras.
- Capturas de `ipconfig /all` antes y después, y de los registros DNS.
- Qué hace cada tipo de registro (A, CNAME, PTR) con un ejemplo de **tu** laboratorio.

---

## 🧠 Lo que has practicado

`DHCP` · `DORA` · `ámbitos, exclusiones, concesiones y reservas` · `opciones DHCP` · `autorización en AD` · `DNS` · `registros A, CNAME y PTR` · `zonas directa e inversa` · `ipconfig /release /renew /flushdns`
