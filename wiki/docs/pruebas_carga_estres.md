
# Pruebas de Carga y Estrés: k6 y Locust

Las **pruebas de carga y estrés** son cruciales para garantizar que una aplicación web pueda soportar múltiples usuarios concurrentes y condiciones extremas sin degradar su rendimiento o fallar.

* **Prueba de carga:** Evalúa el comportamiento de la aplicación bajo un volumen esperado de usuarios.
* **Prueba de estrés:** Lleva el sistema más allá de su capacidad para identificar su punto de quiebre y cómo se recupera.

---

## k6

k6 es una herramienta de código abierto para pruebas de carga y rendimiento, diseñada para desarrolladores y equipos DevOps. Permite escribir scripts en JavaScript para simular usuarios virtuales y obtener métricas de rendimiento detalladas.

**Casos de uso:** Puede ser utilizada para **smoke tests**, pruebas de **carga**, **estrés**, **spike tests** (picos de usuarios) y **soak tests** (pruebas de larga duración).

---

### Instalación paso a paso

**Requisitos previos:**

* **Chocolatey** instalado en Windows.
* **Node.js** instalado (necesario para algunos módulos y el dashboard web).

**1. Instalar Node.js con Chocolatey:**

```bash
choco install nodejs
```

**2. Instalar k6 con Chocolatey:**

```bash
choco install k6
```

-----

### Activar el dashboard web de k6

Para visualizar los resultados en tiempo real, activa el dashboard web con la siguiente variable de entorno:

```powershell
$env:K6_WEB_DASHBOARD = "true"
```

Esto habilitará un dashboard accesible en `http://localhost:5665` durante la ejecución de la prueba.

-----

### Ejemplo de prueba de estrés (estres.js)

```javascript
import http from 'k6/http';
import { sleep } from 'k6';

export let options = {
  stages: [
    { duration: '1m', target: 100 },
    { duration: '1m', target: 300 },
    { duration: '1m', target: 500 },
    { duration: '1m', target: 700 },
    { duration: '1m', target: 1000 },
    { duration: '2m', target: 0 },
  ],
};

export default function () {
  http.get('[http://74.179.196.8/autos](http://74.179.196.8/autos)');
  sleep(1);
}
```

-----

### Ejemplo de prueba de carga con autenticación (carga.js)

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';
import { parseHTML } from 'k6/html';

export let options = {
    vus: 10,
    duration: '1m',
};

const BASE_URL = '[http://74.179.196.8](http://74.179.196.8)';
const COOKIE_NAME = 'SESS5ef5840d62e4dfffe889c8045e5c8867';
const COOKIE_VALUE = '206b21fae221bd3d2214364a30c2ee2c';

export default function () {
    let res = http.get(`${BASE_URL}/node/add/auto`, {
        headers: { 'Cookie': `${COOKIE_NAME}=${COOKIE_VALUE}` },
    });

    let doc = parseHTML(res.body);
    let form_build_id = doc.find('input[name="form_build_id"]').first().attr('value');
    let form_token = doc.find('input[name="form_token"]').first().attr('value');
    let form_id = doc.find('input[name="form_id"]').first().attr('value');

    let payload = {
        'title': `Prueba k6 ${__VU}-${__ITER}`,
        'form_build_id': form_build_id,
        'form_token': form_token,
        'form_id': form_id,
        'op': 'Save',
    };

    let postRes = http.post(`${BASE_URL}/node/add/auto`, payload, {
        headers: {
            'Cookie': `${COOKIE_NAME}=${COOKIE_VALUE}`,
            'Content-Type': 'application/x-www-form-urlencoded',
        },
    });

    check(postRes, { 'status is 200': (r) => r.status === 200 });
    sleep(1);
}
```

-----

### Ejecución y exportación de resultados

**1. Ejecutar la prueba:**

```
k6 run estres.js
```

o

```
k6 run carga.js
```

**2. Ver el dashboard web:**

Abre `http://localhost:5665` en tu navegador.

**3. Exportar resultados:**

k6 permite exportar métricas a diferentes formatos y sistemas.

  * **JSON:**

    ```
    k6 run --out json=resultado.json estres.js
    ```

  * **InfluxDB (para visualización avanzada con Grafana):**

    ```
    k6 run --out influxdb=http://localhost:8086/k6db estres.js
    ```

-----

