# 🧾 Reporte de Vulnerabilidades
---

## Vulnerabilidad 1 — [Critical] Contraseñas almacenadas en texto plano

**Descripción del Riesgo**
La tabla `users` almacena contraseñas sin hashing, permitiendo su recuperación directa en caso de filtración de la base de datos o acceso no autorizado.

**Impacto**
Compromiso inmediato de todas las cuentas y posible escalada de privilegios. Violación de normativas de privacidad (por ejemplo, GDPR, PCI-DSS).

**Remediación Técnica**

* Implementar hashing robusto y salado (Argon2 o bcrypt) usando funciones nativas del framework (por ejemplo `Hash::make` en Laravel).
* Forzar cambio de contraseña en el próximo inicio de sesión y/o enviar proceso de restablecimiento seguro para todas las cuentas.
* Eliminar cualquier almacenamiento de contraseñas en sesiones o respuestas API.
* Asegurar tránsito con TLS/HTTPS.
* Auditar repositorios y backups para evitar contraseñas en claro.

**Aspecto ISO/IEC 27001 (2022)**

| Control relevante                     | Recomendación para el tratamiento                                                                           |
| ------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| A.5.17 (Información de autenticación) | Implementar hashing unidireccional y salado (Argon2/bcrypt). Políticas de complejidad, longitud y rotación. |
| A.8.24 (Uso de criptografía)          | Asegurar almacenamiento criptográfico aprobado y protección en tránsito (TLS/HTTPS).                        |

---

## Vulnerabilidad 2 — [Critical] Rutas API sin autenticación ni autorización

**Descripción del Riesgo**
Múltiples rutas API críticas (ej. `/api/auth/admin-login` y módulos contables) carecen de middleware de autenticación/autorización. Se detectó un token de bypass estático (`admin_bypass_token_123`).

**Impacto**
Acceso total a datos y funciones administrativas por actores no autenticados; compromiso completo del sistema.

**Remediación Técnica**

* Aplicar middleware de autenticación (p. ej. Sanctum/Passport en Laravel).
* Implementar autorización por roles y políticas (RBAC) y principio de mínimo privilegio.
* Eliminar cualquier token o mecanismo de bypass estático y revisar el historial de commits.
* Revisar y proteger rutas en `backend/routes/web.php` (ej. líneas indicadas: `L49-55`).
* Añadir logging y alertas a accesos administrativos.

**Aspecto ISO/IEC 27001 (2022)**

| Control relevante                                       | Recomendación para el tratamiento                                                       |
| ------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| A.5.15 (Control de acceso) / A.8.2 (Derechos de acceso) | Implementar autenticación obligatoria para endpoints sensibles; RBAC.                   |
| A.8.5 (Autenticación segura)                            | Usar mecanismos de autenticación de alta entropía; prohibir tokens estáticos de bypass. |

---

## Vulnerabilidad 3 — [Critical] SQL Injection en múltiples endpoints

**Descripción del Riesgo**
Se usan consultas SQL mediante interpolación directa de strings (ej. `username = '$username' AND password = '$password'`), permitiendo inyección SQL.

**Impacto**
Bypass de autenticación, extracción o manipulación masiva de datos (finanzas, usuarios), y posible ejecución de comandos a través de vectores encadenados.

**Remediación Técnica**

* Reescribir consultas usando Eloquent ORM o Query Builder con parámetros preparados (prepared statements).
* Añadir validación y saneamiento de entrada en todos los endpoints (whitelisting cuando sea posible).
* Implementar pruebas de seguridad automatizadas (SAST/DAST) que detecten inyecciones.
* Revisar logs y auditorías por indicios de explotación.

**Aspecto ISO/IEC 27001 (2022)**

| Control relevante                                      | Recomendación para el tratamiento                                       |
| ------------------------------------------------------ | ----------------------------------------------------------------------- |
| A.8.28 (Seguridad del desarrollo, prueba y aceptación) | Aplicar principios de desarrollo seguro; usar consultas parametrizadas. |
| A.8.25 (Validación de entrada de aplicación)           | Implementar validación, saneamiento y desinfección de inputs.           |

---

## Vulnerabilidad 4 — [Critical] Ejecución Remota de Código (RCE) vía `eval()`

**Descripción del Riesgo**
El endpoint `/api/debug/php-execute` ejecuta directamente código PHP (`eval($code);`) sin autenticación ni validación.

**Impacto**
Ejecución arbitraria de código PHP con privilegios del servidor web. Compromiso total del servidor y riesgo de pivot a otros sistemas.

**Remediación Técnica**

* **Eliminar completamente** el endpoint y cualquier uso de `eval()` en producción.
* Mantener funcionalidades de depuración únicamente en entornos de desarrollo/QA con controles estrictos.
* Revisar el repositorio para remover código de depuración antes del despliegue.
* Auditar permisos y asegurar separación entre entornos dev/prod.

**Aspecto ISO/IEC 27001 (2022)**

| Control relevante                   | Recomendación para el tratamiento                                                    |
| ----------------------------------- | ------------------------------------------------------------------------------------ |
| A.8.9 (Gestión de la configuración) | Prohibir funciones inseguras (eval) en producción; mantener configuraciones seguras. |
| A.8.21 (Separación de entornos)     | Asegurar que código de depuración no sea accesible en producción.                    |

---

## Vulnerabilidad 5 — [Critical] Ejecución Remota de Comandos (RCE) vía `shell_exec()`

**Descripción del Riesgo**
Endpoints (`/api/debug/execute-command`, `/api/admin/execute`) ejecutan comandos del sistema vía `shell_exec($command . ' 2>&1')` sin sanitización.

**Impacto**
Ejecución arbitraria de comandos del SO, exposición de credenciales, acceso a archivos sensibles y compromiso del servidor.

**Remediación Técnica**

