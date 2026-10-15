# 🤖 Laboratorio 09 · Introducción a Ansible

**Nivel:** 🟠 Avanzado · **Sistema:** 🐧 Linux · **Tiempo estimado:** 100 minutos

---

## 🎯 Escenario

Imagina que la empresa compra **20 servidores nuevos** y todos necesitan lo mismo: los mismos usuarios, los mismos paquetes, el mismo firewall, el mismo servidor web. Hacerlo uno a uno, con los comandos que has aprendido, sería lento y, peor aún, **inconsistente**: cada servidor acabaría con pequeñas diferencias.

La solución es la **automatización**: describir en un archivo **cómo debe quedar** un servidor y dejar que una herramienta lo aplique en todos. **Ansible** es la más usada en administración de sistemas.

> 💾 **Snapshot** de las dos VMs. **Nodo de control** (desde donde lanzas Ansible): `ubuntu-server-lab` (`192.168.56.10`). **Nodo gestionado**: `ubuntu-cliente` (`192.168.56.11`). Necesitas la **autenticación SSH por clave** del laboratorio 08 del nivel intermedio, desde el control al gestionado.

---

## 📖 Conceptos nuevos

### Qué es Ansible

**Ansible** es una herramienta de **automatización**. Sus características clave:

- **Sin agente** (*agentless*): no instalas nada en los servidores gestionados; usa **SSH** (que ya tienes) y Python.
- **Declarativa**: no escribes **los pasos**, describes **el estado final** deseado ("el paquete `nginx` debe estar instalado").
- **Idempotente**: puedes ejecutarla **mil veces** y el resultado es el mismo; si ya está como debe, **no toca nada**. Es su rasgo más importante.

> **Analogía:** en vez de dar instrucciones a un albañil ("pon un ladrillo, otro ladrillo..."), le entregas **el plano** de la casa terminada. Si ya está construida a medias, solo hace lo que falta.

### Piezas principales

| Pieza | Qué es |
| ----- | ------ |
| **Nodo de control** | La máquina donde está instalado Ansible y desde la que lanzas las órdenes |
| **Nodos gestionados** | Los servidores que Ansible configura |
| **Inventario** | Un archivo con la **lista de servidores**, agrupados |
| **Módulo** | Una unidad de trabajo (`apt`, `user`, `copy`, `service`...) |
| **Tarea** (*task*) | Una llamada a un módulo con sus parámetros |
| **Playbook** | Un archivo **YAML** con una lista de tareas para un grupo de servidores |
| **Handler** | Una tarea que **solo se ejecuta si otra ha cambiado algo** (por ejemplo, reiniciar un servicio) |
| **Plantilla** (*template*) | Un archivo con **variables** (formato Jinja2) que se rellena para cada servidor |

### Inventario de ejemplo

```ini
[web]
ubuntu-cliente ansible_host=192.168.56.11 ansible_user=TU_USUARIO

[todos:children]
web
```

### Esqueleto de un playbook

Completa las `???`:

```yaml
---
- name: Configuración básica de servidores
  hosts: ???
  become: ???          # ¿necesitan privilegios de administrador?

  tasks:
    - name: Instalar paquetes básicos
      ansible.builtin.apt:
        name: [htop, tree, curl]
        state: ???
        update_cache: true

    - name: Crear el usuario ana
      ansible.builtin.user:
        name: ana
        state: present
        shell: /bin/bash
```

### Comandos de este laboratorio

| Comando | Qué hace |
| ------- | -------- |
| `sudo apt install ansible` | Instala Ansible en el **nodo de control** |
| `ansible all -i inventario.ini -m ping` | Prueba la conexión con **todos** los servidores (`-m` = módulo; `ping` aquí no es el ping de red, comprueba que Ansible puede entrar y ejecutar Python) |
| `ansible web -i inventario.ini -m command -a "uptime"` | Ejecuta **un comando suelto** en el grupo `web` (`-a` = argumentos). Son las llamadas **ad-hoc** |
| `ansible-playbook -i inventario.ini playbook.yml` | **Ejecuta** un playbook |
| `ansible-playbook ... --check --diff` | **Simula**: dice qué **cambiaría** (`--check`) y muestra las **diferencias** (`--diff`) sin tocar nada |
| `ansible-inventory -i inventario.ini --list` | Muestra cómo interpreta tu inventario |
| `ansible-doc ansible.builtin.apt` | **Documentación** de un módulo, offline |
| `ansible-vault create secretos.yml` | Crea un archivo **cifrado** (para contraseñas) |

---

## 🛠️ Qué debes hacer

### Parte A · Preparar

1. En el nodo de control, instala Ansible y crea una carpeta `~/ansible-lab`.
2. Crea el archivo de **inventario** con el nodo gestionado (usa `ansible_host` y `ansible_user`).
3. Comprueba con el módulo `ping` que Ansible **llega** al nodo gestionado sin pedir contraseña. Si no, revisa tu clave SSH.

