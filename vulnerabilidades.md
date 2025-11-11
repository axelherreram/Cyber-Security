21. Vulnerabilidad 21: [High] Reset de contraseña sin verificación 
•	Descripción del Riesgo: El endpoint /api/auth/reset-password permite cambiar contraseñas sin verificar la identidad del usuario ni validar un token de recuperación. 
•	Impacto: Compromiso de cuentas mediante cambio no autorizado de contraseñas y acceso no autorizado al sistema. 
•	Remediación Técnica: Implementar tokens de recuperación seguros y con expiración. Verificar identidad mediante email o SMS. Requerir contraseña actual para cambio. Implementar rate limiting. 
Aspecto de la 
ISO/IEC 27001 
(2022) 	Control Relevante 	Recomendación ISO 27001 para el Tratamiento 
Autenticación segura y gestión de credenciales. 	A.8.5 
(Autenticación segura) 	Implementar un mecanismo de recuperación de contraseña de dos pasos (enlace único y temporal enviado a un canal verificado, como el correo electrónico o SMS). 
Información de autenticación y protección. 	A.5.17 
(Información de autenticación) 	Aplicar límites de velocidad (rate limiting) para prevenir ataques de fuerza bruta o de enumeración en el proceso de restablecimiento de contraseña. 
  
22. Vulnerabilidad 22: [High] Manipulación de sesión sin autorización 
•	Descripción del Riesgo: El endpoint /api/debug/session permite la manipulación completa de sesiones sin autenticación, incluyendo el establecimiento de roles de administrador. 
•	Impacto: Escalada de privilegios, secuestro de sesiones y bypass de autenticación. 
•	Remediación Técnica: Eliminar endpoint de manipulación de sesión en producción. Implementar protección de datos de sesión críticos. Validar cambios de rol en el servidor. 
Aspecto de la 
ISO/IEC 27001 
(2022) 	Control Relevante 	Recomendación ISO 27001 para el Tratamiento 
Control de acceso y gestión de la sesión. 	A.5.15 (Control de acceso) 	Eliminar completamente cualquier endpoint de manipulación de sesión en producción. La información crítica de sesión (como roles de usuario) debe ser inalterable por el usuario. 
Separación de entornos y protección de sistemas. 	A.8.21 (Separación de los entornos de desarrollo, prueba y producción) 	Asegurar que este tipo de funcionalidad de depuración o manipulación no sea accesible en el entorno de producción. 
  
23. Vulnerabilidad 23: [High] Exposición de backups de base de datos 
•	Descripción del Riesgo: El endpoint /api/admin/backup crea backups sin encriptación y expone credenciales de base de datos en el comando ejecutado. 
•	Impacto: Exposición de credenciales en logs y procesos, backups no encriptados accesibles y compromiso completo de datos. 
•	Remediación Técnica: Encriptar backups. No exponer credenciales en comandos. 
Restringir acceso a backups. Usar variables de entorno para credenciales. 
Aspecto de la 
ISO/IEC 27001 (2022) 	Control Relevante 	Recomendación ISO 27001 para el Tratamiento 
Protección contra pérdida de datos y confidencialidad. 	A.8.12 (Copias de seguridad de la información) 	Los backups deben ser cifrados antes de su almacenamiento (en reposo) y su transferencia. Implementar un proceso para probar la recuperación de manera periódica. 
Gestión de claves criptográficas y secretos. 	A.8.11 (Gestión de claves criptográficas) 	Nunca exponer credenciales sensibles (como contraseñas de DB) en comandos o logs de ejecución. Usar variables de entorno protegidas o 
		un gestor de secretos. 
  
24. Vulnerabilidad 24: [High] Almacenamiento de sesiones en localStorage 
•	Descripción del Riesgo: El frontend almacena tokens de sesión en localStorage, accesible mediante JavaScript y vulnerable a XSS. 
•	Impacto: Robo de tokens mediante XSS, acceso no autorizado a cuentas y persistencia de sesión vulnerable. 
•	Remediación Técnica: Usar httpOnly cookies para tokens. Implementar SameSite cookies. Usar sessionStorage en lugar de localStorage si es necesario. Implementar refresh tokens. 
Aspecto de la 
ISO/IEC 27001 
(2022) 	Control Relevante 	Recomendación ISO 27001 para el Tratamiento 
Protección contra malware y código malicioso. 	A.8.7 (Protección contra malware) 	Almacenar los tokens de sesión crítica en HttpOnly cookies para hacerlos inaccesibles a scripts de frontend y mitigar el robo por XSS. 
Seguridad de la información en la transmisión. 	A.8.19 
(Seguridad de 
la red) 	Implementar la directiva Secure y 
SameSite=Strict en las cookies de sesión para prevenir la transmisión sobre canales no cifrados y mitigar ataques CSRF. 
  
25. Vulnerabilidad 25: [High] Exposición de información de usuario completa 
•	Descripción del Riesgo: Los endpoints retornan información completa de usuarios, incluyendo passwords, API keys y secret keys sin restricción. 
•	Impacto: Exposición masiva de credenciales, compromiso de cuentas y acceso a servicios externos mediante API keys. 
•	Remediación Técnica: No retornar campos sensibles en respuestas. Implementar campos ocultos en modelos. Usar recursos de API (API Resources) para filtrar datos. Validar permisos antes de exponer información. 
Aspecto de la 
ISO/IEC 27001 
(2022) 	Control Relevante 	Recomendación ISO 27001 para el Tratamiento 
Control de acceso y minimización de exposición. 	A.8.2 
(Derechos de acceso) 	Aplicar filtrado estricto a nivel de servidor (usando API Resources o serializadores) para excluir siempre campos sensibles como passwords, hashes, API keys o información personal en la respuesta de la API. 
Garantizar el principio de mínimo privilegio de la información. 	A.5.15 (Control de acceso) 	La respuesta de la API debe incluir solo los datos esenciales necesarios para la función. Si el campo es sensible, debe ser omitido o enmascarado por defecto. 