* **Eliminar** estos endpoints en producción.
* Si la ejecución de comandos es imprescindible para operaciones administrativas, reemplazar por APIs internas seguras con whitelist de comandos y autenticación mutua, además de ejecución en contenedores/entornos con privilegios mínimos.
* Auditar permisos del usuario de la aplicación (p. ej. `www-data`) y reducir al mínimo necesario.
* Implementar monitoreo y alertas para ejecuciones de comandos inusuales.

**Aspecto ISO/IEC 27001 (2022)**

| Control relevante                                      | Recomendación para el tratamiento                                                 |
| ------------------------------------------------------ | --------------------------------------------------------------------------------- |
| A.8.28 (Seguridad del desarrollo, prueba y aceptación) | Evitar funciones que ejecuten shells; usar APIs seguras y con menor privilegio.   |
| A.8.4 (Sistema operativo de seguridad)                 | Ejecutar servidor web con mínimos privilegios; limitar capacidad de `shell_exec`. |

---

## Vulnerabilidad 6: [Critical] Exposición masiva de credenciales y secretos

**Descripción del Riesgo:**  
El sistema expone directamente variables de entorno (`/env`), incluyendo credenciales de base de datos, API keys y contraseñas, lo que permite a cualquier usuario no autenticado acceder a información sensible.

**Impacto:**  
Accesos no autorizados a datos sensibles y violación de la confidencialidad.

**Remediación Técnica:**  
Eliminar los endpoints que devuelvan información del entorno. Implementar un servicio de gestión de secretos (por ejemplo, HashiCorp Vault o AWS Secret Manager) y aplicar sanitización de respuestas. Asegurar que los logs no incluyan credenciales y establecer controles de acceso estrictos en la configuración del servidor.

**Aspecto de la ISO/IEC 27001 (2022)**  
| Control Relevante | Recomendación ISO 27001 para el Tratamiento |
|--------------------|---------------------------------------------|
| A.8.28 (Seguridad del desarrollo, prueba y aceptación) | Eliminar todos los endpoints de información del entorno (/env). Usar un gestor de secretos dedicado para almacenar credenciales sensibles en producción. |
| A.8.25 (Prueba de seguridad en el desarrollo y aceptación) | Implementar controles de respuesta de la API que garanticen que ninguna variable de entorno o secreto sensible se devuelva, incluso en caso de error o depuración. |

---

## Vulnerabilidad 7: [Critical] Admin Bypass Token Hardcodeado

**Descripción del Riesgo:**  
Se encontró un token de administración (`Admin_bypass_token_123`) incrustado en el código, lo que permite el acceso administrativo sin autenticación.

**Impacto:**  
Acceso administrativo completo y escalamiento de privilegios.

**Remediación Técnica:**  
Eliminar el token hardcodeado y reemplazarlo por un sistema de autenticación basado en JWT o OAuth2 con firma criptográfica. Implementar autorización por roles (RBAC) y auditar los accesos administrativos.

**Aspecto de la ISO/IEC 27001 (2022)**  
| Control Relevante | Recomendación ISO 27001 para el Tratamiento |
|--------------------|---------------------------------------------|
| A.5.17 (Información de autenticación) | Implementar un sistema de autenticación dinámico (como JWT o tokens de sesión) en lugar de tokens estáticos. Las credenciales deben ser gestionadas criptográficamente y no hardcodeadas. |
| A.8.28 (Seguridad del desarrollo, prueba y aceptación) | Eliminar el secreto hardcodeado. Establecer revisiones de código regulares (peer review) para evitar la inclusión accidental de credenciales o secretos en el código fuente. |

---

## Vulnerabilidad 8: [Critical] Ejecución SQL directa sin validación

**Descripción del Riesgo:**  
Los endpoints permiten la ejecución directa de consultas SQL sin validación ni autenticación, posibilitando la manipulación total de la base de datos.

**Impacto:**  
Manipulación o eliminación total de la base de datos por atacantes.

**Remediación Técnica:**  
Eliminar endpoints de ejecución libre. Utilizar ORM seguro (Eloquent o Query Builder con parámetros preparados) y validar exhaustivamente la entrada de usuario. Implementar un control de acceso granular a la base de datos.

**Aspecto de la ISO/IEC 27001 (2022)**  
| Control Relevante | Recomendación ISO 27001 para el Tratamiento |
|--------------------|---------------------------------------------|
| A.8.25 (Prueba de seguridad en el desarrollo y aceptación) | Migrar a consultas preparadas o utilizar un ORM/Query Builder seguro para separar los datos de los comandos SQL. |
| A.8.27 (Prevención de fuga de información) | Implementar el principio de mínimo privilegio para el usuario de la base de datos de la aplicación. Auditar y bloquear el uso de comandos administrativos (como DROP TABLE) desde la aplicación. |

---

## Vulnerabilidad 9: [Critical] Eliminación masiva de datos sin confirmación

**Descripción del Riesgo:**  
Existe un endpoint que permite borrar toda la base de datos sin autenticación ni doble confirmación.

**Impacto:**  
Pérdida total de datos y denegación del servicio (DoS).

**Remediación Técnica:**  
Implementar un proceso de confirmación en múltiples pasos, autenticación obligatoria y autorización por rol para acciones destructivas. Usar **soft delete** en lugar de eliminación física.

**Aspecto de la ISO/IEC 27001 (2022)**  
| Control Relevante | Recomendación ISO 27001 para el Tratamiento |
|--------------------|---------------------------------------------|
| A.5.3 (Separación de deberes) | Las acciones destructivas críticas deben requerir autorización por rol, doble confirmación (por ejemplo, contraseña o MFA), o un control dual de aprobación. |
| A.8.12 (Copias de seguridad de la información) | Implementar copias de seguridad automáticas antes de operaciones críticas y establecer procedimientos de recuperación de datos probados y validados. |

---

## Vulnerabilidad 10: [Critical] Exposición del APP_KEY de Laravel

**Descripción del Riesgo:**  
El valor de `APP_KEY`, utilizado para la encriptación y firma de datos en Laravel, se encuentra expuesto en los endpoints `/debug/info` y `/env`.

**Impacto:**  
Riesgo de descifrado de datos, falsificación de tokens y sesiones.