### Apreciaciones y recomendaciones de k6

  * k6 es ideal para integrar pruebas en pipelines **CI/CD** y para scripting avanzado.
  * Permite definir **checks y thresholds** para validar automáticamente el éxito de la prueba.
  * Facilita la **exportación de resultados** para análisis posterior.
  * Aunque no tiene una interfaz gráfica para crear scripts, el dashboard web y la integración con Grafana lo compensan.

-----

## Locust

Locust es una herramienta de código abierto para pruebas de carga y estrés, escrita en Python. Permite simular miles de usuarios concurrentes de forma sencilla y flexible. Los escenarios se definen en scripts Python y se pueden controlar y visualizar desde una interfaz web intuitiva.

-----

### Instalación paso a paso

**Requisitos previos:**

  * Python 3.6 o superior instalado.

**1. Crear y activar un entorno virtual (recomendado):**

```bash
python -m venv venv
.\venv\Scripts\activate
```

**2. Instalar Locust y dependencias:**

```bash
pip install locust beautifulsoup4
```

> Si tu script necesita otras librerías (por ejemplo, `lorem-text` o `names` para datos aleatorios), instálalas con `pip`.

**3. Verificar la instalación:**

```bash
locust --help
```

Si ves la ayuda de Locust, la instalación fue exitosa.

-----

### Ejemplo básico de script (carga.py)

```python
from locust import HttpUser, task, between

class ANMUser(HttpUser):
    wait_time = between(1, 1) # Simula tiempo de espera entre acciones
    @task
    def load_test(self):
        self.client.get("/autos")
```

  * `wait_time` simula el tiempo que un usuario virtual espera entre acciones.
  * Cada método decorado con `@task` representa una acción que el usuario virtual ejecutará.

-----

### Ejemplo avanzado (estres.py)

Puedes crear scripts tan complejos como necesites, incluyendo autenticación, subida de archivos, extracción de tokens, etc. 

```python
import json
import time
import random
from locust import HttpUser, task, between
from bs4 import BeautifulSoup
from datetime import datetime

class DrupalUser(HttpUser):
    wait_time = between(1, 3)

    def on_start(self):
        self.client.cookies.set(
            "SESS5ef5840d62e4dfffe889c8045e5c8867",
            "206b21fae221bd3d2214364a30c2ee2c",
            domain="74.179.196.8"
        )

    @task
    def crear_auto_con_archivo(self):
        # 1. GET al formulario (espera la respuesta)
        response = self.client.get("/node/add/auto")
        soup = BeautifulSoup(response.text, "html.parser")
        form_build_id = soup.find("input", {"name": "form_build_id"})["value"]
        form_token = soup.find("input", {"name": "form_token"})["value"]
        form_id = "node_auto_form"

        # 2. POST AJAX para subir el archivo (espera la respuesta)
        fid = None
        max_retries = 3
        for intento in range(max_retries):
            unique_name = f"k6report_{random.randint(1000,9999)}_{int(time.time()*1000)}.pdf"
            with open('k6report.pdf', 'rb') as f:
                files = {
                    'files[field_archivo_0][]': (unique_name, f, 'application/pdf')
                }
                data = {
                    'form_build_id': form_build_id,
                    'form_token': form_token,
                    'form_id': form_id,
                    '_triggering_element_name': 'field_archivo_0_upload_button',
                    '_triggering_element_value': 'Upload'
                }
                upload_url = "/node/add/auto?element_parents=field_archivo/widget&ajax_form=1&_wrapper_format=drupal_ajax"
                upload_response = self.client.post(upload_url, data=data, files=files)
            try:
                ajax_commands = json.loads(upload_response.text)
                for cmd in ajax_commands:
                    if cmd.get("command") in ["insert", "replaceWith"]:
                        html = cmd.get("data", "")
                        if "File already locked for writing" in html:
                            print("Archivo bloqueado, reintentando...")
                            time.sleep(1)
                            break
                        soup_upload = BeautifulSoup(html, "html.parser")
                        input_fid = soup_upload.find("input", {"name": "field_archivo[0][fids]"})
                        if input_fid and input_fid.has_attr("value") and input_fid["value"]:
                            fid = input_fid["value"]
                            break
                if fid:
                    break
            except Exception as e:
                print("Error parseando la respuesta AJAX:", e)
        if not fid:
            print("No se pudo extraer el fid del archivo subido")
            print(upload_response.text)
            return

        # 3. POST para crear el nodo (espera la respuesta)
        now = datetime.now()
        fecha = now.strftime("%Y-%m-%d")
        hora = now.strftime("%H:%M:%S")
        payload = {
            "changed": str(int(now.timestamp())),
            "title[0][value]": "TEST Locust con archivo",
            "form_build_id": form_build_id,
            "form_token": form_token,
            "form_id": form_id,
            "field_vigencia_texto[0][value]": "",
            "field_archivo[0][_weight]": "0",
            "field_archivo[0][fids]": fid,
            "field_descripcion_documento[0][value]": "Documento descripción con archivo",
            "field_descripcion_documento[0][format]": "full_html",
            "field_tipo_documento[0][value]": "archivo",
            "revision_log[0][value]": "",
            "menu[title]": "",
            "menu[description]": "",
            "menu[menu_parent]": "main:",
            "menu[weight]": "0",
            "book[bid]": "0",
            "book[pid]": "-1",
            "book[weight]": "0",
            "path[0][alias]": "",
            "uid[0][target_id]": "iarrieta (21)",
            "created[0][value][date]": fecha,
            "created[0][value][time]": hora,
            "op": "Save"
        }
        post_response = self.client.post(
            "/node/add/auto",
            data=payload,
            headers={"Content-Type": "application/x-www-form-urlencoded"}
        )
        if post_response.status_code not in [200, 303]:
            print("Status:", post_response.status_code)
            print("Body:", post_response.text)
        else:
            print("Nodo creado con archivo, fid:", fid)
```

