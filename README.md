# Paso a paso LAB5 y LAB6 – Django con entorno virtual

Oct 9, 2026 · @Nancy

## Antes de comenzar

Al terminar ambos laboratorios, cada estudiante tendrá el proyecto Django **INFJMC** con la aplicación **core**, tres vistas con front end propio y un repositorio GitHub con las ramas LAB5 y LAB6 fusionadas en main. Todo el código de esta guía fue ejecutado y probado con Python 3.13 y Django 6.1.

| ID | Requisito | Cómo se cumple |
| --- | --- | --- |
| REQ01 | El proyecto se llama INFJMC | `python -m django startproject INFJMC .` |
| REQ02 | La aplicación principal se llama core | `python manage.py startapp core` |
| REQ03 | Vistas home, carreras y docentes | `core/views.py` + `core/urls.py` + 3 templates |

**Herramientas que cada estudiante necesita instaladas:**

- Python 3.10 o superior (en Windows, marcar *Add python.exe to PATH* al instalar). Verificar con `python --version`.
- Git, con nombre y correo configurados: `git config --global user.name "Nombre Apellido"` y `git config --global user.email "correo@duocuc.cl"`.
- Una cuenta en GitHub.
- Visual Studio Code con la extensión *Python* de Microsoft.

**Convención de comandos:** cuando un comando difiere entre sistemas, se indica **Windows** (PowerShell o CMD) y **macOS/Linux**. En macOS/Linux puede ser necesario usar `python3` en vez de `python`.

**Estructura final esperada** (al terminar LAB6):

```
INFJMC/                  <- carpeta del repositorio
├── venv/                <- entorno virtual (NO se sube a GitHub)
├── .gitignore
├── requirements.txt
├── manage.py
├── INFJMC/              <- configuración del proyecto
│   ├── settings.py
│   └── urls.py
└── core/                <- aplicación principal
    ├── views.py
    ├── urls.py
    ├── templates/core/
    │   ├── base.html    (LAB6)
    │   ├── home.html
    │   ├── carreras.html
    │   └── docentes.html
    └── static/core/     (LAB6)
        ├── css/estilos.css
        ├── img/logo.png
        └── js/main.js
```

## LAB5 – Primeros pasos con Django Framework

En LAB5 se crea el repositorio, el entorno virtual, el proyecto INFJMC y la app core con sus tres vistas, trabajando en la rama LAB5 y fusionándola en main al terminar.

### Paso 1. Crear la carpeta del proyecto

Abrir una terminal (PowerShell - CMD ó en VS Code: *Terminal → New Terminal*) y ejecutar:

```bash
mkdir INFJMC
cd INFJMC
```

Desde ahora, **todos los comandos se ejecutan dentro de esta carpeta**. Abrirla en VS Code con *File → Open Folder* o con `code .`

### Paso 2. Crear y activar el entorno virtual

El entorno virtual aísla las librerías del proyecto (Django y su versión) de las del sistema, para que cada proyecto tenga sus propias dependencias.

```bash
python -m venv venv
```

Activarlo:

- **Windows (PowerShell):** `venv\Scripts\Activate.ps1`
- **Windows (CMD):** `venv\Scripts\activate.bat`
- **macOS/Linux:** `source venv/bin/activate`

Se sabe que está activo porque la terminal muestra `(venv)` al inicio de la línea. **Cada vez que se abra una terminal nueva hay que volver a activarlo.** Para salir del entorno se usa `deactivate`.

Si PowerShell muestra el error *"la ejecución de scripts está deshabilitada"*, ejecutar una sola vez `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`, responder `S` y activar de nuevo.

En VS Code, seleccionar el intérprete del entorno: `Ctrl+Shift+P` → *Python: Select Interpreter* → elegir el que dice `('venv': venv)`.

### Paso 3. Instalar las dependencias con requirements.txt

