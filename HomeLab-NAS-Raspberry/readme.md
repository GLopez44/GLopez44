# HomeLab: Servidor NAS sobre Raspberry Pi

## Descripción del Proyecto
Este proyecto documenta la configuración y despliegue de un servidor NAS (Network Attached Storage) casero utilizando hardware de bajo coste (Raspberry Pi) y sistemas operativos basados en Linux. 

El objetivo principal fue crear una solución de almacenamiento en red centralizada para copias de seguridad, gestión de archivos multimedia y acceso local seguro, simulando un entorno de almacenamiento corporativo a pequeña escala.

---

## Hardware y Tecnologías Utilizadas
- **Hardware:** Raspberry Pi (Arquitectura ARM), unidades de almacenamiento externo.
- **Sistema Operativo:** Distribución basada en Linux (Raspberry Pi OS / Entorno NAS).
- **Protocolos de Red:** TCP/IP, DHCP (IP Estática local).
- **Servicios de Compartición:** SMB/CIFS (Samba), NFS, FTP.
- **Gestión de Discos:** Particionado en Linux (ext4/Btrfs), montaje de volúmenes mediante `fstab`.

---

## Fases de Implementación y Arquitectura

1. **Preparación del Sistema Base:**
   Instalación del sistema operativo Linux sin entorno gráfico (Headless) para optimizar el consumo de RAM y CPU. Configuración de acceso remoto seguro mediante SSH.

2. **Configuración de Red:**
   Asignación de direccionamiento IP estático a la interfaz de red de la Raspberry Pi en el router local para garantizar la localización permanente del servidor NAS.

3. **Gestión de Almacenamiento:**
   Formateo de las unidades externas conectadas por USB y configuración del montaje automático al arrancar el sistema mediante la edición del archivo `/etc/fstab`.

4. **Despliegue de Servicios y Permisos:**
   Instalación de los servicios de compartición de archivos (Samba). Creación de usuarios locales en Linux, asignación de contraseñas de red y configuración de permisos estrictos de lectura/escritura (CHMOD/CHOWN) por carpetas.

---

## Aprendizajes Clave
La realización de este proyecto me permitió consolidar mis conocimientos en administración pura de sistemas Linux, gestión de permisos de usuarios y despliegue de servicios de red en entornos sin interfaz gráfica.
