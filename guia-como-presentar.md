# GUÍA DE PRESENTACIÓN — SUSTENTACIÓN
## Cómo presentar el proyecto HTTPS/TLS en TrazBasProxyService

---

## INFORMACIÓN GENERAL

| Aspecto | Detalle |
|---------|---------|
| Duración total | 20-25 minutos de exposición |
| Preguntas del jurado | 10-15 minutos adicionales |
| Diapositivas | 18 slides |
| Ritmo sugerido | 1-2 minutos por diapositiva |
| Vestimenta | Formal (camisa, pantalón de vestir) |
| Postura | De pie, frente al jurado, no dar la espalda |

---

## ESTRUCTURA DE TIEMPO

| Sección | Slides | Tiempo |
|---------|--------|--------|
| Portada + Agenda | 1-2 | 1 min |
| Contexto + Problema | 3-4 | 3 min |
| Objetivos + Marco teórico | 5-7 | 4 min |
| Metodología + Diagnóstico | 8-9 | 3 min |
| Implementación + Desafíos | 10-11 | 4 min |
| Resultados + Evidencias | 12-14 | 4 min |
| Análisis + Conclusiones | 15-16 | 3 min |
| Recomendaciones + Cierre | 17-18 | 2 min |
| **Total** | **18** | **~24 min** |

---

## GUIÓN — LO QUE DECIR EN CADA DIAPOSITIVA

### Slide 1 — Portada (30 segundos)
> "Buenos días / tardes. Mi nombre es Mario Testa, y el día de hoy voy a presentar mi informe de prácticas profesionales realizado en la empresa Minex, titulado: Implementación de protocolo HTTPS/TLS en el servicio web TrazBasProxyService para la transmisión segura de datos entre el software Trasbaz y la plataforma TrazCloud."

---

### Slide 2 — Agenda (30 segundos)
> "La presentación está organizada en diez puntos. Comenzaremos con el contexto donde se desarrolló el proyecto, el problema identificado, los objetivos, el marco teórico, la metodología utilizada, el diagnóstico inicial, el proceso de implementación, los resultados obtenidos, el análisis de esos resultados, y finalmente las conclusiones y recomendaciones."

---

### Slide 3 — Contexto (1 minuto)
> "Minex cuenta con un sistema de gestión de básculas industriales llamado Trasbaz. Este software captura los datos de pesaje en tiempo real y los envía a una plataforma en la nube llamada TrazCloud, alojada en Microsoft Azure. La comunicación entre ambos sistemas se realiza a través de un servicio web llamado TrazBasProxyService, que es precisamente el componente sobre el que trabajé durante mis prácticas. Este servicio está desarrollado con Windows Communication Foundation, conocido como WCF, en .NET 4.7, y está desplegado sobre IIS 10."

---

### Slide 4 — Problema (1.5 minutos)
> "Al iniciar mis prácticas, encontré que TrazBasProxyService operaba exclusivamente sobre el protocolo HTTP, en el puerto 80, sin ningún tipo de cifrado. Esto significaba que toda la comunicación — incluyendo las credenciales de acceso y los datos de pesaje — viajaba en texto plano por la red interna."

> "Cualquier equipo conectado a esa red podía interceptar y leer esos mensajes con herramientas comunes como Wireshark. Adicionalmente, el contrato del servicio, el WSDL, era públicamente accesible sin ninguna autenticación, y las respuestas del servidor incluían cabeceras que revelaban la tecnología utilizada — la versión de IIS y de ASP.NET — lo que facilita ataques dirigidos."

> "Esta situación corresponde directamente a la vulnerabilidad A02:2021 del estándar OWASP Top 10, denominada Cryptographic Failures, o fallas criptográficas."

---

### Slide 5 — Objetivos (1 minuto)
> "El objetivo general fue implementar HTTPS sobre el servicio existente mediante configuración de IIS y del archivo web.config, sin modificar el código fuente, y validar la mejora mediante un conjunto de métricas cuantitativas."

> "Los objetivos específicos incluyeron: medir el estado inicial, instalar un certificado SSL, habilitar el puerto 443, implementar la redirección de HTTP a HTTPS, configurar cabeceras de seguridad, bloquear verbos inseguros, y documentar una guía para que el equipo de infraestructura replique la configuración en el servidor de producción."