El archivo `requirements.txt` lista las librerías que necesita el proyecto. Con él, cualquier persona recrea el mismo entorno con un solo comando, sin instalar paquete por paquete.

Crear el archivo `requirements.txt` en la raíz de la carpeta INFJMC con este contenido:

```
Django
```

Con `(venv)` visible, actualizar pip e instalar todo lo que indica el archivo:

```bash
python -m pip install --upgrade pip
pip install -r requirements.txt
```

Verificar la instalación:

```bash
pip list
python -m django --version
```

`pip list` debe mostrar `Django`, `asgiref` y `sqlparse` (estas dos se instalan solas porque Django las necesita; en Windows aparece también `tzdata`). Con Python 3.12 o superior se instala Django 6.x; con Python 3.10 u 3.11, pip elige Django 5.2 (LTS). El rango `>=5.2,<7.0` admite ambas y sirve para estos laboratorios.

**Si más adelante el proyecto necesita otra librería**, se agrega una línea al archivo (por ejemplo `Pillow>=10.0` para imágenes) y se vuelve a ejecutar `pip install -r requirements.txt`. Otra opción es instalarla con `pip install` y luego registrar las versiones exactas con `pip freeze > requirements.txt`.

`requirements.txt` **sí se sube a GitHub** (la carpeta `venv`, no). Así, en cualquier computador basta clonar, crear el entorno y ejecutar `pip install -r requirements.txt`.

### Paso 4. Crear el repositorio Git y el .gitignore

Crear el archivo `.gitignore` en la raíz de la carpeta INFJMC con este contenido, para que el entorno virtual y archivos generados no se suban a GitHub:

```
venv/
__pycache__/
*.pyc
db.sqlite3
.vscode/
```

Inicializar el repositorio con la rama principal `main` y hacer el primer commit:

```bash
git init -b main
git add .
git commit -m "Commit inicial: .gitignore y requirements.txt"
```

En GitHub, crear un repositorio **vacío** llamado `INFJMC` (sin README ni .gitignore) y conectarlo:

```bash
git remote add origin https://github.com/USUARIO/INFJMC.git
git push -u origin main
```

### Paso 5. Crear la rama LAB5

```bash
git switch -c LAB5
```

Todo el desarrollo de este laboratorio se hace en esta rama. Comprobar con `git branch` (la rama activa aparece con `*`).

### Paso 6. Crear el proyecto INFJMC (REQ01)

```bash
python -m django startproject INFJMC .
```

El **punto final** es importante: crea el proyecto en la carpeta actual y evita una carpeta INFJMC dentro de otra INFJMC. Quedan `manage.py` y la subcarpeta `INFJMC/` con `settings.py`, `urls.py`, `wsgi.py` y `asgi.py`.

Se usa `python -m django` en lugar de `django-admin` porque en Windows 11 el **Control de aplicaciones** (Smart App Control) puede bloquear `django-admin.exe` con el error *"Una directiva de Control de aplicaciones bloqueó este archivo"*. Ejecutar Django a través de `python` evita ese bloqueo y funciona igual en todos los sistemas. Por la misma razón, si `pip` también se bloquea, usar `python -m pip install -r requirements.txt`.

Revisar que el nombre sea exactamente **INFJMC** (no INFJCM): es el requisito REQ01.

Probar que funciona:

```bash
python manage.py migrate
python manage.py runserver
```

Abrir http://127.0.0.1:8000/ en el navegador: debe aparecer el cohete de bienvenida de Django. Detener el servidor con `Ctrl+C`.

### Paso 7. Crear la aplicación core (REQ02)

```bash
python manage.py startapp core
```

Registrar la app en `INFJMC/settings.py`, agregando `"core"` al final de `INSTALLED_APPS`. En el mismo archivo, ajustar idioma y zona horaria:

```python
INSTALLED_APPS = [
    "django.contrib.admin",
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "django.contrib.staticfiles",
    "core",
]

# ... más abajo en el mismo archivo:
LANGUAGE_CODE = "es-cl"
TIME_ZONE = "America/Santiago"
```

