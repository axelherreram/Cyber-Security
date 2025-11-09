# Reporte Detallado de Vulnerabilidades (26 a 30)

## 26. Vulnerabilidad 26: [High] Falta de validación de entrada

* **Descripción del Riesgo:** Múltiples controladores (AuthController, AccountingController, AdminController, DebugController) carecen de validación de input, lo que permite el ingreso de datos maliciosos.
* **Impacto:** Posibilidad de inyección SQL, XSS, manipulación de datos y otros ataques derivados de entradas no sanitizadas.
* **Remediación Técnica:** Implementar *Form Requests* de Laravel con validación estricta. Utilizar una lista blanca (*whitelist*) para aceptar únicamente datos válidos y esperados.

| Aspecto de la ISO/IEC 27001 (2022) | Control Relevante | Recomendación ISO 27001 para el Tratamiento |
| :--- | :--- | :--- |
| Validación de entrada y control de acceso a nivel de aplicación. | A.8.5 (Control de acceso a nivel de aplicación) | Aplicar validación de datos de entrada en todos los endpoints. Implementar *Form Requests* para asegurar que solo los datos esperados sean procesados. |

---

## 27. Vulnerabilidad 27: [High] Exposición de queries SQL en respuestas

* **Descripción del Riesgo:** Múltiples controladores exponen información sensible, como `'sql_query' => $query`, en las respuestas de error.
* **Impacto:** Permite a un atacante conocer la estructura interna de la base de datos, facilitando la explotación de inyecciones o manipulación de consultas.
* **Remediación Técnica:** Eliminar toda exposición de consultas SQL en mensajes de error. Implementar mensajes genéricos y seguros en entornos de producción.

| Aspecto de la ISO/IEC 27001 (2022) | Control Relevante | Recomendación ISO 27001 para el Tratamiento |
| :--- | :--- | :--- |
| Pruebas de seguridad en desarrollo y aceptación. | A.8.20 (Pruebas de seguridad en el desarrollo y aceptación) | Asegurar que los mensajes de error no revelen información sensible ni estructuras internas de la aplicación o base de datos. |

---

## 28. Vulnerabilidad 28: [High] Exposición de logs con información sensible

* **Descripción del Riesgo:** En `AuthController.php (L19-25)` se registran contraseñas (`password`) en los logs del sistema.
* **Impacto:** Divulgación no autorizada de credenciales, tokens y otra información sensible, comprometiendo la confidencialidad.
* **Remediación Técnica:** No registrar información sensible como contraseñas o tokens. Implementar rotación, control de acceso y encriptación de logs.

| Aspecto de la ISO/IEC 27001 (2022) | Control Relevante | Recomendación ISO 27001 para el Tratamiento |
| :--- | :--- | :--- |
| Almacenamiento seguro de secretos. | A.8.21 (Almacenamiento seguro de secretos) | Configurar políticas de logging seguras que impidan el registro de información sensible. Aplicar encriptación y rotación de logs. |

---

## 29. Vulnerabilidad 29: [Medium] Falta de rate limiting

* **Descripción del Riesgo:** En `backend/routes/api.php (L24)` existe un comentario explícito: `"without rate limiting"`, lo que permite intentos ilimitados de autenticación o peticiones abusivas.
* **Impacto:** Permite ataques de fuerza bruta y denegación de servicio (DoS) por falta de control en la frecuencia de solicitudes.
* **Remediación Técnica:** Implementar *middleware* de Rate Limiting en todos los endpoints de autenticación y funcionalidades críticas.

| Aspecto de la ISO/IEC 27001 (2022) | Control Relevante | Recomendación ISO 27001 para el Tratamiento |
| :--- | :--- | :--- |
| Protección ante ataques de fuerza bruta y abuso de servicio. | A.7.4 (Sistema de gestión de contraseñas) | Aplicar limitación de intentos de acceso y políticas de bloqueo temporal tras múltiples intentos fallidos. |

---

## 30. Vulnerabilidad 30: [Medium] Información del sistema expuesta

* **Descripción del Riesgo:** Los endpoints `/phpinfo`, `/env`, `/api/debug/info`, `/api/admin/system-info` exponen información detallada del sistema, configuración del servidor y versiones.
* **Impacto:** Facilita a un atacante el reconocimiento del entorno y la identificación de vulnerabilidades específicas del sistema.
* **Remediación Técnica:** Eliminar todos los endpoints de información y depuración en producción. Restringir su acceso solo a administradores autenticados si es estrictamente necesario.

| Aspecto de la ISO/IEC 27001 (2022) | Control Relevante | Recomendación ISO 27001 para el Tratamiento |
| :--- | :--- | :--- |
| Instalación, gestión y registro seguro de software en sistemas operativos. | A.8.18 (Instalación, gestión y registro de software en sistemas operativos) | Deshabilitar o eliminar funciones de diagnóstico en producción. Asegurar que la información del sistema solo sea accesible mediante autenticación y privilegios adecuados. |

---