---

### Slide 6 — Marco teórico parte 1 (1.5 minutos)
> "En cuanto al marco teórico, es importante entender la diferencia entre HTTP y HTTPS. HTTP transmite todo en texto plano sobre el puerto 80, mientras que HTTPS cifra la comunicación mediante el protocolo TLS sobre el puerto 443."

> "El servicio utiliza WCF con basicHttpBinding. Este binding soporta dos modos de seguridad: 'None', que es la configuración inicial sin cifrado, y 'Transport', donde el cifrado es manejado por TLS a nivel de la infraestructura. Esto nos permitió activar la seguridad sin cambiar el código del servicio."

---

### Slide 7 — Marco teórico parte 2 (1 minuto)
> "En cuanto a las cabeceras de seguridad HTTP, implementamos cuatro: Strict-Transport-Security, que fuerza el uso de HTTPS en futuros accesos; X-Frame-Options, que previene ataques de clickjacking; X-Content-Type-Options, que evita el MIME sniffing; y X-XSS-Protection, que activa los filtros anti-XSS del cliente."

---

### Slide 8 — Metodología (1.5 minutos)
> "La metodología adoptada es cuantitativa, con un diseño pre-experimental de un solo grupo: se midió el estado inicial, se aplicó la intervención técnica, y se midió el estado final."

> "Se definieron ocho métricas, cada una con un valor objetivo claro y medible. Las herramientas de medición fueron PowerShell para la gestión del servidor, curl para las pruebas de comandos, y Postman para la validación visual desde la estación de trabajo."

---

### Slide 9 — Diagnóstico inicial (1 minuto)
> "La medición inicial reveló que ninguna de las ocho métricas cumplía su objetivo. El resultado fue cero de ocho. El servicio respondía en HTTP con código 200 directamente, el puerto 443 no existía, el WSDL era público, no había ninguna cabecera de seguridad, y el verbo TRACE era procesado sin restricciones."

---

### Slide 10 — Implementación (2 minutos)
> "La implementación se organizó en seis fases secuenciales."

> "Primero, generé un certificado SSL autofirmado con RSA de 2048 bits y validez de dos años usando PowerShell. Segundo, configuré el binding HTTPS en el puerto 443 del sitio IIS y asocié el certificado. Tercero, implementé la redirección HTTP a HTTPS."

> "Cuarto, apliqué el nuevo web.config con todas las cabeceras de seguridad. Quinto, desbloqueé la sección requestFiltering en IIS para poder bloquear el verbo TRACE. Y finalmente, realicé la verificación integral con curl y Postman."

> "Lo más importante: en ningún momento modifiqué el código fuente del servicio en VB.NET. Todos los cambios fueron de configuración."

---

### Slide 11 — Desafíos (1.5 minutos)
> "Durante la implementación encontré varios obstáculos que requirieron soluciones alternativas. El más significativo fue que el módulo URL Rewrite de IIS no estaba instalado en el servidor, y el servidor tampoco tenía acceso a Internet para descargarlo. Este módulo es la forma estándar de implementar la redirección HTTP a HTTPS en IIS."

> "La solución fue usar el archivo Global.asax del sitio de redirección, aprovechando el evento Application_BeginRequest de ASP.NET para interceptar cada solicitud HTTP y redirigirla al canal HTTPS con código 301."

> "Otro problema fue que al escribir el web.config con PowerShell, el archivo se generaba con un marcador de orden de bytes UTF-8, conocido como BOM, que IIS no puede parsear. La solución fue usar la clase UTF8Encoding de .NET con el parámetro false para eliminar el BOM."

---

### Slide 12 — Resultados (1 minuto)
> "Tras la implementación, las ocho métricas alcanzaron sus valores objetivo. El resultado fue ocho de ocho, un 100% de conformidad, en comparación con el cero de ocho del diagnóstico inicial."

> "El puerto 443 está activo con TLS, el HTTP redirige a HTTPS con código 301, las cuatro cabeceras de seguridad están presentes, el TRACE retorna 404 bloqueado, y el certificado tiene 730 días de vigencia."