### Parte B · Órdenes ad-hoc

4. Averigua con un comando ad-hoc el **tiempo encendido** (`uptime`) y el **espacio de disco** del nodo gestionado.
5. Instala `tree` en el nodo gestionado con el módulo `apt` en una **llamada ad-hoc** (necesitarás `--become`).
6. **Repite la misma orden** y observa la diferencia entre la **primera** ejecución (`changed`) y la **segunda** (`ok`). **Esto es la idempotencia.**

### Parte C · Tu primer playbook

7. Escribe un playbook `base.yml` que:
   - instale `htop`, `tree` y `curl`,
   - cree los usuarios `ana` y `luis`,
   - asegure que el servicio `cron` está **iniciado y habilitado**.
8. Ejecútalo con `--check --diff` **primero** y lee qué cambiaría.
9. Ejecútalo de verdad. Comprueba el **resumen final** (`ok`, `changed`, `failed`).
10. **Vuélvelo a ejecutar**: debe indicar `changed=0`. Si no, tu playbook no es idempotente.
11. Comprueba **directamente en el nodo gestionado** que todo está como pedías.

### Parte D · Un servicio completo

12. Escribe un segundo playbook `web.yml` que:
    - instale **Nginx**,
    - despliegue un `index.html` **desde una plantilla** que incluya el **nombre del servidor** (variable `{{ inventory_hostname }}`),
    - arranque y habilite el servicio,
    - use un **handler** para **recargar** Nginx solo cuando cambie la plantilla.
13. Ejecútalo y comprueba la web desde tu navegador (`http://192.168.56.11`; si el firewall bloquea el puerto 80, ¿cómo lo permitirías **también con Ansible**, con el módulo `community.general.ufw` o `ufw`?).
14. **Cambia el texto de la plantilla**, ejecútalo otra vez y comprueba que el handler **recarga** Nginx solo esta vez.

---

## ✅ Comprobación

| Prueba | Qué debes ver |
| ------ | ------------- |
| `ansible all -i inventario.ini -m ping` | `"ping": "pong"` y `SUCCESS` |
| Orden ad-hoc repetida | 1.ª vez `CHANGED`; 2.ª vez `SUCCESS` sin cambios |
| `ansible-playbook base.yml` (1.ª vez) | `changed=N` (varias tareas) |
| `ansible-playbook base.yml` (2.ª vez) | **`changed=0`** |
| En el nodo gestionado: `id ana` y `which htop` | Existen |
| Navegador → `http://192.168.56.11` | La página con el nombre del servidor |
| Tras cambiar la plantilla | El handler se ejecuta y la web cambia |

---

## 🧩 Pistas

- YAML: **espacios, nunca tabuladores**, y la indentación define la estructura (una lista empieza por `- `).
- Si Ansible dice que un módulo no existe, usa su **nombre completo** (`ansible.builtin.apt`) y consulta `ansible-doc`.
- Para saber qué parámetros admite un módulo: `ansible-doc nombre-del-módulo` (y baja hasta los **ejemplos**, que suelen ser muy útiles).
- Si el playbook falla en una tarea, el mensaje te indica **qué tarea** y **qué host**. Léelo entero.
- `become: true` equivale a "usa `sudo`". Si olvidas `become` en una tarea de instalación, fallará por permisos.
- Si te da error de "host key checking", conecta una vez a mano por SSH al nodo para aceptar su huella.

---

## 🚀 Retos extra

1. Convierte tu playbook en un **rol** (`ansible-galaxy init`) con su estructura de carpetas. ¿Qué ventaja da reutilizar roles?
2. Usa **variables** (`vars:` o un archivo `group_vars`) para no repetir listas de paquetes y usuarios.
3. Guarda la **contraseña** de un usuario con **`ansible-vault`**, nunca en claro dentro del playbook.
4. Aplica tus **medidas de hardening** del laboratorio 05 con Ansible (SSH, `sysctl`, `unattended-upgrades`). ¿Qué ganas al tenerlo como código?
5. Sube tu playbook a **Git** y razona por qué *"infraestructura como código"* mejora la **trazabilidad** y los **cambios** en equipo.

---

## 📝 Qué documentar en tu `Solucion.md`

- Tus archivos `inventario.ini`, `base.yml` y `web.yml`, comentados.
- Las salidas de la **primera** y la **segunda** ejecución, y qué te enseña la diferencia.
- Con tus palabras: **idempotencia**, y por qué es clave en automatización.
- Cuándo prefieres Ansible a un **script de Bash** y cuándo no.

---

## 🧠 Lo que has practicado

`automatización` · `Ansible` · `inventario` · `módulos` · `playbooks YAML` · `idempotencia` · `handlers y plantillas` · `--check --diff` · `infraestructura como código`
