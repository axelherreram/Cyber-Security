# Reporte Detallado de Vulnerabilidades (11 a 15)

## 11. Vulnerabilidad 11: [Critical] Exposición de variables de entorno completas

* **Descripción del Riesgo:** Los endpoints /env y /api/debug/dump-state exponen todas las variables de entorno, incluyendo credenciales, API keys y secretos.
* **Impacto:** Exposición completa de la configuración, lo que lleva al compromiso de servicios externos, bases de datos y APIs externas, y una violación masiva de la seguridad.
* **Remediación Técnica:** ELIMINAR COMPLETAMENTE estos endpoints en entornos de producción. Nunca exponer variables de entorno. Usar servicios de gestión de secretos.

| Aspecto de la ISO/IEC 27001 (2022) | Control Relevante | Recomendación ISO 27001 para el Tratamiento |
| :--- | :--- | :--- |
| Configuración segura y prevención de exposición de información. | Α.8.17 (Configuración segura) | Eliminar todos los endpoints que exponen variables de entorno en producción. Usar un gestor de secretos (como Azure Key Vault o AWS Secrets Manager) en lugar de archivos .env directamente. |
| Protección de la información sensible y las credenciales. | A.5.17 (Información de autenticación) y A.8.24 (Uso de criptografía) | Asegurar que las credenciales y las claves criptográficas no se revelen a través de interfaces de depuración o diagnóstico en entornos accesibles al público. |

---

## 12. Vulnerabilidad 12: [Critical] Path Traversal en lectura de archivos

* **Descripción del Riesgo:** El endpoint /api/debug/read-file y /api/admin/logs permiten la lectura de archivos arbitrarios mediante path traversal (../../) sin validación.
* **Impacto:** Acceso no autorizado a archivos sensibles del sistema (configuración, código fuente, /etc/passwd), lo que compromete la confidencialidad.
* **Remediación Técnica:** Validar y sanitizar rutas de archivos. Implementar whitelist de archivos permitidos. Restringir acceso a directorios específicos.

| Aspecto de la ISO/IEC 27001 (2022) | Control Relevante | Recomendación ISO 27001 para el Tratamiento |
| :--- | :--- | :--- |
| Control de acceso a las utilidades del sistema. | A.8.2 (Derechos de acceso) | Validar y sanitizar rigurosamente toda entrada de usuario destinada a rutas de archivos para prevenir secuencias de path traversal (../). |
| Protección contra fuga de información y control de la aplicación. | A.8.25 (Validación de entrada de aplicación) | Implementar una lista blanca (whitelist) de archivos y directorios específicos que la aplicación está autorizada a leer, rechazando cualquier solicitud fuera de esa lista. |

---

## 13. Vulnerabilidad 13: [Critical] Inyección de comandos en backup

* **Descripción del Riesgo:** El endpoint de backup construye comandos shell con interpolación directa de variables, permitiendo la inyección de comandos.
* **Impacto:** Ejecución de comandos arbitrarios, compromiso del servidor y modificación/corrupción de los backups.
* **Remediación Técnica:** Usar funciones seguras de Laravel para backups. Validar y sanitizar todas las variables. Usar prepared commands o parámetros nombrados.

| Aspecto de la ISO/IEC 27001 (2022) | Control Relevante | Recomendación ISO 27001 para el Tratamiento |
| :--- | :--- | :--- |
| Prevención de inyección de código y comandos. | A.8.28 (Seguridad del desarrollo, prueba y aceptación) | Reemplazar la interpolación directa de strings con mecanismos que traten los datos como datos, no como código (por ejemplo, librerías de comandos seguros o parámetros de ejecución preparados). |
| Validación de la entrada y seguridad de la aplicación. | A.8.25 (Validación de entrada de aplicación) | Implementar validación estricta y desinfección de las variables de usuario antes de usarlas en comandos del sistema operativo (CWE-78: OS Command Injection). |

---

## 14. Vulnerabilidad 14: [High] IDOR (Insecure Direct Object References)

* **Descripción del Riesgo:** Los endpoints (backend/app/Http/Controllers/Accounting Controller.php L206-247) permiten acceso a recursos de otros usuarios mediante manipulación de IDs ({id}) sin verificación de propiedad.
* **Impacto:** Acceso no autorizado a datos de otros usuarios, modificación de registros financieros y eliminación de datos críticos (violación de la Confidencialidad e Integridad).
* **Remediación Técnica:** Verificar propiedad del recurso antes de cualquier operación (acceder/modificar/eliminar). Implementar políticas de autorización (Laravel Policies).

| Aspecto de la ISO/IEC 27001 (2022) | Control Relevante | Recomendación ISO 27001 para el Tratamiento |
| :--- | :--- | :--- |
| Control de acceso y políticas de autorización. | A.8.2 (Derechos de acceso) | Para cada solicitud de acceso a un recurso de usuario (por ejemplo, un registro contable), la aplicación debe verificar la relación de propiedad entre el usuario autenticado y el ID del recurso solicitado. |
| Seguridad de la información y minimización de privilegios. | A.5.15 (Control de acceso) | Implementar un control de autorización robusto (por ejemplo, Policies de Laravel) que fuerce la verificación de pertenencia a nivel de código antes de ejecutar cualquier lógica de negocio. |

---

## 15. Vulnerabilidad 15: [High] Cross-Site Scripting (XSS) Reflejado y Almacenado

* **Descripción del Riesgo:** La aplicación no sanitiza outputs de usuario, permitiendo XSS reflejado en búsquedas (backend/app/Http/Controllers/AccountingController.php L100-111) y almacenado en formularios (L371-418).
* **Impacto:** Robo de cookies de sesión, secuestro de cuentas, ejecución de código JavaScript malicioso y phishing.
* **Remediación Técnica:** Implementar *output encoding* en todos los endpoints. Usar *Content Security Policy (CSP)*. Implementar sanitización en *frontend* y *backend*.

| Aspecto de la ISO/IEC 27001 (2022) | Control Relevante | Recomendación ISO 27001 para el Tratamiento |
| :--- | :--- | :--- |
| Prevención de vulnerabilidades de inyección. | A.8.28 (Seguridad del desarrollo, prueba y aceptación) | Aplicar codificación de salida (*output encoding*) por defecto en todas las plantillas y vistas para neutralizar cualquier dato de usuario antes de que se represente en el navegador. |
| Configuración segura del entorno de la aplicación. | A.8.17 (Configuración segura) | Implementar una Política de Seguridad de Contenido (CSP) estricta para mitigar el impacto del XSS, limitando las fuentes desde donde se pueden cargar scripts. |