El ejemplo proporcionado anteriormente para k6, adaptado a Python, sería perfectamente válido y demostraría cómo simular un flujo realista, con manejo de cookies, parsing de HTML y peticiones POST.

-----

### Ejecución de pruebas

**1. Ejecutar Locust:**

```bash
locust -f carga.py
```

o

```bash
locust -f estres.py
```

**2. Abrir la interfaz web:**

Navega a `http://localhost:8089`.

**3. Configurar la prueba:**

  * **Número de usuarios (Users):** Cuántos usuarios virtuales quieres simular.
  * **Tasa de generación (Spawn rate):** Cuántos usuarios por segundo se agregan.
  * **Host:** La URL base de tu aplicación (por ejemplo, `http://74.179.196.8`).

Haz clic en **Start swarming** para iniciar la prueba.

-----

### Métricas y exportación de resultados

La interfaz web de Locust muestra:

  * **Requests:** Total de peticiones realizadas.
  * **Fails:** Peticiones fallidas.
  * **Median, 90%ile, Average, Min, Max:** Tiempos de respuesta.
  * **Current RPS:** Solicitudes por segundo.

Puedes **descargar los resultados en CSV** desde la pestaña **Download Data**.

**Ejecución por línea de comandos (sin interfaz web):**

```bash
locust -f carga.py --headless -u 100 -r 10 --host [http://74.179.196.8](http://74.179.196.8) --run-time 1m --csv=resultado
```

  * `-u`: Número de usuarios.
  * `-r`: Tasa de generación.
  * `--run-time`: Duración de la prueba.
  * `--csv=resultado`: Exporta los resultados a archivos CSV.

-----

### Buenas prácticas y recomendaciones con Locust

  * **Siempre usa un entorno virtual** para aislar las dependencias del proyecto.
  * **Nunca pruebes en producción** sin la autorización adecuada y una planificación cuidadosa.
  * **Simula usuarios reales:** Incorpora `wait_time`, asigna diferentes pesos a las tareas y utiliza datos aleatorios para un comportamiento más realista.
  * **Divide tareas:** Puedes definir varios tipos de usuarios (clases) para simular diferentes roles o flujos de usuario.
  * **Valida respuestas:** Utiliza asserts o checks para asegurar que las respuestas de la aplicación sean correctas.
  * **Automatiza:** Ejecuta Locust en modo headless y exporta los resultados para integrarlo en pipelines de CI/CD.
  * **Escala horizontalmente:** Locust soporta ejecución distribuida (cluster) para simular un gran volumen de usuarios concurrentes.

-----

## Resumen y mejores prácticas generales

  * **Define objetivos claros** antes de cada prueba (por ejemplo, número de usuarios, tiempos de respuesta aceptables).
  * **Realiza pruebas en entornos controlados**, nunca en producción sin autorización explícita.
  * **Analiza los resultados** meticulosamente para identificar cuellos de botella y oportunidades de optimización.
  * **Integra las pruebas en tu pipeline de CI/CD** para garantizar la calidad continua de la aplicación.

-----

## Descargar plantilla de informe

[![Descargar](https://img.shields.io/badge/📥%20Descargar%20Word-blue)](files/Reporte_ANM_Carga_Estres.docx)
