# Integración de Proxmox VE & Plesk Obsidian

**Santiago Quirós Arenas**

Este repositorio contiene la documentación técnica para integrar **Plesk** dentro de un ecosistema **Proxmox**, optimizando la seguridad, el rendimiento y la gestión de certificados SSL.

---

## Índice
1. [Despliegue de Plesk en Proxmox](#1-despliegue-de-plesk-en-proxmox)
2. [Script de Instalación Automatizada](#2-script-de-instalación-automatizada)
3. [Proxy Inverso (Acceso Seguro a Proxmox)](#3-proxy-inverso-acceso-seguro-a-proxmox)
4. [Integración con Proxmox Mail Gateway (PMG)](#4-integración-con-proxmox-mail-gateway-pmg)
5. [Ventajas de la Infraestructura](#5-ventajas-de-la-infraestructura-2026)

---

## 1. Despliegue de Plesk en Proxmox

Plesk funciona como el panel de gestión de hosting dentro de una instancia virtualizada administrada por Proxmox:

*   **Máquinas Virtuales (VM):** Es el método recomendado para máxima compatibilidad de kernel. Se pueden utilizar imágenes **QCOW2** de Plesk y preconfigurarlas mediante la utilidad `cloud-init` para establecer contraseñas y red rápidamente.
*   **Contenedores LXC:** Permite instalar Plesk en contenedores ligeros de Linux para optimizar el uso de hardware. Se puede usar la consola nativa de Proxmox para ejecutar los comandos de instalación sin necesidad de habilitar SSH inicialmente, aumentando la seguridad.

---

## 2. Script de Instalación Automatizada

Para desplegar Plesk en una instancia (VM o LXC) con Ubuntu o Debian, sigue estos pasos desde la terminal:

### Paso A: Preparación del sistema

    apt update && apt upgrade -y && apt install wget curl -y 

### Paso B: Ejecución del instalador oficial
    
    sh <(curl autoinstall.plesk.com || wget -O - autoinstall.plesk.com)
 

## 3. Proxy Inverso (Acceso Seguro a Proxmox)

Es posible configurar Plesk para que actúe como un proxy inverso hacia la interfaz web de Proxmox (puerto 8006). Esto permite acceder al panel de Proxmox a través de un subdominio gestionado en Plesk (ej. proxmox.tu-dominio.com) y utilizar certificados Let's Encrypt gestionados por Plesk Obsidian.

**Directivas de Nginx en Plesk:**
En la sección "Configuración de Apache y Nginx" del subdominio, añade lo siguiente en el campo de Directivas adicionales de nginx:

    location / {
        proxy_pass https://TU_IP_LOCAL_PROXMOX:8006;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # Soporte crítico para la consola NoVNC de Proxmox
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        
        # Timeouts largos para evitar cortes en la consola
        proxy_connect_timeout 3600s;
        proxy_read_timeout 3600s;
        proxy_send_timeout 3600s;
    }

## 4. Integración con Proxmox Mail Gateway (PMG)

Una integración común es utilizar **Proxmox Mail Gateway** como filtro de correo (spam y virus) antes de que los mensajes lleguen al servidor Plesk:

* **Relay de Correo:** Se configura el dominio en PMG para que actúe como relay hacia la dirección IP del servidor Plesk.
* **Registros MX:** En el panel de DNS de Plesk, se deben ajustar los registros MX para que apunten exclusivamente al nodo de Proxmox Mail Gateway.
* **Eficiencia:** Esto desplaza la carga de procesamiento de correo malicioso fuera del servidor de hosting, mejorando el rendimiento de las webs alojadas.

---

## 5. Ventajas de la Infraestructura

* **Snapshots y Backups:** Proxmox permite realizar copias de seguridad completas y snapshots de la instancia de Plesk sin tiempo de inactividad, facilitando recuperaciones ante fallos críticos mediante **Proxmox Backup Server**.
* **Migración Facilitada:** Si necesitas mover tu servidor Plesk a un nuevo hardware, puedes usar la función de **Migración en caliente** de Proxmox para transferir la VM entre nodos con mínimos clics.
* **Escalabilidad:** Permite aumentar RAM, CPU y almacenamiento en caliente desde la interfaz de Proxmox sin necesidad de reinstalar Plesk.