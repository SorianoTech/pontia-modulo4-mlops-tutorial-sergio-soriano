# Proyecto MLOps - Clasificacion de Ingresos (Adult Dataset)

Este repositorio implementa un flujo completo de MLOps para:
- entrenar un modelo de Machine Learning,
- validar el codigo con pruebas automatizadas,
- publicar artefactos versionados,
- y desplegar una API de inferencia.

El modelo entrenado es un `RandomForestClassifier` para predecir si una persona gana mas de 50K al anio, usando el dataset Adult Income de UCI.

## Guias por sprint

- [README Sprint 1](README_sprint1.md)
- [README Sprint 2](README_sprint2.md)
- [README Sprint 3](README_sprint3.md)

## 1. Estructura de directorios

```text
.
|- .github/
|  \- workflows/
|     |- build.yml
|     |- deploy.yml
|     \- integration.yml
|- data/
|  \- raw/
|     \- .gitkeep
|- deployment/
|  |- requirements.txt
|  \- app/
|     |- __init__.py
|     \- main.py
|- model_tests/
|  |- __init__.py
|  \- test_model.py
|- models/
|- src/
|  |- __init__.py
|  |- data_loader.py
|  |- evaluate.py
|  |- main.py
|  \- model.py
|- unit_tests/
|  |- __init__.py
|  |- test_data_loader.py
|  |- test_evaluate.py
|  \- test_model.py
|- pytest.ini
|- README.md
\- requirements.txt
```

Resumen rapido:
- `src/`: pipeline de entrenamiento y evaluacion.
- `unit_tests/`: pruebas unitarias de funciones.
- `model_tests/`: pruebas sobre artefactos del modelo entrenado.
- `models/`: artefactos generados (`model.pkl`, `scaler.pkl`, `encoders.pkl`).
- `deployment/app/`: API FastAPI para servir predicciones.
- `.github/workflows/`: automatizacion CI/CD con GitHub Actions.

## 2. Explicacion del codigo (para que sirve)

### 2.1 Entrenamiento y evaluacion

- `src/main.py`
	- Orquesta todo el pipeline:
		1. Carga datos desde `data/raw/adult.data` y `data/raw/adult.test`.
		2. Preprocesa (encoding + escalado).
		3. Entrena el modelo.
		4. Evalua metricas en test.
		5. Guarda artefactos en `models/`.
	- Ademas escribe logs en consola y en `training.log`.

- `src/data_loader.py`
	- Define columnas del dataset Adult.
	- `load_data(...)`:
		- lee train/test,
		- limpia valores faltantes,
		- normaliza la etiqueta de test (quita el punto final).
	- `preprocess_data(...)`:
		- codifica variables categoricas con `LabelEncoder`,
		- convierte el target `income` a binario (0/1),
		- aplica `StandardScaler`,
		- devuelve matrices listas para modelado y los objetos de preprocesado.

- `src/model.py`
	- `train_model(...)` crea y entrena un `RandomForestClassifier(random_state=42)`.

- `src/evaluate.py`
	- `evaluate(...)` calcula `accuracy` y `classification_report`.
	- Escribe resultados en logs.

### 2.2 Serving (despliegue)

- `deployment/app/main.py`
	- Levanta una API con FastAPI.
	- En el arranque (`lifespan`) descarga de la ultima GitHub Release:
		- `model.pkl`
		- `scaler.pkl`
		- `encoders.pkl`
	- Carga esos artefactos en memoria y expone endpoints:
		- `GET /health`: estado del servicio.
		- `POST /predict`: recibe un JSON con features y devuelve prediccion.
		- `GET /metrics`: contador simple de predicciones.

## 3. Workflows de GitHub Actions

El repositorio tiene tres workflows: `integration`, `build` y `deploy`.

## 3.1 integration.yml (Integracion / CI en PR)

Disparadores:
- `pull_request`
- `workflow_dispatch`

