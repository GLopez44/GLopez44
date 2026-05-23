# Bot de Scraping Automatizado con API de Telegram

## Descripción del Proyecto
Este repositorio documenta la arquitectura lógica y el flujo de trabajo de un script de automatización desarrollado en Python. El sistema estaba diseñado para extraer información de páginas web (Web Scraping) de forma desatendida y enviar alertas en tiempo real mediante la API oficial de Telegram.

---

## Tecnologías y Librerías Utilizadas
- **Lenguaje Base:** Python 3.
- **Sistema Operativo:** Linux (Ejecución en segundo plano).
- **Automatización:** Crontab (Programador de tareas de Linux).
- **Librerías de Scraping:** `requests` (para peticiones HTTP) y `BeautifulSoup` (para el parseo de HTML).
- **Integración:** API REST de Telegram (`pyTelegramBotAPI` / `telebot`).

---

## Flujo Lógico de la Automatización

El sistema funcionaba siguiendo el siguiente ciclo de vida automatizado:

1. **Ejecución Programada (Cronjob):**
   El sistema Linux utilizaba el demonio `cron` para ejecutar el script de Python en intervalos regulares de tiempo, sin intervención humana.

2. **Extracción de Datos (Scraping):**
   El script realizaba una petición HTTP GET a la página web objetivo. Mediante el uso de librerías de parseo, se analizaba la estructura del DOM (HTML) para aislar y extraer únicamente los datos de interés (noticias, precios, avisos).

3. **Procesamiento y Limpieza:**
   Los datos extraídos se limpiaban de etiquetas HTML y se formateaban en cadenas de texto legibles, preparándolas para su envío.

4. **Notificación Push (API Telegram):**
   A través de un Bot de Telegram previamente creado con *BotFather*, el script realizaba una petición POST a la API de Telegram incluyendo el `chat_id` destino y el mensaje de texto, entregando la alerta directamente en el dispositivo móvil.

---

## Aprendizajes Clave
Este proyecto fue fundamental para comprender la interacción entre aplicaciones de terceros mediante APIs REST, la manipulación de datos estructurados en Python y la importancia de la automatización de tareas en sistemas operativos Linux mediante `cron`.