**Remediación Técnica:**  
Eliminar los endpoints de depuración en entornos de producción. Rotar inmediatamente la clave `APP_KEY` y regenerar tokens o sesiones dependientes.

**Aspecto de la ISO/IEC 27001 (2022)**  
| Control Relevante | Recomendación ISO 27001 para el Tratamiento |
|--------------------|---------------------------------------------|
| A.8.11 (Gestión de claves criptográficas) | Rotar inmediatamente la clave expuesta. Implementar un proceso formal para la generación, almacenamiento y uso seguro de claves criptográficas. |
| A.8.17 (Configuración segura) | Asegurar que los entornos de desarrollo y producción estén completamente separados y que los endpoints de depuración sean inaccesibles en producción. |

## Vulnerabilidad 11: [Critical] Exposición de variables de entorno completas

* **Descripción del Riesgo:** Los endpoints /env y /api/debug/dump-state exponen todas las variables de entorno, incluyendo credenciales, API keys y secretos.
* **Impacto:** Exposición completa de la configuración, lo que lleva al compromiso de servicios externos, bases de datos y APIs externas, y una violación masiva de la seguridad.
* **Remediación Técnica:** ELIMINAR COMPLETAMENTE estos endpoints en entornos de producción. Nunca exponer variables de entorno. Usar servicios de gestión de secretos.

| Aspecto de la ISO/IEC 27001 (2022) | Control Relevante | Recomendación ISO 27001 para el Tratamiento |
| :--- | :--- | :--- |
| Configuración segura y prevención de exposición de información. | Α.8.17 (Configuración segura) | Eliminar todos los endpoints que exponen variables de entorno en producción. Usar un gestor de secretos (como Azure Key Vault o AWS Secrets Manager) en lugar de archivos .env directamente. |
| Protección de la información sensible y las credenciales. | A.5.17 (Información de autenticación) y A.8.24 (Uso de criptografía) | Asegurar que las credenciales y las claves criptográficas no se revelen a través de interfaces de depuración o diagnóstico en entornos accesibles al público. |

---

## Vulnerabilidad 12: [Critical] Path Traversal en lectura de archivos

* **Descripción del Riesgo:** El endpoint /api/debug/read-file y /api/admin/logs permiten la lectura de archivos arbitrarios mediante path traversal (../../) sin validación.
* **Impacto:** Acceso no autorizado a archivos sensibles del sistema (configuración, código fuente, /etc/passwd), lo que compromete la confidencialidad.
* **Remediación Técnica:** Validar y sanitizar rutas de archivos. Implementar whitelist de archivos permitidos. Restringir acceso a directorios específicos.

| Aspecto de la ISO/IEC 27001 (2022) | Control Relevante | Recomendación ISO 27001 para el Tratamiento |
| :--- | :--- | :--- |
| Control de acceso a las utilidades del sistema. | A.8.2 (Derechos de acceso) | Validar y sanitizar rigurosamente toda entrada de usuario destinada a rutas de archivos para prevenir secuencias de path traversal (../). |
| Protección contra fuga de información y control de la aplicación. | A.8.25 (Validación de entrada de aplicación) | Implementar una lista blanca (whitelist) de archivos y directorios específicos que la aplicación está autorizada a leer, rechazando cualquier solicitud fuera de esa lista. |

---

## Vulnerabilidad 13: [Critical] Inyección de comandos en backup

* **Descripción del Riesgo:** El endpoint de backup construye comandos shell con interpolación directa de variables, permitiendo la inyección de comandos.
* **Impacto:** Ejecución de comandos arbitrarios, compromiso del servidor y modificación/corrupción de los backups.
* **Remediación Técnica:** Usar funciones seguras de Laravel para backups. Validar y sanitizar todas las variables. Usar prepared commands o parámetros nombrados.

| Aspecto de la ISO/IEC 27001 (2022) | Control Relevante | Recomendación ISO 27001 para el Tratamiento |
| :--- | :--- | :--- |
| Prevención de inyección de código y comandos. | A.8.28 (Seguridad del desarrollo, prueba y aceptación) | Reemplazar la interpolación directa de strings con mecanismos que traten los datos como datos, no como código (por ejemplo, librerías de comandos seguros o parámetros de ejecución preparados). |
| Validación de la entrada y seguridad de la aplicación. | A.8.25 (Validación de entrada de aplicación) | Implementar validación estricta y desinfección de las variables de usuario antes de usarlas en comandos del sistema operativo (CWE-78: OS Command Injection). |

---

## Vulnerabilidad 14: [High] IDOR (Insecure Direct Object References)

* **Descripción del Riesgo:** Los endpoints (backend/app/Http/Controllers/Accounting Controller.php L206-247) permiten acceso a recursos de otros usuarios mediante manipulación de IDs ({id}) sin verificación de propiedad.
* **Impacto:** Acceso no autorizado a datos de otros usuarios, modificación de registros financieros y eliminación de datos críticos (violación de la Confidencialidad e Integridad).
* **Remediación Técnica:** Verificar propiedad del recurso antes de cualquier operación (acceder/modificar/eliminar). Implementar políticas de autorización (Laravel Policies).

| Aspecto de la ISO/IEC 27001 (2022) | Control Relevante | Recomendación ISO 27001 para el Tratamiento |
| :--- | :--- | :--- |
| Control de acceso y políticas de autorización. | A.8.2 (Derechos de acceso) | Para cada solicitud de acceso a un recurso de usuario (por ejemplo, un registro contable), la aplicación debe verificar la relación de propiedad entre el usuario autenticado y el ID del recurso solicitado. |
| Seguridad de la información y minimización de privilegios. | A.5.15 (Control de acceso) | Implementar un control de autorización robusto (por ejemplo, Policies de Laravel) que fuerce la verificación de pertenencia a nivel de código antes de ejecutar cualquier lógica de negocio. |

---

## Vulnerabilidad 15: [High] Cross-Site Scripting (XSS) Reflejado y Almacenado

