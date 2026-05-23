Este repositorio contiene la arquitectura técnica, código de integración y scripts de automatización del proyecto **EduCloud**, una plataforma de servicios educativos e infraestructura de red corporativa diseñada para centralizar, securizar y automatizar los recursos informáticos de un centro docente.

El proyecto destaca por la convergencia de entornos **Open Source (Linux)** y **Propietarios (Microsoft Windows Server)**, gobernados bajo una arquitectura de red perimetral estricta y respaldados de forma automática y desacoplada en la nube (**AWS**).

---

## 🛠️ Stack Tecnológico

* **Infraestructura y Red:** OPNsense (Firewall Perimetral), pfSense (Firewall Interno), Enrutamiento DMZ/LAN, NAT/PAT.
* **Sistemas y Directorio:** Windows Server 2019/2022 (Active Directory DS, DNS, DHCP, AD CS).
* **Servidor de Aplicaciones:** Debian GNU/Linux, Apache2, PHP 8.x (Extensiones LDAP y PDO).
* **Persistencia de Datos:** MariaDB / MySQL Server (Modelo Relacional).
* **Seguridad y Criptografía:** Protocolo LDAPS (Puerto 636), Certificados TLS, Expresiones Regulares (Regex) anti-inyección, Chroot Jail (Jaulas SFTP), Bind Mounts, Principio de Mínimo Privilegio.
* **Cloud & DevOps:** Amazon Web Services (AWS S3, IAM Gestión de Identidades), AWS CLI, Programación Bash Scripting, Automatización con CronTab.

---

## 📐 Arquitectura del Sistema

El ecosistema se distribuye en tres zonas lógicas para garantizar el aislamiento de seguridad y la tolerancia a fallos:

1.  **Red Local (LAN):** Segmento protegido (Subred `10.0.135.0/24`) donde se ubican los clientes docentes y alumnos, quienes obtienen direccionamiento dinámico y resolución DNS local a través de la infraestructura corporativa.
2.  **Zona Desmilitarizada (DMZ):** Segmento aislado de alta seguridad (Subred `10.2.167.0/24`) donde residen los servidores expuestos internamente: el Controlador de Dominio de Active Directory (`10.2.167.2`) y la Plataforma Web/SFTP EduCloud (`10.2.167.4`).
3.  **Capa Cloud (AWS):** Repositorio inmutable y aislado para copias de seguridad de datos, código fuente y logs de monitorización.

---

## 🚀 Características Técnicas Destacadas