### Paso 8. Crear las tres vistas (REQ03)

Reemplazar el contenido de `core/views.py`. Por ahora los datos son listas en Python; en laboratorios posteriores vendrán de modelos y base de datos.

```python
from django.shortcuts import render


def home(request):
    return render(request, "core/home.html")


def carreras(request):
    lista_carreras = [
        {"nombre": "Ingeniería en Informática", "jornada": "Diurna", "duracion": "8 semestres"},
        {"nombre": "Analista Programador", "jornada": "Vespertina", "duracion": "5 semestres"},
        {"nombre": "Ingeniería en Conectividad y Redes", "jornada": "Diurna", "duracion": "8 semestres"},
    ]
    return render(request, "core/carreras.html", {"carreras": lista_carreras})


def docentes(request):
    lista_docentes = [
        {"nombre": "Ana Pérez", "asignatura": "Taller Lenguaje de Programación"},
        {"nombre": "Carlos Soto", "asignatura": "Base de Datos"},
        {"nombre": "María Rojas", "asignatura": "Desarrollo Web"},
    ]
    return render(request, "core/docentes.html", {"docentes": lista_docentes})
```

### Paso 9. Configurar las URLs

Crear el archivo nuevo `core/urls.py`:

```python
from django.urls import path
from . import views

urlpatterns = [
    path("", views.home, name="home"),
    path("carreras/", views.carreras, name="carreras"),
    path("docentes/", views.docentes, name="docentes"),
]
```

Editar `INFJMC/urls.py` para incluir las URLs de core:

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path("admin/", admin.site.urls),
    path("", include("core.urls")),
]
```

### Paso 10. Crear los templates

Crear las carpetas `core/templates/core/` (la carpeta `core` repetida dentro de `templates` evita choques de nombres entre apps). Dentro, crear tres archivos.

`core/templates/core/home.html`:

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>INFJMC - Inicio</title>
</head>
<body>
    <nav>
        <a href="{% url 'home' %}">Inicio</a> |
        <a href="{% url 'carreras' %}">Carreras</a> |
        <a href="{% url 'docentes' %}">Docentes</a>
    </nav>
    <h1>Área de Informática</h1>
    <p>Bienvenidos al sitio del Área de Informática. Aquí encontrarás información
       sobre carreras, docentes, actividades y comunicados.</p>
</body>
</html>
```

`core/templates/core/carreras.html`:

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>INFJMC - Carreras</title>
</head>
<body>
    <nav>
        <a href="{% url 'home' %}">Inicio</a> |
        <a href="{% url 'carreras' %}">Carreras</a> |
        <a href="{% url 'docentes' %}">Docentes</a>
    </nav>
    <h1>Carreras del Área</h1>
    <ul>
        {% for c in carreras %}
            <li><strong>{{ c.nombre }}</strong> - {{ c.jornada }} ({{ c.duracion }})</li>
        {% empty %}
            <li>No hay carreras registradas.</li>
        {% endfor %}
    </ul>
</body>
</html>
```

`core/templates/core/docentes.html`:

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>INFJMC - Docentes</title>
</head>
<body>
    <nav>
        <a href="{% url 'home' %}">Inicio</a> |
        <a href="{% url 'carreras' %}">Carreras</a> |
        <a href="{% url 'docentes' %}">Docentes</a>
    </nav>
    <h1>Docentes del Área</h1>
    <ul>
        {% for d in docentes %}
            <li><strong>{{ d.nombre }}</strong> - {{ d.asignatura }}</li>
        {% empty %}
            <li>No hay docentes registrados.</li>
        {% endfor %}
    </ul>
</body>
</html>git
```

Notar que el menú `<nav>` se repite en los tres archivos. Esa duplicación es justamente lo que LAB6 resuelve con herencia de templates.

