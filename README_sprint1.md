# README Sprint 1 - Minima operativa

## Objetivo
Disponer de una guia corta para poner en marcha el proyecto en local, entrenar el modelo y ejecutar pruebas.

## Alcance de Sprint 1
1. Setup local del entorno.
2. Entrenamiento de pipeline ML.
3. Ejecucion de pruebas unitarias.
4. Verificacion de artefactos generados.

## Requisitos previos
- Python 3.10 o superior.
- Git instalado.
- Conexion a internet para descargar dependencias.

## 1) Setup local

### 1.1 Clonar repositorio
Comando:
- git clone <URL_DEL_REPOSITORIO>
- cd pontia-modulo4-mlops-tutorial-sergio-soriano

### 1.2 Crear y activar entorno virtual
Windows PowerShell:
- python -m venv .venv
- .\.venv\Scripts\Activate.ps1

Linux o macOS:
- python3 -m venv .venv
- source .venv/bin/activate

### 1.3 Instalar dependencias
Comando:
- python -m pip install --upgrade pip
- pip install -r requirements.txt

## 2) Preparar datos
El pipeline espera estos archivos:
- data/raw/adult.data
- data/raw/adult.test

Si no existen, crea carpeta y descarga:
- mkdir -p data/raw
- curl -o data/raw/adult.data https://archive.ics.uci.edu/ml/machine-learning-databases/adult/adult.data
- curl -o data/raw/adult.test https://archive.ics.uci.edu/ml/machine-learning-databases/adult/adult.test

Nota para Windows sin curl:
- Puedes descargar ambos archivos manualmente y colocarlos en data/raw.

## 3) Entrenamiento local
Comando:
- python src/main.py

Resultado esperado:
- Se crea training.log.
- Se generan artefactos en models:
  - models/model.pkl
  - models/scaler.pkl
  - models/encoders.pkl

## 4) Pruebas unitarias

### 4.1 Ejecutar pruebas de unit_tests
Linux o macOS:
- PYTHONPATH=src pytest unit_tests/

Windows PowerShell:
- $env:PYTHONPATH = "src"
- pytest unit_tests/

### 4.2 Ejecutar con cobertura
Linux o macOS:
- PYTHONPATH=src pytest unit_tests/ --cov=model --cov=evaluate --cov=data_loader --cov-report=term --cov-report=html

Windows PowerShell:
- $env:PYTHONPATH = "src"
- pytest unit_tests/ --cov=model --cov=evaluate --cov=data_loader --cov-report=term --cov-report=html

Resultado esperado:
- Todos los tests en verde.
- Reporte de cobertura en consola.
- Carpeta htmlcov generada.

## 5) Pruebas de modelo entrenado
Precondicion:
- Haber ejecutado antes python src/main.py.

Comando:
- python model_tests/test_model.py

Resultado esperado:
- El modelo se carga correctamente.
- El script de test finaliza sin errores.

## 6) Checklist de cierre Sprint 1
- Entorno virtual creado y activo.
- Dependencias instaladas sin errores.
- Datos disponibles en data/raw.
- Entrenamiento ejecutado correctamente.
- Artefactos presentes en models.
- Pruebas unitarias en verde.
- Prueba de modelo ejecutada sin errores.

## Troubleshooting rapido
- Error de imports en tests:
  - Asegura PYTHONPATH=src antes de ejecutar pytest.

- Error por archivos de datos no encontrados:
  - Verifica rutas en data/raw/adult.data y data/raw/adult.test.

- Error al cargar artefactos del modelo:
  - Reentrena con python src/main.py y confirma que existen los .pkl en models.
