# README Sprint 2 - CI y Build de artefactos

## Objetivo
Automatizar calidad en Pull Requests y construir artefactos del modelo en main, publicandolos en GitHub Releases.

## Alcance de Sprint 2
1. Integracion continua en PR con tests y cobertura.
2. Build automatizado del modelo en main.
3. Publicacion de model.pkl, scaler.pkl y encoders.pkl.
4. Validacion end-to-end del flujo Integration -> Build.

## Requisitos previos
- Repositorio ya funcional en local (Sprint 1).
- GitHub Actions habilitado en el repositorio.
- Permisos para abrir PR y hacer merge en main.
- Acceso a pestaña Actions y Releases en GitHub.

## Archivos clave del Sprint
- .github/workflows/integration.yml
- .github/workflows/build.yml
- model_tests/test_model.py
- requirements.txt
- src/main.py

## 1) Setup local para validacion previa

### 1.1 Activar entorno e instalar dependencias
Windows PowerShell:
- .\.venv\Scripts\Activate.ps1
- python -m pip install --upgrade pip
- pip install -r requirements.txt

Linux o macOS:
- source .venv/bin/activate
- python -m pip install --upgrade pip
- pip install -r requirements.txt

### 1.2 Ejecutar tests unitarios en local
Linux o macOS:
- PYTHONPATH=src pytest unit_tests/ --cov=model --cov=evaluate --cov=data_loader --cov-report=term --cov-report=html

Windows PowerShell:
- $env:PYTHONPATH = "src"
- pytest unit_tests/ --cov=model --cov=evaluate --cov=data_loader --cov-report=term --cov-report=html

Resultado esperado:
- Tests unitarios en verde.
- Cobertura mostrada en terminal.
- Carpeta htmlcov generada.

## 2) Workflow de Integration en Pull Request

## Que hace
1. Se activa en pull_request y workflow_dispatch.
2. Instala dependencias.
3. Ejecuta pytest sobre unit_tests con cobertura.
4. Guarda resultados en test-results.
5. Comenta los resultados en la PR.
6. Falla el job si fallan tests.

## Pasos para ejecutarlo
1. Crear rama de trabajo:
- git checkout -b feature/ci-validation

2. Hacer cambios y commit:
- git add .
- git commit -m "chore: validar workflow de integracion"

3. Subir rama:
- git push -u origin feature/ci-validation

4. Abrir Pull Request hacia main.

5. Verificar en GitHub Actions:
- Job Integration en estado success.
- Comentario automatico en la PR con resumen de tests.

## Comandos utiles de diagnostico local
- pytest unit_tests/ -q
- pytest unit_tests/test_model.py -q
- pytest unit_tests/test_data_loader.py -q
- pytest unit_tests/test_evaluate.py -q

## 3) Workflow de Build en main

## Que hace
1. Se activa en push a main y workflow_dispatch.
2. Descarga Adult dataset en data/raw.
3. Ejecuta python src/main.py.
4. Ejecuta model_tests/test_model.py.
5. Publica artefactos en GitHub Release.

## Pasos para dispararlo por push a main
1. Hacer merge de PR con Integration en verde.
2. Verificar que inicia Build Model automaticamente.

## Pasos para dispararlo manualmente
1. Ir a GitHub -> Actions -> Build Model.
2. Ejecutar Run workflow sobre main.

## Validaciones post-build
1. Revisar logs del job:
- Descarga de adult.data y adult.test completada.
- Entrenamiento completado.
- model_tests completado.

2. Revisar Release creada:
- Tag tipo model-<run_number>.
- Assets presentes:
  - model.pkl
  - scaler.pkl
  - encoders.pkl

## 4) Comandos de validacion local del build

Si quieres simular Build en local antes de merge:
1. Crear carpeta de datos:
- mkdir -p data/raw

2. Descargar dataset:
- curl -o data/raw/adult.data https://archive.ics.uci.edu/ml/machine-learning-databases/adult/adult.data
- curl -o data/raw/adult.test https://archive.ics.uci.edu/ml/machine-learning-databases/adult/adult.test

3. Entrenar:
- python src/main.py

4. Ejecutar test de modelo:
- python model_tests/test_model.py

Resultado esperado:
- Archivos en models:
  - models/model.pkl
  - models/scaler.pkl
  - models/encoders.pkl

## 5) Checklist de cierre Sprint 2
- Integration corre en cada PR y falla correctamente cuando hay errores.
- Build corre en main y finaliza en success.
- Release de modelo se crea con los 3 artefactos.
- Se puede trazar el artefacto al run de Actions.
- README principal refleja flujo Integration -> Build -> Deploy.

## Troubleshooting rapido

- Error ModuleNotFoundError en tests:
  - Definir PYTHONPATH=src antes de pytest.

- Error por dataset no encontrado en build local:
  - Confirmar data/raw/adult.data y data/raw/adult.test.

- Error en model_tests por modelo ausente:
  - Ejecutar primero python src/main.py.

- Build publica release sin un asset:
  - Revisar paso Upload model to GitHub Release y existencia de archivos en models.

- Integration no comenta en PR:
  - Verificar que el workflow se ejecuto en contexto pull_request y que el job llego al paso de comentario.