### 1. Securización de Identidades e Integración LDAPS (Capa de Aplicación)
Se ha desarrollado un portal de autogestión en PHP integrado directamente con Active Directory, permitiendo el inicio de sesión único (SSO) y la modificación de credenciales de dominio desde la Intranet.
* **Canal Seguro:** Se fuerza la conexión mediante **LDAPS por el puerto 636** cifrada con TLS mediante una Autoridad Certificadora (CA) interna gestionada con **AD CS**.
* **Lógica de Doble Bind (Doble Autenticación):** El código PHP realiza una primera comprobación con los datos actuales del usuario. Tras validarse, ejecuta un segundo *bind* empleando una cuenta de servicio delegada (`svc_password`) confinada dentro del grupo **Account Operators** (Principio de Mínimo Privilegio).
* **Codificación Binaria Estricta:** Para cumplir las directivas criptográficas de Microsoft, el script localiza dinámicamente el *Distinguished Name* (DN) del alumno mediante `ldap_search` y transforma la nueva contraseña en un flujo binario **UTF-16 Little Endian** envuelto en comillas dobles, requisito indispensable para el atributo `unicodePwd`.
* **Filtros de Entrada:** Implementación de Expresiones Regulares (**Regex**) para sanitizar los formularios web, bloqueando metacaracteres del estándar LDAP (`*`, `()`, `\`) y neutralizando ataques de inyección.

### 2. Capa de Datos y Persistencia Empresarial (SQL & PDO)
La plataforma ha sido migrada de archivos JSON planos a un motor relacional robusto **MariaDB**, aportando las siguientes ventajas:
* **Integración mediante PDO:** Toda la comunicación entre la web y la base de datos se realiza parametrizando las consultas con PHP Data Objects (PDO), eliminando cualquier vulnerabilidad de Inyección SQL.
* **Integridad y Concurrencia:** Prevención de colisiones y corrupción de datos ante escrituras simultáneas por parte de cientos de alumnos entregando prácticas concurrentemente.

### 3. Fortificación de Entornos Linux (Aislamiento Web-FTP)
Para la entrega de apuntes y descarga de recursos se unificó el servidor Apache con un servicio SFTP seguro:
* **Jaulas Securizadas (Chroot Jail):** Los profesores acceden vía SFTP de manera enjaulada, impidiéndoles recorrer la jerarquía del sistema operativo o comprometer la seguridad del servidor Linux.
* **Espejo de Directorios (Bind Mount):** Enlace virtual a nivel de Kernel entre la ruta de almacenamiento FTP (`/var/ftp/recursos`) y el directorio raíz de Apache (`/var/www/html/recursos`). Permite que los archivos subidos de forma aislada por SFTP se rendericen de inmediato en la Intranet de forma nativa.

### 4. Estrategia de DevOps, Automatización y Cloud (AWS S3)
Aseguramiento de la continuidad de negocio mediante políticas de copias de seguridad automatizadas basadas en el **Desacoplamiento (Decoupling) y Ciclo de Vida del Dato**:
* **Estrategia de Backup Aislado:** Ejecutado diariamente a las 3:00 AM mediante tareas desatendidas en **CronTab**, un script de bash fragmenta el sistema en 3 bloques independientes para optimizar el RTO (Recovery Time Objective):
    1.  Código estático de la aplicación web comprimido en `.tar.gz`.
    2.  Instantánea coherente y limpia de la base de datos relacional mediante `mysqldump` en formato `.sql`.
    3.  Almacenamiento masivo de recursos de profesores recolectado directamente del origen físico del *Bind Mount*.
* **Políticas IAM restrictivas:** El servidor Linux se comunica con la infraestructura Cloud a través de la **API REST de Amazon** (`awscli`). Las claves criptográficas configuradas localmente pertenecen a un usuario de **AWS IAM** limitado mediante políticas JSON estrictas; solo tiene permisos de escritura (`s3:PutObject`) en el bucket, imposibilitando que un atacante que comprometa el servidor local pueda borrar o alterar los respaldos históricos en la nube.
* **Políticas del Bucket:** El bucket en AWS S3 cuenta con la directiva **"Block Public Access"** activada a nivel global, impidiendo filtraciones de datos desde el exterior.

---

## 💻 Código Destacado del Repositorio

### Script de Respaldo Desacoplado Cloud (`backup_aws.sh`)
Ubicado en el Apéndice de Automatización, este script demuestra el flujo completo de empaquetado, volcado SQL y transferencia segura a Amazon S3:

Salida de código
File README-EduCloud.md created successfully.

```bash
#!/bin/bash
# ==============================================================================
# Script de Respaldo Automatizado Cloud - EduCloud Infrastructure
# Autor: Gonzalo López Álvarez
# Política: Desacoplamiento de componentes y Mínimo Privilegio IAM
# ==============================================================================
set -e

FECHA=$(date +%F)
BUCKET="s3://backup-asir-grupo7-2026"

echo "[+] 1. Iniciando respaldo del código fuente (Excluyendo Bind Mount)..."
tar --exclude='/var/www/html/recursos' -cvzf /root/app_educloud_$FECHA.tar.gz /var/www/html/

echo "[+] 2. Ejecutando instantánea coherente de Base de Datos MariaDB (mysqldump)..."
mysqldump -u edu_user -p'EduCloud2026!' educloud > /root/datos_entregas_$FECHA.sql

echo "[+] 3. Respaldando almacenamiento masivo de recursos de profesores (SFTP)..."
tar -cvzf /root/recursos_profesores_$FECHA.tar.gz /var/ftp/recursos/

echo "[+] 4. Transfiriendo bloques desacoplados a la API REST de Amazon S3..."
aws s3 cp /root/app_educloud_$FECHA.tar.gz $BUCKET/backups/aplicacion/
aws s3 cp /root/datos_entregas_$FECHA.sql $BUCKET/backups/base_datos/
aws s3 cp /root/recursos_profesores_$FECHA.tar.gz $BUCKET/backups/recursos_sftp/

echo "[+] 5. Purgando artefactos temporales del almacenamiento local..."
rm /root/app_educloud_$FECHA.tar.gz
rm /root/datos_entregas_$FECHA.sql
rm /root/recursos_profesores_$FECHA.tar.gz

echo "[✓] Proceso de backup Cloud completado con éxito el $FECHA"
