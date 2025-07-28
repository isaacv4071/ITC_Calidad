# Automatización con Playwright y Pytest

## Instalación y Preparación del Entorno

1. **Instalar Python**
   Descarga desde [python.org](https://www.python.org/downloads/).

2. **Generar entorno virtual**
   Aísla el entorno de pruebas:

   ```  
   python -m venv venv
   ```

3. **Activar entorno virtual**

   * En **Windows**:

     ```  
     .\venv\Scripts\activate
     ```

   * En **macOS/Linux**:

     ```  
     source venv/bin/activate
     ```

4. **Instalar dependencias necesarias**

   Si usas un `requirements.txt`:

   ```  
   pip install -r requirements.txt
   ```

   O directamente:

   ```  
   pip install pytest-playwright
   ```

5. **Instalar navegadores de Playwright**

   ```  
   playwright install
   ```

---

## 🎯 Generador de Código

Puedes generar código automáticamente mientras navegas una página web con:

```  
playwright codegen <URL>
```

### Generar en Python:

```  
playwright codegen --target python <URL>
```

> 💡 Útil para grabar flujos que luego puedes convertir en pruebas automatizadas.

---

## ▶️ Ejecución de Tests

* **Ejecutar todos los tests**:

  ```  
  pytest
  ```

* **Ejecutar un archivo específico**:

  ```  
  pytest tests/test_login.py
  ```

* **Ejecutar una función específica**:

  ```  
  pytest tests/test_login.py::test_login_correcto
  ```

> 🧩 Usa `-v` para modo detallado y `-s` para ver prints:

```  
pytest -v -s
```

---

## ⚙️ Estructura y Organización

Organiza tus pruebas dentro de una carpeta como `tests/`:

```plaintext
/tests
    ├── test_login.py
    └── test_registro.py
```

---

## 🧠 Tipos de Funciones

### Función `test_`

* Define un escenario de prueba.
* Debe comenzar con `test_` para que `pytest` la reconozca.

```python
def test_login_correcto(page):
    ...
```

### Funciones auxiliares

* Encapsulan lógica reutilizable.
* No se ejecutan por sí solas.

```python
def realizar_login(page, usuario, contraseña):
    page.goto("https://ejemplo.com/login")
    page.fill("#usuario", usuario)
    page.fill("#contraseña", contraseña)
    page.click("text=Ingresar")
```

> 🛠️ Úsalas para mantener tus tests limpios y reutilizables.

---

## ⚙️ Configuración de Pytest

Cuando utilizas **pytest-playwright**, puedes personalizar el comportamiento global mediante el archivo `pytest.ini`.

### Archivo `pytest.ini`

```ini
[pytest]
addopts = --headed
```

Esto permite que los tests se ejecuten con navegador visible (modo no headless) por defecto.

> ⚠️ **Nota:** Esta configuración **no aplica** si usas **Playwright puro** (`sync_playwright`). En ese caso, el control del modo headless o no headless se realiza directamente en el script:

```python
browser = p.chromium.launch(headless=False)
```

---

## 🧪 Ejemplos

### 🧪 Ejemplo extendido: ejecución dinámica de flujos con `Pytest + Playwright`

#### 📁 Estructura resumida

```plaintext
project/
│
├── app.py                ← Selector por consola
├── conftest.py           ← Argumentos y fixtures
├── flujos.py             ← Lógica por tipo
└── test_solicitud.py     ← Ejecuta el flujo correcto según la entrada
```

---

#### `flujos.py` – Definición de flujos independientes

```python
def flujo_convocatoria(page, tipo_contrato, valor_inicial):
    page.goto("https://example.com")
    print(f"[Convocatoria] Contrato: {tipo_contrato}, Valor: {valor_inicial}")

def flujo_contrato(page, tipo_contrato, valor_inicial):
    page.goto("https://example.com/contrato")
    print(f"[Contrato] Contrato: {tipo_contrato}, Valor: {valor_inicial}")

FLUJOS = {
    "convocatoria": flujo_convocatoria,
    "contrato": flujo_contrato
}
```

---

#### `test_solicitud.py` – Ejecuta el flujo basado en el tipo elegido

```python
from flujos import FLUJOS

def test_solicitud(page, tipo_solicitud, tipo_contrato, valor_inicial):
    flujo = FLUJOS.get(tipo_solicitud)
    assert flujo, f"Flujo no encontrado para tipo '{tipo_solicitud}'"
    flujo(page, tipo_contrato, valor_inicial)
```

---

#### `conftest.py` – Argumentos y validación

```python
import pytest

@pytest.fixture
def tipo_solicitud(request):
    return request.config.getoption("--tipo")

@pytest.fixture
def tipo_contrato(request):
    return request.config.getoption("--tipocontrato")

@pytest.fixture
def valor_inicial(request):
    return request.config.getoption("--valorinicial")

def pytest_addoption(parser):
    parser.addoption("--tipo", default="convocatoria")
    parser.addoption("--tipocontrato", default="administrativo")
    parser.addoption("--valorinicial", default="1000000")
```

---

#### `app.py` – Entrada por consola

```python
import subprocess

TIPOS_SOLICITUD = {"1": "convocatoria", "2": "contrato"}
TIPOS_CONTRATO = {"1": "administrativo", "2": "asistencial"}

def elegir(diccionario, nombre):
    print(f"\nElige {nombre}:")
    for k, v in diccionario.items():
        print(f"{k}. {v}")
    while True:
        opcion = input(f"Ingrese el número de {nombre}: ")
        if opcion in diccionario:
            return diccionario[opcion]
        print("Opción inválida.")

def main():
    tipo_solicitud = elegir(TIPOS_SOLICITUD, "tipo de solicitud")
    tipo_contrato = elegir(TIPOS_CONTRATO, "tipo de contrato")
    valor_inicial = input("Valor inicial: ")

    subprocess.run([
        "pytest", "test_solicitud.py",
        f"--tipo={tipo_solicitud}",
        f"--tipocontrato={tipo_contrato}",
        f"--valorinicial={valor_inicial}"
    ])

if __name__ == "__main__":
    main()
```

---

#### ✅ Ventajas del enfoque `FLUJOS[tipo]()`

* Código desacoplado y modular por tipo de flujo.
* Más fácil de escalar si se agregan 10+ tipos.
* El test actúa como **punto de entrada**, no como ejecutor de lógica.

---

### 🧪 Ejemplo de Playwright puro

#### 📁 Estructura general

```plaintext
project/
│
├── tests/
│   ├── test_NewUser.py
│   └── ...otros tests...
│
├── runner.py         ← Punto de entrada (consola interactiva)
└── videos/           ← Carpeta donde se guardan los videos de cada ejecución
```

---

#### `runner.py` – Lanzador interactivo de pruebas

```python
import os, glob, subprocess, sys
import questionary
from rich.console import Console
from rich.panel import Panel
from rich.progress import Progress, SpinnerColumn, TextColumn

console = Console()

def get_test_files(test_dir="tests"):
    return glob.glob(os.path.join(test_dir, "test_*.py"))

def get_test_description(file_path):
    with open(file_path, encoding="utf-8") as f:
        for line in f:
            if line.strip().startswith("#d"):
                return line.strip()[2:].strip()
    return os.path.basename(file_path)

def main():
    console.print(Panel("[bold cyan]▶ Playwright Test Runner[/bold cyan]"))

    test_files = get_test_files()
    if not test_files:
        console.print("[red]No hay tests en la carpeta 'tests/'[/red]")
        sys.exit(1)

    while True:
        choices = [
            f"{get_test_description(f)} ({os.path.basename(f)})"
            for f in test_files
        ] + ["Cerrar"]

        choice = questionary.select("Selecciona un test:", choices=choices).ask()
        if choice == "Cerrar":
            break

        selected_test = test_files[choices.index(choice)]
        console.print(Panel(f"[yellow]Ejecutando:[/yellow] {selected_test}"))

        with Progress(SpinnerColumn(), TextColumn("{task.description}"), transient=True) as progress:
            task = progress.add_task("🧪 Corriendo test...", start=False)
            progress.start_task(task)

            result = subprocess.run([sys.executable, selected_test], capture_output=True, text=True)

            progress.update(task, description="✅ Finalizado")
            progress.stop_task(task)

        console.print(Panel.fit(
            result.stdout or "[sin salida]",
            title="[bold blue]Salida[/bold blue]", border_style="blue"
        ))

        if result.returncode == 0:
            console.print(Panel("✅ [green]Test PASÓ correctamente[/green]", border_style="green"))
        else:
            console.print(Panel("❌ [red]Test FALLÓ[/red]", border_style="red"))

        console.print(Panel("[magenta]=== Fin de la ejecución ===[/magenta]"))

if __name__ == "__main__":
    main()
```

---

#### `tests/test_NewUser.py` – Ejemplo de test desacoplado

```python
#d Solicitar nuevo usuario

from playwright.sync_api import sync_playwright
from datetime import datetime
import os

def test_NewUser():
    with sync_playwright() as p:
        browser = p.chromium.launch(headless=True)
        context = browser.new_context(
            viewport={"width": 1280, "height": 720},
            record_video_dir="videos/",
            record_video_size={"width": 1280, "height": 720}
        )
        page = context.new_page()

        # --- Lógica del test ---
        page.goto("https://example.com/registro")
        page.get_by_label("Nombres").fill("Juan")
        page.get_by_label("Apellidos").fill("Pérez")
        page.get_by_label("Correo").fill("juan.perez@yopmail.com")
        page.get_by_label("Celular").fill("3001234567")
        page.get_by_role("button", name="Confirmar").click()
        # --- Fin del test ---

        context.close()
        browser.close()

        # Guardar el video con timestamp
        video_path = page.video.path()
        now = datetime.now().strftime("%Y%m%d_%H%M%S")
        final_path = f"videos/test_NewUser_{now}.webm"
        os.rename(video_path, final_path)
```

---

#### 🎥 Evidencia automática (video)

* Cada ejecución guarda automáticamente un **video del test**.
* El video se almacena en `videos/` con nombre único basado en fecha y hora:

```plaintext
videos/test_NewUser_20250728_145201.webm
```

Esto facilita la trazabilidad y la revisión manual si el test falla.

---

#### 💡 ¿Por qué esta estructura?

* ✅ **Independencia**: cada test es un archivo ejecutable por sí mismo (`python test_X.py`)
* ✅ **Reutilización**: puedes importar utilidades (`from utils import ...`) sin acoplar tests entre sí
* ✅ **Escalabilidad**: solo agregas nuevos `test_xxx.py` en `/tests` y ya aparecen en el menú
* ✅ **Descripción visible**: el tag `#d` se usa para mostrar una breve descripción en la consola

---

## 🧠 Tip final

Si necesitas que tus tests usen variables sensibles (como credenciales), puedes usar variables de entorno (`os.environ`) o preguntas interactivas dentro de cada test. Así los mantienes seguros y reutilizables.

## Distribución temporal del código para su uso

En este proyecto se incluyen dos scripts batch para facilitar la preparación y ejecución del entorno de pruebas automatizadas en Windows.  
Estos scripts automatizan la creación del entorno virtual, la instalación de dependencias y navegadores, y la ejecución de la aplicación.

### 1. `env.bat` — Inicialización del entorno (ejecutar **una sola vez**)

Este script debe ejecutarse **solo la primera vez** que se prepara el entorno en una nueva máquina o instalación limpia.  
Sus pasos son:

1. **Crea el entorno virtual** si no existe.
2. **Activa el entorno virtual**.
3. **Instala las dependencias** listadas en `requirements.txt`.
4. **Instala los navegadores de Playwright**.
5. **Inicializa el OCR** (opcional, si tu proyecto lo requiere).
6. Muestra un mensaje de confirmación.

**Contenido típico de `env.bat`:**

```batch
@echo off
REM 1. Crea el entorno virtual si no existe
if not exist venv (
    echo Creando entorno virtual...
    python -m venv venv
    echo Esperando 3 segundos para asegurar la creación del entorno...
    timeout /t 3 /nobreak > nul
)

REM 2. Activa el entorno virtual
call venv\Scripts\activate.bat

REM 3. Instala dependencias
echo Instalando dependencias desde requirements.txt...
pip install --upgrade pip
pip install -r requirements.txt

REM 4. Instala navegadores de Playwright
echo Instalando navegadores de Playwright...
python -m playwright install

REM 5. Inicializa el ocr  
echo Inicializando OCR...  
python ocr.py

echo.
echo Todo listo. El entorno virtual está preparado y Playwright instalado.
pause
```

**¿Cuándo usarlo?**  
Ejecuta `env.bat` **una sola vez** al clonar el proyecto o al preparar el entorno por primera vez en tu equipo.

---

### 2. `run.bat` — Ejecución de la aplicación

Este script es para **ejecutar la aplicación** o los tests una vez que el entorno ya está preparado.

1. **Activa el entorno virtual**.
2. **Ejecuta la aplicación principal** (por ejemplo, `app.py`).

**Contenido típico de `run.bat`:**

```batch
@echo off
REM Activa el entorno virtual
call venv\Scripts\activate.bat

REM Ejecuta tu aplicación
python app.py
```

**¿Cuándo usarlo?**  
Ejecuta `run.bat` cada vez que quieras lanzar la aplicación o correr los tests, después de haber inicializado el entorno con `env.bat`.

---

### Notas y buenas prácticas

- **No ejecutes `env.bat` más de una vez** a menos que necesites reinstalar el entorno desde cero.
- **No incluyas la carpeta `venv` en tu control de versiones** (agrega `venv/` a tu `.gitignore`).
- Si cambias las dependencias, actualiza `requirements.txt` y vuelve a ejecutar la parte de instalación de dependencias.
- Estos scripts están diseñados para **Windows**. Para sistemas Linux/Mac, deberás adaptar los comandos de activación del entorno virtual.