### Paso 11. Probar

```bash
python manage.py runserver
```

| URL | Debe mostrar |
| --- | --- |
| http://127.0.0.1:8000/ | Página de inicio con el menú |
| http://127.0.0.1:8000/carreras/ | Listado de 3 carreras |
| http://127.0.0.1:8000/docentes/ | Listado de 3 docentes |

### Paso 12. Commit, push y merge de LAB5 a main

```bash
git add .
git commit -m "LAB5: proyecto INFJMC, app core y vistas home, carreras y docentes"
git push -u origin LAB5

git switch main
git merge LAB5
git push origin main
```

Alternativa recomendada para que quede registro en GitHub: en lugar de `git merge` local, abrir un **Pull Request** de LAB5 hacia main en la web de GitHub y fusionarlo ahí; luego en la terminal `git switch main` y `git pull`.

Verificar en GitHub que main contiene `manage.py`, `INFJMC/` y `core/`, y que **no** aparece la carpeta `venv/`.

## LAB6 – Templates y archivos estáticos (STATIC)

En LAB6 se construye el front end del sitio: una plantilla base que las tres páginas heredan con `{% extends %}`, y una carpeta static con CSS, imágenes y JavaScript cargados con `{% static %}`. Se trabaja en la rama LAB6 y se fusiona en main al terminar.

Nota para el enunciado: el texto de LAB6 dice *"para que el resultado de LAB4 y LAB5 se fusionen"*; corresponde a **LAB5 y LAB6**.

### Paso 1. Retomar el proyecto con el entorno virtual activo

**Mismo computador:** abrir la carpeta INFJMC y activar el entorno (`venv\Scripts\Activate.ps1` en Windows o `source venv/bin/activate` en macOS/Linux).

**Otro computador (ej. laboratorio de la sede):** la carpeta `venv` no está en GitHub, así que se recrea a partir de `requirements.txt`:

```bash
git clone https://github.com/USUARIO/INFJMC.git
cd INFJMC
python -m venv venv
# activar el entorno (ver Paso 2 de LAB5)
pip install -r requirements.txt
python manage.py migrate
```

### Paso 2. Crear la rama LAB6 desde main actualizado

```bash
git switch main
git pull
git switch -c LAB6
```

Así LAB6 parte con todo lo que ya se fusionó de LAB5.

### Paso 3. Crear la estructura de archivos estáticos

Dentro de la app core, crear estas carpetas (de nuevo, `core` repetido dentro de `static` para evitar choques de nombres):

```
core/static/core/css/
core/static/core/img/
core/static/core/js/
```

No se necesita configuración extra: `django.contrib.staticfiles` ya está en `INSTALLED_APPS` y `STATIC_URL = "static/"` ya viene en `settings.py`. Django busca automáticamente en la carpeta `static/` de cada app. Copiar en `core/static/core/img/` el logo de la institución con el nombre `logo.png` (o el recurso publicado en el aula).

Opcional: si se quiere una carpeta static global (fuera de las apps), crear `static/` en la raíz y agregar en `settings.py` la línea `STATICFILES_DIRS = [BASE_DIR / "static"]`.

### Paso 4. Hoja de estilos

`core/static/core/css/estilos.css`:

```css
* { box-sizing: border-box; margin: 0; padding: 0; }

body {
    font-family: Arial, Helvetica, sans-serif;
    background-color: #f4f6f9;
    color: #333;
    min-height: 100vh;
    display: flex;
    flex-direction: column;
}

.encabezado {
    background-color: #003366;
    padding: 10px 30px;
    display: flex;
    align-items: center;
    justify-content: space-between;
}

.logo { height: 60px; background: #fff; padding: 4px; border-radius: 4px; }

.menu a {
    color: #fff;
    text-decoration: none;
    margin-left: 20px;
    font-weight: bold;
}

.menu a:hover, .menu a.activo { text-decoration: underline; color: #ffcc00; }

.contenido { flex: 1; padding: 30px; max-width: 1000px; margin: 0 auto; width: 100%; }

.contenido h1 { color: #003366; margin-bottom: 20px; }

.tarjetas { display: flex; flex-wrap: wrap; gap: 20px; }

.tarjeta {
    background: #fff;
    border-left: 5px solid #003366;
    padding: 15px;
    width: 280px;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
}

.tabla { width: 100%; border-collapse: collapse; background: #fff; }
.tabla th { background: #003366; color: #fff; padding: 10px; text-align: left; }
.tabla td { padding: 10px; border-bottom: 1px solid #ddd; }

.pie { background: #003366; color: #fff; text-align: center; padding: 15px; }
```

### Paso 5. JavaScript

`core/static/core/js/main.js` (marca en el menú la página actual):

```javascript
document.querySelectorAll(".menu a").forEach(function (enlace) {
    if (enlace.getAttribute("href") === window.location.pathname) {
        enlace.classList.add("activo");
    }
});
```

### Paso 6. Template base

Crear `core/templates/core/base.html`. Contiene todo lo común (head, menú, pie) y define **bloques** que cada página rellena. La primera línea `{% load static %}` habilita la etiqueta `{% static %}`.

```html
{% load static %}
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block titulo %}INFJMC{% endblock %} | Área de Informática</title>
    <link rel="stylesheet" href="{% static 'core/css/estilos.css' %}">
</head>
<body>
    <header class="encabezado">
        <img src="{% static 'core/img/logo.png' %}" alt="Logo institucional" class="logo">
        <nav class="menu">
            <a href="{% url 'home' %}">Inicio</a>
            <a href="{% url 'carreras' %}">Carreras</a>
            <a href="{% url 'docentes' %}">Docentes</a>
        </nav>
    </header>

    <main class="contenido">
        {% block contenido %}
        {% endblock %}
    </main>

    <footer class="pie">
        <p>Área de Informática &copy; 2026</p>
    </footer>

    <script src="{% static 'core/js/main.js' %}"></script>
</body>
</html>
```

### Paso 7. Templates hijos (extensiones)

Reemplazar el contenido de los tres templates de LAB5. Cada uno empieza con `{% extends %}` y solo define los bloques; el menú ya no se repite. Las vistas y URLs de LAB5 **no cambian**.

`core/templates/core/home.html`:

```html
{% extends 'core/base.html' %}

{% block titulo %}Inicio{% endblock %}

{% block contenido %}
    <h1>Área de Informática</h1>
    <p>Bienvenidos al sitio del Área de Informática. Aquí encontrarás información
       sobre carreras, docentes, actividades y comunicados.</p>
{% endblock %}
```

`core/templates/core/carreras.html`:

```html
{% extends 'core/base.html' %}

{% block titulo %}Carreras{% endblock %}

{% block contenido %}
    <h1>Carreras del Área</h1>
    <div class="tarjetas">
        {% for c in carreras %}
            <div class="tarjeta">
                <h3>{{ c.nombre }}</h3>
                <p>Jornada: {{ c.jornada }}</p>
                <p>Duración: {{ c.duracion }}</p>
            </div>
        {% empty %}
            <p>No hay carreras registradas.</p>
        {% endfor %}
    </div>
{% endblock %}
```

`core/templates/core/docentes.html`:

```html
{% extends 'core/base.html' %}

{% block titulo %}Docentes{% endblock %}

{% block contenido %}
    <h1>Docentes del Área</h1>
    <table class="tabla">
        <thead>
            <tr><th>Nombre</th><th>Asignatura</th></tr>
        </thead>
        <tbody>
            {% for d in docentes %}
                <tr><td>{{ d.nombre }}</td><td>{{ d.asignatura }}</td></tr>
            {% empty %}
                <tr><td colspan="2">No hay docentes registrados.</td></tr>
            {% endfor %}
        </tbody>
    </table>
{% endblock %}
```