* **Descripción del Riesgo:** La aplicación no sanitiza outputs de usuario, permitiendo XSS reflejado en búsquedas (backend/app/Http/Controllers/AccountingController.php L100-111) y almacenado en formularios (L371-418).
* **Impacto:** Robo de cookies de sesión, secuestro de cuentas, ejecución de código JavaScript malicioso y phishing.
* **Remediación Técnica:** Implementar *output encoding* en todos los endpoints. Usar *Content Security Policy (CSP)*. Implementar sanitización en *frontend* y *backend*.

| Aspecto de la ISO/IEC 27001 (2022) | Control Relevante | Recomendación ISO 27001 para el Tratamiento |
| :--- | :--- | :--- |
| Prevención de vulnerabilidades de inyección. | A.8.28 (Seguridad del desarrollo, prueba y aceptación) | Aplicar codificación de salida (*output encoding*) por defecto en todas las plantillas y vistas para neutralizar cualquier dato de usuario antes de que se represente en el navegador. |
| Configuración segura del entorno de la aplicación. | A.8.17 (Configuración segura) | Implementar una Política de Seguridad de Contenido (CSP) estricta para mitigar el impacto del XSS, limitando las fuentes desde donde se pueden cargar scripts. |











# Reporte Detallado de Vulnerabilidades (26 a 30)

## Vulnerabilidad 26: [High] Falta de validación de entrada

* **Descripción del Riesgo:** Múltiples controladores (AuthController, AccountingController, AdminController, DebugController) carecen de validación de input, lo que permite el ingreso de datos maliciosos.
* **Impacto:** Posibilidad de inyección SQL, XSS, manipulación de datos y otros ataques derivados de entradas no sanitizadas.
* **Remediación Técnica:** Implementar *Form Requests* de Laravel con validación estricta. Utilizar una lista blanca (*whitelist*) para aceptar únicamente datos válidos y esperados.

| Aspecto de la ISO/IEC 27001 (2022) | Control Relevante | Recomendación ISO 27001 para el Tratamiento |
| :--- | :--- | :--- |
| Validación de entrada y control de acceso a nivel de aplicación. | A.8.5 (Control de acceso a nivel de aplicación) | Aplicar validación de datos de entrada en todos los endpoints. Implementar *Form Requests* para asegurar que solo los datos esperados sean procesados. |

---

## Vulnerabilidad 27: [High] Exposición de queries SQL en respuestas

* **Descripción del Riesgo:** Múltiples controladores exponen información sensible, como `'sql_query' => $query`, en las respuestas de error.
* **Impacto:** Permite a un atacante conocer la estructura interna de la base de datos, facilitando la explotación de inyecciones o manipulación de consultas.
* **Remediación Técnica:** Eliminar toda exposición de consultas SQL en mensajes de error. Implementar mensajes genéricos y seguros en entornos de producción.

| Aspecto de la ISO/IEC 27001 (2022) | Control Relevante | Recomendación ISO 27001 para el Tratamiento |
| :--- | :--- | :--- |
| Pruebas de seguridad en desarrollo y aceptación. | A.8.20 (Pruebas de seguridad en el desarrollo y aceptación) | Asegurar que los mensajes de error no revelen información sensible ni estructuras internas de la aplicación o base de datos. |

---

## Vulnerabilidad 28: [High] Exposición de logs con información sensible

* **Descripción del Riesgo:** En `AuthController.php (L19-25)` se registran contraseñas (`password`) en los logs del sistema.
* **Impacto:** Divulgación no autorizada de credenciales, tokens y otra información sensible, comprometiendo la confidencialidad.
* **Remediación Técnica:** No registrar información sensible como contraseñas o tokens. Implementar rotación, control de acceso y encriptación de logs.

| Aspecto de la ISO/IEC 27001 (2022) | Control Relevante | Recomendación ISO 27001 para el Tratamiento |
| :--- | :--- | :--- |
| Almacenamiento seguro de secretos. | A.8.21 (Almacenamiento seguro de secretos) | Configurar políticas de logging seguras que impidan el registro de información sensible. Aplicar encriptación y rotación de logs. |

---

## Vulnerabilidad 29: [Medium] Falta de rate limiting

* **Descripción del Riesgo:** En `backend/routes/api.php (L24)` existe un comentario explícito: `"without rate limiting"`, lo que permite intentos ilimitados de autenticación o peticiones abusivas.
* **Impacto:** Permite ataques de fuerza bruta y denegación de servicio (DoS) por falta de control en la frecuencia de solicitudes.
* **Remediación Técnica:** Implementar *middleware* de Rate Limiting en todos los endpoints de autenticación y funcionalidades críticas.

| Aspecto de la ISO/IEC 27001 (2022) | Control Relevante | Recomendación ISO 27001 para el Tratamiento |
| :--- | :--- | :--- |
| Protección ante ataques de fuerza bruta y abuso de servicio. | A.7.4 (Sistema de gestión de contraseñas) | Aplicar limitación de intentos de acceso y políticas de bloqueo temporal tras múltiples intentos fallidos. |

---

## Vulnerabilidad 30: [Medium] Información del sistema expuesta

* **Descripción del Riesgo:** Los endpoints `/phpinfo`, `/env`, `/api/debug/info`, `/api/admin/system-info` exponen información detallada del sistema, configuración del servidor y versiones.
* **Impacto:** Facilita a un atacante el reconocimiento del entorno y la identificación de vulnerabilidades específicas del sistema.
* **Remediación Técnica:** Eliminar todos los endpoints de información y depuración en producción. Restringir su acceso solo a administradores autenticados si es estrictamente necesario.

| Aspecto de la ISO/IEC 27001 (2022) | Control Relevante | Recomendación ISO 27001 para el Tratamiento |
| :--- | :--- | :--- |
| Instalación, gestión y registro seguro de software en sistemas operativos. | A.8.18 (Instalación, gestión y registro de software en sistemas operativos) | Deshabilitar o eliminar funciones de diagnóstico en producción. Asegurar que la información del sistema solo sea accesible mediante autenticación y privilegios adecuados. |

