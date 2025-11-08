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