---

### Slides 13-14 — Evidencias Postman (1 minuto)
> "Aquí pueden ver las evidencias capturadas con Postman desde mi estación de trabajo, accediendo al servidor 10.1.1.66."

> "En la primera evidencia, con la opción de seguir redirects desactivada, la solicitud HTTP retorna código 301 con el header Location apuntando a la URL HTTPS. Esto confirma la métrica M5."

> "En la segunda evidencia, en la pestaña de headers de la respuesta HTTPS, se pueden ver las cuatro cabeceras de seguridad implementadas, y se verifica la ausencia de X-Powered-By y X-AspNet-Version. Esto confirma la métrica M3."

---

### Slide 15 — Análisis (1 minuto)
> "El análisis de resultados confirma tres puntos principales. Primero, la hipótesis de trabajo fue correcta: es completamente viable implementar HTTPS en un servicio WCF legacy sin modificar su código. Segundo, la vulnerabilidad OWASP A02 fue eliminada: toda la comunicación ahora viaja cifrada. Tercero, la superficie de ataque del servicio se redujo significativamente al eliminar la exposición del WSDL en texto plano y suprimir las cabeceras informativas."

---

### Slide 16 — Conclusiones (1 minuto)
> "En conclusión: las ocho métricas definidas pasaron de cero a ocho conformes. El servicio ahora opera sobre TLS. La implementación no requirió modificar el código fuente. Y se produjo una guía técnica lista para que el equipo de infraestructura aplique los mismos cambios en el servidor de producción 10.0.0.4."

---

### Slide 17 — Recomendaciones (45 segundos)
> "Como trabajo futuro, recomiendo prioritariamente deshabilitar TLS 1.0 y 1.1 en el servidor, dejando únicamente TLS 1.2 y 1.3 habilitados. También recomiendo reemplazar el certificado autofirmado por uno emitido por una Autoridad de Certificación reconocida antes de pasar a producción."

---

### Slide 18 — Cierre (30 segundos)
> "Termino citando a Bruce Schneier: 'La seguridad no es un producto, es un proceso.' Este proyecto es un paso en ese proceso continuo para Minex."

> "Quedo a disposición del jurado para responder cualquier pregunta. Muchas gracias."

---

## PREGUNTAS FRECUENTES DEL JURADO — Y CÓMO RESPONDERLAS

### "¿Por qué usaste un certificado autofirmado y no uno de una CA?"
> "El servidor de pruebas 10.1.1.66 es un entorno interno sin acceso a Internet, por lo que no era posible obtener un certificado de una CA pública como Let's Encrypt. Un certificado autofirmado es suficiente para validar el funcionamiento técnico en el entorno de pruebas. Para producción, la guía que elaboré indica específicamente el proceso de importar un certificado de CA corporativa."

---

### "¿Por qué no modificaste el código fuente?"
> "Esa fue una restricción deliberada del proyecto. Modificar el código fuente requeriría recompilar el servicio, lo que implica riesgo de regresión, ciclos de prueba adicionales y coordinación con el equipo de desarrollo. Al limitarse a configuración de infraestructura, los cambios son más seguros, reversibles y aplicables por el equipo de infraestructura sin conocimientos del código."

---

### "¿Qué es WCF y por qué se usa aquí?"
> "WCF es Windows Communication Foundation, el framework de Microsoft para servicios web SOAP en .NET. Se usa en TrazBasProxyService porque implementa el estándar SOAP 1.1 que muchos sistemas industriales legacy requieren. El servicio fue desarrollado antes de que REST se convirtiera en el estándar predominante."

---

### "¿Cuál es la diferencia entre TLS y SSL?"
> "SSL es el protocolo original, ahora obsoleto y con vulnerabilidades conocidas. TLS es su sucesor y la versión actual que se usa en la práctica. Cuando hablamos de HTTPS hoy en día, técnicamente estamos usando TLS, aunque coloquialmente se sigue llamando SSL. Las versiones actuales recomendadas son TLS 1.2 y TLS 1.3."

---