---



Informe de Soluciones a Vulnerabilidades 31-35 

Basado en los lineamientos de la ISO/IEC 27001:2022, OWASP Top 10 y prácticas de Aseguramiento de la Calidad del Software (SQA).

Vulnerabilidad 31: Componentes con vulnerabilidades conocidas

ISO 27001 referencia: A.8.8 – Gestión de vulnerabilidades técnicas, A.9.4 – Seguridad de los sistemas de desarrollo y soporte.
Solución técnica:
1. Actualizar dependencias a versiones soportadas: Laravel 10 LTS, Vue 3.x, Axios 1.x.
2. Automatizar escaneos con Dependabot, Snyk o GitHub Security Alerts.
3. Establecer política de revisión trimestral de librerías y pruebas de regresión en QA.
Control SQA: Verificar compatibilidad mediante pruebas unitarias y de integración.

Vulnerabilidad 32: Falta de headers de seguridad HTTP

ISO 27001 referencia: A.8.24 – Protección de la información en tránsito, A.8.23 – Seguridad de la información en los servicios de red.
Solución técnica:
1. Implementar middleware de headers de seguridad (CSP, X-Frame-Options, HSTS, etc.).
2. Configurar las cabeceras también en el servidor web (Nginx o Apache).
3. Validar con pruebas o herramientas como securityheaders.com.
Control SQA: Agregar pruebas de cabeceras HTTP en el checklist de QA.

Vulnerabilidad 33: CSRF no protegido

ISO 27001 referencia: A.8.28 – Seguridad de los servicios web y API, A.8.25 – Control de acceso a la información.
Solución técnica:
1. Aplicar middleware CSRF en rutas web y usar meta tags en formularios.
2. En APIs REST usar JWTs con expiración corta y validar cabeceras de origen.
3. Usar SameSite cookies para prevenir ataques.
Control SQA: Verificar que todas las peticiones POST, PUT y DELETE incluyan tokens válidos.

Vulnerabilidad 34: Catch-all route con información de debug

ISO 27001 referencia: A.8.12 – Registro y monitoreo de actividades, A.8.16 – Manejo de errores y excepciones.
Solución técnica:
1. Retornar mensajes 404 genéricos sin exponer información interna.
2. Registrar intentos en logs internos sin mostrar al usuario.
3. Desactivar APP_DEBUG en producción y proteger el archivo .env.
Control SQA: Validar que respuestas 404 no incluyan estructuras internas.

Vulnerabilidad 35: Mapeo inseguro de módulos a tablas

ISO 27001 referencia: A.8.29 – Validación de entrada de datos, A.8.26 – Separación de entornos de desarrollo y producción.
Solución técnica:
1. Implementar lista blanca de módulos válidos.
2. Evitar concatenación dinámica en consultas SQL.
3. Validar entradas y sanitizar datos.
4. Realizar pruebas de inyección SQL en QA.
Control SQA: Verificar que los parámetros alterados no accedan a tablas no definidas.
Plan general de mejora continua
1. Establecer un proceso de revisión de código seguro (DevSecOps).
2. Implementar un sistema de gestión de vulnerabilidades (SGV).
3. Mantener repositorio con control de cambios documentado.
4. Realizar pruebas de penetración y auditorías semestrales.
5. Alinear el plan de calidad del software con controles de ISO 27001.









## Vulnerabilidad 36: [Medium] Exposición de `phpinfo()`

### 📝 Resolución de la Vulnerabilidad (ISO 27001/27002)

Esta vulnerabilidad se relaciona directamente con el control de la **Configuración Segura** y la **Gestión de Vulnerabilidades**.

#### 1. Control Clave de la ISO/IEC 27002

* **A.8.20 (2013) / 5.14 (2022) - Configuración Segura:**
    * **Objetivo:** Asegurar que los sistemas de información, incluyendo el software de aplicación, estén configurados de manera segura.
    * **Aplicación:** El uso de `phpinfo()` en producción es una **configuración insegura**.

#### 2. Pasos de Remediación Basados en la ISO 27001/27002

1.  **Eliminar o Desactivar la Exposición (Medida de Control):**
    * **Acción:** Eliminar completamente el endpoint `/phpinfo` del archivo `backend/routes/web.php` en los entornos de Staging y Producción.
    * **Justificación:** Cumple con la directriz de **Configuración Segura**.
2.  **Restricción de Acceso (Principio de Mínimo Privilegio):**
    * **Acción:** Si fuera estrictamente necesario, restringir el acceso mediante:
        * **Autenticación y Autorización:** Solo para usuarios con roles de administración de sistemas.
        * **Restricción de Acceso IP:** Permitir el acceso únicamente desde IPs internas o de VPNs de administración específicas.
    * **Justificación:** A.9.2.3 / 5.15 (**Gestión de Derechos de Acceso**) y A.12.1.2 / 8.21 (**Seguridad de los Servicios de Red**).
3.  **Asegurar el Proceso de Desarrollo (Gestión del Cambio):**
    * **Acción:**
        * **Revisión de Código (Peer Review):** Proceso obligatorio para verificar que no se incluyan funciones de diagnóstico o depuración en ramas de Staging o Producción.
        * **Pruebas de Seguridad (Testing):** Incluir la búsqueda de endpoints de depuración en los escaneos de vulnerabilidades.
    * **Justificación:** A.14.2.8 / 8.29 (**Desarrollo de Política de Desarrollo Seguro**) y A.14.2.2 / 8.27 (**Procedimiento de Control de Cambios**).

| Referencia | Concepto de Seguridad | Relación con ISO 27001/27002 |
| :--- | :--- | :--- |
| **OWASP A05:2021** – Security Misconfiguration | Resultado directo de una mala configuración de seguridad. | Cubierto por el control **A.8.20 / 5.14** (Configuración Segura). |
| **CWE-200** – Information Exposure | Exposición de información sensible (configuración de servidor, rutas). | Cubierto por controles de Acceso (**A.9**) y Gestión de Incidentes (**A.16**). |

