# SMS Processing & Certified PDF Service

Este sistema es una solución robusta para la recepción, reenvío y certificación legal de mensajes SMS. Utiliza una arquitectura basada en eventos (Event-Driven Architecture) con **FastAPI** y **RabbitMQ** para asegurar que el procesamiento sea asíncrono, escalable y tolerante a fallos.

## 🚀 Flujo del Sistema

1.  **API (FastAPI):** Recibe un SMS de un proveedor externo, lo registra en la base de datos (SQLite/PostgreSQL) y lo encola.
2.  **Worker Resend:** Toma el mensaje, lo reenvía a un endpoint externo de entrega y, si tiene éxito, marca el estado como `delivered`.
3.  **Worker PDF:** Genera un certificado PDF con los detalles de la entrega, le aplica una **firma digital PAdES-B-LT** y un **sello de tiempo (TSA)** para validez legal.
4.  **Worker Distribución:** Envía el certificado firmado por correo electrónico (SMTP) y lo sube a un servidor remoto (FTP o SFTP).

## 🛠️ Tecnologías utilizadas

*   **Backend:** FastAPI (Python 3.9+)
*   **Base de Datos:** SQLAlchemy ORM (compatible con SQLite, PostgreSQL, etc.)
*   **Cola de Mensajes:** RabbitMQ (Pika)
*   **Generación de PDF:** ReportLab
*   **Firma Digital:** PyHanko (Sello de tiempo e integridad PAdES)
*   **Logs:** Módulo `logging` personalizado para trazabilidad completa.

## 📋 Requisitos Previos

*   Python 3.9 o superior.
*   Instancia de **RabbitMQ** activa (Local o CloudAMQP).
*   Un certificado digital en formato `.p12` o `.pfx` para el sellado de PDFs.
*   Servidor SMTP (Gmail, SendGrid, etc.) y/o servidor FTP/SFTP.

## 🔧 Instalación

1.  **Clonar el repositorio:**
    ```bash
    git clone <url-del-repositorio>
    cd sms-certification-service
    ```

2.  **Crear y activar un entorno virtual:**
    ```bash
    python -m venv venv
    source venv/bin/activate  # En Windows: venv\Scripts\activate
    ```

3.  **Instalar dependencias:**
    ```bash
    pip install fastapi uvicorn sqlalchemy pika requests reportlab pyhanko python-dotenv pysftp
    ```

4.  **Configurar variables de entorno:**
    Crea un archivo `.env` en la raíz del proyecto basándote en lo siguiente:
    ```env
    RABBITMQ_HOST=localhost
    SMTP_HOST=smtp.gmail.com
    SMTP_PORT=587
    SMTP_USER=tu_correo@gmail.com
    SMTP_PASS=tu_contraseña_de_aplicacion
    SMTP_SENDER=no-reply@empresa.com
    REMOTE_STORAGE_TYPE=SFTP
    REMOTE_HOST=ftp.empresa.com
    REMOTE_USER=usuario_ftp
    REMOTE_PASS=pass_ftp
    ```

## 🚀 Ejecución

Para que el sistema funcione, deben estar corriendo simultáneamente la API y los tres workers.

### 1. Iniciar la API
```bash
uvicorn api:app --reload --port 8000
```

### 2. Iniciar Worker de Reenvío (Resend)
```bash
python worker_resend.py
```

### 3. Iniciar Worker de Certificación (PDF + Firma)
*Asegúrate de tener el archivo `empresa.p12` en la raíz.*
```bash
python worker_pdf.py
```

### 4. Iniciar Worker de Distribución (Email + FTP)
```bash
python worker_distribucion.py
```

## 📂 Estructura del Proyecto

*   `api.py`: Puntos de entrada HTTP (Endpoints).
*   `database.py` / `models.py`: Configuración de la DB y esquemas de tablas.
*   `productorRabbitmq.py`: Lógica para enviar mensajes a las colas.
*   `worker_resend.py`: Lógica de integración con proveedores de SMS externos.
*   `worker_pdf.py`: Motor de generación de documentos y firma criptográfica.
*   `worker_distribucion.py`: Manejo de envíos SMTP y protocolos de archivos.
*   `setupLog.py`: Configuración centralizada de logs detallados.

## 📝 Notas de Implementación

*   **Firma Digital:** El sistema está configurado para buscar un certificado `empresa.p12`. Puedes cambiar la ruta y contraseña en `worker_pdf.py`.
*   **Seguridad:** En producción, asegúrate de cambiar `SQLALCHEMY_DATABASE_URL` a una base de datos robusta como PostgreSQL.
*   **Escalabilidad:** Puedes levantar múltiples instancias de los workers en diferentes contenedores para procesar grandes volúmenes de SMS en paralelo.

---
*Desarrollado para la gestión de comunicaciones certificadas.*