### "¿Qué es OWASP?"
> "OWASP es el Open Web Application Security Project, una fundación sin fines de lucro que publica estándares y guías de seguridad web. Su lista Top 10 identifica las diez vulnerabilidades más críticas en aplicaciones web. La situación inicial del servicio correspondía a A02:2021 — Cryptographic Failures, que es la segunda vulnerabilidad más crítica según el estándar 2021."

---

### "¿Por qué el resultado de M4 es 404 y no 405?"
> "Cuando el módulo requestFiltering de IIS bloquea un verbo HTTP, retorna un código 404 en lugar de 405. Esto es un comportamiento intencional de IIS: el 404 no confirma ni niega la existencia del recurso, mientras que un 405 indicaría que el recurso existe pero el método no está permitido. El 404 es más seguro desde el punto de vista de la revelación de información."

---

### "¿Cómo garantizas que los cambios en producción serán iguales?"
> "Elaboré una guía técnica paso a paso, documentada en el repositorio, con los comandos PowerShell exactos necesarios para replicar cada fase de la implementación en el servidor de producción 10.0.0.4. La guía incluye también la consideración de que en producción se debe usar un certificado de CA corporativa en lugar del autofirmado."

---

### "¿Cuál fue el mayor desafío técnico?"
> "El mayor desafío fue implementar la redirección HTTP a HTTPS sin el módulo URL Rewrite, que es la solución estándar en IIS pero no estaba disponible en el servidor ni podía descargarse por falta de acceso a Internet. La solución alternativa usando Global.asax con el evento Application_BeginRequest me obligó a entender en profundidad el pipeline integrado de ASP.NET, lo cual resultó ser un aprendizaje valioso."

---

## CONSEJOS GENERALES PARA LA PRESENTACIÓN

### Antes de presentar:
- [ ] Practica el guión completo al menos 3 veces en voz alta
- [ ] Cronométrate para verificar que estés en el rango de 20-25 minutos
- [ ] Prepara las capturas de Postman impresas como respaldo
- [ ] Lleva el informe impreso por si el jurado pide ver detalles
- [ ] Llega 15 minutos antes para revisar el proyector y la sala

### Durante la presentación:
- [ ] Habla despacio y con voz clara — el nerviosismo acelera el habla
- [ ] Mira al jurado, no a la pantalla
- [ ] Usa el puntero o el cursor para señalar los datos en las tablas
- [ ] Si pierdes el hilo, mira el slide y retoma desde el último punto
- [ ] No te disculpes si te equivocas — corrígete y continúa

### Al responder preguntas:
- [ ] Escucha la pregunta completa antes de responder
- [ ] Si no sabes algo, di: "Eso está fuera del alcance de este proyecto, pero la recomendación sería..."
- [ ] Apoya tus respuestas en los datos de las métricas
- [ ] Mantén la calma — el jurado no busca que sepas todo, busca que razonés correctamente

### Frases útiles:
- "Como se puede observar en la tabla..."
- "Los datos muestran que..."
- "La métrica M[X] pasó de [ANTES] a [DESPUÉS], lo que representa..."
- "Esa es una excelente observación. En el contexto de este proyecto..."

---

## DISEÑO SUGERIDO PARA LAS DIAPOSITIVAS

### Paleta de colores recomendada:
- **Primario:** Azul oscuro (#1B3A6B) — autoridad, tecnología
- **Secundario:** Verde (#27AE60) — éxito, conformidad
- **Alerta:** Rojo (#E74C3C) — problemas, riesgos
- **Neutro:** Gris claro (#F5F5F5) — fondos de tablas

### Fuentes:
- **Títulos:** Calibri Bold o Montserrat Bold, 28-32pt
- **Cuerpo:** Calibri o Open Sans, 18-20pt
- **Tablas:** 14-16pt

### Regla de diseño:
> Máximo 6 líneas de texto por diapositiva. Si hay más, divide en dos slides.

### Imágenes sugeridas (buscar en Flaticon o Freepik):
- Candado cerrado verde → slides de resultados positivos
- Candado abierto rojo → slide del problema
- Diagrama de red → slide de contexto
- Logo OWASP → slide de marco teórico
- Escudo con check → slide de conclusiones