---

## Vulnerabilidad 37: [Medium] Falta de validación de tipo de archivo

### 📝 Resolución de la Vulnerabilidad (ISO 27001/27002)

Esta vulnerabilidad afecta directamente a la **Seguridad de Aplicaciones** y al control de **Integridad**.

#### 1. Control Clave de la ISO/IEC 27002

* **A.14.2.5 (2013) / 8.24 (2022) - Prueba de seguridad de la información durante el desarrollo:**
    * **Objetivo:** Asegurar que las vulnerabilidades se identifiquen y corrijan antes de la puesta en producción.
* **A.14.2.8 (2013) / 8.29 (2022) - Desarrollo de la Política de Desarrollo Seguro:**
    * **Objetivo:** Establecer reglas y estándares para el desarrollo de código seguro.
    * **Aplicación:** Prohibir la confianza en las propiedades de archivos proporcionadas por el cliente (extensión o MIME type).

#### 2. Pasos de Remediación Basados en la ISO 27001/27002

1.  **Implementación de Controles Técnicos (Configuración Segura):**
    * **Validación de Contenido Real (Magic Bytes):** Inspeccionar los primeros bytes del archivo (firmas) para determinar su tipo real, independientemente de la extensión.
    * **Lista Blanca (Whitelist):** Solo permitir los tipos de archivo explícitamente requeridos (ej. `.pdf`, `.jpg`).
    * **Cambio de Nombre y Ubicación:** Almacenar los archivos con nombre aleatorio en un **directorio no ejecutable** por el servidor web.
2.  **Protección contra Código Malicioso (Gestión de Amenazas):**
    * **Escaneo Antivirus:** Integrar un servicio de escaneo antivirus en el proceso de subida para bloquear scripts maliciosos.
    * **Justificación:** **A.12.2.1 / 8.2** (**Controles contra malware**).
3.  **Formalización del Proceso (Pruebas de Seguridad):**
    * **Revisión de Código:** Hacer obligatorio que las funciones de subida de archivos sean revisadas específicamente para validar el *magic byte*.
    * **Pruebas de Penetración:** Incluir pruebas de subida de archivos maliciosos en los planes de prueba de seguridad recurrentes.

| Referencia | Concepto de Seguridad | Relación con ISO 27001/27002 |
| :--- | :--- | :--- |
| **OWASP A03:2021** – Injection | Permite la inyección de código ejecutable (webshell). | Cubierto por **A.14.2.5 / 8.24** (Prueba de Seguridad) y **A.14.2.8 / 8.29** (Desarrollo Seguro). |
| **CWE-434** – Unrestricted Upload of File with Dangerous Type | Describe la falla al confiar en datos de entrada del cliente. | Cubierto por **A.14.2.8 / 8.29** (Desarrollo Seguro), exigiendo validación del lado del servidor. |

---

## Vulnerabilidad 38: [Medium] Credenciales de base de datos en `docker-compose.yml`

### 📝 Resolución de la Vulnerabilidad (ISO 27001/27002)

Esta vulnerabilidad se relaciona con la **Gestión de la Configuración**, el **Control de Acceso** y la **Seguridad del Desarrollo**.

#### 1. Controles Clave de la ISO/IEC 27002

* **A.9.4.1 (2013) / 5.10 (2022) - Control de la Información de Autenticación:**
    * **Objetivo:** Asegurar la protección de la información de autenticación (contraseñas).
    * **Aplicación:** Almacenar contraseñas en texto plano en un repositorio viola este control.
* **A.14.2.8 (2013) / 8.29 (2022) - Desarrollo de Política de Desarrollo Seguro:**
    * **Objetivo:** Prohibir las prácticas inseguras como el *hardcoding* de credenciales.

#### 2. Pasos de Remediación Basados en la ISO 27001/27002

1.  **Implementación de Gestión de Secretos (Medida de Control):**
    * **Acción:** Migrar las credenciales a un sistema de gestión de secretos adecuado.
    * **Docker:** Utilizar **Docker Secrets** o un sistema externo (ej. HashiCorp Vault) que inyecte las credenciales en tiempo de ejecución.
    * **Transición:** El archivo que contenga secretos (ej. `.env`) debe estar listado en el `.gitignore`.
    * **Justificación:** **A.9.4.1 / 5.10** (Proteger Autenticación) y **A.10.1.2 / 8.6** (**Gestión de Claves**).
2.  **Rotación Inmediata de Credenciales (Gestión de Incidentes):**
    * **Acción:** Rotar inmediatamente las credenciales expuestas (`vulnerable_password_123` y `vuln_pass_123`) en la base de datos real.
    * **Justificación:** **A.16.1.7 / 6.8** (Recopilación de Evidencia y Toma de Decisiones de Respuesta) exige una acción correctiva inmediata.
3.  **Auditoría de Seguridad en Desarrollo (Proceso de SGSI):**
    * **Acción:**
        * **Hooks de Pre-Commit/CI/CD:** Implementar herramientas automatizadas de **escaneo de secretos** (linters) en el pipeline para detener *commits* con credenciales *hardcodeadas*.
        * **Capacitación:** Educar a los desarrolladores sobre el manejo adecuado de secretos.

| Referencia | Concepto de Seguridad | Relación con ISO 27001/27002 |
| :--- | :--- | :--- |
| **OWASP A05:2021** – Security Misconfiguration | El archivo de configuración incluye datos sensibles. | Cubierto por **A.8.20 / 5.14** (Configuración Segura). |
| **CWE-798** – Use of Hard-coded Credentials | Práctica insegura de incrustar credenciales en el código/configuración. | Cubierto por **A.14.2.8 / 8.29** (Desarrollo Seguro) y **A.9.4.1 / 5.10** (Control de Autenticación). |

---

## Vulnerabilidad 39: [Medium] Redis sin autenticación

### 📝 Resolución de la Vulnerabilidad (ISO 27001/27002)

