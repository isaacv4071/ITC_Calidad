# Documentación de Infraestructura de Pruebas Automatizadas, Pruebas de Carga y Estrés, y Gestión de Vulnerabilidades  
## Área de Calidad – Intelecto Services

### 1. [Pruebas Automatizadas](./pruebas_automatizadas.md)

En Intelecto Services, la automatización de pruebas se realiza principalmente con **Playwright**, permitiendo validar funcionalidades críticas de las aplicaciones web de forma rápida y confiable.

- **Playwright** se utiliza para pruebas end-to-end, integración y regresión, cubriendo los principales flujos de usuario en las aplicaciones web.
- Las pruebas se ejecutan de forma manual, sin integración directa con pipelines CI/CD.
- Los resultados de las pruebas se documentan mediante videos generados automáticamente por los scripts de Playwright, lo que permite visualizar el comportamiento de la aplicación durante la ejecución de cada caso de prueba.
- Estos videos facilitan la revisión, el análisis de errores y la trazabilidad de los casos probados, permitiendo compartir fácilmente la evidencia con el equipo de desarrollo y otras áreas interesadas.

### 2. [Pruebas de Carga y Estrés](./pruebas_carga_estres.md)

Para asegurar el rendimiento y la estabilidad de los sistemas bajo diferentes niveles de demanda, se emplean dos herramientas principales:

- **K6**: Utilizada para pruebas de carga automatizadas, simulando múltiples usuarios concurrentes y midiendo tiempos de respuesta, throughput y estabilidad del sistema.
- **Locust**: Permite crear escenarios personalizados de carga y estrés, con scripts en Python, facilitando la simulación de comportamientos complejos y la identificación de cuellos de botella.
- Ambas herramientas generan reportes detallados y métricas clave, que se analizan para tomar decisiones sobre optimización y escalabilidad.

### 3. [Gestión de Vulnerabilidades](./gestion_vulnerabilidades.md)

La seguridad es prioritaria en el área de calidad. Para la detección y gestión de vulnerabilidades se utilizan:

- **OWASP ZAP**: Herramienta de análisis dinámico (DAST) para identificar vulnerabilidades en aplicaciones web, como inyecciones, XSS y otros riesgos OWASP Top 10.
- **SonarQube**: Plataforma de análisis estático de código (SAST) que detecta vulnerabilidades, bugs y problemas de calidad en el código fuente antes de llegar a producción.
- Los hallazgos de ambas herramientas se documentan en reportes PDF generados automáticamente tras cada análisis.
- Estos reportes PDF contienen el detalle de las vulnerabilidades encontradas, su criticidad y recomendaciones de remediación.
- Los reportes se comparten con los equipos responsables para su revisión y priorización, facilitando la toma de decisiones y el seguimiento de la resolución de los hallazgos.

### 4. Buenas Prácticas

- Todos los scripts y configuraciones deben estar versionados en repositorios Git.
- Se realizan revisiones periódicas de los resultados y se ajustan los escenarios de prueba según la evolución de los sistemas.
- El equipo de calidad colabora estrechamente con desarrollo y seguridad para garantizar la cobertura y la mejora continua.