`{% extends %}` debe ser la **primera línea** del template hijo; si algo va antes, Django lanza error.

### Paso 8. Probar

```bash
python manage.py runserver
```

| URL | Debe mostrar |
| --- | --- |
| http://127.0.0.1:8000/ | Encabezado azul con logo y menú, bienvenida, pie de página |
| http://127.0.0.1:8000/carreras/ | Carreras como tarjetas; "Carreras" destacado en amarillo en el menú |
| http://127.0.0.1:8000/docentes/ | Docentes en tabla con encabezado azul |
| http://127.0.0.1:8000/static/core/css/estilos.css | El texto del CSS (confirma que static funciona) |

Si el CSS no se aplica tras un cambio, recargar sin caché con `Ctrl+F5`.

### Paso 9. Commit, push y merge de LAB6 a main

```bash
git add .
git commit -m "LAB6: template base, extensiones y archivos static"
git push -u origin LAB6

git switch main
git merge LAB6
git push origin main
```

O bien, como en LAB5, mediante un Pull Request de LAB6 hacia main en GitHub. Si se instaló algún paquete nuevo, actualizar antes `pip freeze > requirements.txt`.

Al finalizar, main debe contener LAB5 + LAB6, y en GitHub deben verse las ramas `main`, `LAB5` y `LAB6`.

## Errores frecuentes

| Síntoma | Causa | Solución |
| --- | --- | --- |
| `No module named django` o `django-admin` no se reconoce | Entorno virtual no activado | Activar `venv` y verificar `(venv)` en la terminal |
| PowerShell: la ejecución de scripts está deshabilitada | Política de ejecución de Windows | `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned` |
| Carpeta INFJMC dentro de otra INFJMC | Faltó el punto en `startproject INFJMC .` | Borrar y repetir con el punto |
| `TemplateDoesNotExist at /` | Template mal ubicado o `core` no está en `INSTALLED_APPS` | Revisar ruta `core/templates/core/` y settings |
| `NoReverseMatch` | Nombre en `{% url %}` distinto al `name=` de `core/urls.py` | Igualar los nombres |
| `Invalid block tag 'static'` | Falta `{% load static %}` en el template | Agregarlo como primera línea de `base.html` |
| CSS o imagen no cargan (404 en consola) | Ruta en `{% static %}` no coincide con la carpeta | Debe ser `core/static/core/...` y `{% static 'core/...' %}` |
| La carpeta `venv` aparece en GitHub | `.gitignore` creado después del primer `git add` | `git rm -r --cached venv`, commit y push |
| Cambios de LAB5 no aparecen en LAB6 | Rama LAB6 creada antes del merge | `git switch LAB6` y `git merge main` |
| Una directiva de Control de aplicaciones bloqueó este archivo (django-admin.exe o pip.exe) | Smart App Control de Windows bloquea los .exe del entorno virtual | Usar python -m django ... y python -m pip ... |

## Checklist de revisión

**LAB5**

- [ ] Entorno virtual creado y `requirements.txt` en el repositorio
- [ ] `venv/` excluido mediante `.gitignore`
- [ ] Proyecto llamado INFJMC (REQ01)
- [ ] App core creada y registrada en `INSTALLED_APPS` (REQ02)
- [ ] Vistas home, carreras y docentes funcionando en sus URLs (REQ03)
- [ ] Rama LAB5 en GitHub y fusionada en main

**LAB6**

- [ ] Rama LAB6 creada desde main actualizado
- [ ] `base.html` con bloques y `{% load static %}`
- [ ] Los tres templates usan `{% extends 'core/base.html' %}`
- [ ] Carpeta `core/static/core/` con CSS, imagen y JS cargados correctamente
- [ ] Rama LAB6 en GitHub y fusionada en main (main = LAB5 + LAB6)