Esta vulnerabilidad incumple los principios de **Control de Acceso** y **Configuración Segura**.

#### 1. Controles Clave de la ISO/IEC 27002

* **A.9.2.1 (2013) / 5.15 (2022) - Gestión de Derechos de Acceso:**
    * **Objetivo:** Restringir el acceso a la información y a las funciones de procesamiento a usuarios autorizados.
    * **Aplicación:** Un servicio sin contraseña viola el principio de necesidad de saber.
* **A.12.1.2 (2013) / 8.21 (2022) - Seguridad de los Servicios de Red:**
    * **Objetivo:** Proteger los servicios de red contra el acceso no autorizado.
    * **Aplicación:** Dejar un servicio de red como Redis sin autenticación es una falla de seguridad de red.

#### 2. Pasos de Remediación Basados en la ISO 27001/27002

1.  **Implementación de Autenticación (Control de Acceso):**
    * **Acción:** Configurar una **contraseña fuerte** para Redis utilizando la directiva `requirepass`.
    * **Integración Docker:** Inyectar esta contraseña como un **secreto de Docker** o variable de entorno segura.
    * **Justificación:** Cumple con **A.9.2.1 / 5.15** (Gestión de Derechos de Acceso).
2.  **Restricción de Acceso a la Red (Seguridad de Red):**
    * **Acción:** Asegurar que Redis solo sea accesible desde la **red interna de Docker** o desde la **dirección IP del servidor de aplicaciones** (loopback o subred privada). **Nunca** debe ser expuesto al host o al público.
    * **Justificación:** **A.12.1.2 / 8.21** (Seguridad de los Servicios de Red) exige controles de acceso a la red (firewalls).
3.  **Uso de Cifrado (Protección de Comunicaciones):**
    * **Acción:** Si el servicio fuese accesible a través de una red no confiable, se debe implementar **TLS/SSL (cifrado)** para proteger el tráfico de datos.
    * **Justificación:** **A.10.1.1 / 5.23** (Cifrado).

| Referencia | Concepto de Seguridad | Relación con ISO 27001/27002 |
| :--- | :--- | :--- |
| **OWASP A05:2021** – Security Misconfiguration | Configuración predeterminada insegura del servicio. | Cubierto por **A.8.20 / 5.14** (Configuración Segura), que requiere la eliminación de configuraciones por defecto inseguras. |
| **CWE-306** – Missing Authentication for Critical Function | Ausencia de un mecanismo de autenticación para acceso a datos críticos (cache/sesiones). | Cubierto por **A.9.2.1 / 5.15** (Gestión de Derechos de Acceso). |

---

## Vulnerabilidad 40: [Medium] Falta de logging de eventos de seguridad

### 📝 Resolución de la Vulnerabilidad (ISO 27001/27002)

Esta vulnerabilidad viola los requisitos de **Monitoreo**, **Revisión** y **Gestión de Incidentes**.

#### 1. Control Clave de la ISO/IEC 27002

* **A.12.4.1 (2013) / 8.16 (2022) - Registro de Eventos (Logging):**
    * **Objetivo:** Asegurar que las actividades de los usuarios, excepciones y **eventos de seguridad** sean registrados para ayudar en la detección y el análisis forense.
    * **Aplicación:** La ausencia de registro de eventos críticos (inicios de sesión, acciones administrativas) es una violación directa de este control.

#### 2. Pasos de Remediación Basados en la ISO 27001/27002

1.  **Definición y Captura de Eventos Críticos (A.12.4.1 / 8.16):**
    * **Acción:** Implementar un sistema de logging **estructurado y detallado** para registrar, como mínimo:
        * Eventos de **Autenticación** (exitosos y fallidos).
        * Cambios de **Privilegios** (asignación/revocación de roles).
        * Actividades **Administrativas**.
        * Acceso a **Datos Sensibles**.
    * **Justificación:** El registro debe ser útil para futuras investigaciones forenses.
2.  **Gestión, Almacenamiento y Protección de Logs (A.12.4.2 / 8.17):**
    * **Acción:**
        * **Centralización:** Enviar los logs a un sistema centralizado **SIEM** (Security Event and Information Management).
        * **Protección de Integridad:** Proteger los logs contra manipulación (servidor externo de solo anexión).
        * **Retención:** Establecer una política de retención clara.
    * **Justificación:** **A.12.4.2 / 8.17** (**Protección de la Información de Registro**) exige que la evidencia esté protegida contra la manipulación.
3.  **Monitoreo y Respuesta a Incidentes (A.16.1.7 / 6.8):**
    * **Acción:** Implementar **alertas automáticas** en el sistema SIEM (ej. alerta por múltiples inicios de sesión fallidos).
    * **Justificación:** **A.16.1.7 / 6.8** (Recopilación de Evidencia y Toma de Decisiones de Respuesta) requiere la capacidad de detectar incidentes, lo cual depende del logging efectivo.

| Referencia | Concepto de Seguridad | Relación con ISO 27001/27002 |
| :--- | :--- | :--- |
| **OWASP A09:2021** – Security Logging and Monitoring Failures | La falla en el logging previene la detección y respuesta a incidentes. | Cubierto por **A.12.4.1 / 8.16** (Registro de Eventos) y **A.16.1.7 / 6.8** (Respuesta a Incidentes). |
| **CWE-778** – Insufficient Logging | La aplicación no proporciona el nivel de detalle necesario para la investigación forense. | Cubierto por **A.12.4.1 / 8.16**, que exige registrar información suficiente. |











# Reporte Detallado de Vulnerabilidades (41 a 45)

## 41. Vulnerabilidad 41: [Low] Exposición de rutas de archivos en errores

* **Descripción del Riesgo:** Los mensajes de error revelan rutas internas del sistema, exponiendo la estructura del servidor y detalles del entorno.
* **Impacto:** Permite a un atacante obtener información sobre la estructura de directorios, posibles rutas sensibles y configuraciones internas del sistema.
* **Remediación Técnica:** Configurar manejo seguro de errores. Mostrar mensajes genéricos al usuario y registrar detalles técnicos únicamente en logs protegidos de acceso restringido.

