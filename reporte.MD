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