Que hace:
1. Checkout del repositorio.
2. Configura Python 3.10.
3. Instala dependencias de `requirements.txt`.
4. Ejecuta pruebas unitarias en `unit_tests/` con cobertura (`pytest-cov`).
5. Guarda resultados (`test-results/results.log`, `test-results/results.xml`).
6. Comenta en la PR el resultado de tests + coverage.
7. Falla el job si las pruebas fallaron.

Carpetas/archivos que necesita:
- `unit_tests/`
- `src/`
- `requirements.txt`
- `pytest.ini`

## 3.2 build.yml (Entrenamiento y publicacion de artefactos)

Disparadores:
- `push` a `main`
- `workflow_dispatch`

Que hace:
1. Checkout del repositorio.
2. Configura Python 3.10.
3. Instala dependencias de `requirements.txt`.
4. Descarga dataset Adult a `data/raw/`.
5. Ejecuta `python src/main.py` para entrenar.
6. Ejecuta pruebas de modelo en `model_tests/test_model.py`.
7. Publica artefactos (`models/model.pkl`, `models/scaler.pkl`, `models/encoders.pkl`) en una GitHub Release.

Carpetas/archivos que necesita:
- `src/`
- `model_tests/`
- `requirements.txt`
- `data/raw/` (la crea/usa durante el workflow)
- `models/` (se generan artefactos)

## 3.3 deploy.yml (Despliegue)

Disparador:
- `workflow_dispatch` (manual)

Que hace:
1. Checkout del repositorio.
2. Llama al `RENDER_DEPLOY_HOOK` para desplegar en Render.

Carpetas/archivos que necesita:
- Para ejecutar el workflow: no depende de carpetas especificas de codigo (solo del repositorio y del secreto `RENDER_DEPLOY_HOOK`).
- Para que la app desplegada funcione correctamente:
	- `deployment/app/`
	- `deployment/requirements.txt`
	- Artefactos publicados previamente por `build` en GitHub Releases.

## 3.4 Orden recomendado: build, deploy e integration

Si hablamos de flujo de calidad y entrega de extremo a extremo, el orden recomendado es:

1. **Integration (primero)**
	 - valida codigo y pruebas en cada PR antes de mergear.
2. **Build (segundo)**
	 - una vez en `main`, entrena, valida el modelo y publica artefactos versionados.
3. **Deploy (tercero)**
	 - despliega la aplicacion consumiendo la ultima release de artefactos.

Nota importante:
- En este repo `deploy` es manual, asi que debe ejecutarse despues de un `build` exitoso para asegurar que existan artefactos actualizados.

## 4. Como se realizan las pruebas

## 4.1 Pruebas unitarias

Objetivo:
- validar funciones del codigo fuente (`load_data`, `preprocess_data`, `train_model`, `evaluate`).

Ubicacion:
- `unit_tests/`

Comando recomendado:

```bash
PYTHONPATH=src pytest unit_tests/ --cov=model --cov=evaluate --cov=data_loader --cov-report=term --cov-report=html
```

En Windows PowerShell:

```powershell
$env:PYTHONPATH = "src"
pytest unit_tests/ --cov=model --cov=evaluate --cov=data_loader --cov-report=term --cov-report=html
```

Salida esperada:
- reporte de tests,
- porcentaje de cobertura,
- carpeta `htmlcov/` con reporte HTML.

## 4.2 Pruebas de modelo entrenado

Objetivo:
- validar que el artefacto del modelo existe, se puede cargar y predice correctamente.

Ubicacion:
- `model_tests/test_model.py`

Precondicion:
- haber entrenado antes (`python src/main.py`) para generar `models/model.pkl`.

Comando:

```bash
python model_tests/test_model.py
```

## 4.3 Flujo local sugerido

```bash
pip install -r requirements.txt
mkdir -p data/raw
# descargar adult.data y adult.test en data/raw
python src/main.py
PYTHONPATH=src pytest unit_tests/
python model_tests/test_model.py
```

## 5. Dependencias principales

Entrenamiento y pruebas (`requirements.txt`):
- scikit-learn
- pandas
- numpy
- joblib
- pytest
- pytest-cov

Serving (`deployment/requirements.txt`):
- fastapi
- uvicorn
- requests
- scikit-learn
- pandas
- numpy
- joblib
