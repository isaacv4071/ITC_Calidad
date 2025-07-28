## Detección de Vulnerabilidades con OWASP ZAP y SonarQube

Esta sección describe cómo utilizar **OWASP ZAP** y **SonarQube** para identificar vulnerabilidades en aplicaciones web y en el código fuente, respectivamente.

---

### OWASP ZAP — Análisis de Seguridad Dinámico (DAST)

OWASP ZAP (Zed Attack Proxy) es una herramienta open source para encontrar vulnerabilidades en aplicaciones web mediante pruebas dinámicas (DAST).

#### Paso a paso para usar OWASP ZAP

1. **Instalación**
   - Descarga OWASP ZAP desde [la página oficial](https://www.zaproxy.org/download/).
   - Instala la aplicación según tu sistema operativo.

2. **Configuración inicial**
   - Abre OWASP ZAP.
   - Elige el modo de protección (por defecto, “Standard” está bien para la mayoría de los casos).
   - Configura el navegador para que use el proxy de ZAP (por defecto: localhost:8080).

3. **Escaneo de una aplicación web**
   - Ingresa la URL de la aplicación a analizar en el campo “URL to attack”.
   - Selecciona el tipo de escaneo:  
     - **Quick Start**: para un escaneo rápido.
     - **Automated Scan**: para un análisis más profundo.
   - Haz clic en “Attack” o “Start Scan”.

4. **Revisión de resultados**
   - Al finalizar el escaneo, revisa la pestaña de “Alerts” para ver las vulnerabilidades detectadas.
   - Haz clic en cada alerta para ver detalles, recomendaciones y posibles soluciones.

5. **Exportar reporte**
   - Ve a `Report > Generate Report`.
   - Elige el formato (HTML, XML, Markdown, etc.) y guarda el archivo.

#### Notas y buenas prácticas

- Realiza los escaneos en entornos de pruebas, nunca en producción sin autorización.
- Puedes automatizar ZAP usando scripts o integrarlo en pipelines CI/CD.
- Mantén ZAP actualizado para detectar las vulnerabilidades más recientes.

---

### SonarQube — Análisis de Código Estático (SAST)

SonarQube es una plataforma para el análisis estático del código fuente, detectando bugs, vulnerabilidades y code smells.

#### Paso a paso para usar SonarQube

1. **Instalación**
   - Descarga SonarQube Community Edition desde [la página oficial](https://www.sonarqube.org/downloads/).
   - Descomprime el archivo y sigue las instrucciones de instalación para tu sistema operativo.
   - Asegúrate de tener Java instalado (JDK 11 o superior recomendado).

2. **Inicio del servidor SonarQube**
   - Ejecuta el script de inicio:
     - En Windows: `bin\windows-x86-xx\StartSonar.bat`
     - En Linux/Mac: `bin/linux-x86-xx/sonar.sh start`
   - Accede a la interfaz web en [http://localhost:9000](http://localhost:9000).

3. **Configuración inicial**
   - Inicia sesión (usuario y contraseña por defecto: `admin` / `admin`).
   - Cambia la contraseña por seguridad.
   - Crea un nuevo proyecto desde la interfaz.

4. **Análisis de un proyecto**
   - Instala el scanner de SonarQube adecuado para tu lenguaje ([guía oficial](https://docs.sonarqube.org/latest/analysis/scan/sonarscanner/)).
   - Desde la raíz de tu proyecto, ejecuta el análisis. Ejemplo con SonarScanner CLI:
     ```
     sonar-scanner \
       -Dsonar.projectKey=nombre_proyecto \
       -Dsonar.sources=. \
       -Dsonar.host.url=http://localhost:9000 \
       -Dsonar.login=TOKEN_GENERADO
     ```
   - Espera a que finalice el análisis.

5. **Revisión de resultados**
   - Ingresa a la interfaz web de SonarQube.
   - Revisa los issues detectados: bugs, vulnerabilidades, code smells, duplicaciones, etc.
   - Haz clic en cada issue para ver detalles y recomendaciones.

#### Notas y buenas prácticas

- Integra SonarQube en tu pipeline de CI/CD para análisis automáticos en cada commit o pull request.
- Configura reglas de calidad y umbrales de aceptación según las necesidades del proyecto.
- Mantén el scanner y el servidor SonarQube actualizados.


### ¿Cómo documentar vulnerabilidades en un consolidado?

El **consolidado de vulnerabilidades** es un informe estructurado que centraliza los hallazgos críticos y medios detectados durante auditorías de seguridad, facilitando su análisis, priorización y remediación.  
A continuación, se describe la estructura recomendada y las mejores prácticas para su elaboración.

---

#### Estructura del consolidado

1. **Encabezado**
   - Título del informe
   - Fecha del reporte
   - Herramientas utilizadas (por ejemplo: OWASP ZAP, SonarQube)

2. **Resumen Ejecutivo**
   - Breve descripción del alcance, contexto y principales hallazgos.
   - Impacto general sobre la confidencialidad, integridad y disponibilidad.

3. **Listado de Vulnerabilidades**
   - **Número y nombre de la vulnerabilidad**
   - **Descripción:** Explicación clara del problema.
   - **Archivos/Endpoints afectados:** Ubicación exacta (archivos, rutas, fragmentos de código, endpoints).
   - **Evidencia:** Fragmentos de código, configuraciones, capturas o ejemplos que demuestren la vulnerabilidad.
   - **Repercusión:** Riesgos y posibles impactos.
   - **Referencia:** CWE, OWASP Top 10 u otra normativa relevante.
   - **Recomendación:** Acciones concretas para mitigar o eliminar la vulnerabilidad.

4. **Conclusión y Recomendaciones Generales**
   - Resumen de acciones prioritarias.
   - Sugerencias para mejorar procesos, capacitación y controles.

---

#### Ejemplo de entrada en el consolidado

```markdown
##### 1. Librerías JavaScript Vulnerables

**Descripción:** Se identificaron bibliotecas JavaScript desactualizadas con vulnerabilidades conocidas.

**Archivos afectados:**  
- /Web/wwwroot/js/libs/jquery.js  
- /Web/wwwroot/js/libs/bootstrap.js

**Repercusión:** Exposición a ataques XSS, ejecución de código malicioso o secuestro de sesión.

**Referencia:** OWASP Top 10 - A06:2021 | CWE-1395

**Recomendación:** Actualizar todas las librerías a versiones seguras. Implementar control de versiones y revisión periódica de dependencias.
```

---

#### Buenas prácticas para la documentación

- **Sé específico:** Indica archivos, endpoints y fragmentos de código exactos.
- **Incluye evidencia:** Copia fragmentos relevantes o capturas de pantalla.
- **Referencia estándares:** Usa CWE, OWASP Top 10, etc., para clasificar la vulnerabilidad.
- **Recomendaciones claras:** Propón acciones concretas y realistas.
- **Prioriza:** Distingue entre vulnerabilidades críticas, medias y bajas.
- **Actualiza el consolidado periódicamente** tras cada auditoría o revisión.

---

#### Plantilla sugerida para cada vulnerabilidad

```markdown
##### [Número]. [Nombre de la vulnerabilidad]

**Descripción:** [Explicación clara del hallazgo]

**Archivos/Endpoints afectados:**  
- [Lista de archivos, rutas o endpoints]

**Evidencia:**  
[Fragmento de código, configuración, ejemplo, captura, etc.]

**Repercusión:** [Impacto potencial]

**Referencia:** [CWE, OWASP Top 10, etc.]

**Recomendación:** [Acción concreta para mitigar]
```