| Aspecto de la ISO/IEC 27001 (2022) | Control Relevante | Recomendación ISO 27001 para el Tratamiento |
| :--- | :--- | :--- |
| Manejo seguro de errores y excepciones. | A.8.12 (Registro de eventos) | Implementar un sistema de manejo de errores centralizado que capture las excepciones sin exponer información sensible al usuario final. |
| Protección de información sensible en logs. | A.8.16 (Monitoreo de actividades) | Asegurar que los logs que contengan información técnica o rutas del sistema estén protegidos mediante controles de acceso. |
| Seguridad del desarrollo y pruebas. | A.14.2.5 (Seguridad en los procesos de desarrollo) | Evitar el despliegue de configuraciones de depuración o mensajes detallados en entornos de producción. |

---

## 42. Vulnerabilidad 42: [Low] Falta de timeout en sesiones

* **Descripción del Riesgo:** Las sesiones de usuario no expiran automáticamente después de un periodo de inactividad.
* **Impacto:** Aumenta el riesgo de secuestro de sesión y acceso no autorizado a cuentas activas en dispositivos desatendidos.
* **Remediación Técnica:** Configurar un tiempo de expiración automático para las sesiones. Regenerar periódicamente el ID de sesión e invalidar las inactivas.

| Aspecto de la ISO/IEC 27001 (2022) | Control Relevante | Recomendación ISO 27001 para el Tratamiento |
| :--- | :--- | :--- |
| Gestión de sesiones seguras. | A.5.17 (Gestión de identidad y autenticación) | Establecer políticas de expiración de sesión adecuadas y regeneración de tokens tras autenticación o cambios críticos. |
| Protección contra accesos no autorizados. | A.5.16 (Control de acceso a la información) | Forzar la invalidación de sesiones inactivas y aplicar logout automático tras un periodo definido. |
| Seguridad del entorno de aplicación. | A.8.23 (Gestión de sesiones en aplicaciones web) | Implementar un control de sesión robusto que impida el uso prolongado de tokens o cookies sin renovación. |

---

## 43. Vulnerabilidad 43: [Low] Permisos solo en cliente

* **Descripción del Riesgo:** Las validaciones de permisos se realizan únicamente en el lado del cliente (frontend), lo que permite manipulación y acceso no autorizado a funciones o datos.
* **Impacto:** Permite a un atacante eludir restricciones y acceder a funcionalidades o información restringida mediante la manipulación del cliente.
* **Remediación Técnica:** Implementar validaciones de permisos y roles en el servidor. No confiar en las validaciones del lado del cliente.

| Aspecto de la ISO/IEC 27001 (2022) | Control Relevante | Recomendación ISO 27001 para el Tratamiento |
| :--- | :--- | :--- |
| Control de acceso y autorización. | A.5.16 (Control de acceso) | Asegurar que toda verificación de permisos se ejecute en el servidor antes de permitir operaciones críticas. |
| Seguridad del desarrollo de software. | A.8.28 (Seguridad del desarrollo, prueba y aceptación) | Implementar políticas de autorización en el backend (por ejemplo, middleware o policies) y no depender de la lógica del cliente. |
| Validación de seguridad en procesos de desarrollo. | A.14.2.1 (Principios de seguridad en el desarrollo) | Aplicar controles de acceso en capas múltiples y auditar periódicamente la lógica de autorización. |

---

## 44. Vulnerabilidad 44: [Low] Falta de validación de contenido XML (XXE)

* **Descripción del Riesgo:** Los parsers XML permiten entidades externas (XXE), lo que podría provocar lectura de archivos locales o ataques SSRF.
* **Impacto:** Permite exposición de archivos del sistema, denegación de servicio o acceso no autorizado a recursos internos.
* **Remediación Técnica:** Deshabilitar la carga de entidades externas en parsers XML. Usar parsers seguros y validar/sanitizar toda entrada XML.

| Aspecto de la ISO/IEC 27001 (2022) | Control Relevante | Recomendación ISO 27001 para el Tratamiento |
| :--- | :--- | :--- |
| Validación de entrada y manejo seguro de datos. | A.8.25 (Validación de entrada de aplicación) | Validar y sanitizar todos los datos XML antes de su procesamiento. Deshabilitar DTD y entidades externas. |
| Protección contra ataques de inyección. | A.8.9 (Protección de datos durante el procesamiento) | Configurar los parsers XML para impedir la resolución de recursos externos o internos. |
| Seguridad en el ciclo de desarrollo. | A.14.2.5 (Seguridad en los procesos de desarrollo) | Utilizar librerías actualizadas y configuraciones seguras en todos los parsers XML empleados. |

---

## 45. Vulnerabilidad 45: [Low] Race Conditions y fallos de lógica de negocio

* **Descripción del Riesgo:** Las operaciones financieras no son atómicas, lo que permite condiciones de carrera en transacciones concurrentes.
* **Impacto:** Puede provocar inconsistencias en balances, duplicación de transacciones o errores en la lógica de negocio.
* **Remediación Técnica:** Implementar transacciones ACID, validar balances antes de ejecutar operaciones y aplicar bloqueos (locks) en operaciones críticas.

| Aspecto de la ISO/IEC 27001 (2022) | Control Relevante | Recomendación ISO 27001 para el Tratamiento |
| :--- | :--- | :--- |
| Integridad de los procesos y datos. | A.8.24 (Integridad de la información) | Asegurar la ejecución atómica de operaciones críticas mediante mecanismos de bloqueo y control de concurrencia. |
| Protección de los sistemas de información. | A.14.1.2 (Seguridad en los procesos operativos) | Implementar validaciones previas y posteriores en operaciones financieras o transaccionales. |
| Control de cambios y consistencia. | A.12.1.2 (Gestión de cambios en sistemas) | Aplicar mecanismos de auditoría y monitoreo que detecten condiciones de carrera o inconsistencias. |

---
